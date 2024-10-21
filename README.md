# WordPress Website Hosting on AWS

This README provides an overview of the project where a WordPress website was hosted on AWS using various services and configurations for optimal performance and security.

## Project Overview

The project involved setting up a high-availability WordPress website using multiple AWS services. Below is a summary of the infrastructure and steps taken throughout the deployment process.

### AWS Resources Utilized

1. **Virtual Private Cloud (VPC):**
   - Configured a VPC with both public and private subnets across two different availability zones.

2. **Internet Gateway:**
   - Deployed an Internet Gateway to enable connectivity between VPC instances and the wider Internet.

3. **Security Groups:**
   - Established Security Groups as network firewall mechanisms to control access to instances.

4. **Availability Zones:**
   - Leveraged multiple Availability Zones to enhance system reliability and fault tolerance.

5. **Public and Private Subnets:**
   - Used Public Subnets for infrastructure components like the NAT Gateway and Application Load Balancer.
   - Positioned web servers (EC2 instances) within Private Subnets for enhanced security.

6. **NAT Gateway:**
   - Enabled instances in private subnets to access the Internet via the NAT Gateway.

7. **Web Hosting:**
   - Hosted the website on EC2 Instances.

8. **Application Load Balancer:**
   - Employed an Application Load Balancer and a target group for evenly distributing web traffic to an Auto Scaling Group of EC2 instances.

9. **Auto Scaling Group:**
   - Utilized an Auto Scaling Group to automatically manage EC2 instances, ensuring availability, scalability, fault tolerance, and elasticity.

10. **Shared File System (EFS):**
    - Used Amazon EFS for shared file storage.

11. **Database (RDS):**
    - Implemented Amazon RDS for the database management.

12. **SSL Certificates:**
    - Secured application communications using AWS Certificate Manager.

13. **Monitoring and Notifications:**
    - Configured Simple Notification Service (SNS) to alert about activities within the Auto Scaling Group.

14. **Domain Management:**
    - Registered a domain name and set up a DNS record using Route 53.

## Installation Scripts

### Script for Installing WordPress
```bash
#!/bin/bash
# create to root user
sudo su
# update the software packages on the EC2 instance
sudo yum update -y
# create an html directory
sudo mkdir -p /var/www/html
# environment variable
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com
# mount the EFS to the html directory
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html
# install the apache web server
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd
# install PHP and necessary extensions
sudo dnf install -y php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath php-ctype php-fileinfo php-openssl php-pdo php-tokenizer
# install MySQL 8 community repository and server
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf install -y mysql-community-server
# start MySQL server
sudo systemctl start mysqld
sudo systemctl enable mysqld
# set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html
# download WordPress files
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/
# create and edit the wp-config.php file
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php
sudo vi /var/www/html/wp-config.php
# restart the webserver
sudo service httpd restart
```

### Script for Auto Scaling Group Launch Template
```bash
#!/bin/bash
# update the software packages on the EC2 instance
sudo yum update -y
# install the apache web server
sudo yum
