# [LINUX] Cách khôi phục GRUB trên RHEL

Owner: Nam Tran
Last edited time: March 7, 2026 9:23 AM

Bài viết này sẽ hướng dẫn chi tiết các bước để khôi phục GRUB trên RHEL 7 / CentOS 7. Hỏng hóc hoặc mất GRUB là một trong những vấn đề phổ biến nhất trên các máy chủ/máy trạm Linux. Nguyên nhân có thể do đĩa cứng hỏng, lỗi firmware hoặc tắt hệ thống đột ngột. Hệ thống được cài đặt với firmware BIOS/UEFI và bạn cần biết hệ điều hành hiện tại đang sử dụng. Dưới đây là cách xác định[BIOS so với UEFI trên máy chủ Linux.](https://unixarena.com/2018/05/how-to-find-linux-is-under-bios-or-uefi-mode.html)Để khôi phục GRUB trên RHEL 7 / CentOS 7, bạn phải có đĩa DVD hoặc tệp ISO mới nhất.

Lỗi thường gặp trong GRUB:

Nếu GRUB bị hỏng hoặc mất, hệ thống sẽ không khởi động được và sẽ bị kẹt ở màn hình GRUB như sau.

```
GNU GRUB version 0.97 (638K lower / 3143616K uper memory)

[ Minimal BASH-like line editing is supported. For the first word. TAB
lists possible command completions. Anywhere else TAB lists the possible
completions of a device/filename.]

grub>
```

Hệ thống sẽ dừng lại với thông báo sau.

```
GRUB loading stage 2
```

Trong trường hợp này, vui lòng làm theo các hướng dẫn dưới đây.

# **Khôi phục/Phục hồi GRUB – Hệ thống dựa trên BIOS:**

1. Cắm đĩa DVD RHEL 7 / CentOS 7 mới nhất vào máy chủ hoặc kết nối tệp ISO thông qua ILO.

2. Trong trường hợp máy ảo, kết nối tệp ISO với máy ảo.

3. Khởi động máy chủ bằng đĩa DVD/hình ảnh ISO.

4. Chọn tùy chọn khắc phục sự cố sau khi hệ thống đã khởi động từ DVD/ISO.

![**RHEL7 CentOS7 – Chế độ khắc phục sự cố**](https://unixarena.com/wp-content/uploads/2018/05/RHEL7-CentOS7-Troubleshooting-Rescue-Mode.jpg)

**RHEL7 CentOS7 – Chế độ khắc phục sự cố**

5. Chọn chế độ cứu hộ.

![**RHEL 7 – Chế độ Khôi phục**](https://unixarena.com/wp-content/uploads/2018/05/RHEL-7-Rescue-Mode.jpg)

**RHEL 7 – Chế độ Khôi phục**

6. Nhấn phím 1 để tiếp tục tìm hình ảnh hệ điều hành và nhận dấu nhắc lệnh.

![**Tiếp tục – Chế độ cứu hộ – RHEL**](https://unixarena.com/wp-content/uploads/2018/05/Continue-Rescue-Mode.jpg)

**Tiếp tục – Chế độ cứu hộ – RHEL**

7. Chuyển sang chế độ chroot vào hình ảnh hệ điều hành.

![chroot to OS Image - RHEL](https://unixarena.com/wp-content/uploads/2018/05/chroot-to-OS-Image.jpg)

**chroot vào hình ảnh hệ điều hành – RHEL**

8. Bắt đầu từ RHEL 7/CentOS 7, GRUB 2 là trình tải khởi động mặc định. Tệp cấu hình GRUB 2 là**/boot/grub2/grub.cfg. Cài đặt**trình tải khởi động GRUB trên đĩa gốc. (Mặc định: /dev/sda).

![grub2-install - RHEL 7 CentOS7](https://unixarena.com/wp-content/uploads/2018/05/grub2-install-RHEL-7-CentOS7.jpg)

**grub2-install – RHEL 7 CentOS7**

9. Truy cập vào thư mục /boot/grub2 và xác nhận sự tồn tại của tệp “grub.cfg”. Ở đây, tệp “grub.cfg” không tồn tại.

![grub.cfg is missing - RHEL 7](https://unixarena.com/wp-content/uploads/2018/05/grub.cfg-is-missing-RHEL-7.jpg)

**grub.cfg bị thiếu – RHEL 7**

10. Hãy tạo tệp “grub.cfg”.

![grub2-mkconfig - Recreate grub.cfg](https://unixarena.com/wp-content/uploads/2018/05/grub2-mkconfig-Recreate-grub.cfg_.jpg)

**grub2-mkconfig – Tạo lại grub.cfg**

Nếu bạn thiếu lệnh grub2-mkconfig, hãy cài đặt gói “grub2-tools.x86_64” trong chế độ cứu hộ. Vì bạn đã khởi động từ DVD, gói này nên có sẵn trong đó.

![Grub.cfg recreated using grub2-mkconfig](https://unixarena.com/wp-content/uploads/2018/05/Grub.cfg-recreated-using-grub2-mkconfig.jpg)

Tạo lại grub.cfg bằng grub2-mkconfig

11. Thoát khỏi chroot và khởi động lại hệ thống.

Điều này chỉ hoạt động trên các máy chủ X86 dựa trên BIOS và máy ảo, nhưng không hoạt động với các máy chủ và máy ảo sử dụng firmware UEFI.

[Dưới đây là liên kết để xác định loại firmware được sử dụng để khởi động hệ thống.](https://unixarena.com/2018/05/how-to-find-linux-is-under-bios-or-uefi-mode.html)