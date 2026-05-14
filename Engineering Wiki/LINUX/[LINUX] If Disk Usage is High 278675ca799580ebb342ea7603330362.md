# [LINUX] If Disk Usage is High

Owner: Nam Tran
Last edited time: September 24, 2025 10:15 AM

**Pro Tip: If Disk Usage is High**

- Run `df -h` to check free space per partition.
- Use `du -ah / | sort -rh | head -10` to identify the largest files/folders.
- Vacuum old logs (`journalctl --vacuum-time=`) and clean package cache (`apt autoclean, etc.`).
- Remove old or unused kernels (on Ubuntu/Debian systems).

**If Server Becomes Unresponsive:**

- Switch to a TTY (`Ctrl+Alt+F2, etc.`) or use console/hypervisor access.
- Check system load with `uptime`, and use `top` / `htop` to locate runaway processes and terminate them if necessary.
- Inspect logs with `tail`, `dmesg`, or `journalctl` for error details.
- If the system is completely frozen, use **SysRq magic keys** (if enabled) for a safe reboot.