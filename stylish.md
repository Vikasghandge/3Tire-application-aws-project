🎯 3-Tier AWS Architecture Project Guide ✅

✅ Project Requirements
- Web Layer: EC2 + NGINX
- App Layer: Node.js backend on EC2
- Database Layer: MySQL using Amazon RDS

🧱 Step 1: Create VPC
- Name: demo-vpc
- CIDR: 192.168.0.0/22
- No IPv6
- 2 Public Subnets, 4 Private Subnets
- NAT Gateway: Yes in 1 AZ

Rename Subnets:
Private: demo-vpc-app1-subnet, demo-vpc-app2-subnet, demo-vpc-db1-subnet, demo-vpc-db2-subnet
Public: web-1, web-2

(Optional) Create Security Groups:
- Web-Tier SG
- App-Tier SG
- DB-Tier SG
- LoadBalancer SG
OR just use the default SG (allow all traffic)

🪣 Step 2: Create S3 Bucket & IAM Role
- S3 Bucket: enable versioning, upload application code
- IAM Role for EC2:
  - Permissions: AmazonEC2RoleforSSM, EC2AdministratorAccess
  - Role name: demo-ec2-role-ssm

🐬 Step 3: Configure RDS (MySQL)
- Create DB Subnet Group: name it db-subnet, use db1 and db2 subnets
- Create RDS:
  - Engine: MySQL
  - DB Name: database-1
  - Username: admin
  - Password: admin123
  - Subnet group: db-subnet
  - VPC: demo-vpc
  - Security Group: default

⚙️ Step 4: Setup App Tier (Node.js on EC2)
- Launch EC2:
  - Name: App-Tier-Instance
  - AMI: Amazon Linux 2
  - Type: t2.micro
  - Subnet: app1-subnet
  - IAM Role: demo-ec2-role-ssm

Connect via Session Manager:
sudo su
cd /home/ec2-user
ping 8.8.8.8
passwd ec2-user (optional)

Install MySQL Client:
sudo yum install mysql -y

Connect to RDS:
mysql -h <RDS-endpoint> -u admin -padmin123

Create DB:
CREATE DATABASE webappdb;
USE webappdb;
CREATE TABLE IF NOT EXISTS transactions(id INT NOT NULL AUTO_INCREMENT, amount DECIMAL(10,2), description VARCHAR(100), PRIMARY KEY(id));
INSERT INTO transactions (amount, description) VALUES ('400', 'groceries');
SELECT * FROM transactions;

☁️ Step 5: Update App Code and Upload to S3
- Edit app-tier config:
DB_HOST = "<RDS-ENDPOINT>"
DB_USER = "admin"
DB_PASSWORD = "admin123"
DB_DATABASE = "webappdb"
- Upload folder again to S3 (new version will be tracked)

🚀 Step 6: Setup Node.js in App Tier
curl -o- https://raw.githubusercontent.com/avizway1/aws_3tier_architecture/main/install.sh | bash
source ~/.bashrc
nvm install 16
nvm use 16
npm install -g pm2

Download Code:
sudo aws s3 cp s3://<your-bucket-name>/application-code/app-tier/ app-tier --recursive
cd app-tier
npm install
pm2 start index.js
pm2 status
pm2 startup
pm2 save
curl http://localhost:4000/health

🧲 Step 7: Create Internal Load Balancer
- Target Group:
  - Name: App-Internal-TG
  - Type: Instance
  - Port: 4000
  - Health Check: /health
- Load Balancer:
  - Name: App-Internal-LB
  - Type: Internal
  - Subnets: app1 and app2
  - Forward traffic to target group

Update nginx.conf with internal LB DNS and upload it to S3 again

🌍 Step 8: Setup Web Tier (NGINX + React)
- Launch EC2:
  - Name: Web-Tier-Instance
  - Subnet: web-1
  - Public IP: Yes
  - IAM Role: demo-ec2-role-ssm

Connect via Session Manager
cd /home/ec2-user

Install Node:
curl -o- https://raw.githubusercontent.com/avizway1/aws_3tier_architecture/main/install.sh | bash
source ~/.bashrc
nvm install 16
nvm use 16
npm install -g pm2

Download Frontend Code:
sudo aws s3 cp s3://<your-bucket-name>/application-code/web-tier/ web-tier --recursive
cd web-tier
npm install
npm run build

Install & Configure NGINX:
sudo amazon-linux-extras install nginx1 -y
cd /etc/nginx
sudo rm nginx.conf
sudo aws s3 cp s3://<your-bucket-name>/application-code/nginx.conf .
sudo service nginx restart
sudo chkconfig nginx on
chmod -R 755 /home/ec2-user

Access App:
http://<Web-Tier-EC2-Public-IP>

✅ Final Checklist
- VPC created ✅
- S3 + IAM Role ✅
- RDS Setup ✅
- App Layer working ✅
- Load Balancer created ✅
- Web Layer serving app ✅
- End-to-End test passed ✅

🏁 Congratulations! Your 3-Tier AWS Project is Successfully Deployed 🎉
