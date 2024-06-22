# Docker Installation on Arch Linux


1. Installing Docker package:

```bash
sudo pacman -Syu docker
```

2. Start and enable the Docker service:
```bash
sudo systemctl enable docker.service
sudo systemctl start docker.service
```

3. Reboot the system to apply the changes.

4. Check the status of the Docker service:

```bash
sudo systemctl status docker.service
```

5. Verify that Docker Engine is installed correctly by running the hello-world image.

```bash
sudo docker run hello-world
```

--------------------------------------------------------------------------------

# Use Docker without sudo

1. Add docker group:

```bash
sudo groupadd docker
```

2. Add your user to the docker group:

```bash
sudo usermod -aG docker $USER
```

3. Activate the changes to groups:

```bash
newgrp docker
```

4. Verify that you can run docker commands without sudo:

```bash
docker run hello-world
```

--------------------------------------------------------------------------------

# Docker-Compose Installation

1. Install Docker-Compose:

```bash
sudo pacman -Syu docker-compose
```

2. Check the version of Docker Compose installed on the system.

```bash
docker-compose --version
```

--------------------------------------------------------------------------------