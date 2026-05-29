# jenkins-cicd-devops-guide
End-to-end Jenkins CI/CD DevSecOps pipeline guide covering GitHub integration, Docker automation, SonarQube code quality analysis, Trivy security scanning, OWASP dependency checks, and automated deployment using Docker Compose. Includes real-world AWS EC2 deployment workflow with webhook-based automation and secure credential management.

# Jenkins CI/CD + DevSecOps Complete Guide

## What is CI/CD?

CI/CD (Continuous Integration, Continuous Delivery, Continuous Deployment) is a DevOps methodology that automates the software development lifecycle.

- Continuous Integration (CI): Automatically build and test code on every GitHub push.
- Continuous Delivery (CD): Prepares code for release automatically.
- Continuous Deployment: Automatically deploys applications to production.

This is implemented using Jenkins, an open-source automation server.

When code is pushed to GitHub:
- Jenkins triggers via webhook
- Pulls source code
- Builds application
- Runs tests
- Performs security scans
- Builds Docker image
- Pushes image to Docker Hub
- Deploys application using Docker Compose

---

## Jenkins Installation (Ubuntu EC2)

sudo apt update
sudo apt install fontconfig openjdk-21-jre -y
java -version

sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian/jenkins.io-2026.key

echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null

sudo apt update
sudo apt install jenkins -y

sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins

http://<EC2-Public-IP>:8080

cat /var/lib/jenkins/secrets/initialAdminPassword

---

## Docker Installation

sudo apt update
sudo apt install docker.io -y
sudo apt install docker-compose-v2 -y

sudo usermod -aG docker jenkins
sudo systemctl restart docker
sudo systemctl restart jenkins

---

## Jenkins Plugins

GitHub Integration Plugin  
Pipeline Plugin  
Docker Plugin  
SonarQube Scanner Plugin  
OWASP Dependency Check Plugin  
Stage View Plugin  

---

## SonarQube Setup

docker run -d --name sonarqube -p 9000:9000 sonarqube:lts-community

http://<EC2-IP>:9000

wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-5.0.1.3006-linux.zip
unzip sonar-scanner-cli-5.0.1.3006-linux.zip
sudo mv sonar-scanner-5.0.1.3006-linux /opt/sonar-scanner

Jenkins Path:
/opt/sonar-scanner

---

## GitHub Webhook

Jenkins Job:
Enable: GitHub hook trigger for SCM polling

GitHub:
Settings → Webhooks → Add Webhook

http://<JENKINS-IP>:8080/github-webhook/
Content-Type: application/json
Event: push

---

## Trivy Security Scanner

Trivy is a container vulnerability scanner.

sudo apt install wget apt-transport-https gnupg lsb-release -y

wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -

echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list

sudo apt update
sudo apt install trivy -y

---

## Docker Hub Integration

withCredentials([usernamePassword(
credentialsId: "docker-cred",
usernameVariable: "dockerHubUser",
passwordVariable: "dockerHubPass"
)]) {

sh 'echo $dockerHubPass | docker login -u $dockerHubUser --password-stdin'

sh "docker image tag app:latest $dockerHubUser/app:latest"

sh "docker push $dockerHubUser/app:latest"
}

---

## Docker Image Tagging

docker image tag old-image:latest username/image:version
docker push username/image:version

---

## CI/CD PIPELINE FLOW

Developer → GitHub → Jenkins → Trivy → SonarQube → Build → Docker Hub → Docker Compose Deployment

---

## DevSecOps PRACTICES

- Code Quality → SonarQube  
- Security Scan → Trivy  
- Dependency Scan → OWASP  
- CI/CD Automation → Jenkins  
- Secure Credentials → Jenkins Vault  

---

## 🏗️ FINAL ARCHITECTURE

Developer → GitHub → Jenkins → Security Scan → Code Quality → Docker Build → Docker Hub → Deployment
