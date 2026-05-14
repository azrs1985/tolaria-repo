# [LINUX] Tăng kích thước phân vùng tiêu chuẩn / Mở rộng phân vùng gốc bằng fdisk

Owner: Nam Tran
Last edited time: March 8, 2026 3:07 PM

![](https://tekneed.com/wp-content/uploads/2020/06/how-to-increase-the-root-filesystem-standard-partition-using-fdisk.jpg)

Học quy trình từng bước để tăng kích thước phân vùng tiêu chuẩn và mở rộng phân vùng gốc bằng fdisk trực tuyến trên Linux mà không gây gián đoạn hoặc mất dữ liệu

Trong một trong các bài viết trên trang web này, tôi đã giải thích về[**phân vùng tiêu chuẩn**](https://tekneed.com/storage-management-in-linux-explained-with-examples/#what-is-standard-partition-in-linux)và[**phân vùng LVM**](https://tekneed.com/understanding-lvm-with-examples-advantages-of-lvm/#what-is-lvm-in-linux). Tôi cũng đã giải thích quy trình từng bước[**để mở rộng và thu nhỏ phân vùng LVM.**](https://tekneed.com/how-to-extend-or-reduce-lvm-partition-in-linux/)

Trong bài viết này, chúng ta sẽ tìm hiểu cách mở rộng/tăng kích thước phân vùng tiêu chuẩn trực tuyến (không gián đoạn) mà không mất dữ liệu và sẽ sử dụng phân vùng gốc (/) làm ví dụ.

[**Đề xuất: Câu hỏi và câu trả lời luyện thi RHCSA 8/EX200**](https://tekneed.com/rhcsa-8-ex200-exam-practice-question-answer-pdf-2021/)

[**Đề xuất: Câu hỏi và câu trả lời thực hành cho kỳ thi RHCE 8/EX294**](https://tekneed.com/rhce-8-ex294-exam-practice-question-answer-collections/)

[**Bài viết đề xuất: Quản lý lưu trữ trong Linux được giải thích bằng ví dụ**](https://tekneed.com/storage-management-in-linux-explained-with-examples/)

---

## **Cách tăng/mở rộng phân vùng gốc (/) trong Linux bằng công cụ fdisk**

LƯU Ý 1: Sao lưu hệ thống của bạn nếu có thể. Nếu đó là máy ảo (VM) trên Azure hoặc bất kỳ nhà cung cấp dịch vụ đám mây nào khác, hãy tạo bản sao lưu (snapshot) của đĩa hệ điều hành.

LƯU Ý 2: Lý do sao lưu là để khôi phục lại nếu có sự cố xảy ra. Nếu hệ thống tệp của bạn khỏe mạnh, rất hiếm khi xảy ra vấn đề. Các bước này là những bước tôi thường thực hiện và đã sử dụng trong môi trường sản xuất trước đây và thành công. Vì vậy, đừng lo lắng.

LƯU Ý 3: Khi bạn sử dụng tùy chọn “d” để xóa phân vùng và sau đó sử dụng tùy chọn “n”, điều bạn đang làm thực chất là tạo bảng phân vùng và nó vẫn ở trong bộ nhớ, không xóa toàn bộ phân vùng, vì vậy đừng lo lắng.

## **Quy trình từng bước**

### **1. Kiểm tra kích thước hệ thống tệp gốc (/)**

![](https://tekneed.com/wp-content/uploads/2020/06/image-9.png)

```
[root@Tekneed ~]# lsblk

NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   15G  0 disk
├─sda1   8:1    0  238M  0 part /boot
├─sda2   8:2    0  200M  0 part [SWAP]
└─sda3   8:3    0  6.5G  0 part /
sr0     11:0    1  7.3G  0 rom  /run/media/root/RHEL-8-1-0-BaseOS-x86_64
```

### **2. Kiểm tra loại hệ thống tệp gốc (/)**

```
[root@Tekneed ~]# lsblk -fs /dev/sda3

NAME  FSTYPE LABEL UUID                                 MOUNTPOINT
sda3  xfs          0ba4bfe5-9f93-4725-bea2-b0d9c5175bbf /
└─sda
```

Bạn có thể thấy rằng loại hệ thống tệp gốc là “xfs”

### **3. Khởi tạo /dev/sda bằng công cụ fdisk**

```
[root@Tekneed ~]# fdisk /dev/sda

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
```

**Nhập chữ cái “p” để hiển thị tất cả các phân vùng trên sda**

```
Command (m for help): p
Disk /dev/sda: 15 GiB, 16106127360 bytes, 31457280 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x9550a04d

Device     Boot  Start      End  Sectors  Size Id Type
/dev/sda1  *      2048   489471   487424  238M 83 Linux
/dev/sda2       489472   899071   409600  200M 82 Linux swap / Solaris
/dev/sda3       899072 14569471 13670400  6.5G 83 Linux

Command (m for help):
```

**Nhập chữ cái “d” để xóa một phân vùng**

```
Command (m for help): d
Partition number (1-3, default 3):
```

**Nhập số phân vùng 3 hoặc nhấn Enter để giữ nguyên giá trị mặc định là 3**

```
Partition 3 has been deleted.
```

**BÂY GIỜ, NHẬP CHỮ “n” ĐỂ TÁI TẠO PHÂN VÙNG VỚI KÍCH THƯỚC THEO Ý MUỐN**

```
Command (m for help): n
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p):
```

**Nhập chữ cái “p” để thiết lập phân vùng chính. Phân vùng của bạn có thể là phân vùng thứ cấp tùy thuộc vào số lượng phân vùng bạn có. Một đĩa chỉ có thể có tối đa bốn phân vùng chính**

```
Select (default p): p
Partition number (3,4, default 3):
```

**Nhập số phân vùng là 3 hoặc nhấn Enter để giữ nguyên mặc định là 3**

```
Partition number (3,4, default 3):
First sector (899072-31457279, default 899072):
```

**Nhấn phím Enter lần nữa để đến sector cuối cùng**

```
Last sector, +sectors or +size{K,M,G,T,P} (899072-31457279, default 31457279):
```

**Nhập kích thước phân vùng mới hoặc nhấn phím Enter để sử dụng toàn bộ không gian trống trên sda. Trong trường hợp này, chúng ta sử dụng toàn bộ không gian trống**

```
Last sector, +sectors or +size{K,M,G,T,P} (899072-31457279, default 31457279):

Created a new partition 3 of type 'Linux' and of size 14.6 GiB.
Partition #3 contains a xfs signature.

Do you want to remove the signature? [Y]es/[N]o:
```

**Nhập “no” để không xóa chữ ký**

```
Do you want to remove the signature? [Y]es/[N]o: no

Command (m for help):
```

**Nhập chữ “w” để ghi hoặc lưu các thay đổi và nhấn Enter**

```
Command (m for help): w
The partition table has been altered.
Syncing disks.
```

[**Bài viết bổ sung: Cách bảo vệ đĩa/hệ thống tệp trong Linux**](https://tekneed.com/how-to-encrypt-a-drive-in-linux-secure-a-filesystem/)

### **4. Kiểm tra sự gia tăng của phân vùng**

```
[root@Tekneed ~]# lsblk

NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sda      8:0    0   15G  0 disk
├─sda1   8:1    0  238M  0 part /boot
├─sda2   8:2    0  200M  0 part [SWAP]
└─sda3   8:3    0 14.6G  0 part /
sr0     11:0    1  7.3G  0 rom  /run/media/root/RHEL-8-1-0-BaseOS-x86_64
[root@Tekneed ~]#
```

![](https://tekneed.com/wp-content/uploads/2020/06/how-to-increase-the-root-filesystem-standard-partition-using-fdisk.jpg)

### **5. Thay đổi kích thước hệ thống tệp,**

Nếu hệ thống tệp là XFS, sử dụng lệnh,

```
[root@Tekneed ~]# xfs_growfs /dev/sda3

xfs_growfs: /dev/sda3 is not a mounted XFS filesystem
```

Nếu gặp lỗi trên, sử dụng lệnh,

```
[root@Tekneed ~]# xfs_growfs /

meta-data=/dev/sda3              isize=512    agcount=9, agsize=427200 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1,
.........
```

Nếu hệ thống tệp là ext (2,3,4), hãy sử dụng “resize2fs” thay thế

```
[root@Tekneed ~]# resize2fs /dev/sda3
```

```
[root@Tekneed ~]# resize2fs /
```

### **6. Bạn cũng có thể chạy lệnh dưới đây để áp dụng thay đổi ngay lập tức cho kernel**

```
[root@Tekneed ~]# partprobe
```

### **7. Kiểm tra kích thước mới của hệ thống tệp**

```
[root@Tekneed ~]# df -h

Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        887M     0  887M   0% /dev
tmpfs           904M     0  904M   0% /dev/shm
tmpfs           904M  9.7M  894M   2% /run
tmpfs           904M     0  904M   0% /sys/fs/cgroup
/dev/sda3        15G  4.2G   11G  29% /
/dev/sda1       233M  150M   84M  65% /boot
tmpfs           181M  1.2M  180M   1% /run/user/42
tmpfs           181M  4.6M  177M   3% /run/user/0
/dev/sr0        7.4G  7.4G     0 100% /run/media/root/RHEL-8-1-0-BaseOS-x86_64
[root@Tekneed ~]#
```

![extend root partition using fdisk](https://tekneed.com/wp-content/uploads/2020/06/extend-standard-partition-in-Linux-online-without-data-loss.jpg)

extend root partition using fdisk

Bạn có thể thấy kích thước đã tăng lên.