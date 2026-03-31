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

'''
Generate SSH Key (Recommended)

Run this on your local machine (Git Bash / terminal):

ssh-keygen -t rsa -b 4096 -C "github-actions"
'''

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






# 🚀 Deploy HTML Site to S3 using GitHub Actions

This guide shows how to host a static website using AWS S3 and automate deployment with GitHub Actions.

---

## 📦 Tech Stack

* GitHub (Code Repository)
* GitHub Actions (CI/CD)
* AWS S3 (Static Hosting)

---

## 🪣 1. Create S3 Bucket

* Go to AWS → S3
* Create bucket
* Bucket name must be **globally unique**

Example:

```
my-html-site-123
```

---

## 🌐 2. Enable Static Website Hosting

In S3 bucket:

* Go to **Properties**
* Enable:

  * Static website hosting
* Set:

  * Index document → `index.html`

You’ll get a URL like:

```
http://your-bucket-name.s3-website-region.amazonaws.com
```

---

## 🔓 3. Make Bucket Public

Go to:
**Permissions → Block public access**

* Disable "Block all public access"

Add bucket policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicRead",
      "Effect": "Allow",
      "Principal": "*",
      "Action": ["s3:GetObject"],
      "Resource": ["arn:aws:s3:::your-bucket-name/*"]
    }
  ]
}
```

---

## 🔐 4. Create IAM User

Go to:
AWS → IAM

Create user with:

* Programmatic access

Attach policy:

* `AmazonS3FullAccess` (or limited custom)

Save:

* Access Key
* Secret Key

---

## 🔑 5. Add GitHub Secrets

In your GitHub repo:

**Settings → Secrets → Actions**

Add:

| Secret                | Value            |
| --------------------- | ---------------- |
| AWS_ACCESS_KEY_ID     | your key         |
| AWS_SECRET_ACCESS_KEY | your secret      |
| AWS_REGION            | e.g. ap-south-1  |
| S3_BUCKET             | your bucket name |

---

## ⚙️ 6. GitHub Actions Workflow

Create:

```
.github/workflows/deploy.yml
```

Paste:

```yaml
name: Deploy to S3

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          region: ${{ secrets.AWS_REGION }}

      - name: Deploy to S3
        run: |
          aws s3 sync . s3://${{ secrets.S3_BUCKET }} --delete
```

---

## 🚀 7. Deploy

1. Push code to `main`
2. GitHub Actions runs
3. Files uploaded to S3
4. Website is live 🎉

---

## 🌍 Access Website

Open:

```
http://your-bucket-name.s3-website-region.amazonaws.com
```

---

## ⚡ Optional Improvements

### ✅ Use CloudFront (CDN)

* Faster global delivery
* HTTPS support

### ✅ Custom Domain

* Route53 → point domain to S3/CloudFront

### ✅ Cache Control

```bash
aws s3 sync . s3://bucket --cache-control "max-age=3600"
```

---

## 🧪 Debugging

### Check files uploaded

```bash
aws s3 ls s3://your-bucket-name
```

### Fix public access issue

* Ensure bucket policy is correct
* Public access block disabled

---

## ❗ Common Issues

* Access denied → wrong IAM permissions
* Website not loading → static hosting not enabled
* 403 error → bucket not public
* Wrong region in URL

---

## 🧠 Notes

* S3 is best for static sites (HTML, CSS, JS)
* No server needed
* Very low cost

---

## ✅ Done

Now you have a serverless deployment pipeline 🚀
