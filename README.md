# DockJenk
## Docker Installation Steps
To set up Docker on this machine, run the following commands:

```bash
sudo apt update
sudo apt install docker.io docker-compose-v2 -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
newgrp docker
