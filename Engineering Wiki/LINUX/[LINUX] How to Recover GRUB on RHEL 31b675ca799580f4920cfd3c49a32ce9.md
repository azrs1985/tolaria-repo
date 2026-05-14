# [LINUX] How to Recover GRUB on RHEL

Owner: Nam Tran
Last edited time: March 7, 2026 9:21 AM

This article will provide the step by step procedure to recover the GRUB on REHL 7 / CentOS 7. GRUB corruption / lost is one of the most common issues on Linux servers/workstations. The possible reasons for grub corruption could be due to bad disk/bug on the firmware or powered off the system abruptly. Systems ships with BIOS/ UEFI firmware and you should know what OS is using currently. Here is the way to identify [BIOS vs UEFI on Linux servers.](https://unixarena.com/2018/05/how-to-find-linux-is-under-bios-or-uefi-mode.html)  To recover GRUB on RHEL7 /CentOS 7, you must have the latest DVD or ISO image.

Common Errors in GRUB:

If GRUB is corrupted or lost, the system will not boot and it will be stuck in grub like below.

```
GNU GRUB version 0.97 (638K lower / 3143616K uper memory)

[ Minimal BASH-like line editing is supported. For the first word. TAB
lists possible command completions. Anywhere else TAB lists the possible
completions of a device/filename.]

grub>
```

The system gets stop with the following message.

```
GRUB loading stage 2
```

In such cases, Please follow the below instructions.

# **Recover/Restore the GRUB – BIOS Based system:**

1. Insert RHEL 7 / CentOS 7 latest DVD on the server or attach ISO image using ILO.

2. In case of a Virtual machine, attach the ISO image to the VM.

3. Boot the server using the DVD/ISO image.

4. Choose troubleshooting option once the system is booted in DVD/ISO.

![**RHEL7 CentOS7 – Troubleshooting Rescue Mode**](https://unixarena.com/wp-content/uploads/2018/05/RHEL7-CentOS7-Troubleshooting-Rescue-Mode.jpg)

**RHEL7 CentOS7 – Troubleshooting Rescue Mode**

5. Choose the rescue mode.

![RHEL 7 - Rescue Mode](https://unixarena.com/wp-content/uploads/2018/05/RHEL-7-Rescue-Mode.jpg)

**RHEL 7 – Rescue Mode**

6. Press 1 to continue to find the OS image and get the shell prompt.

![Continue - Rescue Mode - RHEL](https://unixarena.com/wp-content/uploads/2018/05/Continue-Rescue-Mode.jpg)

**Continue – Rescue Mode – RHEL**

7.chroot to the OS image.

![chroot to OS Image - RHEL](https://unixarena.com/wp-content/uploads/2018/05/chroot-to-OS-Image.jpg)

**chroot to OS Image – RHEL**

8. Starting RHEL 7/CentOS 7,  GRUB 2 is the default  bootloader. The GRUB 2 configuration file is **/boot/grub2/grub.cfg.** Install grub boot loader on root disk. (Default : /dev/sda).

![grub2-install - RHEL 7 CentOS7](https://unixarena.com/wp-content/uploads/2018/05/grub2-install-RHEL-7-CentOS7.jpg)

**grub2-install – RHEL 7 CentOS7**

9. Navigate to /boot/grub2 directory and confirm the existence of “grub.cfg” .  Here “grub.cfg” file is not exists.

![grub.cfg is missing - RHEL 7](https://unixarena.com/wp-content/uploads/2018/05/grub.cfg-is-missing-RHEL-7.jpg)

**grub.cfg is missing – RHEL 7**

10. Let’s generate “grub.cfg”  file.

![grub2-mkconfig - Recreate grub.cfg](https://unixarena.com/wp-content/uploads/2018/05/grub2-mkconfig-Recreate-grub.cfg_.jpg)

**grub2-mkconfig – Recreate grub.cfg**

If you are missing grub2-mkconfig command, install “grub2-tools.x86_64” package in rescue mode . Since you have booted from DVD, the package should be available in that.

![Grub.cfg recreated using grub2-mkconfig](https://unixarena.com/wp-content/uploads/2018/05/Grub.cfg-recreated-using-grub2-mkconfig.jpg)

Grub.cfg recreated using grub2-mkconfig

11. Exit from the chroot and reboot the system.

This will work only on BIOS-based X86 servers and virtual machines but will not work with UEFI firmware servers and VMs.

[Here is the link to identify that what firmware type was used to boot the system.](https://unixarena.com/2018/05/how-to-find-linux-is-under-bios-or-uefi-mode.html)