# CI/CD Deployment: Flask + Express on EC2 with Jenkins

This project demonstrates deploying a **Flask backend** and an **Express frontend** on a **single EC2 instance**, and automating the deployment using **Jenkins CI/CD pipelines**.

---

## 📌 Part 1: Manual Deployment on EC2

### 🎯 Objective
Deploy Flask and Express applications manually on a single EC2 instance.

### ✅ Steps

#### 1. Launch EC2 Instance
- Go to AWS EC2 dashboard.
- Launch a **free-tier Amazon Linux 2/Ubuntu** instance.
- Add a security group rule to allow:
  - TCP **port 3000** (for Express)
  - TCP **port 5000** (for Flask)
  - TCP **port 8080** (for Jenkins)
  - TCP **port 22** (for SSH)

#### 2. SSH into EC2
```bash
ssh -i your-key.pem ec2-user@<public-ip>
```

#### 3. Install Required Software
```bash
# Update
sudo apt update

# Install Python3 and pip
sudo apt install python3-pip -y

# Install Node.js and npm
sudo apt install nodejs npm -y

# Install Git
sudo apt install git -y
```

#### 4. Clone Project Repository
```bash
git clone https://github.com/AzamCloudOps/flask-express-jenkins-pipeline.git
cd flask-express-jenkins-pipeline
```

#### 5. Run Flask App
```bash
cd flask-backend
pip3 install -r requirements.txt
nohup python3 app.py > flask.log 2>&1 &
```

#### 6. Run Express App
```bash
cd ../express-frontend
npm install
nohup node app.js > express.log 2>&1 &
```

#### ✅ Access the Apps
- Flask App: `http://<EC2-IP>:5000/`
- Express App: `http://<EC2-IP>:3000/`

---

## 📌 Part 2: CI/CD using Jenkins

### 🎯 Objective
Automate deployment using Jenkins pipelines for both Flask and Express apps.

---

### ✅ Steps

#### 1. Install Jenkins
```bash
# Java is required
sudo apt install openjdk-11-jdk -y

# Add Jenkins key and repo
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt install jenkins -y

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

#### 2. Access Jenkins
- Visit: `http://<EC2-IP>:8080`
- Get initial password:
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

- Complete setup and install **suggested plugins**.
- Install Git, NodeJS, and Python plugins if needed.

---

### 🛠 Create Jenkins Pipelines

#### Folder Structure
```
Jenkins-backend/Jenkinsfile     <-- Flask pipeline
Jenkins-frontend/Jenkinsfile    <-- Express pipeline
```

#### Jenkinsfile for Flask App (`Jenkins-backend/Jenkinsfile`)
```groovy
pipeline {
    agent any
    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/AzamCloudOps/flask-express-jenkins-pipeline.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                dir('flask-backend') {
                    sh 'pip3 install -r requirements.txt'
                }
            }
        }
        stage('Run Flask App') {
            steps {
                dir('flask-backend') {
                    sh 'pkill -f app.py || true'
                    sh 'nohup python3 app.py > flask.log 2>&1 &'
                }
            }
        }
    }
}
```

#### Jenkinsfile for Express App (`Jenkins-frontend/Jenkinsfile`)
```groovy
pipeline {
    agent any
    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/AzamCloudOps/flask-express-jenkins-pipeline.git'
            }
        }
        stage('Install Dependencies') {
            steps {
                dir('express-frontend') {
                    sh 'npm install'
                }
            }
        }
        stage('Run Express App') {
            steps {
                dir('express-frontend') {
                    sh 'pkill -f app.js || true'
                    sh 'nohup node app.js > express.log 2>&1 &'
                }
            }
        }
    }
}
```

---

## 🚀 Trigger CI/CD via GitHub Webhook (Optional)

1. Go to your GitHub repo → Settings → Webhooks
2. Payload URL: `http://<your-jenkins-ip>:8080/github-webhook/`
3. Content type: `application/json`
4. Events: Just the push event
5. Save

---

## 📸 Deliverables

- ✅ EC2 instance running Flask and Express
- ✅ Jenkins pipelines working on code push
- ✅ Screenshots:
  - EC2 instance public IP
  - Flask and Express running in browser
  - Jenkins build console output (success)

---

## 📎 Repo Links

- GitHub Repo: [flask-express-jenkins-pipeline](https://github.com/AzamCloudOps/flask-express-jenkins-pipeline)

---

## ✅ Credits
Project by **Mohd Azam Uddin**
Deployed using AWS, Jenkins, Python, Node.js
