# 🚀 CI/CD Pipeline for Node.js Backend & Frontend

![CI/CD](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-blue)
![Node](https://img.shields.io/badge/Node.js-22.x-green)
![AWS](https://img.shields.io/badge/Deployed%20on-AWS%20EC2-orange)
![Nginx](https://img.shields.io/badge/Web%20Server-Nginx-brightgreen)

---

## 📌 Overview

This repository demonstrates a **production-ready CI/CD pipeline** for a **Node.js backend and frontend application**. The setup ensures **automatic build and deployment** whenever code is pushed to the `main` branch.

---

## 🧱 Technology Stack

| Layer           | Technology                |
| --------------- | ------------------------- |
| Runtime         | Node.js (v22.x)           |
| CI/CD           | GitHub Actions            |
| Cloud           | AWS EC2 (Ubuntu)          |
| Backend Process | PM2                       |
| Frontend Server | Nginx                     |
| Runner          | GitHub Self-hosted Runner |

---

## 🔄 CI/CD Architecture Diagram

```text
Developer Push (main branch)
        ↓
GitHub Actions Workflow
        ↓
Self-Hosted Runner (AWS EC2)
        ↓
Install Dependencies (npm ci)
        ↓
Build Application
        ↓
Backend → PM2 Restart
Frontend → Served via Nginx
```

---

## ⚙️ Backend CI/CD Workflow

### Trigger

* Runs automatically on every push to `main`

### Responsibilities

* Backup and restore `.env`
* Install dependencies
* Build backend (if required)
* Restart backend using PM2

### Workflow File

```yaml
name: Node.js CI

on:
  push:
    branches: ["main"]

jobs:
  build:
    runs-on: self-hosted

    strategy:
      matrix:
        node-version: [22.x]

    steps:
      - name: Backup .env
        run: |
          if [ -f .env ]; then mv .env /tmp/env_backup; fi

      - uses: actions/checkout@v4

      - name: Restore .env
        run: |
          if [ -f /tmp/env_backup ]; then mv /tmp/env_backup .env; fi

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22.x
          cache: npm

      - run: npm ci
      - run: npm run build --if-present
      - run: sudo pm2 restart backend
```

---

## 🎨 Frontend CI/CD Workflow

### Responsibilities

* Install dependencies
* Build frontend
* Generate production-ready `dist/` folder

### Workflow File

```yaml
name: Node.js CI

on:
  push:
    branches: ["main"]

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [22.x]

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 22.x
          cache: npm

      - run: npm ci
      - run: npm run build --if-present
      - run: npm run build
```

---

## ☁️ AWS EC2 Server Setup

### Security Group

* Inbound Rule:

  * Port: **80**
  * Protocol: **TCP**
  * Source: **0.0.0.0/0**

### Install Required Packages

```bash
sudo apt update
sudo apt install nodejs npm -y
sudo npm install -g pm2
sudo apt install nginx -y
```

---

## 🖥️ Backend Deployment (Self-Hosted Runner)

1. Add **Self-hosted Runner (Linux)** from GitHub repository settings
2. Run the provided download commands on EC2

### Configure Runner as Service

```bash
sudo ./svc.sh install
sudo ./svc.sh start
```

### Create Environment File

```bash
nano .env
```

### Start Backend

```bash
sudo pm2 start server.js --name=backend
```

---

## 🌍 Frontend Deployment with Nginx

### Runner Setup

```bash
sudo ./svc.sh install
sudo ./svc.sh start
```

### Nginx Configuration

1. Get build path

```bash
cd dist
pwd
```

2. Edit Nginx config

```bash
cd /etc/nginx/sites-available
sudo nano default
```

3. Update `root` path with `dist` directory

4. Restart services

```bash
sudo systemctl restart nginx
sudo systemctl reload nginx
```

5. Fix permission

```bash
sudo chmod o+x /home/ubuntu/frontend-runner/_work/pocketpuls-frontend/pocketpuls-frontend
```

---

## ✅ Final Outcome

* Automated CI/CD pipeline
* Backend auto-restart with PM2
* Frontend served via Nginx
* Secure and scalable deployment
* Production-ready workflow

---

## 🚀 Future Improvements

* HTTPS with SSL (Let's Encrypt)
* Zero-downtime deployment
* GitHub Actions cache optimization
* Docker-based deployment
* Monitoring & logging

---

**Maintained by:** Shihab
