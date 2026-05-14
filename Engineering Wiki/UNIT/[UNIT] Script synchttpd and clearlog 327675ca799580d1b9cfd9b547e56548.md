# [UNIT] Script synchttpd and clearlog

Owner: Nam Tran
Last edited time: March 22, 2026 4:00 PM

```bash
/opt/scripts/sync.sh
#!/bin/bash
set -euo pipefail

now=`date +"%Y/%m/%d %T"`
filenow=`date +"%Y%m%d_%T"`

synclog_dir="/appvol/logs/sync_logs"
synclog_file="$synclog_dir/sync_$filenow.log"

mkdir -p "$synclog_dir"

echo "$now Begin delete backup files/log files older than 7 days" | tee -a "$synclog_file"
find "$synclog_dir" -type f -mtime +7 -name 'sync_*.log' -execdir rm -- '{}' +
now=`date +"%Y/%m/%d %T"`
echo "$now End delete backup files/log files older than 7 days" | tee -a "$synclog_file"

echo "$now Begin backup Reverse Proxy config and Certificate" | tee -a "$synclog_file"
rsync -azvi --progress --delete root@rpdevvm01.unit.local:/etc/httpd/conf/ /etc/httpd/conf/ | tee -a "$synclog_file"
rsync -azvi --progress --delete root@rpdevvm01.unit.local:/etc/httpd/conf.d/ /etc/httpd/conf.d/ | tee -a "$synclog_file"
rsync -azvi --progress --delete root@rpdevvm01.unit.local:/etc/letsencrypt/ /etc/letsencrypt/ | tee -a "$synclog_file"
rsync -azvi --progress root@rpdevvm01.unit.local:/etc/ssl/certs/_.unit.vn.crt /etc/ssl/certs/_.unit.vn.crt | tee -a "$synclog_file"
rsync -azvi --progress root@rpdevvm01.unit.local:/etc/ssl/certs/_.unit.vn.key /etc/ssl/certs/_.unit.vn.key | tee -a "$synclog_file"
rsync -azvi --progress root@rpdevvm01.unit.local:/etc/ssl/certs/_.unit.vn.ca-bundle /etc/ssl/certs/_.unit.vn.ca-bundle | tee -a "$synclog_file"
rsync -azvi --progress --delete root@rpdevvm01.unit.local:/etc/pki/tls/certs/ /etc/pki/tls/certs/ | tee -a "$synclog_file"
rsync -azvi --progress --delete root@rpdevvm01.unit.local:/appvol/logs/httpd/ /appvol/logs/httpd/ | tee -a "$synclog_file"
now=`date +"%Y/%m/%d %T"`
echo "$now End backup Reverse Proxy config and Certificate" | tee -a "$synclog_file"

echo "$now Checking httpd configuration..." | tee -a "$synclog_file"
if ! /usr/sbin/httpd -t | tee -a "$synclog_file"; then
    now=`date +"%Y/%m/%d %T"`
    echo "$now CRITICAL: httpd configuration is invalid! Aborting restart." | tee -a "$synclog_file"
    exit 1
fi

echo "$now Safely restarting httpd service (if running)..." | tee -a "$synclog_file"
/usr/bin/systemctl try-restart httpd.service
now=`date +"%Y/%m/%d %T"`
echo "$now httpd service restart attempt completed." | tee -a "$synclog_file"
```

```jsx
/opt/scripts/notify_failure.sh 
#!/bin/bash

# Configuration
EMAIL_TO="namtp@unit.com.vn"  # User's email from mail.json
EMAIL_FROM="sysadmin@unit.vn"
SUBJECT="ALERT: Sync HTTPD Service Failed - $(hostname)"
HOSTNAME=$(hostname)
DATE=$(date +"%Y/%m/%d %H:%M:%S")

# Message Body
MESSAGE="
Hệ thống đồng bộ dữ liệu HTTPD đã gặp lỗi.

Chi tiết:
- Hostname: $HOSTNAME
- Thời gian: $DATE
- Service: synchttpd.service

Vui lòng kiểm tra log hệ thống bằng lệnh:
journalctl -u synchttpd.service -n 50

---
Đây là thông báo tự động từ hệ thống.
"

# Send Email using mail command (s-nail on RHEL 9, mailx on RHEL 7/8)
# -r flag sets the "From" address
echo "$MESSAGE" | mail -s "$SUBJECT" -r "$EMAIL_FROM" "$EMAIL_TO"

if [ $? -eq 0 ]; then
    echo "Notification email sent to $EMAIL_TO"
else
    echo "Failed to send notification email."
fi
```

```bash
/etc/systemd/system/synchttpd.service
[Unit]
Description=Sync HTTPD Config
OnFailure=synchttpd-failure.service

[Service]
Type=oneshot
ExecStart=/usr/bin/flock -n /var/run/sync_httpd.lock /bin/bash /opt/scripts/sync.sh

User=root
Group=root
Nice=10
```

```bash
/etc/systemd/system/synchttpd.timer
[Unit]
Description=Run Sync HTTPD Config script at every hours

[Timer]
OnCalendar=hourly
Persistent=true
AccuracySec=1min
RandomizedDelaySec=5m
Unit=synchttpd.service

[Install]
WantedBy=timers.target
```

```bash
/opt/scripts/clear_large_logs.sh
#!/bin/bash
set -euo pipefail

# Định dạng ngày giờ: tránh ký tự ":" trong tên file
now="$(date +"%Y/%m/%d %H:%M:%S")"
filenow="$(date +"%Y%m%d_%H%M%S")"

# Thư mục log nguồn
APPVOL_LOG_DIR="/appvol/logs/httpd"   # Directory containing log files
VAR_LOG_DIR="/var/log/httpd"

# Thư mục và file log của script
SCRIPTS_LOG_DIR="/appvol/logs/clear_logs"
mkdir -p "$SCRIPTS_LOG_DIR"
synclog="$SCRIPTS_LOG_DIR/clear_log_${filenow}.log"

# Giới hạn size: 100MB = 104857600 bytes
LIMIT=104857600

# Bật nullglob để for glob không match thì trả về rỗng (tránh lỗi)
shopt -s nullglob

echo "$now Clear logs HTTPD on directories: $APPVOL_LOG_DIR and $VAR_LOG_DIR" | tee -a "$synclog"

# --- Hàm truncate nếu vượt ngưỡng ---
process_dir() {
  local dir="$1"
  local pattern="$2"  # ví dụ: "*.log" hoặc "*.*"
  local label="$3"

  if [ -d "$dir" ]; then
    echo "Checking log files in $dir..." | tee -a "$synclog"

    for f in "$dir"/$pattern; do
      # Chỉ xử lý file thường
      if [ -f "$f" ]; then
        # Lấy kích thước file
        local size
        size=$(stat -c%s "$f" 2>/dev/null || echo 0)

        if [ "$size" -gt "$LIMIT" ]; then
          echo "[$label] File $f exceeds $LIMIT bytes ($size). Truncating..." | tee -a "$synclog"
          : > "$f"  # truncate về 0 byte
          echo "[$label] Truncated: $f" | tee -a "$synclog"
        else
          echo "[$label] File $f size $size bytes. No action." | tee -a "$synclog"
        fi
      fi
    done
  else
    echo "Directory $dir does not exist!" | tee -a "$synclog"
  fi
}

# Xử lý hai thư mục log nguồn
process_dir "$APPVOL_LOG_DIR" "*.log" "APPVOL"
process_dir "$VAR_LOG_DIR" "*.*" "VARLOG"

# --- Clear các file log của script cũ hơn 1 ngày ---
# --- Delete script logs older than 1 day ---
if [ -d "$SCRIPTS_LOG_DIR" ]; then
  echo "Deleting script logs older than 1 day in $SCRIPTS_LOG_DIR..." | tee -a "$synclog"

  find "$SCRIPTS_LOG_DIR" \
    -type f \
    -name "clear_log_*.log" \
    -mtime +1 \
    -print -delete \
    | tee -a "$synclog"

  echo "Done deleting old script logs (>1 day)." | tee -a "$synclog"
else
  echo "Scripts log directory $SCRIPTS_LOG_DIR does not exist!" | tee -a "$synclog"
fi

echo "Completed at $(date +"%Y/%m/%d %H:%M:%S")" | tee -a "$synclog"
```

```bash
/etc/systemd/system/clearlogs.service
[Unit]
Description=Clear HTTPD logs exceeding size limit

[Service]
Type=oneshot
ExecStart=/usr/bin/flock -n /var/run/clear_httpd_logs.lock /bin/bash /opt/scripts/clear_large_logs.sh

User=root
Group=root
Nice=10
```

```bash
/etc/systemd/system/clearlogs.timer
[Unit]
Description=Run Clear HTTPD log script at every hours

[Timer]
OnCalendar=hourly
Persistent=true
AccuracySec=1min
RandomizedDelaySec=5m
Unit=clearlogs.service

[Install]
WantedBy=timers.target
```