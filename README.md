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
```

## Jenkins Installation Steps
To set up Jenkins on this machine, run the following commands:

```bash
#We need to install Java first
sudo apt update
sudo apt install fontconfig openjdk-21-jre
java -version
#Now we install Jenkins
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins
```
