# project amazon-app 


# 🚀 Amazon Clone Deployment using Terraform + AWS + Jenkins + SonarQube + Docker

This guide provides complete step‑by‑step instructions to deploy the **Amazon Clone** project using AWS EC2, Terraform, Jenkins, SonarQube, Docker, Trivy, OWASP, and Node.js.

---

## 📌 **Prerequisites**

* Ubuntu/Linux machine
* AWS account
* Jenkins installed (or EC2 instance for Jenkins)
* SonarQube instance
* DockerHub account

---

## 🖥️ **1. Setup Local Machine**

### ▶️ Install AWS CLI & Terraform

```bash
sudo apt update
sudo apt install awscli -y
sudo apt install terraform -y
```

### ▶️ Configure AWS

```bash
aws configure
```

Add:

* Access Key
* Secret Key
* Region

---

## 📁 **2. Create Project Directory**

```bash
mkdir amazon
cd amazon
```

### ▶️ Clone the repository

```bash
git clone https://github.com/roshand33/Project-Amazon-Clone.git
ls
```

You will see a folder like **Project-Amazon-Clone** → go inside `JENKINS.TS`.

---

## ⚙️ **3. Configure Terraform Files**

### Edit provider.tf → update **AWS region**

### Edit main.tf → update

* AMI ID
* instance type (ex: `t2.micro`, `t3.medium`)

### Run Terraform

```bash
terraform init
terraform validate
terraform plan
terraform apply --auto-approve
```

---

## 🔑 **4. Access EC2 Instance for Jenkins**

Copy EC2 public IP and open:

```
http://EC2-IP:8080
```

Retrieve Jenkins initial password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Login to Jenkins.

---

## 🔌 **5. Install Required Jenkins Plugins**

1. Eclipse Temurin Installer
2. SonarQube Scanner
3. NodeJs Plugin
4. OWASP Dependency Check
5. Prometheus Metrics
6. Docker (First four + CloudBees)
7. Stage View

---

## 🛠️ **6. Configure Credentials**

### ▶️ Sonar Token

* Open SonarQube → `http://EC2-IP:9000`
* Go to **Administration → Security → Generate Token**
* Name: `sonar-token`
* Copy the token

Then in Jenkins:

* Manage Jenkins → Credentials → Secret Text → ID: `sonar-token`

### ▶️ DockerHub

Add DockerHub username/password → ID: `docker`

---

## ⚙️ **7. Configure Jenkins Tools**

Manage Jenkins → Tools:

* **JDK** → `jdk17` (install version 17)
* **NodeJS** → `node16` (install 16.2.0)
* **Sonar Scanner** → default
* **Docker** → add installer

Add SonarQube server in System Configuration:

* Server name: `sonar-server`
* Credentials: `sonar-token`

---

## 🧪 **8. Jenkins Pipeline Script**

```groovy
pipeline{
    agent any
    tools{
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        stage('clean workspace'){
            steps{
                cleanWs()
            }
        }
        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/roshand33/Project-Amazon-Clone.git'
            }
        }
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonar-server') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Amazon \
                    -Dsonar.projectKey=Amazon '''
                }
            }
        }
        stage("quality gate"){
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'jenkins'
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }
        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){   
                       sh "docker build -t amazon-clone ."
                       sh "docker tag amazon-clone YOUR_DOCKER_USERNAME/amazon-clone:latest "
                       sh "docker push YOUR_DOCKER_USERNAME/amazon-clone:latest "
                    }
                }
            }
        }
        stage("TRIVY"){
            steps{
                sh "trivy image YOUR_DOCKER_USERNAME/amazon-clone:latest > trivyimage.txt"
            }
        }
        stage('Deploy to container'){
            steps{
                sh 'docker run -d --name amazon-clone -p 3000:3000 YOUR_DOCKER_USERNAME/amazon-clone:latest'
            }
        }
    }
}
```

---

## ✨ **9. Required Changes**

✔ Replace `YOUR_DOCKER_USERNAME` with your DockerHub username
✔ Fork repo or use original
✔ Ensure names match:

* sonar-server
* sonar-token
* sonar-scanner

---

## 🔄 **10. Restart Jenkins**

Visit:

```
http://EC2-IP:8080/restart
```

Login → Run pipeline → Wait 10–15 minutes.

If Quality Gate causes issues, refer repo:

```
https://github.com/roshand33/webhook_sonar_jenkins.git
```

---

## 🎉 Deployment Complete!

Your Amazon Clone will be running on:

```
http://EC2-IP:3000
```

---

## ⭐ Author

**Roshan D**  (GitHub: `roshand33`)

---

Let me know if you want badges, images, or a more advanced README design!
