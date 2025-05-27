#                                                    3Tire-application-aws-project

3Tire Application project.
requirements:
### web layer:   
### app-layer:
### database layer: 

### 1) Creation of Vpc:
- in vpc select vpc and more.
- vpc-name: demo-vpc.
- cidr: 192.168.0.0/22.
- ipv6: No required.
- Tendency: Default.
- Public Subnets: 2.
- Private Subnet: 4.
- Nat: in 1 AZ.
- then create VPC.


- after vpc got created sucessfully.
- change the name of subnets of understanding
- like we have 6 subnets 2 public and 4 private.
- add name in first private subent like //  demo-vpc-app1-subnet, demo-vpc-app2-subnet
- another two private subnet: // demo-vpc-db1-subnet, demo-vpc-db2-subnet 
- and to public subnets // give name them web-1 and web2

- if you wanto to create proper security groups u can create separat security geoups like web tire, app tire, db tire, loadbalancer.
  of use default security make sure enable all-traffic-allow.


### 2) Creation of S3 Bucket and IAM Role.
- Create S3 Bucket With Default Permissions and Public Access.
- name: add any globally unique name:
- versioning: enabled # it should be enabled make sure
- now then first the you need to save application code into loal file manager. save it like repo and keep know path of file.

### 3) Now Need to IAM Role.
- go to AWS Console and Select IAM Create Role which selecting Role Select EC2-Role and then Add permissions 
- in Permsiions search AmazoneEC2RoleforSSM select optional select EC2-Adminstratir 
- Name: demo-ec2-role-ssm

### 4) congiure RDS DATABASE:-
- In RDS we need to configure Subnet Group First.
- Create Subnet Group: db-subnet
- VPC: your created vpc like demo-vpc select it.
- AZS: ap-south-1a, ap-south-1b
- Subnets: select db1 and db2 subnets.

- Go Back and Create RDS Database:
- Standard:
- Engine: Mysql
- Template: Free 
- DB Name: database-1
- Username: admin       ## set according to you
- Password:  admin123  ## set according to you

- Virtual Private Cloud(vpc)
- Select vpc which you have created.  demo-vpc
- Subnet Group: which you have created  example: db-subnet

- SG: Select default for temp 

- avz: no preference

- every will remain same and create database.

### 5) Application Tire Setup.
- Launch EC2 Instance.
- Name: AppTireinstance
- Image: Amazone 
- AMI: Amazone Linux 2 AMI (HVM)
- InstanceType: T2-Micro or small.
- KayPair: Proceed Without Key pair
- Network Setting:  select vpc which you have created 
- subent: app
- Ip Address: no assign 
- security group: Default 


Go to Advance Setting.
IAM Role Profile  --> add Iam which u have Created  example: demo-ec2-role-ssm   ## dont forget to add.

- Then Launch Instance.
- Connect to the Instance with Session Manager.
- login into server via session manager.
```
sudo su
```
```
whoami
```
```
cd ..
```
```
cd /home/ec2-user
```
Check the Internet is Available or Not.
```
ping 8.8.8.8
```
optional thre any password issue use passwd ec2-user and create password fo user.

Install Mysql to Access database.
```
sudo yum install mysql -y 

```

login into database follow below give command

```
mysql -h <rds //database endpoint// > -u < username > -p < password >

```
example.
```
mysql -h database-1.cvu4cuyqyi5s.ap-south-1.rds.amazonaws.com -u admin -padmin123
```

To Check Available Databases;
```
show databases;  
```
create database
```
CREATE DATABASE webappdb;

```

use database.
```
use webappdb;
```

add table into your database.
```
CREATE TABLE IF NOT EXISTS transactions(
  id INT NOT NULL AUTO_INCREMENT, 
  amount DECIMAL(10,2), 
  description VARCHAR(100), 
  PRIMARY KEY(id)
);

```

add value into your table.
```
INSERT INTO transactions (amount, description) VALUES ('400', 'groceries');

```

If You Want See Data Inside the Table use Command.
```
SELECT * FROM transactions;
```

then exit;
```
exit;
```

- now go to the that file folder in your local file file manager where you have cloned 
- folder application code -- app-tire--DbConfig
- add DB HOST: 'rds database endpoint'
- DB_USER: 'admin'
- DB : 'admin123'
- DB_DATABASE: 'webappdb'   ## which means only replace values

- go s3 bucket do same path application - apptire- dbConfig replace updated file with this make sure bucket versioning should be enable.
or uplode that folder again in same s3 bucet it will create new version like in your s3 bucket there is folder available name application-code
- select add folder add new updated folder into your s3 bucket.

Go back on Your Server and Install Node.js By Below Given Command.

```
curl -o- https://raw.githubusercontent.com/avizway1/aws_3tier_architecture/main/install.sh | bash
```
update bachrc
```
source ~/.bashrc
```
```
nvm install 16    # node version manager
```
```
nvm use 16
```
```
npm install -g pm2  (you will see found 0 vaulnarbalites )
```
```
cd ..
```
```
cd ..
```
```
cd ~/
```

copy app tire file form s3  to your machine by using given command.
```
sudo aws s3 cp s3://<bucket name>/application-code/app-tire/ app-tire --recursive

```
Example.
```
 sudo aws s3 cp s3://three-tire-bucket-ssm/application-code/app-tier/ app-tier --recursive

```
```
ls
```
- this will copy folder app-tire 
```
cd app-tire

```
```
npm install   ## it will create index.js file
```
```
pm2 start index.js    ## this will start index.js
```

```
pm2 status   # will show status
```

```
pm2 startup
```
``` 
pm2 save 
```

```
curl http://localhost:4000/health  # output  this is health check.
```

- crate loadbalancer
- before that create target groups  
- Target Type: Instance
- Target Group Name:   App-Internal-TG
- protocol: http  Port no: 4000
- vpc -- creatred by you
- Health check  http:   path /health
- next select AppTireInstance

  - include as pending below  then create target group.
 
    
- now create load balancer  appication loadbalaner
- Name: App-Internal-LB
- choose: Internal
- Network: created vpc 
- avz : select both 1a   secect app subnet and 1b  select app2 subnet

- sg -- add default

- Listerener and routing
- target group App-Internal-TG

- now go the your local system and find ngnix.conf

- there scroll down it will show dns of internal loadbalaner which is old copy dns of your current load balancer and paste it there.
- go the the s3 bucket and replace  ngnix.conf which new one.

###  web tire 
 - Lanuch ec2 instance:  `Web-Tire-Instance`
 - image: `amazone`
- ami: `Amazone Linux 2 AMI`
- InstanceType: `t2.micro`
- key pair: `proceed without key pair.`
- Network: `Selecte Created Vpc` 
- Subnet: Public Subnet 1a
- Ip: yes
- sg: default

- go to the advance setting
- select existing IAM role and launch instance.

- connect to your web ec2 instance by using session manager.
```
sudo ec2-user
```
```
cd /home/ec2-user

```

### install Node.js

```
curl -o- https://raw.githubusercontent.com/avizway1/aws_3tier_architecture/main/install.sh | bash
```
```
source ~/.bashrc
```
```
nvm install 16    # node version manager
```
```
nvm use 16
```
```
npm install -g pm2  (you will see found 0 vaulnarbalites )
```
```
sudo aws s3 cp s3://<your s3 bucket name> /application-code/app-tire/ app-tire --recursive
```
```
sudo aws s3 cp s3://three-tire-bucket-ssm/application-code/web-tier/ web-tier --recursive
```
```
cd web-tire
```
```
npm install
```

```
npm run build
```


#install nginx
```
sudo amazon-linux-extras install nginxl -y
```
```
cd /etc/nginx
```  
# find nginx.conf]
```
```
sudo rm nginx.conf
```
````
sudo aws s3 cp s3://<bucket name>/application-code/nginx.conf .
```
eample.
```
sudo aws s3 cp s3://three-tire-bucket-ssm/application-code/nginx.conf .
```

```
sudo service nginx restart   # install nginx
```
```
chmod -R 755 /home/ec2-user
```
```
sudo chkconfig nginx on
```

``` 

now copy the public IP address of web-instance  = http://<ip address>
```






























