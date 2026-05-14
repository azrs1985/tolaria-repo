# [XCP] Clone hệ điều hành từ Source VM (LVM) sang Destination VM (Standard Partition)

Owner: Nam Tran
Last edited time: March 5, 2026 11:42 PM

Đây là hướng dẫn chi tiết từng bước để clone hệ điều hành từ **Source VM (LVM)** sang **Destination VM (Standard Partition)**.

Vì cấu trúc đĩa thay đổi (từ LVM sang Partition thường), chúng ta không thể chỉ copy dữ liệu là xong. Quy trình bắt buộc phải bao gồm việc **sửa cấu hình boot** và **tạo lại initramfs**.

### Chuẩn bị

1. **Source VM:** Đang chạy bình thường.
2. **Destination VM:**
    - Khởi động bằng **Live CD** hoặc **RHEL Installation ISO** (chọn chế độ *Troubleshooting > Rescue a Red Hat Enterprise Linux system*).
    - Lý do: Bạn không thể format hay restore lên ổ đĩa `/` khi hệ điều hành đang chạy trên chính ổ đĩa đó.
    - Đảm bảo Destination VM đã có mạng và SSH service đang chạy (trong môi trường Rescue).
    - Kích hoạt mạng trong Rescue Mode (đối với RHEL 8)
        
        Cấu hình IP thủ công (Ví dụ đặt IP là 192.168.1.101):
        
        ```bash
        ip addr add 192.168.1.101/24 dev eth0
        ip link set eth0 up
        ```
        

---

### Bước 1: Chia đĩa và Format trên Destination VM

*(Thực hiện tại môi trường Rescue của Destination VM)*

Giả sử ổ đĩa mới là `/dev/xvda`. Chúng ta sẽ tạo 3 phân vùng giống cấu trúc cơ bản: `/boot`, `swap`, và `/`.

1. **Dùng fdisk tạo partition:**
    - `/dev/xvda1`: 1GB (cho `/boot`)
    - `/dev/xvda2`: Dung lượng còn lại (cho `/`)
2. **Format và kích hoạt:**
    
    ```bash
    mkfs.xfs -f /dev/xvda1        # Boot
    mkfs.xfs -f /dev/xvda2        # Root
    
    ```
    
3. **Mount hệ thống file để chuẩn bị nhận dữ liệu:**
    
    ```bash
    mkdir -p /mnt/target
    mount /dev/xvda2 /mnt/target          # Mount Root trước
    mkdir -p /mnt/target/boot
    mount /dev/xvda1 /mnt/target/boot     # Mount Boot vào trong Root
    
    ```
    

---

### Bước 2: Clone dữ liệu từ Source sang Destination

*(Chạy lệnh này từ **Source VM**)*

Giả sử IP của Destination VM (đang ở Rescue mode) là `192.168.1.100`.

1. **Clone phân vùng Root (/):***Lưu ý: Loại bỏ thư mục /boot ra khỏi lần dump này nếu /boot nằm riêng, nhưng xfsdump thường dump theo device. Nếu Source VM có /boot là partition riêng (ví dụ xvda1) và root là LVM, ta dump root LVM trước.*
    
    ```bash
    # Dump Logical Volume Root sang partition xvda3 bên kia
    ssh root@192.168.1.100 "xfsdump -l 0 -J - /" | xfsrestore -J - /mnt/target
    
    ```
    
2. **Clone phân vùng Boot (/boot):***(Nếu Source VM có phân vùng /boot riêng)*
    
    ```bash
    ssh root@192.168.1.100 "xfsdump -l 0 -J - /boot" | xfsrestore -J - /mnt/target/boot
    ```
    
    **Giải thích lệnh:**
    
    - `ssh root@192.168.1.100`: Kết nối sang máy cũ.
    - `xfsdump -l 0 -J - /`: Backup mức 0, tham số `J` để tắt ghi log (tránh lỗi file log), dấu `-` để xuất dữ liệu ra chuẩn `stdout` thay vì lưu thành file.
    - `|`: Chuyển dữ liệu vừa xuất ra sang máy mới qua mạng.
    - `xfsrestore -J - /mnt/target`: Nhận dữ liệu từ `stdin` và bung trực tiếp vào thư mục `/mnt/target` (phân vùng mới).

---

### Bước 3: Chroot và Sửa cấu hình (QUAN TRỌNG NHẤT)

*(Thực hiện tại **Destination VM**)*

Sau khi copy dữ liệu xong, bạn cần vào bên trong hệ thống mới để chỉnh sửa.

1. **Mount các hệ thống file ảo:**
Để các lệnh như `grub2-mkconfig` hay `dracut` chạy được, cần mount `/dev`, `/proc`, `/sys`.
    
    ```bash
    mount --bind /dev /mnt/target/dev
    mount --bind /proc /mnt/target/proc
    mount --bind /sys /mnt/target/sys
    ```
    
2. **Chroot vào hệ thống mới:**
    
    ```bash
    chroot /mnt/target
    ```
    
    *Bây giờ bạn đang đứng "bên trong" ổ cứng của VM mới.*
    
3. **Lấy UUID mới của các phân vùng:**
Gõ lệnh `blkid` và copy lại UUID của `/dev/xvda1` (boot), `/dev/xvda2` (root).
    
    ```bash
    blkid
    ```
    
4. **Sửa file `/etc/fstab`:**
    
    ```bash
    vi /etc/fstab
    ```
    
    - Xóa các dòng cũ tham chiếu đến `/dev/mapper/rhel_dns...`.
    - Thêm mới (hoặc sửa) bằng UUID vừa lấy:
    
    ```
    UUID=xxxxx-xxxx-xxxx-xxxx /                       xfs     defaults        0 0
    UUID=yyyyy-yyyy-yyyy-yyyy /boot                   xfs     defaults        0 0
    ```
    
5. **Sửa file `/etc/default/grub`:**
    
    ```bash
    vi /etc/default/grub
    ```
    
    Tìm dòng `GRUB_CMDLINE_LINUX`, sửa thành dạng Standard Partition:
    
    - Xóa: `rd.lvm.lv=...` (xóa hết các đoạn liên quan đến lvm)
    - Sửa `resume=...`: Trỏ về UUID của partition SWAP mới.
    - Thêm/Sửa `root=...`: Trỏ về UUID của partition ROOT mới (thường Grub tự nhận, nhưng tốt nhất nên xóa các tham số LVM đi).
    
    *Ví dụ sau khi sửa:*
    
    ```bash
    GRUB_CMDLINE_LINUX="crashkernel=auto root=UUID=xxxxx-xxxx-xxxx-xxxx rhgb quiet"
    
    ```
    
6. **Tạo lại Initramfs (Dracut):**
Đây là bước bắt buộc để Kernel biết cách boot mà không cần LVM.
    
    ```bash
    # Backup file cũ cho chắc
    cp /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
    
    # Tạo lại initramfs mới, force ghi đè
    dracut -f
    ```
    
    *Lưu ý: Nếu phiên bản kernel đang chạy của Rescue CD khác với kernel đã cài trong ổ cứng, bạn cần chỉ định rõ phiên bản kernel. Xem version bằng `ls /lib/modules` và điền vào lệnh dracut: `dracut -f /boot/initramfs-4.18.0-xxx.el8.x86_64.img 4.18.0-xxx.el8.x86_64`*
    
7. **Cài đặt và Update Grub Bootloader:**
    - **Nếu boot chuẩn Legacy (BIOS):**
        
        ```bash
        grub2-mkconfig -o /boot/grub2/grub.cfg
        grub2-install /dev/xvda
        
        ```
        
    - **Nếu boot chuẩn UEFI:**
        
        ```bash
        grub2-mkconfig -o /boot/efi/EFI/redhat/grub.cfg
        # Cần đảm bảo partition EFI đã được mount nếu dùng UEFI
        
        ```
        
8. **Xử lý SELinux (Để tránh không login được):**
Vì file mới được tạo lại, nhãn SELinux có thể bị sai.
    
    ```bash
    touch /.autorelabel
    
    ```
    

### Bước 4: Hoàn tất

1. Thoát chroot: `exit`
2. Unmount:
    
    ```bash
    umount /mnt/target/boot
    umount /mnt/target/sys
    umount /mnt/target/proc
    umount /mnt/target/dev
    umount /mnt/target
    
    ```
    
3. Reboot VM: `reboot`