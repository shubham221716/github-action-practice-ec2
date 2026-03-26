# 🚀 HTML Site Deployment to EC2 using GitHub Actions

This repository demonstrates how to deploy a static HTML website to an AWS EC2 instance using GitHub Actions.

---

## 📦 Tech Stack

* GitHub (Code Repository)
* GitHub Actions (CI/CD)
* AWS EC2 (Hosting Server)
* Nginx (Web Server)

---

## ⚙️ Prerequisites

### 1. Create EC2 Instance

* Launch Ubuntu EC2 instance
* Allow inbound traffic:

  * HTTP (80)
  * HTTPS (443)
  * SSH (22)

---

### 2. Connect to EC2

```bash
ssh -i your-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

---

### 3. Install Nginx

```bash
sudo apt update
sudo apt install nginx -y
```

Start and enable:

```bash
sudo systemctl start nginx
sudo systemctl enable nginx
```

---

### 4. Setup Website Directory

```bash
sudo chown -R ubuntu:ubuntu /var/www/html
```

---

## 🔐 GitHub Secrets Setup

Go to:
**Repo → Settings → Secrets and variables → Actions**

Add:

| Secret Name | Value                      |
| ----------- | -------------------------- |
| EC2_HOST    | EC2 Public IP              |
| EC2_USER    | ubuntu                     |
| EC2_SSH_KEY | Private key (.pem content) |
| EC2_PORT    | 22                         |

---

## 🔑 SSH Key Notes

* Use **private key** (NOT `.pub`)
* Format must include:

```
-----BEGIN RSA PRIVATE KEY-----
...
-----END RSA PRIVATE KEY-----
```

---

## ⚙️ GitHub Actions Workflow

Create file:

```
.github/workflows/deploy.yml
```

Paste:

```yaml
name: Deploy HTML to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repo
        uses: actions/checkout@v4

      - name: Copy files to EC2
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          source: "*"
          target: "/var/www/html"

      - name: Restart Nginx
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            sudo systemctl restart nginx
```

---

## 🚀 Deployment Steps

1. Push code to `main` branch
2. GitHub Actions runs automatically
3. Files copied to EC2
4. Nginx serves the updated website

---

## 🌐 Access Website

Open browser:

```
http://YOUR_EC2_PUBLIC_IP
```

---

## 🧪 Debugging

### Check Nginx

```bash
sudo systemctl status nginx
```

### Test locally

```bash
curl localhost
```

### Fix permissions

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/authorized_keys
```

---

## ❗ Common Issues

* SSH authentication failed → wrong key or user
* Site not loading → port 80 blocked
* Permission denied → fix ownership of `/var/www/html`
* No public access → missing public IP

---

## 🧠 Notes

* EC2 public IP changes on restart → use Elastic IP for stability
* Always keep `.pem` file secure
* Do not commit secrets to repo

---

## ✅ Done

Now you have a fully automated CI/CD pipeline for deploying a static HTML site 🚀
