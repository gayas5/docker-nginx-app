# 🚀 Jenkins CI/CD Pipeline with GitHub, Maven (Java), and Docker

This guide provides a complete CI/CD setup using **Jenkins**, **GitHub**, **Maven**, and **Docker**.  
The pipeline automatically:

1. Pulls code from GitHub  
2. Builds Java project using Maven  
3. Runs tests  
4. Builds Docker image  
5. Pushes Docker image to Docker Hub or ECR  
6. (Optional) Deploys to a server  

This README includes:

- Jenkins + GitHub integration  
- Jenkinsfile for Maven project  
- Docker build & push pipeline  
- Webhook automation  

---

# 📌 Prerequisites

| Component | Requirement |
|----------|-------------|
| Jenkins | Installed (AL2023/Ubuntu/CentOS) |
| GitHub | Repository with Java + Maven project |
| Docker | Installed on Jenkins server |
| Docker Hub / ECR | For storing images |
| Java 17 | Required by Jenkins |
| Maven | Installed on Jenkins node |

---

# 🏗️ 1. Jenkins Setup Checklist

✔ Install Java 17  
✔ Install Jenkins  
✔ Install Docker  
✔ Add Jenkins to Docker group:  

```bash
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins

# 🚀 **Jenkins Installation on Amazon Linux 2023 (FULL STEP-BY-STEP)**

Amazon Linux 2023 uses **dnf**, **systemd**, and **amazon-corretto** Java.

Follow these exact steps:

---

# ✅ **STEP 1 — Update the system**

```bash
sudo dnf update -y
```

---

# ✅ **STEP 2 — Install Java 17 (Required for Jenkins)**

Jenkins latest versions need **Java 17**.

```bash
sudo dnf install java-17-amazon-corretto -y
java -version
```

You must see:

```
openjdk version "17..."
```

---

# ✅ **STEP 3 — Add Jenkins Repository (Correct Path for AL2023)**

Amazon Linux 2023 uses:

```
/etc/yum.repos.d/
```

So run:

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo \
https://pkg.jenkins.io/redhat-stable/jenkins.repo
```

Import Jenkins GPG key:

```bash
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
```

---

# ✅ **STEP 4 — Install Jenkins**

```bash
sudo dnf install jenkins -y
```

This installs:

* Jenkins service
* /var/lib/jenkins (home directory)
* /etc/sysconfig/jenkins (config)

---

# ✅ **STEP 5 — Start & Enable Jenkins**

```bash
sudo systemctl enable --now jenkins
sudo systemctl status jenkins
```

You should see:
`active (running)`

---

# ✅ **STEP 6 — Allow Jenkins Port 8080**

Amazon Linux 2023 does **NOT** use firewalld by default.

So allow port 8080 **from AWS EC2 Security Group**:

Inbound rule →

| Type       | Protocol | Port | Source    |
| ---------- | -------- | ---- | --------- |
| Custom TCP | TCP      | 8080 | 0.0.0.0/0 |

(For security, later allow only your IP.)

---

# ✅ **STEP 7 — Get Jenkins Initial Password**

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Copy the password.

---

# ✅ **STEP 8 — Open Jenkins UI**

Open browser:

```
http://<EC2-PUBLIC-IP>:8080
```

Then:

✔ Enter initial password
✔ Install plugins → **Suggested Plugins**
✔ Create Admin User
✔ Jenkins ready! 🎉

---

# 🎉 **Jenkins Successfully Installed on Amazon Linux 2023!**

---

# 🔧 OPTIONAL BUT RECOMMENDED SETUP

## ✔ Install Git

```bash
sudo dnf install git -y
```

## ✔ Install Maven

```bash
sudo dnf install maven -y
```

## ✔ Install Docker (if you want CI/CD with container builds)

```bash
sudo dnf install docker -y
sudo systemctl enable --now docker
sudo usermod -aG docker ec2-user
```

Logout & login again to apply docker group.

---

# 📌 OPTIONAL: Reverse Proxy with Nginx + SSL (HTTPS)

If you want Jenkins to run on HTTPS (recommended), I can provide:

✔ Nginx config
✔ Let's Encrypt SSL
✔ Domain + DNS setup







# 🚀 EC2 Deployment Guide (Java / Maven / Docker)

This guide explains how to deploy your application to **Amazon EC2** using:

- Java + Maven (non-container deployment)
- Docker container deployment
- Jenkins CI/CD auto-deployment
- Nginx reverse proxy (optional)
- Systemd service (for auto-start on reboot)

---

# 🏗 1. EC2 Server Requirements

| Component | Recommended |
|----------|-------------|
| EC2 Type | t2.micro / t3.micro |
| OS | Amazon Linux 2023 / 2 |
| Storage | 20GB |
| Network | Open port 8080 or 80/443 |
| Jenkins | Optional (for auto-deploy) |

---

# 🔐 2. SSH Into EC2

```bash
ssh -i keypair.pem ec2-user@<ec2-public-ip>
Here’s the **exact fix** for your Jenkins error:

```
ERROR: dockerhub-creds
Finished: FAILURE
```

This means your Jenkins Pipeline tried to use **dockerhub-creds** but **that credential does NOT exist** in Jenkins.

---

# ✅ **Fix the Error: Create Jenkins Credential**

Follow these steps:

---

## **1️⃣ Go to Jenkins → Manage Jenkins**

* Open Jenkins
* Click **Manage Jenkins**

---

## **2️⃣ Open “Credentials”**

* Click **Manage Credentials**
* Choose:
  👉 `Store: Jenkins`
  👉 `Domain: Global`

---

## **3️⃣ Create New Credential**

Click **Add Credentials**

Select:

### **Kind:** Username with password

### **ID:** `dockerhub-creds` ← (must be EXACTLY this)

### **Username:** Your Docker Hub username

### **Password:** Your Docker Hub password

### **Description:** Dockerhub login

→ Click **Create**

---

# ✅ **4️⃣ Use credentials in your pipeline**

Your Jenkinsfile should have:

```groovy
pipeline {
    agent any

    stages {
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', 
                    usernameVariable: 'DOCKER_USER', 
                    passwordVariable: 'DOCKER_PASS')]) {

                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }
    }
}
```

---

# 💡 **Common Mistakes (Check These!)**

| Issue                                 | Fix                                                   |
| ------------------------------------- | ----------------------------------------------------- |
| Credential ID is wrong                | Must match exactly: **dockerhub-creds**               |
| Wrong credential type                 | Use **Username + Password**                           |
| Jenkins doesn't have Docker installed | Install Docker + add jenkins user to docker group     |
| No permission to run Docker           | Run this: `sudo usermod -aG docker jenkins` + restart |

---

# 🔧 **Restart Jenkins after fixing**

```
sudo systemctl restart jenkins
```

Then run the job again.

---

# 📌 If you want, send me your **complete Jenkinsfile**, and I will fix it 100%.

