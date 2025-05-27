# 🚀 3-Tier AWS Application Project

## 📌 Project Overview:

A **3-Tier Web Application** hosted on AWS using VPC, EC2, RDS, S3, IAM, and Node.js.

---

## 🧰 Requirements:

* **Web Layer** 🕸️
* **App Layer** ⚙️
* **Database Layer** 🗄️

---

## 🏗️ 1) VPC Creation:

* Go to **VPC → VPC and more**
* **VPC Name:** `demo-vpc`
* **CIDR Block:** `192.168.0.0/22`
* **IPv6:** Not Required
* **Tenancy:** Default
* **Public Subnets:** 2
* **Private Subnets:** 4
* **NAT Gateway:** In 1 AZ

➡️ Create VPC

### 🔖 Subnet Naming (Post VPC Creation):

* Private Subnets:

  * `demo-vpc-app1-subnet`
  * `demo-vpc-app2-subnet`
  * `demo-vpc-db1-subnet`
  * `demo-vpc-db2-subnet`
* Public Subnets:

  * `web-1`
  * `web-2`

### 🔐 Security Groups:

Create separate SGs (optional):

* Web Tier SG
* App Tier SG
* DB Tier SG
* LoadBalancer SG

Or use default SG (Allow All Traffic)

---

## 🪣 2) S3 Bucket and IAM Role:

### ✅ S3 Bucket:

* **Name:** Globally unique
* **Permissions:** Default (Public Access)
* **Versioning:** Enabled

➡️ Save your application code in a local folder and upload to S3.

### 👤 IAM Role:

* Go to **IAM → Create Role**
* Select **EC2 Role**
* Attach Policies:

  * `AmazonEC2RoleforSSM`
  * `EC2AdministratorAccess`
* **Role Name:** `demo-ec2-role-ssm`

---

## 🗄️ 3) RDS Database:

### Create Subnet Group:

* **Name:** `db-subnet`
* **VPC:** `demo-vpc`
* **AZs:** `ap-south-1a`, `ap-south-1b`
* **Subnets:** `db1`, `db2`

### Create RDS:

* **Engine:** MySQL
* **Template:** Free Tier
* **DB Name:** `database-1`
* **Username:** `admin`
* **Password:** `admin123`
* **VPC:** `demo-vpc`
* **Subnet Group:** `db-subnet`
* **AZ Preference:** No preference
* **SG:** Default (for now)

➡️ Launch DB Instance

---

## ⚙️ 4) App Tier EC2 Setup:

### Launch EC2:

* **Name:** AppTierInstance
* **AMI:** Amazon Linux 2
* **Instance Type:** t2.micro
* **Key Pair:** Without Key Pair
* **VPC:** `demo-vpc`
* **Subnet:** `app-subnet`
* **Auto-assign IP:** Disabled
* **SG:** Default or custom
* **IAM Role:** `demo-ec2-role-ssm`

➡️ Launch and connect using **Session Manager**

---

## 🔧 5) EC2 Configuration Commands:

```bash
sudo su
```

```bash
whoami
```

```bash
cd ..
```

```bash
cd /home/ec2-user
```

```bash
ping 8.8.8.8
```

```bash
sudo yum install mysql -y
```

### Connect to RDS:

```bash
mysql -h <rds-endpoint> -u admin -p
```

### Inside MySQL:

```sql
show databases;
```

```sql
CREATE DATABASE webappdb;
```

```sql
use webappdb;
```

```sql
CREATE TABLE IF NOT EXISTS transactions(
  id INT NOT NULL AUTO_INCREMENT,
  amount DECIMAL(10,2),
  description VARCHAR(100),
  PRIMARY KEY(id)
);
```

```sql
INSERT INTO transactions (amount, description) VALUES ('400', 'groceries');
```

```sql
SELECT * FROM transactions;
```

```sql
exit;
```

---

## 📝 6) Update DB Config in App Code:

* Replace values in the config file with:

  * `DB_HOST`: `<your-rds-endpoint>`
  * `DB_USER`: `admin`
  * `DB_PASSWORD`: `admin123`
  * `DB_DATABASE`: `webappdb`

➡️ Upload updated code/config back to **S3 bucket** (with versioning enabled)

---

## 📦 7) Install Node.js and App Dependencies:

```bash
curl -o- https://raw.githubusercontent.com/avizway1/aws_3tier_architecture/main/install.sh | bash
```

```bash
source ~/.bashrc
```

```bash
nvm install 16
```

```bash
nvm use 16
```

```bash
npm install -g pm2
```

```bash
cd ~/
```

➡️ Done! You're now ready to fetch app code from S3 and run your app using `pm2`.

---

✨ *Good luck with your deployment!* 🚀
