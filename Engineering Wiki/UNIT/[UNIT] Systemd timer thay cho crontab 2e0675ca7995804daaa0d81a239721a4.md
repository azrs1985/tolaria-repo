# [UNIT] Systemd timer thay cho crontab

Owner: Nam Tran
Last edited time: January 9, 2026 9:52 AM

[Clear logs HTTPD](%5BUNIT%5D%20Systemd%20timer%20thay%20cho%20crontab/Clear%20logs%20HTTPD%202e0675ca799580e6bf0bcd4eb73a5917.md)

[Sync HTTPD Config](%5BUNIT%5D%20Systemd%20timer%20thay%20cho%20crontab/Sync%20HTTPD%20Config%202e0675ca799580888c31d2f6714dc89e.md)

`systemd timer` **ổnđịnh, dễ quản lý hơn cron**, có log rõ ràng và không bị mất job khi server shutdown. Dưới đây là **cách chuyển crontab sang systemd timer**, từng bước, đúng chuẩn production.

---

# ✅ Dùng `systemd timer` thay cho `cron`

## 🎯 Mục tiêu

Chạy script:

```
/opt/scripts/clear_httpd_log.sh

```

vào **00:00 và 12:00 mỗi ngày**

---

## 🧱 1. Tạo file **service**

### 📄 `/etc/systemd/system/clear-httpd-log.service`

```bash
[Unit]
Description=Clear HTTPD logs exceeding size limit

[Service]
Type=oneshot
ExecStart=/bin/bash /opt/scripts/clear_httpd_log.sh

# (tuỳ chọn) chạy bằng root
User=root
Group=root

# (tuỳ chọn) môi trường an toàn
Nice=10
```

💡 `Type=oneshot` → task chạy xong là kết thúc (đúng cho script bash)

---

## ⏱️ 2. Tạo file **timer**

### 📄 `/etc/systemd/system/clear-httpd-log.timer`

```bash
[Unit]
Description=Run Clear HTTPD log script at 00:00 and 12:00 daily

[Timer]
OnCalendar=*-*-* 00:00:00
OnCalendar=*-*-* 12:00:00
Persistent=true

[Install]
WantedBy=timers.target
```

### Giải thích nhanh

| Tham số | Ý nghĩa |
| --- | --- |
| `OnCalendar` | Lịch chạy (giống cron nhưng dễ đọc) |
| `Persistent=true` | Server tắt lúc 00h → bật lên sẽ chạy bù |
| `timers.target` | Group timers chuẩn của systemd |

---

## 🚀 3. Reload & enable timer

`systemctl daemon-reexec`

`systemctl daemon-reload`

`systemctl enable --now clear-httpd-log.timer`

✅ `--now` → bật timer ngay không cần reboot

---

## 🔍 4. Kiểm tra timer

### Xem timer có active không

`systemctl list-timers --all | grep clear-httpd`

Ví dụ output:

```
NEXT                         LEFT       LAST                         PASSED     UNIT                      ACTIVATES
2026-01-07 00:00:00 ICT      10h left   2026-01-06 12:00:01 ICT      1h ago     clear-httpd-log.timer    clear-httpd-log.service

```

---

## 📜 5. Xem log (điểm mạnh hơn cron 🚀)

`journalctl -u clear-httpd-log.service`

Hoặc theo thời gian:

`journalctl -u clear-httpd-log.service --since today`

✅ **Không cần redirect `>> logfile` như cron**

---

## 🔐 6. Chống chạy chồng (flock – RẤT KHUYẾN NGHỊ)

Sửa `ExecStart` trong file `.service`:

`ExecStart=/usr/bin/flock -n /var/run/clear*httpd*log.lock /bin/bash /path/to/clear*httpd*log.sh`

✅ Job đang chạy → lần kế tiếp **skip**, không treo

---

## 🧪 7. Test thủ công

`systemctl start clear-httpd-log.service`

Kiểm tra:

`journalctl -u clear-httpd-log.service -n 50`

---

## ✅ So sánh: cron vs systemd timer

| Tiêu chí | cron | systemd timer |
| --- | --- | --- |
| Log | rời rạc | `journalctl` |
| Chạy bù khi server down | ❌ | ✅ |
| Quản lý | khó | `systemctl` |
| Dependency | ❌ | ✅ |
| Production ready | ⚠️ | ✅ |

---

## ✅ Gỡ cron cũ (nếu có)

`crontab -e`

# Xóa dòng cũ

---

## 📌 Tóm tắt

✔ Tạo `.service` để chạy script\
✔ Tạo `.timer` thay cron\
✔ Chạy **00h & 12h hằng ngày**\
✔ Có log, chạy bù, không chồng job\
✔ Chuẩn enterprise Linux

---