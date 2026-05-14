# [UNIT] Giới hạn dung lượng log bằng tay

Owner: Nam Tran
Last edited time: November 25, 2025 11:27 AM

Bạn muốn **giới hạn dung lượng log file tối đa 100MB** trên Ubuntu. Có hai cách phổ biến:

---

### 1. Với **systemd journalctl** (áp dụng cho log hệ thống)

Systemd quản lý log qua `journalctl`. Bạn có thể giới hạn dung lượng bằng lệnh:

```bash
sudo journalctl --vacuum-size=100M

```

- Lệnh này sẽ tự động xóa log cũ, chỉ giữ lại tổng dung lượng log ≤ 100MB.
- Có thể chạy thủ công hoặc đặt cron job để chạy định kỳ.

Nếu muốn cấu hình cố định, chỉnh file cấu hình:

```bash
sudo nano /etc/systemd/journald.conf

```

Thêm hoặc sửa dòng:

```
SystemMaxUse=100M

```

Sau đó khởi động lại dịch vụ:

```bash
sudo systemctl restart systemd-journald

```

---

### 2. Với **log file thường** (ví dụ `/var/log/syslog`)

Nếu bạn muốn truncate thủ công file log về dung lượng ≤ 100MB:

```bash
sudo truncate -s 100M /var/log/syslog

```

- `s 100M` sẽ cắt file về đúng 100MB.
- Nếu file nhỏ hơn 100MB thì giữ nguyên, nếu lớn hơn thì bị cắt ngắn.

---

### ⚠️ Lưu ý

- **Không nên xóa file log** trực tiếp, vì nhiều dịch vụ cần file tồn tại.
- Với log hệ thống (`journalctl`), nên dùng cách 1 để đảm bảo an toàn.
- Với log ứng dụng riêng, có thể dùng `truncate` hoặc thiết lập **logrotate** để tự động xoay vòng và giới hạn dung lượng.

---

👉 Bạn muốn mình hướng dẫn chi tiết cách **cấu hình logrotate** để tự động giới hạn log file 100MB, thay vì phải truncate thủ công không?