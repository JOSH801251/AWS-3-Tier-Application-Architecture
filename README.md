# AWS-3-Tier-Application-Architecture
 Description: In this project, a 3-tier architecture was designed and implemented on AWS, including the
 presentation, logic, and data tiers. A VPC was configured with public and private subnets, EC2
 instances were set up with an Application Load Balancer for scalability, and Auto Scaling was deployed
 for dynamic resource management. An RDS instance was managed for database needs, security groups
 and IAM roles were configured, and performance was monitored using CloudWatch.
 
 
 Tools: Amazon EC2, ELB Route 53, Auto Scaling Group, Amazon Cloud Watch, Amazon RDS,
 Amazon S3, Amazon VPC, Amazon NAT, IAM Role

Block Diagram:
![image](https://github.com/user-attachments/assets/26d6fa14-3718-40ac-98ea-382d34890494)
Procedure:
1.	Create VPC – 10.0.0.0/16
![image](https://github.com/user-attachments/assets/567aec20-c414-4af7-a87c-f98cdf7a5d33)
2. Subnets:
a.	Public-Subet-1 – 10.0.1.0/24(AZ-a)
b.	Public-Subet-2 – 10.0.2.0/24(AZ-b)

c.	Private-Subet-1 – 10.0.3.0/24(AZ-a)
d.	Private-Subet-2 – 10.0.4.0/24(AZ-b)

e.	DB-Subnet – 1 – 10.0.5.0/24(AZ-a)
f.	DB-Subnet – 2 – 10.0.6.0/24(AZ-b)
![image](https://github.com/user-attachments/assets/da710ff4-f774-462c-89f3-fb04e783f906)
3. Route Tables:
g.	Public – RT – Associate Public Subnet 1 & 2
h.	Private – RT – Associate Private Subnet 1 & 2
![image](https://github.com/user-attachments/assets/932f9b68-2cd0-486c-a263-568b1b2a3480)
4. Create an Internet Gateway - Attach to the VPC we’ve created.
![image](https://github.com/user-attachments/assets/404779f1-90fd-4909-9445-517653176924)
5. Create a NAT Gateway (Use public Subnet 2)
![image](https://github.com/user-attachments/assets/0b6036c4-e69f-4b1f-a465-15d2e4faf565)
6. In the Public – RT -- Add a route to Internet 
i.	0.0.0.0/0 -- IGW (Internet Gateway)
![image](https://github.com/user-attachments/assets/55ece915-5840-4c27-95c6-8c0eb4ccf4de)
7. In the Private – RT -- Add a route to Internet
j.	0.0.0.0/0 -- NAT (NAT Gateway)
![image](https://github.com/user-attachments/assets/7d2f1540-2918-4ca8-9287-ea27e3dfd111)
8. Security Groups:
k.	Load Balancer-SG -- Allow HTTP & HTTPS from 0.0.0.0/0
l.	App-Server-SG -- Allow HTTP from Load Balancer-SG
m.	DB-SG -- Allow MySQL from App-Server-SG  
![image](https://github.com/user-attachments/assets/d9a46148-c34b-4d25-9c6d-44cdca0a1075)
9. Create an IAM Role called EC2SSMAgent -- Add the below policy
   Policy Name: AmazonSSMManagedInstanceCore  
![image](https://github.com/user-attachments/assets/628753b4-f603-4c52-8ecb-991c9172f01e)
10. Create an Application-Server
n.	Launch Instance
o.	Select your VPC and Private-Subnet-1
p.	Select Existing SG -- Application-SG
Under Advance Details -- IAM Instance Profile -- Choose the Role you’ve Created (EC2SSM)
![image](https://github.com/user-attachments/assets/85edd418-c3a3-4036-a5e0-11f20080e654)
11. Install HTTPD, Start & Enable it.
q.	Sudo su
r.	Yum install httpd -y
s.	Systemctl start httpd
t.	Systemctl enable httpd
u.	dnf install mariadb105-server
yum install php php-mysqli
12. Go to RDS & Create Subnet Group -- Select AZ a & b, Select your DB Subnets.
![image](https://github.com/user-attachments/assets/54d80a1a-1b21-422d-9a0d-53c02937021a)
![image](https://github.com/user-attachments/assets/f261cf9d-81c2-4713-a7e5-016a356bb756)
13. Login to Database using the below command
v.	Mysql -h <db-hostname> -u <user name> -p
w.	Show databases;
x.	Create database wordpress;
y.	Show databases;
z.	Exit
![image](https://github.com/user-attachments/assets/eeb44cd6-42bc-498c-9d39-52463050022e)
14. Go to Root Directory and Download WordPress
1. Cd /var/www/html
2. Wget https://wordpress.org/latest.zip
3. Rm latest.zip
4. Mv wordpress/* .
![image](https://github.com/user-attachments/assets/f342acca-22bc-4414-ac1e-4600316d58f8)
15. Rename config-sample file
a. Mv wp-config-sample.php wp-config.php
b. Vi wp-config.php
c. Give Database Parameters
i.	Dbname
ii.	Db username
iii.	Db password
iv.  Db endpoint
![image](https://github.com/user-attachments/assets/3e5d3897-8f90-41dc-be7c-af626ee1853b)
16. Systemctl restart httpd
17. Create an AMI of you Application Server
![image](https://github.com/user-attachments/assets/ae6f8dc1-8e9f-41e6-8ec9-dad58d4aefc3)
18. Create a Target Group -- Select your VPC and Add Application Server
![image](https://github.com/user-attachments/assets/8a702df1-ffd2-46e2-ab24-dbe7126fbb9e)
19.	Create an Application Load Balancer -- Select your VPC and Select public subnet 1 & 2, Select the Load Balancer Security Group.

![image](https://github.com/user-attachments/assets/72a228b7-4db0-4718-be75-2023ab4f5f48)
20.Check the Load Balancer DNS and confirm whether the Application is Working or not

![image](https://github.com/user-attachments/assets/3021d52d-105a-4908-9b2c-2df66a52adcc)

![image](https://github.com/user-attachments/assets/babeddeb-6bb0-4d93-9ff0-0881667a5b2a)

![image](https://github.com/user-attachments/assets/91c5d519-d3ea-44d8-87f9-e31146c7e694)

21. Create Autoscaling Group
![image](https://github.com/user-attachments/assets/e4ede581-bf13-4ae2-9fc7-6e0368983a5e)
![image](https://github.com/user-attachments/assets/2a431477-722b-4266-975c-c5e6dc3b95a6)

22. Create SNS and CloudWatch
![image](https://github.com/user-attachments/assets/bddd3c7f-67de-42c0-a48e-83dc6f79ce4c)
![image](https://github.com/user-attachments/assets/f7db7c4f-420c-487b-a602-9eec424cf362)
