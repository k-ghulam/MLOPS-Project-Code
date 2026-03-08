# 🚀 End-to-End CI/CD Pipeline with Jenkins, Docker, Kubernetes & GCP

![Docker](https://img.shields.io/badge/Docker-Containerized-blue)
![Kubernetes](https://img.shields.io/badge/Kubernetes-Orchestrated-blue)
![Jenkins](https://img.shields.io/badge/Jenkins-CI/CD-red)
![Python](https://img.shields.io/badge/Python-3.11-green)
![License](https://img.shields.io/badge/License-MIT-yellow)

This project demonstrates a **complete production-style DevOps CI/CD pipeline** that automatically builds, pushes, and deploys a Python application using:

* **Jenkins CI/CD**
* **Docker**
* **Kubernetes (Minikube)**
* **Google Cloud VM**
* **Jenkins Shared Libraries**
* **GitHub Webhooks**

The pipeline automatically deploys the application to Kubernetes whenever code is pushed to GitHub.

---

# 📌 Architecture

```
Developer
   │
   │ Push Code
   ▼
GitHub Repository
   │
   │ Webhook Trigger
   ▼
Jenkins Pipeline
   │
   ├── Checkout Code
   ├── Build Docker Image
   ├── Push Image → DockerHub
   └── Deploy → Kubernetes
             │
             ▼
         Minikube Cluster
             │
             ▼
        Flask Application
```

---

# 📁 Project Structure

```
project-root
│
├── Dockerfile
├── Jenkinsfile
├── application.py
│
├── k8s
│   ├── deployment.yaml
│   └── service.yaml
│
└── vars
    ├── gitCheckout.groovy
    ├── dockerBuildAndPush.groovy
    ├── installKubectl.groovy
    └── k8sDeploy.groovy
```

---

# 🐍 Python Installation Guide

⚠️ **Use Python 3.11 only**

Higher versions currently have **limited ML library support**.

### Install Python

Download from:

https://www.python.org/downloads/release/python-3110/

During installation:

✔ Enable **Add Python to PATH**

---

# 🐳 Dockerfile

Create a file named:

```
Dockerfile
```

```dockerfile
FROM python:3.11

WORKDIR /app

COPY . /app

RUN pip install --no-cache-dir -e .

EXPOSE 5000

ENV FLASK_APP=application.py

CMD ["python","application.py"]
```

---

# ☸ Kubernetes Configuration

## Deployment

`k8s/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flask-app
  template:
    metadata:
      labels:
        app: flask-app
    spec:
      containers:
      - name: flask-container
        image: your-dockerhub/image:latest
        ports:
        - containerPort: 5000
```

---

## Service

`k8s/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  type: NodePort
  selector:
    app: flask-app
  ports:
  - port: 5000
    targetPort: 5000
    nodePort: 30080
```

---

# ⚙ Jenkins Shared Library

Shared libraries simplify Jenkins pipelines by reusing common code.

### Example

`vars/gitCheckout.groovy`

```groovy
def call(String repoUrl, String branch, String credId) {
    checkout([
        $class: 'GitSCM',
        branches: [[name: branch]],
        userRemoteConfigs: [[credentialsId: credId, url: repoUrl]]
    ])
}
```

---

# 📦 Jenkins Pipeline

`Jenkinsfile`

```groovy
@Library('jenkins-shared') _

pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                gitCheckout('repo-url','main','github-token')
            }
        }

        stage('Build & Push Image') {
            steps {
                dockerBuildAndPush('dockerhub/repo','dockerhub-token')
            }
        }

        stage('Install Kubectl') {
            steps {
                installKubectl()
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                k8sDeploy('kubeconfig')
            }
        }
    }

}
```

---

# ☁ Google Cloud VM Setup

Create a VM instance with:

| Setting | Value        |
| ------- | ------------ |
| Machine | E2 Standard  |
| RAM     | 16GB         |
| Disk    | 150GB        |
| OS      | Ubuntu 24.04 |

Enable:

* HTTP
* HTTPS

---

# 🐳 Install Docker

```
docker run hello-world
```

Enable docker on boot:

```
sudo systemctl enable docker
```

---

# ☸ Install Minikube

Start cluster:

```
minikube start
```

Verify:

```
kubectl get nodes
```

---

# 🧑‍💻 Run Jenkins in Docker

```
docker run -d --name jenkins \
-p 8080:8080 \
-p 50000:50000 \
-v /var/run/docker.sock:/var/run/docker.sock \
-u root \
--network minikube \
jenkins/jenkins:lts
```

Access Jenkins:

```
http://VM_IP:8080
```

---

# 🔐 Configure Credentials in Jenkins

Add credentials:

| Credential        | ID              |
| ----------------- | --------------- |
| GitHub Token      | github-token    |
| DockerHub Token   | dockerhub-token |
| Kubernetes Config | kubeconfig      |

---

# 🔁 GitHub Webhook Automation

Go to:

```
GitHub → Repository → Settings → Webhooks
```

Add:

```
http://JENKINS_IP:8080/github-webhook/
```

Select:

✔ **Push Event**

---

# 🚀 Deploy Application

Push code:

```
git add .
git commit -m "Trigger pipeline"
git push origin main
```

Jenkins will automatically:

1️⃣ Build Docker Image
2️⃣ Push Image to DockerHub
3️⃣ Deploy to Kubernetes

---

# 📊 Verify Deployment

Check pods:

```
kubectl get pods
```

Port forward:

```
kubectl port-forward deployment/flask-deployment 5000:5000 --address 0.0.0.0
```

Open browser:

```
http://VM_IP:5000
```

---

# 🎉 Result

You now have a **fully automated CI/CD pipeline**.

Every GitHub push will automatically:

✔ Build Docker image
✔ Push to DockerHub
✔ Deploy to Kubernetes

---

# 📚 Technologies Used

* Python
* Docker
* Kubernetes
* Jenkins
* Minikube
* GitHub
* Google Cloud Platform

---

# ⭐ If you found this project useful

Give the repository a **star ⭐ on GitHub**.
