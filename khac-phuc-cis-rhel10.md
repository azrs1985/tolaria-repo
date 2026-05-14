---
type: Note
---

# Khắc phục các vấn đề CIS trên Red Hat Enterprise Linux 10 (RHEL 10)

Tài liệu này hướng dẫn cách quét và khắc phục các vấn đề bảo mật theo tiêu chuẩn CIS (Center for Internet Security) trên hệ điều hành Red Hat Enterprise Linux 10.

## 1. Tổng quan
CIS Benchmark cho RHEL 10 cung cấp các hướng dẫn cấu hình bảo mật tối ưu. RHEL 10 hỗ trợ sẵn các cấu hình này thông qua gói `scap-security-guide`.

## 2. Kiểm tra mức độ tuân thủ với OpenSCAP

Trước khi khắc phục, bạn cần biết hệ thống đang vi phạm những lỗi nào.

### Cài đặt công cụ
```bash
sudo dnf install openscap-scanner scap-security-guide
```

### Quét hệ thống
Để thực hiện quét và tạo báo cáo HTML:
```bash
oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_cis \
--report cis-report.html \
/usr/share/xml/scap/ssg/content/ssg-rhel10-ds.xml
```
*Sau khi chạy, bạn có thể mở file `cis-report.html` để xem chi tiết các mục không đạt.*

## 3. Các bước khắc phục phổ biến (Thủ công)

### 3.1 Cấu hình Filesystem
Hạn chế quyền ghi và thực thi trên các phân vùng tạm:
- Chỉnh sửa `/etc/fstab` để thêm các tùy chọn `nodev`, `nosuid`, và `noexec` cho `/tmp`, `/dev/shm`.

### 3.2 Cấu hình SSH
Chỉnh sửa file `/etc/ssh/sshd_config`:
- `PermitRootLogin no` (Cấm đăng nhập root trực tiếp)
- `MaxAuthTries 4` (Giới hạn số lần thử đăng nhập)
- `Protocol 2` (Sử dụng giao thức an toàn)

### 3.3 Chính sách mật khẩu
Cấu hình trong `/etc/security/pwquality.conf`:
- `minlen = 14` (Độ dài tối thiểu 14 ký tự)
- `dcredit = -1`, `ucredit = -1`, `ocredit = -1`, `lcredit = -1` (Yêu cầu chữ hoa, chữ thường, số và ký tự đặc biệt)

## 4. Khắc phục tự động bằng Ansible

Red Hat cung cấp các Playbook để tự động hóa việc sửa lỗi.

1. Tìm kiếm Remediation Roles:
   Các file remediation thường nằm trong `/usr/share/scap-security-guide/ansible/`.

2. Chạy Playbook:
```bash
ansible-playbook -i localhost, -c local rhel10-playbook-cis.yml
```

## 5. Lưu ý quan trọng về FIPS Mode
Trên RHEL 10, **chế độ FIPS phải được bật ngay tại thời điểm cài đặt hệ điều hành** bằng cách thêm tham số `fips=1` vào kernel boot. RHEL 10 không hỗ trợ bật FIPS sau khi đã cài đặt xong.

## 6. Tài liệu tham khảo
- [Trang chủ CIS Security](https://www.cisecurity.org/)
- [Red Hat Customer Portal - Security Hardening](https://access.redhat.com/)

---
*Ghi chú: Luôn thực hiện sao lưu cấu hình trước khi áp dụng các thay đổi bảo mật.*
