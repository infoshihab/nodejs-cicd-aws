# 🚀 CI/CD Pipeline for Node.js Backend & Frontend

![CI/CD](https://img.shields.io/badge/CI%2FCD-GitHub%20Actions-blue)
![Node](https://img.shields.io/badge/Node.js-22.x-green)
![AWS](https://img.shields.io/badge/Deployed%20on-AWS%20EC2-orange)
![Nginx](https://img.shields.io/badge/Web%20Server-Nginx-brightgreen)

---

## 📌 Overview

This repository contains a **complete, production-ready CI/CD implementation** for deploying a **Node.js Backend and Frontend** application using **GitHub Actions**, **AWS EC2**, **PM2**, and **Nginx**.

The setup is designed to be **simple, reliable, and scalable**, making it suitable for real-world production environments.

---

Here’s the corrected **file structure section** for your documentation, reflecting **separate repositories** for backend and frontend:

---

## 🗂️ Repository Structure

> ⚠️ **Note:** Backend and Frontend are maintained in **separate repositories**. Each repository has its own GitHub Actions workflow.

---

### 📦 Backend Repository

```text
.github/
└── workflows/
    └── backend-ci.yml        # Backend CI/CD workflow

server.js                     # Backend entry file
package.json
.env                           # Environment variables (not committed)
```

---

### 🎨 Frontend Repository

```text
.github/
└── workflows/
    └── frontend-ci.yml       # Frontend CI/CD workflow

package.json
dist/                          # Production build output
```

---

## 🔄 CI/CD Architecture Flow

```text
Git Push (main branch)
        ↓
GitHub Actions Workflow
        ↓
Self-Hosted Runner (AWS EC2)
        ↓
Install Dependencies (npm ci)
        ↓
Build Application
        ↓
Backend → Restart via PM2
Frontend → Served via Nginx
```

---

## ⚙️ Backend CI/CD

### 📄 Workflow File

**File:** `.github/workflows/backend-ci.yml`

### 🔹 Trigger

* Runs on every push to the `main` branch

### 🔹 What This Workflow Does

* Backs up the `.env` file before deployment
* Pulls the latest code from GitHub
* Restores `.env` after checkout
* Installs dependencies using `npm ci`
* Builds the backend (if required)
* Restarts the backend using PM2

### 📄 Backend Workflow Configuration

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

## 🎨 Frontend CI/CD

### 📄 Workflow File

**File:** `.github/workflows/frontend-ci.yml`

### 🔹 Trigger

* Runs on every push to the `main` branch

### 🔹 What This Workflow Does

* Installs frontend dependencies
* Builds the frontend project
* Generates production-ready `dist/` folder

### 📄 Frontend Workflow Configuration

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

### 🔐 Security Group Configuration

* Inbound Rule:

  * Type: Custom TCP
  * Port: **80**
  * Source: **Anywhere (0.0.0.0/0)**

---

### 📦 Install Required Packages (EC2)

```bash
sudo apt update
sudo apt install nodejs npm -y
sudo npm install -g pm2
sudo apt install nginx -y
```

---

## 🖥️ Backend Deployment (Self-Hosted Runner)

### Step 1: Add Runner

* GitHub → Repository → Settings → Actions → Runners
* Add **Self-hosted Runner (Linux)**

### Step 2: Configure Runner as a Service

> ⚠️ We do NOT use `./run.sh`

```bash
sudo ./svc.sh install
sudo ./svc.sh start
```

### Step 3: Create Environment File

```bash
nano .env
```

### Step 4: Start Backend with PM2

```bash
sudo pm2 start server.js --name=backend
```

---

## 🌍 Frontend Deployment with Nginx

### Step 1: Add Self-Hosted Runner

* Repeat the runner setup steps for frontend

### Step 2: Configure Runner Service

```bash
sudo ./svc.sh install
sudo ./svc.sh start
```

### Step 3: Environment Variables

* Store frontend environment variables in **GitHub Secrets** (if required)

---

### 🌐 Nginx Configuration

#### 1️⃣ Get Build Path

```bash
cd dist
pwd
```

#### 2️⃣ Edit Nginx Config

```bash
cd /etc/nginx/sites-available
sudo nano default
```

* Update the `root` path with the copied `dist` directory path

#### 3️⃣ Restart Nginx

```bash
sudo systemctl restart nginx
sudo systemctl reload nginx
```

#### 4️⃣ Fix Directory Permissions (Important)

```bash
sudo chmod o+x /home/ubuntu/frontend-runner/_work/pocketpuls-frontend/pocketpuls-frontend
sudo chmod o+x /home/ubuntu/frontend-runner/_work/pocketpuls-frontend
sudo chmod o+x /home/ubuntu/frontend-runner/_work
sudo chmod o+x /home/ubuntu/frontend-runner
sudo chmod o+x /home/ubuntu
sudo chmod o+x /home
```

---

## ✅ Final Result

* Automated CI/CD for backend & frontend
* Backend managed with PM2
* Frontend served using Nginx
* Secure, clean, and scalable deployment
* Suitable for real-world production
---

**Maintained by:** Shihab Uddin
