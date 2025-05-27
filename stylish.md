<h1 align="center">🚀 3-Tier Application Deployment on AWS ☁️</h1>

<p align="center">
  <img src="https://img.shields.io/badge/AWS-Cloud-orange?logo=amazon-aws" />
  <img src="https://img.shields.io/badge/VPC-Network-blue?logo=cloudflare" />
  <img src="https://img.shields.io/badge/Node.js-App-green?logo=node.js" />
  <img src="https://img.shields.io/badge/MySQL-Database-blue?logo=mysql" />
</p>

---

## 📌 Project Overview

This project demonstrates deploying a **3-tier web application** on AWS using:
- 🌐 **Web Tier**: Frontend hosted via NGINX
- ⚙️ **App Tier**: Node.js backend
- 🗄️ **Database Tier**: MySQL on RDS

---

## 📦 Requirements

### 🖥️ Web Layer:
- NGINX
- S3 for hosting code

### ⚙️ App Layer:
- EC2 Instance (Amazon Linux 2)
- Node.js
- PM2

### 🛢️ Database Layer:
- Amazon RDS (MySQL)

---

## 🛠️ Step 1: Create a VPC

> **VPC Setup**

- VPC Name: `demo-vpc`  
- CIDR Block: `192.168.0.0/22`  
- IPv6: ❌ Not Required  
- Tenancy: Default  
- Subnets:  
  - Public: 2 (`web-1`, `web-2`)  
  - Private: 4  
    - App: `demo-vpc-app1-subnet`, `demo-vpc-app2-subnet`  
    - DB: `demo-vpc-db1-subnet`, `demo-vpc-db2-subnet`  
- NAT Gateway: 1 AZ  

✅ Rename subnets after creation for clarity.

---

## 🔐 Step 2: Create S3 Bucket & IAM Role

### S3 Bucket:
- Unique name: `three-tire-bucket-ssm`
- Public access: Blocked
- ✅ Versioning: Enabled

### IAM Role:
- Name: `demo-ec2-role-ssm`
- Role Type: EC2
- Permissions:
  - `AmazonEC2RoleforSSM`
  - `AmazonEC2FullAccess` (optional for admin)

---

## 🧱 Step 3: Configure RDS (MySQL)

### 📦 Subnet Group:
- Name: `db-subnet`
- VPC: `demo-vpc`
- AZs: `ap-south-1a`, `ap-south-1b`
- Subnets: Select DB private subnets

### 🛢️ RDS Settings:
- Engine: MySQL
- Template: Free Tier
- DB Name: `database-1`
- Username: `admin`
- Password: `admin123`
- VPC: `demo-vpc`
- Subnet Group: `db-subnet`
- AZ: No Preference
- SG: Default

---

## ⚙️ Step 4: Setup Application Tier (EC2)

### Instance Config:
- Name: `AppTireInstance`
- AMI: Amazon Linux 2
- Type: `t2.micro`
- No key pair
- Network: `demo-vpc`
- Subnet: App subnet
- IAM Role: `demo-ec2-role-ssm`
- SG: Default (ensure open traffic for testing)

✅ Use **Session Manager** to connect to EC2.

### Initial Setup Commands:
```bash
sudo su
whoami
cd /home/ec2-user
ping 8.8.8.8
