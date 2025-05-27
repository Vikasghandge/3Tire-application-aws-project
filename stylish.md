#  🚀 3-Tier Application AWS Project 🌩️

```diff
+===========================================================+
|                  PROJECT REQUIREMENTS                     |
+===========================================================+
| 🕸️ WEB LAYER    | 🖥️ APP LAYER    | 🗃️ DATABASE LAYER  |
+-----------------+-----------------+-----------------------+

---
<h1 align="center">🚀 3-Tier Application Deployment on AWS ☁️</h1>

<p align="center">
  <img src="https://img.shields.io/badge/AWS-Cloud-orange?logo=amazon-aws" />
  <img src="https://img.shields.io/badge/VPC-Network-blue?logo=cloudflare" />
  <img src="https://img.shields.io/badge/Node.js-App-green?logo=node.js" />
  <img src="https://img.shields.io/badge/MySQL-Database-blue?logo=mysql" />
</p>

---

## 📋 **Project Overview:**

A simple **3-Tier Architecture Web Application** deployment on **AWS Cloud**

### 🏗️ **Architecture Tiers:**

* **Web Layer** (Frontend - Web Tier EC2)
* **App Layer** (Backend - Node.js App on EC2)
* **Database Layer** (RDS MySQL)

---

## 🌐 **1. VPC Setup**

### 🧰 VPC Configuration:

* VPC Name: `demo-vpc`
* CIDR: `192.168.0.0/22`
* IPv6: ❌ Disabled
* Tenancy: Default
* Public Subnets: `2`
* Private Subnets: `4`
* NAT Gateway: `1` (Single AZ)

### 📝 Rename Subnets:

* App Subnets: `demo-vpc-app1-subnet`, `demo-vpc-app2-subnet`
* DB Subnets: `demo-vpc-db1-subnet`, `demo-vpc-db2-subnet`
* Web Subnets: `web-1`, `web-2`

### 🔐 Security Groups:

* Create security groups or use default ones.
* Allow All Traffic temporarily for testing.

---

## 🪣 **2. S3 Bucket & IAM Role**

### ✅ S3 Bucket:

* Name: **Globally unique**
* Versioning: ✅ **Enabled**

### 👮 IAM Role:

* Name: `demo-ec2-role-ssm`
* Type: EC2 Role
* Permissions: `AmazonEC2RoleforSSM`, `EC2AdminAccess`

---

## 🛢️ **3. RDS Setup (MySQL)**

### 📦 DB Subnet Group:

* Name: `db-subnet`
* VPC: `demo-vpc`
* AZs: `ap-south-1a`, `ap-south-1b`
* Subnets: Select DB1 and DB2 subnets

### 📚 Create RDS DB:

* Engine: MySQL
* Template: Free Tier
* DB Name: `database-1`
* Username: `admin`
* Password: `admin123`
* VPC: `demo-vpc`
* Subnet Group: `db-subnet`
* Security Group: Default

---

## ⚙️ **4. App Tier Setup**

### 🚀 Launch EC2 Instance:

* Name: `AppTireInstance`
* AMI: Amazon Linux 2
* Type: t2.micro
* Key Pair: None
* Network: `demo-vpc`
* Subnet: app1
* IAM Role: `demo-ec2-role-ssm`

### 🔗 Connect & Setup:

```bash
sudo su
ping 8.8.8.8  # Test internet
sudo yum install mysql -y
```

### 📂 Connect to RDS:

```bash
mysql -h <rds-endpoint> -u admin -p
# Example:
mysql -h database-1.cvu4cuyqyi5s.ap-south-1.rds.amazonaws.com -u admin -padmin123
```

### 🛠️ Configure MySQL:

```sql
CREATE DATABASE webappdb;
USE webappdb;
CREATE TABLE IF NOT EXISTS transactions(
  id INT NOT NULL AUTO_INCREMENT,
  amount DECIMAL(10,2),
  description VARCHAR(100),
  PRIMARY KEY(id)
);
INSERT INTO transactions (amount, description) VALUES ('400', 'groceries');
SELECT * FROM transactions;
exit;
```

### 💾 Application Code Setup:

* Update DB config in your local `app-tier/DbConfig`
* Upload updated code to S3: `application-code/app-tier/`

### 💻 Install Node.js & App:

```bash
curl -o- https://raw.githubusercontent.com/avizway1/aws_3tier_architecture/main/install.sh | bash
source ~/.bashrc
nvm install 16
nvm use 16
npm install -g pm2
```

### 📥 Copy App Files from S3:

```bash
sudo aws s3 cp s3://<bucket-name>/application-code/app-tier/ app-tier --recursive
cd app-tier
npm install
pm2 start index.js
pm2 status
pm2 startup
pm2 save
curl http://localhost:4000/health
```

---

## 📡 **5. Internal Load Balancer**

### 🎯 Target Group:

* Name: `App-Internal-TG`
* Protocol: HTTP, Port: 4000
* Health Check: `/health`

### 📶 Load Balancer:

* Name: `App-Internal-LB`
* Type: Internal
* VPC: `demo-vpc`
* AZs: `app1`, `app2` subnets
* SG: Default
* Routing: Attach `App-Internal-TG`

### ⚙️ Update NGINX Config:

* Replace DNS in `nginx.conf` with internal LB DNS
* Upload to S3 bucket

---

## 🌍 **6. Web Tier Setup**

### 🧾 EC2 Instance:

* Name: `Web-Tier-Instance`
* AMI: Amazon Linux 2
* Type: t2.micro
* Network: `demo-vpc`
* Subnet: Public
* IAM Role: `demo-ec2-role-ssm`

### 🔧 Setup:

```bash
sudo su
curl -o- https://raw.githubusercontent.com/avizway1/aws_3tier_architecture/main/install.sh | bash
source ~/.bashrc
nvm install 16
nvm use 16
npm install -g pm2
```

### 📁 Get Web Files:

```bash
sudo aws s3 cp s3://<bucket-name>/application-code/web-tier/ web-tier --recursive
cd web-tier
npm install
npm run build
```

### 🌐 Install & Configure NGINX:

```bash
sudo amazon-linux-extras install nginx1 -y
cd /etc/nginx
sudo rm nginx.conf
sudo aws s3 cp s3://<bucket-name>/application-code/nginx.conf .
sudo service nginx restart
chmod -R 755 /home/ec2-user
sudo chkconfig nginx on
```

📡 **Access your app via public IP**:

```bash
http://<Web-Tier-Instance-Public-IP>
```

✅ **Deployment Complete!**

---

🎉 **Enjoy your working 3-Tier Application on AWS!**
