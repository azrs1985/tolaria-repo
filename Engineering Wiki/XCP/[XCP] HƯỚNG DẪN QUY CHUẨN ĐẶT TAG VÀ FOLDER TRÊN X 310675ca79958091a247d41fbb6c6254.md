# [XCP] HƯỚNG DẪN QUY CHUẨN ĐẶT TAG VÀ FOLDER TRÊN XCP-NG

Owner: Nam Tran
Last edited time: March 18, 2026 3:47 PM

## 1. Mục đích

- Tăng khả năng nhận diện chức năng của VM ngay lập tức.
- Hỗ trợ lọc (filter) và tìm kiếm nhanh trên Xen Orchestra (XO).
- Phân quyền quản trị theo nhóm folder.
- Tự động hóa các tác vụ backup và script dựa trên Tag.

---

## 2. Quy định đặt tên VM (VM Naming Convention)

Trước khi phân loại vào Folder, bản thân tên VM phải tuân thủ cấu trúc:

**[Môi trường]-[Dịch vụ]-[Số thứ tự]**

- **Môi trường (Env):** PROD (Production), STAG (Staging), DEV (Development), TEST.
- **Dịch vụ (Service):** Viết tắt chức năng (ví dụ: DB, WEB, K8S-Master, APP, LB).
- **Số thứ tự:** Luôn dùng 2 chữ số (01, 02...).

> **Ví dụ:** PROD-PostgreSQL-01, DEV-XWiki-01,...
> 

---

## 3. Cấu trúc cây thư mục (Folder Structure)

Folder trong XCP-ng giúp gom nhóm các tài nguyên có cùng mục đích sử dụng hoặc thuộc cùng một dự án.

### Cấu trúc đề xuất:

- **📂 Infrastructure:** Chứa các VM cốt lõi (AD, DNS, Mail Server, Firewall).
- **📂 Production:** Chứa các hệ thống đang chạy thực tế.
- **📂 Development/Testing:** Các môi trường thử nghiệm.
- **📂 Templates:** Nơi chứa các bản Snapshot hoặc VM gốc để nhân bản.
- **📂 Decommissioned:** Các VM đã tắt, chờ xóa hoặc lưu trữ.

---

## 4. Hệ thống Tag (Tagging Strategy)

Tag là công cụ mạnh mẽ nhất trong Xen Orchestra để lọc và chạy backup. Một VM nên có ít nhất 3 loại Tag sau:

### 4.1. Tag theo Nhóm kỹ thuật (Technical Tags)

Dùng để xác định hệ điều hành hoặc công nghệ.

- Ubuntu-22.04, RHEL-9, Windows-2022.
- Docker, K8s, Database.

### 4.2. Tag theo Chính sách Backup (Backup Tags)

Sử dụng các Tag này để các **Backup Jobs** trong Xen Orchestra tự động nhận diện VM.

- Backup-Daily: Sao lưu hàng ngày.
- Backup-Weekly: Sao lưu hàng tuần.
- No-Backup: Các VM tạm, không cần lưu trữ.

### 4.3. Tag theo Trạng thái bảo mật (Security Tags)

- Hardened: VM đã được cấu hình bảo mật (ví dụ: RHEL 9 minimal + AIDE).
- DMZ: VM nằm trong vùng mạng công cộng.
- Internal: VM chỉ kết nối nội bộ.

---

## 5. Bảng tổng hợp ví dụ

| **Tên VM** | **Folder** | **Tags** |
| --- | --- | --- |
| PROD-K8S-Worker-01 | Production/K8S-Cluster | Ubuntu-22.04, K8s, Backup-Daily |
| STAG-Oracle-02 | Development | RHEL-8, Database, No-Backup |
| PROD-Proxy-01 | Infrastructure | Hardened, DMZ, Backup-Weekly |

---

## 6. Quy trình thực hiện (Best Practices)

1. **Luôn viết thường (Lowercase):** Đối với Tag, nên sử dụng chữ thường hoặc kebab-case (ví dụ: web-server) để tránh nhầm lẫn khi gõ tìm kiếm.
2. **Gắn Tag ngay khi khởi tạo:** Không bao giờ để một VM "không tên, không tag".
3. **Dọn dẹp định kỳ:** Hàng tháng, quản trị viên kiểm tra folder Decommissioned để xóa vĩnh viễn các VM không còn giá trị.