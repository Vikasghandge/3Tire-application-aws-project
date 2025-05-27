# 🎯 3-Tier AWS Architecture Project Guide 🚀

---

## ✅ Project Requirements

- **Web Layer:** Serves the frontend using EC2 & NGINX.
- **App Layer:** Node.js backend, EC2 instance.
- **Database Layer:** MySQL using Amazon RDS.

---

## 🧱 Step 1: Create VPC 🛰️

- VPC Name: `demo-vpc`
- CIDR Block: `192.168.0.0/22`
- IPv6: ❌ No
- Tenancy: Default
- Public Subnets: 2
- Private Subnets: 4
- NAT Gateway: ✅ In 1 AZ

🔧 Rename Subnets:
Private:

demo-vpc-app1-subnet

demo-vpc-app2-subnet

demo-vpc-db1-subnet

demo-vpc-db2-subnet

Public:

web-1

web-2

yaml
Copy
Edit

🔐 Create Security Groups (Optional):
- Web-Tier SG
- App-Tier SG
- DB-Tier SG
- LoadBalancer SG

📢 OR allow all traffic on Default SG for simplicity.

---

## 🪣 Step 2: Create S3 Bucket & IAM Role

- Create an S3 bucket (globally unique name)
- Enable: ✅ Versioning
- Upload: Application Code files (Keep path reference)

👮 IAM Role:
- Role Type: EC2 Role
- Permissions: 
  - `AmazonEC2RoleforSSM`
  - `EC2AdministratorAccess`
- Role Name: `demo-ec2-role-ssm`

---

## 🐬 Step 3: Configure RDS (Database Layer)

🔹 Create DB Subnet Group:
- Name: `db-subnet`
- VPC: `demo-vpc`
- Subnets: `db1` & `db2`

🔹 Launch RDS Instance:
- Engine: MySQL
- Template: Free Tier
- DB Name: `database-1`
- Username: `admin`
- Password: `admin123`
- VPC: `demo-vpc`
- Subnet Group: `db-subnet`
- AZ: No preference
- SG: Default

---

## ⚙️ Step 4: Configure App Tier (Node.js on EC2)

📌 Launch EC2 Instance:
- Name: `App-Tier-Instance`
- AMI: Amazon Linux 2
- Instance Type: `t2.micro`
- VPC: `demo-vpc`
- Subnet: `app1-subnet`
- IAM Role: `demo-ec2-role-ssm`

🔗 Connect using Session Manager:
```bash
sudo su
whoami
cd /home/ec2-user
ping 8.8.8.8  # Check Internet
🔐 (Optional) Set password:

bash
Copy
Edit
passwd ec2-user
📦 Install MySQL:

bash
Copy
Edit
sudo yum install mysql -y
🔌 Connect to RDS:

bash
Copy
Edit
mysql -h <RDS-endpoint> -u admin -padmin123
📊 Setup Database:

sql
Copy
Edit
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
☁️ Step 5: Upload Updated App Code to S3
Edit config:

js
Copy
Edit
DB_HOST = '<RDS-ENDPOINT>';
DB_USER = 'admin';
DB_PASSWORD = 'admin123';
DB_DATABASE = 'webappdb';
Re-upload app folder to S3 (new version will be created if versioning is ON)

🚀 Step 6: Setup Node.js App in App-Tier EC2
bash
Copy
Edit
curl -o- https://raw.githubusercontent.com/avizway1/aws_3tier_architecture/main/install.sh | bash
source ~/.bashrc
nvm install 16
nvm use 16
npm install -g pm2
📂 Copy files from S3:

bash
Copy
Edit
sudo aws s3 cp s3://<bucket-name>/application-code/app-tier/ app-tier --recursive
cd app-tier
npm install
pm2 start index.js
pm2 status
pm2 startup
pm2 save
curl http://localhost:4000/health
🧲 Step 7: Create Load Balancer & Target Group
🌀 Create Target Group:

Name: App-Internal-TG

Type: Instance

Port: 4000

Protocol: HTTP

Health check path: /health

Register App EC2 instance

🌐 Create Load Balancer:

Name: App-Internal-LB

Type: Internal

VPC: demo-vpc

Subnets: app1 & app2

SG: Default

Listener: Forward to App-Internal-TG

🛠️ Update nginx.conf with internal LB DNS & upload to S3 again

🌍 Step 8: Configure Web Tier (Frontend)
📌 Launch EC2 Instance:

Name: Web-Tier-Instance

Subnet: web-1

Public IP: ✅ Yes

IAM Role: demo-ec2-role-ssm

🔗 Connect using Session Manager:

bash
Copy
Edit
cd /home/ec2-user
⚙️ Install Node & Build App:

bash
Copy
Edit
curl -o- https://raw.githubusercontent.com/avizway1/aws_3tier_architecture/main/install.sh | bash
source ~/.bashrc
nvm install 16
nvm use 16
npm install -g pm2
📂 Copy files:

bash
Copy
Edit
sudo aws s3 cp s3://<your-s3-bucket>/application-code/web-tier/ web-tier --recursive
cd web-tier
npm install
npm run build
🕸️ Install NGINX:

bash
Copy
Edit
sudo amazon-linux-extras install nginx1 -y
cd /etc/nginx
sudo rm nginx.conf
sudo aws s3 cp s3://<your-s3-bucket>/application-code/nginx.conf .
sudo service nginx restart
sudo chkconfig nginx on
chmod -R 755 /home/ec2-user
📡 Access your App via Public IP:

cpp
Copy
Edit
http://<Web-Tier-EC2-Public-IP>
✅ Final Checklist
✅ Task	Status
VPC Created	✅
IAM Role & S3 Bucket Setup	✅
RDS DB Created & Connected	✅
App Layer with Node.js + DB	✅
Load Balancer with Health Checks	✅
Web Tier with NGINX Setup	✅
End-to-End App Health Check	✅

🏁 Congratulations! Your 3-Tier Application on AWS is Fully Deployed! 🎉🎉
yaml
Copy
Edit

---

Let me know if you'd like this as a downloadable **PDF**, **Notion import**, or **README file** for GitHub.





