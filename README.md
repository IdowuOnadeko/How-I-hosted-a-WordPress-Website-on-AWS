# WordPress on AWS Deployment

## Project Overview

This project demonstrates how to host a WordPress website on AWS using a range of AWS services to ensure scalability, fault tolerance, and high availability. The deployment architecture leverages best practices in cloud infrastructure and security, using resources such as VPC, EC2, RDS, EFS, ALB, Auto Scaling, and more.

## Architecture Diagram

Refer to the architecture diagram uploaded in the [GitHub repository](link-to-repo) (My-Illustration-of-WordPress-Website-on-AWS) for a visual representation of the infrastructure setup.

## AWS Resources Utilized

1. **VPC Configuration:**

   - Created a Virtual Private Cloud (VPC) with both public and private subnets across two different availability zones for high availability and fault tolerance.

2. **Internet Gateway:**

   - Deployed an Internet Gateway to facilitate internet connectivity for instances within the VPC.

3. **Security Groups:**

   - Established Security Groups to act as a firewall, managing inbound and outbound traffic to the EC2 instances and other AWS services.

4. **Availability Zones:**

   - Utilized two Availability Zones to enhance reliability and distribute resources.

5. **Subnets:**

   - Public Subnets: Hosted infrastructure components like the NAT Gateway and Application Load Balancer.
   - Private Subnets: Positioned web servers (EC2 instances) and databases for enhanced security.

6. **EC2 Instance Connect Endpoint:**

   - Implemented secure connections to assets within both public and private subnets.

7. **NAT Gateway:**

   - Enabled instances in private subnets to access the Internet securely.

8. **EC2 Instances:**

   - Hosted the WordPress website on EC2 instances located in private subnets.

9. **Application Load Balancer (ALB):**

   - Used to distribute incoming web traffic evenly across an Auto Scaling Group of EC2 instances.

10. **Auto Scaling Group:**

    - Automatically managed EC2 instances to ensure availability, scalability, and fault tolerance.

11. **GitHub:**

    - Stored web files for version control and collaboration.

12. **AWS Certificate Manager:**

    - Secured application communications with SSL/TLS certificates.

13. **Simple Notification Service (SNS):**

    - Configured to send alerts about activities within the Auto Scaling Group.

14. **Route 53:**

    - Registered domain names and set up DNS records for the website.

15. **Elastic File System (EFS):**

    - Used for shared file storage across multiple EC2 instances.

16. **Relational Database Service (RDS):**

    - Hosted the MySQL database for WordPress.

---

## Deployment Instructions

### Prerequisites

- AWS CLI configured with appropriate IAM permissions.
- Git installed on your local machine.

### Clone the Repository

```bash
git clone <link-to-repo>
cd <repository-directory>
```

### Infrastructure Deployment

1. **VPC Setup:**

   - Follow the provided CloudFormation/Terraform scripts to set up the VPC, subnets, and networking components.

2. **Deploy EC2 Instances:**

   - Use the provided scripts to launch EC2 instances within private subnets.

### WordPress Installation on EC2

Run the following script to install WordPress on your EC2 instance:

```bash
# Switch to root user
sudo su

# Update software packages
sudo yum update -y

# Create HTML directory
sudo mkdir -p /var/www/html

# Set environment variable for EFS DNS
EFS_DNS_NAME=fs-064e9505819af10a4.efs.us-east-1.amazonaws.com

# Mount EFS to the HTML directory
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2,noresvport "$EFS_DNS_NAME":/ /var/www/html

# Install Apache web server
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP and necessary extensions
sudo dnf install -y php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath php-ctype php-fileinfo php-openssl php-pdo php-tokenizer

# Install MySQL server
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Set permissions
sudo usermod -a -G apache ec2-user
sudo chown -R ec2-user:apache /var/www
sudo chmod 2775 /var/www && find /var/www -type d -exec sudo chmod 2775 {} \;
sudo find /var/www -type f -exec sudo chmod 0664 {} \;
chown apache:apache -R /var/www/html

# Download and configure WordPress
wget https://wordpress.org/latest.tar.gz
tar -xzf latest.tar.gz
sudo cp -r wordpress/* /var/www/html/
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php

# Edit wp-config.php with RDS details
sudo vi /var/www/html/wp-config.php

# Restart Apache
sudo service httpd restart
```

### Auto Scaling Group Launch Template Script

```bash
#!/bin/bash

# Update packages
sudo yum update -y

# Install Apache
sudo yum install -y httpd
sudo systemctl enable httpd
sudo systemctl start httpd

# Install PHP and necessary extensions
sudo dnf install -y php php-cli php-cgi php-curl php-mbstring php-gd php-mysqlnd php-gettext php-json php-xml php-fpm php-intl php-zip php-bcmath php-ctype php-fileinfo php-openssl php-pdo php-tokenizer

# Install MySQL
sudo wget https://dev.mysql.com/get/mysql80-community-release-el9-1.noarch.rpm
sudo dnf install -y mysql80-community-release-el9-1.noarch.rpm
sudo rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf repolist enabled | grep "mysql.*-community.*"
sudo dnf install -y mysql-community-server
sudo systemctl start mysqld
sudo systemctl enable mysqld

# Mount EFS
EFS_DNS_NAME=fs-02d3268559aa2a318.efs.us-east-1.amazonaws.com
echo "$EFS_DNS_NAME:/ /var/www/html nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" >> /etc/fstab
mount -a

# Set permissions
chown apache:apache -R /var/www/html

# Restart Apache
sudo service httpd restart
```

---

## Domain Configuration

1. Register your domain via **Route 53**.
2. Set up DNS records to point to the Application Load Balancer.
3. Secure your domain using **AWS Certificate Manager**.

---

## Notifications

Set up **SNS** to monitor and send notifications about the health and scaling activities of your Auto Scaling Group.

---

## Conclusion

This project showcases a robust and scalable deployment of a WordPress website on AWS, leveraging various services to ensure performance, security, and high availability.

For more details and scripts, refer to the [GitHub repository](link-to-repo).

(My-Illustration-of-WordPress-Website-on-AWS)
