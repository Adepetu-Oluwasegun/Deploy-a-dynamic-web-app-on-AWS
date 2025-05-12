# Deploy-a-dynamic-web-app-on-AWS
Deploy a dynamic web app on AWS using management console

# Dynamic Website Deployment on AWS – DevOps Project

This project demonstrates the deployment of a dynamic website using AWS cloud infrastructure. It implements best practices in network architecture, scalability, security, and automation using various AWS services and DevOps principles. All relevant architecture diagrams and deployment scripts are available in this repository.

---

## Infrastructure Overview

The following AWS resources and services were used to build and deploy this project:

### Networking & Security
- **VPC with Public and Private Subnets** across **2 Availability Zones** for high availability and fault tolerance.
- **Internet Gateway** to allow internet access to resources in the public subnet.
- **NAT Gateway** placed in the public subnet to provide internet access to instances in the private subnets.
- **Security Groups** to define granular access control rules for EC2 instances and other services.
- **EC2 Instance Connect Endpoint** for secure shell access to instances in both public and private subnets.

### Compute & Scaling
- **EC2 Instances** hosted in **private subnets** for enhanced security.
- **Application Load Balancer (ALB)** deployed in public subnets to route traffic to the backend EC2 instances.
- **Auto Scaling Group (ASG)** to manage EC2 instances based on demand, ensuring high availability and elasticity.
- **SNS (Simple Notification Service)** configured to notify about scaling activities within the Auto Scaling Group.

### Storage & Deployment
- **S3 Bucket** used for storing and retrieving application files and code artifacts.
- **Apache Web Server** and **PHP** installed via a user-data script to serve the dynamic web application.
- **MySQL Server** installed on EC2 instances to support the application backend.

### Domain & Certificate Management
- **Amazon Route 53** used for DNS management and domain registration.
- **AWS Certificate Manager (ACM)** used to provision and manage SSL/TLS certificates for secure communication.

---

## 🛠 Deployment Script

The EC2 user-data script automates the installation and configuration of the LAMP stack and the deployment of the website files:

<details>
<summary>Click to view full script</summary>

```bash
#!/bin/bash
sudo yum update -y
sudo yum install -y httpd
sudo systemctl enable httpd 
sudo systemctl start httpd

sudo dnf install -y \
php php-pdo php-openssl php-mbstring php-exif php-fileinfo \
php-xml php-ctype php-json php-tokenizer php-curl php-cli php-fpm \
php-mysqlnd php-bcmath php-gd php-cgi php-gettext php-intl php-zip

sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm 
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server 
sudo systemctl start mysqld
sudo systemctl enable mysqld

sudo sed -i '/<Directory "\/var\/www\/html">/,/<\/Directory>/ s/AllowOverride None/AllowOverride All/' /etc/httpd/conf/httpd.conf

S3_BUCKET_NAME=aosnote-shopwise-web-files
sudo aws s3 sync s3://"$S3_BUCKET_NAME" /var/www/html
cd /var/www/html
sudo unzip shopwise.zip
sudo cp -R shopwise/. /var/www/html/
sudo rm -rf shopwise shopwise.zip

sudo chmod -R 777 /var/www/html
sudo chmod -R 777 storage/

sudo vi .env
sudo service httpd restart
