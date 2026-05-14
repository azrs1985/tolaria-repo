# [RHEL] Installing Docker on Red Hat Enterprise Linux (RHEL)

Owner: Nam Tran
Last edited time: August 28, 2025 3:23 PM

**1. Remove Conflicting Packages (Optional):**

If you have Podman or Buildah installed, it is recommended to remove them to avoid potential conflicts with Docker.

Code

```bash
sudo dnf remove -y podman buildah
```

**2. Add the Docker Repository:**

Docker packages are not included in the default RHEL repositories, so you need to add the official Docker repository.

Code

```bash
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
```

Verify that the repository has been added successfully:

Code

```bash
sudo dnf repolist
```

**3. Install Docker Engine:**

Install the Docker Engine and its associated components using `dnf`.

Code

```bash
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```

**4. Start and Enable Docker:**

Start the Docker daemon and configure it to start automatically on system boot.

Code

```bash
sudo systemctl start docker
sudo systemctl enable docker
```

**5. Verify Installation:**

Confirm that Docker is running and installed correctly by checking its status and version.

Code

```bash
systemctl is-active docker
docker version
```

**6. (Optional) Manage User Permissions:**

To run Docker commands without `sudo`, add your user to the `docker` group.

Code

```bash
sudo usermod -aG docker $USER
```

You will need to log out and log back in for this change to take effect.