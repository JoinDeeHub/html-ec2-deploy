**Deployment on EC2 using GitHub Actions**
===================

A simple project demonstrating **CI/CD deployment of a static HTML website to an AWS EC2 (Ubuntu) instance using GitHub Actions and Nginx**.

This project is part of a hands-on DevOps exercise where you:

-   Create an EC2 Ubuntu server

-   Create a dedicated DevOps user

-   Configure SSH keys

-   Create a GitHub repository

-   Add a simple HTML page

-   Configure GitHub Actions for CI/CD

-   Automatically deploy to EC2 on every push to `main`

* * * * *

🗓️ **Day 1 --- EC2 Setup & User Configuration**
----------------------------------------------

### **1\. Launch EC2 Instance**

-   AMI: **Ubuntu 22.04 / 20.04**

-   Instance Type: **t3.micro** (Free tier eligible)

-   Open inbound ports:

    -   **22** → SSH

    -   **80** → HTTP

SSH into the instance:

`ssh -i test.pem ubuntu@<EC2-PUBLIC-IP>`

### **2\. Create DevOps User**

`sudo adduser devopsuser
sudo usermod -aG sudo devopsuser`

### **3\. Configure SSH Key Authentication**

Generate key on **local machine**:

`ssh-keygen -t ed25519 -C "devopsuser"`

Copy the public key to the EC2 server:

`cat ~/.ssh/devopsuser.pub | ssh -i test.pem ubuntu@<EC2-PUBLIC-IP> "sudo tee -a /home/devopsuser/.ssh/authorized_keys"`

Login as devopsuser:

`ssh -i ~/.ssh/devopsuser devopsuser@<EC2-PUBLIC-IP>`

<img width="1365" height="444" alt="Screenshot from 2025-11-19 00-30-52" src="https://github.com/user-attachments/assets/f2b5b495-8831-4a88-b1cc-52d48ca95b52" />


* * * * *

🗓️ **Day 2 --- Create HTML Application**
---------------------------------------

### **1\. Create GitHub Repository**

`Repository Name: html-ec2-deploy`

### **2\. Add Simple HTML File**

`index.html`:

`<!DOCTYPE html>
<html>
<head>
    <title>DevOps Demo</title>
</head>
<body>
    <h1>Hello from GitHub Actions & EC2!</h1>
</body>
</html>`

Push changes to GitHub:

`git add index.html
git commit -m "Add HTML page"
git push origin main`

* * * * *

🗓️ **Day 3 --- CI/CD With GitHub Actions**
-----------------------------------------

### **1\. Add Repository Secrets**

Go to:\
**GitHub → Repo → Settings → Secrets → Actions**

Add:

| Secret Name | Value |
| --- | --- |
| `EC2_HOST` | Public IPv4 of EC2 |
| `EC2_USER` | devopsuser |
| `EC2_SSH_KEY` | Private key content |

<img width="1365" height="444" alt="Screenshot from 2025-11-19 02-41-30" src="https://github.com/user-attachments/assets/5ae47981-5095-4edc-812e-b97de9b8b14e" />


### **2\. Create GitHub Actions Workflow**

Create:

`.github/workflows/deploy.yml`

Add:

`name: Deploy HTML to EC2

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repo
        uses: actions/checkout@v3

      - name: Copy files to EC2
        uses: appleboy/scp-action@v0.1.7
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          source: "./*"
          target: "/home/${{ secrets.EC2_USER }}/app"

      - name: Restart Web Server
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.EC2_HOST }}
          username: ${{ secrets.EC2_USER }}
          key: ${{ secrets.EC2_SSH_KEY }}
          script: |
            sudo apt update
            sudo apt install -y nginx
            sudo cp /home/${{ secrets.EC2_USER }}/app/index.html /var/www/html/index.html
            sudo systemctl restart nginx `

<img width="1365" height="444" alt="Screenshot from 2025-11-19 01-20-53" src="https://github.com/user-attachments/assets/cdb3ed6f-5bbd-4c45-b99d-ea9f899aa0dc" />

            

### 🚀 How Deployment Works

Whenever you push code to **main**, GitHub Actions:

1.  Connects to the EC2 instance over SSH

2.  Copies updated project files

3.  Installs/updates Nginx

4.  Places `index.html` under `/var/www/html`

5.  Restarts the web server

Open your browser:

`http://<EC2-PUBLIC-IP>`
