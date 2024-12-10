# Jenkins CI/CD with SonarQube, Docker, and GitHub Webhooks on AWS

This project sets up a robust Jenkins CI/CD pipeline integrated with SonarQube for code quality analysis, Docker for containerization, and GitHub Webhooks for automated builds. The infrastructure leverages three AWS EC2 instances to isolate and streamline the deployment of Jenkins, SonarQube, and Docker servers.

---

## Project Architecture

- **EC2 Instances**:  
  - **Jenkins Server**: Manages the CI/CD pipeline.  
  - **SonarQube Server**: Performs static code analysis and reports on code quality.  
  - **Docker Server**: Builds and runs containerized applications.  
- **Operating System**: Ubuntu (t2.medium instances).  
- **Source Code Management**: GitHub repository named `Jenkins-Sonar-Docker-CICD`.

---

## Prerequisites

1. **GitHub Repository**  
   - Push project files to a GitHub repository.
   
2. **AWS EC2 Instances**  
   - Provision **3 Ubuntu-based t2.medium EC2 instances** for:  
     - Jenkins  
     - SonarQube  
     - Docker  

---

## Configuration Steps

### 1. Configure Jenkins Server

#### Connect to Jenkins EC2 Instance
```bash
ssh -i <keyfile.pem> ubuntu@<Jenkins_Public_IP>
sudo hostnamectl set-hostname jenkins
/bin/bash
sudo apt update
```

#### Install JDK
```bash
sudo apt install fontconfig openjdk-17-jre
java -version
```
#### Install Jenkins
```bash
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list
sudo apt-get update
sudo apt-get install jenkins
```

#### Configure Jenkins
1. Start Jenkins: sudo systemctl start jenkins
2. Get Jenkins admin password:
   ```bash
   sudo cat /var/lib/jenkins/secrets/initialAdminPassword
   ````
3. Access Jenkins at http://<Jenkins_Public_IP>:8080.

#### Create Jenkins Pipeline
1. New item → Freestyle project: Automated-Pipeline.
2. Configure the following:
    - Git Repository URL: Add GitHub URL.
    - Build Triggers: Check "GitHub hook trigger for GITScm polling".
      
#### Set Up GitHub Webhooks
1. In GitHub repo → Settings → Webhooks.
2. Add webhook:
    - Payload URL: http://<Jenkins_Public_IP>:8080/github-webhook/
    - Events: Select "Pushes" and "Pull Requests".

<p align="center">      
  <img src="https://github.com/user-attachments/assets/0c285f44-a5ac-41c0-a90d-d6bd9cc0aba4" alt="Build and SonarQube Pass Screenshot">
</p>

### 2. Configure SonarQube Server
#### Connect to SonarQube EC2
```bash
ssh -i <keyfile.pem> ubuntu@<SonarQube_Public_IP>
sudo hostnamectl set-hostname sonarqube
/bin/bash
sudo apt update
```
#### Install Prerequisites and SonarQube
```bash
sudo apt install fontconfig openjdk-17-jre unzip -y
wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-24.12.0.100206.zip
unzip sonarqube-24.12.0.100206.zip
cd sonarqube-24.12.0.100206/bin/linux-x86-64
./sonar.sh console
```

#### Configure Security Group
1. Add Inbound Rule: Custom TCP, Port 9000.

#### Access SonarQube
1. Open http://<SonarQube_Public_IP>:9000 in browser.
2. Login credentials:
      - Username: admin
      - Password: admin.

<p align="center">
  <img src="https://github.com/user-attachments/assets/1525c5bc-c78d-4831-b511-f7c1e461e6cf" alt="Build and SonarQube Pass Screenshot">
</p>
        
#### Generate Token and Configure Jenkins
1. In SonarQube: My Account → Security → Generate Token.
2. In Jenkins:
      - Manage Jenkins → System → SonarQube servers.
      - Add SonarQube server details and token.

### 3. Configure Docker Server
#### Connect to Docker EC2
```bash
ssh -i <keyfile.pem> ubuntu@<Docker_Public_IP>
sudo hostnamectl set-hostname docker
/bin/bash
sudo apt update
```
#### Install Docker
```bash
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo docker run hello-world
```

### 4. Integrate Docker with Jenkins
#### Allow Jenkins to Access Docker EC2
1. Update SSH settings in Docker EC2:
```bash
sudo nano /etc/ssh/sshd_config
```
   - Change:
     - PubkeyAuthentication yes
     - PasswordAuthentication yes
```bash
sudo systemctl restart sshd
passwd ubuntu
```
2. Add Jenkins public key to Docker EC2:
```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
# Paste Jenkins public key here
chmod 600 ~/.ssh/authorized_keys
```
#### Configure Jenkins to Use Docker Server
1. Manage Jenkins → System → Node Configuration.
2. Add Docker server:
      - Group: docker-server
      - Name: docker-1
      - Host: <Docker_Public_IP>
3. Test with a pipeline build:
      - Add a shell step: touch test.sh.
  
<p align="center">
  <img src="https://github.com/user-attachments/assets/c3017414-4bad-4d6e-b170-8a01386066cb" alt="SonarQube Pass State Screenshot">
</p>

<p align="center">
  🎉 Build Triggered Successfully! 🚀 Data added correctly, pipeline completed, and SonarQube passed with flying colors. ✅ Here's the proof! 📸
</p>

