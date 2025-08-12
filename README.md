# 🚀 Complete Guide: Express App Deployment on AWS EC2 with Nginx, Domain & SSL

## 📋 Table of Contents
1. [Introduction to AWS Services](#introduction-to-aws-services)
2. [Prerequisites](#prerequisites)
3. [AWS Setup & Configuration](#aws-setup--configuration)
4. [EC2 Instance Setup](#ec2-instance-setup)
5. [Express App Deployment](#express-app-deployment)
6. [Nginx Configuration](#nginx-configuration)
7. [Domain & SSL Setup](#domain--ssl-setup)

---

## 🌐 Introduction to AWS Services

### Amazon Web Services (AWS)
AWS is a comprehensive cloud computing platform offering over 200 services including computing power, storage, databases, and networking used by millions of businesses worldwide.

### Key AWS Services

#### 🖥️ Amazon EC2 (Elastic Compute Cloud)
Virtual servers in the cloud providing scalable computing capacity with pay-only-for-what-you-use pricing.

```
┌─────────────────────────────────────────┐
│            AWS EC2 Instance             │
├─────────────────────────────────────────┤
│  Operating System: Ubuntu/Amazon Linux │
│  CPU: Variable (t2.micro = 1 vCPU)     │
│  RAM: Variable (t2.micro = 1 GB)       │
│  Storage: EBS Volume                    │
│  Network: VPC with Security Groups     │
└─────────────────────────────────────────┘
```

#### 📦 Amazon S3 (Simple Storage Service)
Object storage service providing 99.999999999% (11 9's) durability with unlimited storage capacity and built-in security.

#### 🚀 AWS Elastic Beanstalk
Platform-as-a-Service (PaaS) offering automatic scaling, health monitoring, and easy deployment without infrastructure management.

### 🔄 Reverse Proxy Architecture
```
Client Request Flow:
Internet → Domain → Load Balancer → Nginx (Reverse Proxy) → Express App

┌─────────┐    ┌─────────┐    ┌─────────────┐    ┌─────────────┐
│ Browser │───▶│ Domain  │───▶│   Nginx     │───▶│ Express App │
│         │    │ (SSL)   │    │(Port 80/443)│    │ (Port 3000) │
└─────────┘    └─────────┘    └─────────────┘    └─────────────┘
```

**Nginx Benefits:** SSL termination, load balancing, caching, compression, security filtering

---

## 📋 Prerequisites

### Required Accounts & Tools
- [ ] AWS Account (Free tier available)
- [ ] Domain name (from registrar like Namecheap, GoDaddy, etc.)
- [ ] SSH client (Terminal on Mac/Linux, PuTTY on Windows)
- [ ] Basic knowledge of Linux commands
- [ ] Node.js Express application ready for deployment

### Local Development Setup
```bash
# Ensure you have Node.js installed
node --version
npm --version

# Your Express app structure should look like:
my-express-app/
├── package.json
├── app.js (or index.js)
├── routes/
├── public/
└── views/
```

---

## 🔧 AWS Setup & Configuration

### Step 1: Create AWS Account
1. Go to [aws.amazon.com](https://aws.amazon.com)
2. Click "Create an AWS Account"
3. Follow the registration process
4. Add payment method (Free tier available)
5. Verify your identity

### Step 2: Access AWS Management Console
```
AWS Console Structure:
├── Services (200+ services)
├── EC2 Dashboard
├── S3 Dashboard
├── IAM (Identity & Access Management)
├── VPC (Virtual Private Cloud)
└── Route 53 (DNS Service)
```

### Step 3: Understanding AWS Regions
Choose a region close to your target audience:
- **US East (N. Virginia)** - us-east-1
- **US West (Oregon)** - us-west-2
- **Europe (Ireland)** - eu-west-1
- **Asia Pacific (Singapore)** - ap-southeast-1

---

## 🖥️ EC2 Instance Setup

### Step 1: Launch EC2 Instance

#### 1.1 Navigate to EC2 Dashboard
```
AWS Console → Services → EC2 → Launch Instance
```

#### 1.2 Choose AMI (Amazon Machine Image)
| AMI Type | Use Case | Package Manager |
|----------|----------|-----------------|
| Ubuntu 22.04 LTS | General purpose, large community | apt |
| Amazon Linux 2 | AWS optimized | yum |
| CentOS | Enterprise focused | yum |

**Recommended: Ubuntu 22.04 LTS**

#### 1.3 Choose Instance Type
| Instance Type | vCPUs | Memory | Use Case |
|---------------|-------|--------|----------|
| t2.micro | 1 | 1 GB | Free tier, small apps |
| t2.small | 1 | 2 GB | Light production |
| t2.medium | 2 | 4 GB | Medium traffic |
| t3.small | 2 | 2 GB | Better performance |

**For beginners: t2.micro (Free tier eligible)**

#### 1.4 Configure Security Group
Create a new security group with these rules:

| Type | Protocol | Port Range | Source | Description |
|------|----------|------------|--------|-------------|
| SSH | TCP | 22 | My IP | SSH access |
| HTTP | TCP | 80 | 0.0.0.0/0 | Web traffic |
| HTTPS | TCP | 443 | 0.0.0.0/0 | Secure web traffic |
| Custom TCP | TCP | 3000 | 0.0.0.0/0 | Express app (temporary) |

#### 1.5 Create Key Pair
```bash
# AWS will generate a .pem file
# Download and store securely
# Example: my-ec2-key.pem
```

### Step 2: Connect to EC2 Instance

#### 2.1 Get Instance Details
After launching, note:
- **Public IPv4 address**: e.g., 3.15.45.67
- **Public IPv4 DNS**: e.g., ec2-3-15-45-67.us-east-2.compute.amazonaws.com

#### 2.2 Connect via SSH

**For Mac/Linux:**
```bash
# Make key file secure
chmod 400 my-ec2-key.pem

# Connect to instance
ssh -i "my-ec2-key.pem" ubuntu@your-public-ip

# Example:
ssh -i "my-ec2-key.pem" ubuntu@3.15.45.67
```

**For Windows (using PuTTY):**
1. Convert .pem to .ppk using PuTTYgen
2. Use PuTTY with the .ppk key
3. Connect to ubuntu@your-public-ip

#### 2.3 Initial Server Setup
```bash
# Update package list
sudo apt update && sudo apt upgrade -y

# Install essential packages
sudo apt install -y curl wget git htop nginx  

# Verify installation
nginx -v
```

---

## 📱 Express App Deployment

### Step 1: Install Node.js on EC2

#### 1.1 Install Node.js via NodeSource
```bash
# Add NodeSource repository
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -

# Install Node.js
sudo apt-get install -y nodejs

# Verify installation
node --version
npm --version
```

#### 1.2 Alternative: Using NVM (Node Version Manager)
```bash
# Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# Reload bash profile
source ~/.bashrc

# Install latest LTS Node.js
nvm install --lts
nvm use --lts
```

### Step 2: Deploy Your Express App

#### 2.1 Clone Your Repository
```bash
# If using Git
git clone https://github.com/yourusername/your-express-app.git
cd your-express-app

# Or upload files using SCP
# scp -i "key.pem" -r ./local-app ubuntu@ip:/home/ubuntu/
```

#### 2.2 Install Dependencies
```bash
# Install production dependencies
npm install --production

# If you have a build step
npm run build
```

#### 2.3 Configure Environment Variables
```bash
# Create environment file
nano .env

# Add your variables
NODE_ENV=production
PORT=3000
DB_CONNECTION_STRING=your_db_url
API_KEY=your_api_key
```

#### 2.4 Test Your Application
```bash
# Start the application
npm start

# Or if you have a start script
node app.js
```

Test in browser: `http://your-ec2-ip:3000`

### Step 3: Setup Process Manager (PM2)

#### 3.1 Install PM2
```bash
# Install PM2 globally
sudo npm install -g pm2

# Verify installation
pm2 --version
```

#### 3.2 Start App with PM2
```bash
# Start your app
pm2 start app.js --name "my-express-app"

# Or use ecosystem file
pm2 start ecosystem.config.js
```

#### 3.3 PM2 Ecosystem Configuration
Create `ecosystem.config.js`:
```javascript
module.exports = {
  apps: [{
    name: 'my-express-app',
    script: 'app.js',
    instances: 'max',
    exec_mode: 'cluster',
    env: {
      NODE_ENV: 'development',
      PORT: 3000
    },
    env_production: {
      NODE_ENV: 'production',
      PORT: 3000
    },
    error_file: './logs/err.log',
    out_file: './logs/out.log',
    log_file: './logs/combined.log',
    time: true
  }]
};
```

#### 3.4 PM2 Management Commands
```bash
# List running processes
pm2 list

# Monitor processes
pm2 monit

# View logs
pm2 logs

# Restart app
pm2 restart my-express-app

# Stop app
pm2 stop my-express-app

# Auto-start on server reboot
pm2 startup
pm2 save
```

---

## ⚡ Nginx Configuration

### Step 1: Understanding Nginx Structure
```
/etc/nginx/
├── nginx.conf (main configuration)
├── sites-available/ (available site configs)
├── sites-enabled/ (active site configs)
├── conf.d/ (additional configurations)
└── snippets/ (reusable config snippets)
```

### Step 2: Create Nginx Server Block

#### 2.1 Create Configuration File
```bash
# Create new site configuration
sudo nano /etc/nginx/sites-available/your-domain.com
```

#### 2.2 Basic Nginx Configuration
```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
    
    # Serve static files directly
    location /static/ {
        alias /home/ubuntu/your-express-app/public/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
    
    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header Referrer-Policy "no-referrer-when-downgrade" always;
    add_header Content-Security-Policy "default-src 'self' http: https: data: blob: 'unsafe-inline'" always;
}
```

#### 2.3 Enable Site Configuration
```bash
# Create symbolic link to enable site
sudo ln -s /etc/nginx/sites-available/your-domain.com /etc/nginx/sites-enabled/

# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx
```

### Step 3: Advanced Nginx Features

#### 3.1 Rate Limiting
```nginx
# Add to http block in /etc/nginx/nginx.conf
http {
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/m;
    
    server {
        location /api/ {
            limit_req zone=api burst=5 nodelay;
            proxy_pass http://localhost:3000;
        }
    }
}
```

#### 3.2 Load Balancing (Multiple App Instances)
```nginx
upstream app_servers {
    server localhost:3000 weight=3;
    server localhost:3001 weight=2;
    server localhost:3002 weight=1;
}

server {
    location / {
        proxy_pass http://app_servers;
    }
}
```

#### 3.3 Caching Configuration
```nginx
# Add caching
proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=10g 
                 inactive=60m use_temp_path=off;

server {
    location / {
        proxy_cache my_cache;
        proxy_cache_valid 200 1h;
        proxy_cache_use_stale error timeout http_500 http_502 http_503 http_504;
        proxy_pass http://localhost:3000;
    }
}
```

---

## 🌐 Domain & SSL Setup

### Step 1: Domain Configuration

#### 1.1 Purchase Domain
Popular registrars:
- Namecheap
- GoDaddy 
- Google Domains
- Cloudflare

#### 1.2 Configure DNS Records
Add these DNS records at your domain registrar:

| Type | Name | Value | TTL |
|------|------|-------|-----|
| A | @ | your-ec2-ip | 3600 |
| A | www | your-ec2-ip | 3600 |
| CNAME | * | your-domain.com | 3600 |

**Example:**
```
A     @     3.15.45.67     3600
A     www   3.15.45.67     3600
```

#### 1.3 Verify DNS Propagation
```bash
# Check DNS resolution
nslookup your-domain.com
dig your-domain.com

# Online tools:
# - whatsmydns.net
# - dnschecker.org
```

### Step 2: SSL Certificate with Let's Encrypt

#### 2.1 Install Certbot
```bash
# Install Certbot
sudo apt install snapd
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot

# Create symlink
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

#### 2.2 Obtain SSL Certificate
```bash
# Automatic configuration with Nginx
sudo certbot --nginx -d your-domain.com -d www.your-domain.com

# Manual certificate only
sudo certbot certonly --nginx -d your-domain.com -d www.your-domain.com
```

#### 2.3 Automatic Renewal Setup
```bash
# Test automatic renewal
sudo certbot renew --dry-run

# Check renewal timer
sudo systemctl status snap.certbot.renew.timer

# Manual renewal if needed
sudo certbot renew
```

### Step 3: Updated Nginx Configuration with SSL

After Certbot, your Nginx config will look like:
```nginx
server {
    listen 80;
    server_name your-domain.com www.your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com www.your-domain.com;
    
    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;
    include /etc/letsencrypt/options-ssl-nginx.conf;
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem;
    
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

---

## 📊 Monitoring & Maintenance

### Step 1: Server Monitoring

#### 1.1 System Resource Monitoring
```bash
# Check system resources
htop
free -h
df -h

# Check running processes
ps aux | grep node

# Monitor network connections
netstat -tulpn | grep :80
netstat -tulpn | grep :443
```

#### 1.2 Application Logs
```bash
# PM2 logs
pm2 logs --lines 100

# Nginx logs
sudo tail -f /var/log/nginx/access.log
sudo tail -f /var/log/nginx/error.log

# System logs
sudo journalctl -f
```

#### 1.3 Setup Log Rotation
```bash
# Configure PM2 log rotation
pm2 install pm2-logrotate

# Configure Nginx log rotation (usually pre-configured)
sudo nano /etc/logrotate.d/nginx
```

### Step 2: Backup Strategy

#### 2.1 Application Backup
```bash
# Create backup script
nano backup-app.sh

#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
tar -czf /home/ubuntu/backups/app_backup_$DATE.tar.gz /home/ubuntu/your-express-app
```

#### 2.2 Database Backup (if applicable)
```bash
# MongoDB backup example
mongodump --out /home/ubuntu/backups/mongodb_$DATE

# MySQL backup example
mysqldump -u username -p database_name > /home/ubuntu/backups/mysql_$DATE.sql
```

#### 2.3 Automated Backups with Cron
```bash
# Edit crontab
crontab -e

# Add daily backup at 2 AM
0 2 * * * /home/ubuntu/backup-app.sh

# Weekly full system backup
0 3 * * 0 tar -czf /home/ubuntu/backups/full_backup_$(date +%Y%m%d).tar.gz /home/ubuntu/
```

### Step 3: Security Best Practices

#### 3.1 Firewall Configuration (UFW)
```bash
# Install and configure UFW
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 'Nginx Full'
sudo ufw status
```

#### 3.2 Fail2Ban for SSH Protection
```bash
# Install Fail2Ban
sudo apt install fail2ban

# Configure
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local

# Start service
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

#### 3.3 Regular Security Updates
```bash
# Update packages regularly
sudo apt update && sudo apt upgrade -y

# Auto-update security patches
sudo apt install unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

---

## 🔄 Alternative Deployment Methods

### Option 1: AWS Elastic Beanstalk

#### When to Use Elastic Beanstalk:
- Quick deployment without server management
- Automatic scaling and load balancing
- Built-in monitoring and health checks
- Easy rollbacks and version management

#### Deployment Steps:
```bash
# Install EB CLI
pip install awsebcli

# Initialize EB application
eb init -p node.js my-express-app

# Create environment and deploy
eb create production-env
eb deploy
```

#### Beanstalk vs EC2 Comparison:
| Feature | EC2 | Elastic Beanstalk |
|---------|-----|-------------------|
| Control | Full control | Managed service |
| Setup Time | Hours | Minutes |
| Scaling | Manual | Automatic |
| Cost | Lower for simple apps | Higher but includes management |
| Learning Curve | Steep | Gentle |

### Option 2: AWS S3 + CloudFront (Static Sites)

For static sites generated from Express (using build tools):

#### 2.1 Build Static Files
```bash
# If using a static site generator
npm run build
# This creates a 'build' or 'dist' folder
```

#### 2.2 S3 Bucket Setup
```bash
# Install AWS CLI
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install

# Configure AWS CLI
aws configure

# Create S3 bucket
aws s3 mb s3://your-domain.com

# Upload files
aws s3 sync ./build s3://your-domain.com --delete
```

#### 2.3 Configure S3 for Static Hosting
```bash
# Enable static website hosting
aws s3 website s3://your-domain.com --index-document index.html --error-document error.html
```

### Option 3: Docker on EC2

#### 3.1 Create Dockerfile
```dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY . .

EXPOSE 3000

CMD ["node", "app.js"]
```

#### 3.2 Build and Deploy
```bash
# On your local machine
docker build -t my-express-app .
docker save my-express-app | gzip > my-express-app.tar.gz

# Transfer to EC2
scp -i "key.pem" my-express-app.tar.gz ubuntu@ec2-ip:/home/ubuntu/

# On EC2
sudo apt install docker.io
docker load < my-express-app.tar.gz
docker run -d -p 3000:3000 --name my-app my-express-app
```

---

## 🔧 Troubleshooting

### Common Issues and Solutions

#### Issue 1: Cannot Connect to EC2 Instance
```bash
# Check security group allows SSH (port 22)
# Verify correct key file permissions
chmod 400 your-key.pem

# Use correct username (ubuntu for Ubuntu AMI)
ssh -i "key.pem" ubuntu@ip-address

# Debug SSH connection
ssh -i "key.pem" -v ubuntu@ip-address
```

#### Issue 2: Nginx Returns 502 Bad Gateway
```bash
# Check if your app is running
pm2 status
curl http://localhost:3000

# Check Nginx error logs
sudo tail -f /var/log/nginx/error.log

# Verify proxy_pass URL in Nginx config
sudo nginx -t
```

#### Issue 3: SSL Certificate Issues
```bash
# Check certificate status
sudo certbot certificates

# Renew certificate
sudo certbot renew --force-renewal

# Check Nginx SSL configuration
sudo nginx -t
```

#### Issue 4: Domain Not Resolving
```bash
# Check DNS propagation
nslookup your-domain.com
dig your-domain.com

# Verify A records point to correct IP
# Wait for DNS propagation (up to 48 hours)
```

#### Issue 5: High Memory Usage
```bash
# Check memory usage
free -h
htop

# Optimize PM2 configuration
pm2 delete all
pm2 start ecosystem.config.js

# Consider upgrading instance type
```

### Performance Optimization

#### 1. Node.js Optimization
```javascript
// Enable compression
const compression = require('compression');
app.use(compression());

// Set proper cache headers
app.use(express.static('public', {
  maxAge: '1d',
  etag: false
}));
```

#### 2. Nginx Optimization
```nginx
# Enable gzip compression
gzip on;
gzip_vary on;
gzip_min_length 1024;
gzip_types text/plain text/css application/json application/javascript text/xml application/xml;

# Enable HTTP/2
listen 443 ssl http2;
