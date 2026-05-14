# [Ubuntu]  Disable Ubuntu Pro's news notifications on Ubuntu Server

Owner: Nam Tran
Last edited time: October 1, 2025 5:22 PM

To disable Ubuntu Pro's news notifications on Ubuntu Server 24.04, open the terminal and run `sudo pro config set apt_news=false` to stop the "additional updates" messages during `apt` operations. If you want to disable the ESM (Expanded Security Maintenance) functionality entirely, you will need to stop and disable the `ubuntu-advantage` service using `sudo systemctl stop ubuntu-advantage.service` and `sudo systemctl disable ubuntu-advantage.service`, then uncomment the ESM apt sources in `/etc/apt/sources.list.d/`.

**Disable Ubuntu Pro News & Ads**

This is the easiest method to stop the nag messages during `apt` commands.

1. **Open the terminal**: on your Ubuntu Server.
2. **Enter the following command**: and press Enter:

```bash
sudo pro config set apt_news=false
```

1. **Enter your password**: if prompted.

**Completely Disable Ubuntu Pro (Including ESM)**

This method disables the service and removes the configuration for ESM updates.

1. **Stop the Ubuntu Pro service:**

```bash
sudo systemctl stop ubuntu-advantage.service
```

1. **Disable the service**: from starting on boot:

```bash
sudo systemctl disable ubuntu-advantage.service
```

**Comment out the ESM apt sources:**

- Open the file `/etc/apt/sources.list.d/ubuntu-advantage.sources` in a text editor using root privileges (e.g., `sudo nano /etc/apt/sources.list.d/ubuntu-advantage.sources`).
- Add a `#` at the beginning of each line that starts with `deb` to comment them out.
- Save the file and exit the editor.

**If you want to remove the messages from the MOTD (Message of the Day):**

1. **Create an empty file**: in your home directory:

```bash
touch ~/.hide-esm-in-motd
```

1. **Move the file**: to the correct location to prevent the messages:

```bash
sudo mv ~/.hide-esm-in-motd /var/lib/update-notifier/
```