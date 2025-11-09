# 🚀 DevOps CI/CD Pipeline – Datascientest Project

## 🎯 Project Objective

The goal of this project is to implement a complete **CI/CD pipeline** for a simple Nginx web application that:
- automatically builds a Docker image from a GitHub repository,
- pushes the image to Docker Hub,
- deploys the application to a Kubernetes cluster (`dev` namespace),
- allows manual promotion to `prod` after confirmation in Jenkins.

---

## ⚙️ Architecture Overview

GitHub (source)
↓
Jenkins (running in Docker)
↓
Docker build & push
↓
Docker Hub (registry)
↓
Kubernetes (k3s cluster)


**Components:**
| Component | Description |
|------------|-------------|
| **GitHub** | Stores the source code (`Dockerfile`, `index.html`, `k8s/`) |
| **Jenkins** | Orchestrates the CI/CD pipeline |
| **Docker Hub** | Stores built container images |
| **Kubernetes (k3s)** | Target environment for deployment |
| **Namespaces (`dev`, `prod`)** | Separate environments for testing and production |

---

## 🧩 Repository Structure


.
├── Dockerfile
├── Jenkinsfile
├── index.html
└── k8s/
├── base/
│ ├── deployment.yaml
│ ├── service.yaml
│ ├── dev/
│ │ └── ns.yaml
│ ├── prod/
│ │ └── ns.yaml


---

## 🧱 **CI/CD Pipeline**

### 1️⃣ **Checkout**
Jenkins клонирует проект из GitHub:
```groovy
checkout scm
...


2️⃣ Build Docker image

```
   FROM nginx:alpine
   COPY index.html /usr/share/nginx/html/index.html
```
   Jenkins performs:
```
docker build -t plavez/devops-pipeline:latest .
docker tag plavez/devops-pipeline:latest plavez/devops-pipeline:${BUILD_NUMBER}
```
### 3️⃣ Push to DockerHub
   Jenkins authenticates and pushes the image:

```
docker push plavez/devops-pipeline:latest
docker push plavez/devops-pipeline:${BUILD_NUMBER}
```
### 4️⃣ Deploy to dev
   Automatic deployment to Kubernetes:
   ```
   kubectl apply -f k8s/dev/ns.yaml
kubectl -n dev apply -f k8s/base/deployment.yaml
kubectl -n dev apply -f k8s/base/service.yaml
kubectl -n dev set image deployment/devops-web web=plavez/devops-pipeline:${BUILD_NUMBER}
kubectl -n dev rollout status deployment/devops-web
```
### 5️⃣ Deploy to prod
   Manual confirmation:
```
input message: 'Deploy to PRODUCTION?'
```

After confirmation, Jenkins deploys to the prod namespace using:
```
kubectl apply -f k8s/prod/ns.yaml
kubectl -n prod apply -f k8s/base/deployment.yaml
kubectl -n prod apply -f k8s/base/service.yaml
kubectl -n prod set image deployment/devops-web web=plavez/devops-pipeline:${BUILD_NUMBER}
kubectl -n prod rollout status deployment/devops-web
```
## 🧾 Jenkins Stage View (example)
Stage	Duration	Result
Declarative: Checkout SCM	1s	✅
Checkout	1s	✅
Build Docker image	1s	✅
Push to DockerHub	7s	✅
Deploy to dev (auto)	6s	✅
Deploy to prod (manual)	—	✅ (after confirmation)

(Stage View screenshot attached — all stages green)


Verifying Kubernetes Deployment

After a successful pipeline run:
```
kubectl get all -n dev
```

Expected Output:

```
NAME                              READY   STATUS    RESTARTS   AGE
pod/devops-web-7c95fdb78c-7kqmh   1/1     Running   0          10s

NAME                 TYPE        CLUSTER-IP     PORT(S)   AGE
service/devops-svc   ClusterIP   10.43.185.27   80/TCP    10s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/devops-web   1/1     1            1           10s
```

Jenkins Configuration (Docker Compose)
```
services:
  jenkins:
    image: jenkins/jenkins:lts
    container_name: jenkins
    user: root
    restart: unless-stopped
    ports:
      - "8080:8080"
      - "50000:50000"
    environment:
      - TZ=Europe/Berlin
      - JAVA_OPTS=-Djenkins.install.runSetupWizard=false
    volumes:
      - /opt/jenkins_home:/var/jenkins_home
      - /var/run/docker.sock:/var/run/docker.sock
      - /usr/bin/docker:/usr/bin/docker
      - ~/.kube/config.jenkins:/root/.kube/config
```
Conclusion

This project demonstrates the implementation of a complete CI/CD pipeline using modern DevOps tools:

## ✅ Key achievements:

Automated pipeline from GitHub → Jenkins → DockerHub → Kubernetes

Dynamic tagging and deployment with version control

Separate environments for development and production

Secure credentials management in Jenkins

End-to-end automation and monitoring via Jenkins UI

📎 Attachments

Screenshot of Jenkins Stage View (all stages successful)

Output of kubectl get all -n dev

Jenkinsfile (final version)

Dockerfile and Kubernetes manifests

```

---

Would you like me to generate the **architecture diagram** (in PNG) as well — showing the GitHub → Jenkins → DockerHub → K8s flow with arrows and icons?
That’s the last thing Datascientest usually expects in the submission report.
```