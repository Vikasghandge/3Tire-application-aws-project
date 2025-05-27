markdown
# 🚀 3-Tier Application AWS Project 🌩️

```diff
+===========================================================+
|                  PROJECT REQUIREMENTS                     |
+===========================================================+
| 🕸️ WEB LAYER    | 🖥️ APP LAYER    | 🗃️ DATABASE LAYER  |
+-----------------+-----------------+-----------------------+
🔧 1) VPC Creation (Network Foundation)
bash
🔵 [LOG] Starting VPC creation...
✅ VPC Name: demo-vpc
✅ CIDR: 192.168.0.0/22
✅ IPv6: Disabled
✅ Tenancy: Default
✅ Public Subnets: 2 (Web Tier)
✅ Private Subnets: 4 (App+DB Tiers)
✅ NAT Gateway: 1 AZ
🏷️ Subnet Naming Convention
markdown
🟢 Public Subnets:
- web-1 (AZ1a)
- web-2 (AZ1b)

🔴 Private Subnets:
- demo-vpc-app1-subnet (AZ1a)
- demo-vpc-app2-subnet (AZ1b)
- demo-vpc-db1-subnet (AZ1a)
- demo-vpc-db2-subnet (AZ1b)
🔐 Security Groups
diff
! Important: Either create dedicated SGs or allow all traffic temporarily
+ web-tier-sg
+ app-tier-sg 
+ db-tier-sg
+ lb-sg
📦 2) S3 Bucket Setup
bash
🟣 [LOG] Configuring S3 for code storage...
🔷 Bucket Name: three-tire-bucket-ssm (must be globally unique)
🔷 Versioning: ✅ ENABLED (critical for rollbacks)
🔷 Permissions: Public access blocked (recommended)
👮 3) IAM Role Configuration
bash
🟡 [LOG] Creating EC2 instance role...
🛠️ Service: EC2
🔑 Permissions:
   - AmazonEC2RoleforSSM (required for Session Manager)
   - EC2-Administrator (temporary for setup)
📛 Role Name: demo-ec2-role-ssm
🗄️ 4) RDS Database Setup
sql
-- [DATABASE CONFIGURATION LOG] --
1️⃣ Create Subnet Group:
   - Name: db-subnet
   - VPC: demo-vpc
   - AZs: ap-south-1a, ap-south-1b
   - Subnets: db1 & db2

2️⃣ Launch RDS Instance:
   - Engine: MySQL
   - Template: Free Tier
   - Identifier: database-1
   - Credentials: 
     👨‍💻 User: admin
     🔑 Pass: admin123
   - VPC: demo-vpc
   - Connectivity: Default SG (temporary)
🛠️ Database Initialization
sql
CREATE DATABASE webappdb;

USE webappdb;

CREATE TABLE IF NOT EXISTS transactions(
  id INT NOT NULL AUTO_INCREMENT, 
  amount DECIMAL(10,2), 
  description VARCHAR(100), 
  PRIMARY KEY(id)
);

INSERT INTO transactions VALUES (1, 400, 'groceries');
-- Verify with:
SELECT * FROM transactions;
🖥️ 5) Application Tier Setup
bash
🟠 [LOG] Launching App Tier EC2...
🖥️ Instance Name: AppTierInstance
📀 AMI: Amazon Linux 2
⚙️ Type: t2.micro
🔗 IAM Role: demo-ec2-role-ssm (CRITICAL!)
🌐 Network: 
   - VPC: demo-vpc
   - Subnet: app1-subnet
🔄 App Server Configuration
bash
# Connect via Session Manager
sudo su
cd /home/ec2-user

# Install MySQL client
sudo yum install mysql -y

# Test DB connection
mysql -h [RDS_ENDPOINT] -u admin -padmin123

# Node.js Setup
curl -o- https://raw.githubusercontent.com/avizway1/aws_3tier_architecture/main/install.sh | bash
source ~/.bashrc
nvm install 16
nvm use 16
npm install -g pm2

# Deploy App Code
sudo aws s3 cp s3://three-tire-bucket-ssm/application-code/app-tier/ app-tier --recursive
cd app-tier
npm install
pm2 start index.js
pm2 save
pm2 startup

# Verify health check
curl http://localhost:4000/health
⚖️ Load Balancer Configuration
markdown
1️⃣ Target Group:
   - Name: App-Internal-TG
   - Protocol: HTTP:4000
   - Health Check: /health
   - Targets: AppTierInstance

2️⃣ Application LB:
   - Name: App-Internal-LB
   - Scheme: Internal
   - Subnets: app1 & app2
   - Security Group: default
   - Listener: Forward to App-Internal-TG
🌐 Web Tier Setup
bash
🔵 [LOG] Launching Web Tier...
🖥️ Instance Name: WebTierInstance
🌍 Public IP: Enabled
🔗 IAM Role: demo-ec2-role-ssm

# After connection:
sudo su
cd /home/ec2-user

# Node.js Installation (same as app tier)

# Deploy Web Code
sudo aws s3 cp s3://three-tire-bucket-ssm/application-code/web-tier/ web-tier --recursive
cd web-tier
npm install
npm run build

# NGINX Setup
sudo amazon-linux-extras install nginx1 -y
cd /etc/nginx
sudo rm nginx.conf
sudo aws s3 cp s3://three-tire-bucket-ssm/application-code/nginx.conf .
sudo service nginx restart
chmod -R 755 /home/ec2-user
sudo chkconfig nginx on
🎉 Final Verification
bash
🌐 Access via: http://<Web-Tier-Public-IP>
🔍 Check all components:
   - Web Tier (NGINX) → App Tier (PM2) → DB Tier (RDS)
   - All health checks passing
   - Data flowing end-to-end
diff
+===========================================================+
|                🎯 PROJECT DEPLOYMENT SUCCESS! 🎯           |
+===========================================================+
|  Web Tier: Running on Public IP                           |
|  App Tier: Healthy in Private Subnets                     |
|  Database: Accessible only from App Tier                  |
|  Security: Properly isolated 3-tier architecture          |
+-----------------------------------------------------------+
💡 Pro Tips:

Always enable versioning for S3 buckets

Restrict RDS security groups to only app tier later

Consider using Parameter Store for credentials

Set up proper monitoring for all tiers

Implement auto-scaling for web/app tiers

🔧 Troubleshooting:

Check CloudWatch logs for each component

Verify security group rules

Test connectivity between each tier

Confirm IAM roles are properly attached
