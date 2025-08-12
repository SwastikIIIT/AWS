# Express App Deployment on AWS EC2 with Nginx, Domain, and SSL

This guide explains how to deploy a Node.js + Express app on **AWS EC2** using **Nginx reverse proxy**, attach a **custom domain**, and enable **SSL with Let's Encrypt**.

---

## 1. Launch an EC2 Instance
1. Go to [AWS EC2 Console](https://console.aws.amazon.com/ec2/)
2. **Launch Instance**
   - **AMI**: Ubuntu 22.04 LTS (or Amazon Linux 2)
   - **Instance type**: t2.micro (free tier)
   - **Key pair**: Create/choose a `.pem` key
   - **Security group**: Allow ports:
     - **22** (SSH)
     - **80** (HTTP)
     - **443** (HTTPS)
3. Launch the instance.

---

## 2. Connect to EC2 via SSH
```bash
ssh -i "your-key.pem" ubuntu@your-ec2-public-ip
