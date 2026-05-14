# [LINUX] Increase Standard / Extend Root Partition Using fdis

Owner: Nam Tran
Last edited time: March 8, 2026 3:06 PM

![](https://tekneed.com/wp-content/uploads/2020/06/how-to-increase-the-root-filesystem-standard-partition-using-fdisk.jpg)

Learn the step by step process of how to increase a standard partition & extend root partition using fdisk online in Linux without downtime or losing data

In one of the articles on this site, I have explained what a [**standard partition**](https://tekneed.com/storage-management-in-linux-explained-with-examples/#what-is-standard-partition-in-linux) and a [**LVM partition**](https://tekneed.com/understanding-lvm-with-examples-advantages-of-lvm/#what-is-lvm-in-linux) are. I have also explained the step by step process of [**how to extend and reduce an LVM partition.**](https://tekneed.com/how-to-extend-or-reduce-lvm-partition-in-linux/)

In this article, we will look at how to extend/increase a standard partition online (no downtime) without losing data and we will use the root partition (/) as an example.

[**Suggested: RHCSA 8/EX200 Exam Practice Questions&Answers**](https://tekneed.com/rhcsa-8-ex200-exam-practice-question-answer-pdf-2021/)

[**Suggested: RHCE 8/EX294 Exam Practice Questions&Answers**](https://tekneed.com/rhce-8-ex294-exam-practice-question-answer-collections/)

[**Suggested Article : Storage Management In Linux Explained With Examples**](https://tekneed.com/storage-management-in-linux-explained-with-examples/)

---

## **How To Increase / Extend The Root (/) Partition In Linux Using The fdisk Utility**

NOTE 1: Take a backup of your system if you can. If it a VM on Azure or any other cloud services provider, take the snapshot of the OS disk

NOTE 2: The reason for the backup is to roll back if anything goes wrong. If your filesystem is healthy, it is very rare for an issue to occur. These steps are the steps I usually take and have used in a production environment before and was successful. So, no worries.

NOTE 3: When you use the “d” option to delete the partition and use the “n” option afterward, what you are doing is actually creating a partition table and remains in memory and not deleting the whole partition, so no worries.

## **Step By Step Process**

### **1. verify the root (/) filesystem size**

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

### **2. verify the root(/) filesystem type**

```
[root@Tekneed ~]# lsblk -fs /dev/sda3

NAME  FSTYPE LABEL UUID                                 MOUNTPOINT
sda3  xfs          0ba4bfe5-9f93-4725-bea2-b0d9c5175bbf /
└─sda
```

You can see that the root filesystem type is “xfs”

### **3. Initialize /dev/sda using the fdisk utility**

```
[root@Tekneed ~]# fdisk /dev/sda

Welcome to fdisk (util-linux 2.32.1).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.
```

**enter the letter “p” to print all the partitions on sda**

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

**enter the letter “d”to delete a partition**

```
Command (m for help): d
Partition number (1-3, default 3):
```

**enter the partition number 3 or press enter to leave it at default which is 3**

```
Partition 3 has been deleted.
```

**NOW, ENTER THE LETTER “n” TO RECREATE THE PARTITION TO YOUR DESIRED SIZE**

```
Command (m for help): n
Partition type
   p   primary (2 primary, 0 extended, 2 free)
   e   extended (container for logical partitions)
Select (default p):
```

**enter the letter “p” to make it a primary partition, yours might be secondary depending on the number of partitions you have. A disk can have only four partitions as primary partitions**

```
Select (default p): p
Partition number (3,4, default 3):
```

**enter the partition number which is 3 or press enter to leave at default which is 3**

```
Partition number (3,4, default 3):
First sector (899072-31457279, default 899072):
```

**press the enter key again to get to the last sector**

```
Last sector, +sectors or +size{K,M,G,T,P} (899072-31457279, default 31457279):
```

**enter the new partition size or press the enter key to use the whole available space on sda. In this scenario, we are using the whole available space**

```
Last sector, +sectors or +size{K,M,G,T,P} (899072-31457279, default 31457279):

Created a new partition 3 of type 'Linux' and of size 14.6 GiB.
Partition #3 contains a xfs signature.

Do you want to remove the signature? [Y]es/[N]o:
```

**enter “no” not to remove the signature**

```
Do you want to remove the signature? [Y]es/[N]o: no

Command (m for help):
```

**enter the letter “w” to write or save the changes and press enter**

```
Command (m for help): w
The partition table has been altered.
Syncing disks.
```

[**Additional Article : How To Protect A Disk/Filesystem In Linux**](https://tekneed.com/how-to-encrypt-a-drive-in-linux-secure-a-filesystem/)

### **4. verify the partition increment**

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

### **5. resize the filesystem,**

If the filesystem is xfs, use the command,

```
[root@Tekneed ~]# xfs_growfs /dev/sda3

xfs_growfs: /dev/sda3 is not a mounted XFS filesystem
```

If you get the error above, use the command,

```
[root@Tekneed ~]# xfs_growfs /

meta-data=/dev/sda3              isize=512    agcount=9, agsize=427200 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1,
.........
```

If the filesystem is ext (2,3,4), use “resize2fs” instead

```
[root@Tekneed ~]# resize2fs /dev/sda3
```

```
[root@Tekneed ~]# resize2fs /
```

### **6. you may also run the command below to make immediate changes to the kernel**

```
[root@Tekneed ~]# partprobe
```

### **7. verify the filesystem new size**

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

You can see that the size has increased.