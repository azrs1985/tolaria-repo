# [Ubuntu] Disable automatic updates on Ubuntu Server

Owner: Nam Tran
Last edited time: October 1, 2025 5:14 PM

To disable automatic updates on Ubuntu Server 24, you should edit the `20auto-upgrades` file to change the `APT::Periodic::Update-Package-Lists` and `APT::Periodic::Unattended-Upgrade` values to `0` , or alternatively, you can stop and disable the `apt-daily-upgrade.timer` and `apt-daily.timer` services using `systemctl`. Another option is to reconfigure the `unattended-upgrades` package to disable automatic installation.

**Using the** `systemctl` **command to disable the timer services**

This method stops and disables the services responsible for daily package list updates and unattended upgrades.

1. **Stop the timer services**:

```bash
sudo systemctl stop apt-daily-upgrade.timer && \
sudo systemctl stop apt-daily.timer
```

1. **Disable the timer services**:

```bash
sudo systemctl disable apt-daily-upgrade.timer && \
sudo systemctl disable apt-daily.timer
```

1. **Mask the timer services**: to prevent them from being re-enabled by other system processes:

```bash
sudo systemctl mask apt-daily-upgrade.timer && \
sudo systemctl mask apt-daily.timer
```

**Editing the** `20auto-upgrades` **file**

This method modifies the APT configuration to disable automatic package list checks and unattended upgrades.

1. **Open the file**: for editing:

```bash
sudo nano /etc/apt/apt.conf.d/20auto-upgrades
```

1. **Modify the values**:
    - Change `APT::Periodic::Update-Package-Lists "1";` to `APT::Periodic::Update-Package-Lists "0";`.
    - Change `APT::Periodic::Unattended-Upgrade "1";` to `APT::Periodic::Unattended-Upgrade "0";`.
2. **Save and exit**: the file (Ctrl+O, then Enter, then Ctrl+X in nano).

**Using** `dpkg-reconfigure`

This command interactively reconfigures the `unattended-upgrades` package to disable automatic updates.

1. **Run the command**:

```bash
sudo dpkg-reconfigure -plow unattended-upgrades
```

1. **Select "No"**: when asked if you want to automatically download and install stable updates.