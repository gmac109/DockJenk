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

1. Install Java
```bash
#We need to install Java first
sudo apt update
sudo apt install fontconfig openjdk-21-jre
java -version
```
2. Add Jenkins repo & install
```bash
#Now we install Jenkins
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt update
sudo apt install jenkins
```
3. Start & Enable Jenkins
```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

4. Initial Jenkins setup
   - Retrieve initial admin password	
   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ```
   - Access the Jenkins dashboard at ```http://<ec2-public-ip>:8080```

   - Paste the password, install suggested plugins, and create an admin user.

## Project Overview
This project sets up a 2-tier web application (Flask + MySQL) containerized 
with Docker and automatically deployed via a Jenkins pipeline triggered by 
GitHub pushes.

The app: TaskFlow — a dark-themed to-do tracker where users can add tasks 
with due dates, mark them complete, and delete them. All data is stored in 
a MySQL database.

---

## Architecture
```
Developer (MacBook)
      |
      |  git push
      v
GitHub Repository (gmac109/DockJenk)
      |
      |  Jenkins pulls on Build
      v
AWS EC2 Instance (Ubuntu 22.04)
      |-- Jenkins (port 8080)
      └── Docker
            |-- Flask Container (port 5000)
            └── MySQL Container (port 3306)
```

---

## Project Structure
```
DockJenk/
├── templates/
│   └── index.html        # Frontend UI (dark theme)
├── app.py                # Flask application
├── requirements.txt      # Python dependencies
├── Dockerfile            # Flask container definition
├── docker-compose.yml    # Multi-container orchestration
└── Jenkinsfile           # CI/CD pipeline definition
```

---

## Tech Stack
| Tool | Purpose |
|------|---------|
| Flask | Python web framework |
| MySQL | Database (via Docker) |
| Docker & Docker Compose | Containerization |
| Jenkins | CI/CD automation |
| AWS EC2 (t2.micro) | Cloud server |
| GitHub | Source code management |

---

## Setup & Installation

### Step 1: AWS EC2 Instance
- Launched Ubuntu 22.04 LTS on a t2.micro instance
- Configured security group inbound rules for ports 22, 80, 5000, and 8080

### Step 2: Install Dependencies on EC2
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install git docker.io docker-compose-v2 -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $USER
```

### Step 3: Install Jenkins
```bash
sudo apt install openjdk-17-jdk -y
sudo apt install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

### Step 4: Configure Jenkins Pipeline
- Accessed Jenkins at http://<ec2-ip>:8080
- Created a new Pipeline job named Payload
- Set pipeline definition to Pipeline script from SCM
- Connected to GitHub repo with a Personal Access Token credential
- Set branch to */main and script path to Jenkinsfile

---

## Flask App Routes
| Route | Method | Description |
|-------|--------|-------------|
| `/` | GET | Display all tasks |
| `/add` | POST | Add a new task |
| `/complete/<id>` | GET | Toggle task complete/incomplete |
| `/delete/<id>` | GET | Delete a task |
| `/health` | GET | Health check for Docker |

---

## CI/CD Pipeline Stages
```
Stage 1: Clone Code          -> Jenkins pulls latest code from GitHub
Stage 2: Build Docker Image  -> Builds Flask app image from Dockerfile
Stage 3: Deploy              -> Runs docker compose up --build
```
Any git push to the main branch triggers a full automated redeploy.

---

## Problems Encountered & Solutions

### 1. Jenkins Waiting for Executor
**Problem:** Jenkins console showed `Waiting for next available executor` and never started.  
**Fix:** Went to Manage Jenkins -> Nodes -> Built-In Node -> Configure and changed executors from 0 to 2.

### 2. /tmp Disk Space Warning
**Problem:** Jenkins kept going offline warning that /tmp had less than 1GB free.  
**Fix:** Lowered the Free Temp Space Threshold to 200MB since /tmp is only 470MB total on a t2.micro.

### 3. Corrupted URL in Jenkinsfile
**Problem:** The Jenkinsfile URL was markdown-formatted causing the error `fatal: protocol '[https' is not supported`.  
**Fix:** Replaced with a plain clean URL: `url: 'https://github.com/gmac109/DockJenk.git'`

### 4. Branch Mismatch (master vs main)
**Problem:** Local machine was on master, GitHub default was main, causing push conflicts.  
**Fix:** Renamed local branch with `git branch -m master main` and set GitHub default branch to main.

### 5. GitHub Authentication Failed
**Problem:** Jenkins failed with `fatal: Authentication failed` because the Clone Code stage had no credentials.  
**Fix:** Created a GitHub Personal Access Token, added it to Jenkins credentials, and updated the Jenkinsfile:
```groovy
git branch: 'main',
    credentialsId: 'github-token',
    url: 'https://github.com/gmac109/DockJenk.git'
```

### 6. Lost Jenkins Access
**Problem:** Got locked out after changing security settings.  
**Fix:** SSH'd into EC2, temporarily set `<useSecurity>false</useSecurity>` in `/var/lib/jenkins/config.xml`, reset credentials, then re-enabled security.

---

## Final Result
After a successful build the TaskFlow app is live at:
```
http://<ec2-public-ip>:5000
```
Jenkins pipeline name: Payload

## Deployment Fix: AWS Security Group

After a successful Jenkins build both containers were running but the 
app was not accessible in the browser.

**Problem:** Port 5000 was not open in the AWS Security Group inbound rules.
**Fix:** Added the following inbound rule in the AWS Console:

| Type | Protocol | Port | Source |
|------|----------|------|--------|
| Custom TCP | TCP | 5000 | 0.0.0.0/0 |

After saving the rule the app was immediately accessible at:
```
http://<ec2-public-ip>:5000
```

---

## Swap Memory Fix

**Problem:** Jenkins build froze and was killed by the OS due to running 
out of RAM on the t2.micro (1GB RAM).  
**Fix:** Added 2GB of swap memory to use disk space as extra RAM:
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```