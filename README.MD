Web Stack Implementation: LAMP Stack on AWS EC2

This repository showcases the step-by-step deployment of a solid, production-ready LAMP (Linux, Apache, MySQL, PHP) web stack hosted on an Amazon Web Services (AWS) EC2 Virtual Machine.

Architecture Overview

The LAMP stack consists of four open-source components arranged in layered tiers to serve dynamic web pages:

Linux (Ubuntu 24.04 LTS): The foundational operating system layer.

Apache HTTP Server: The web server handling HTTP/HTTPS requests.

MySQL Database: The relational database storing system and user state.

PHP: The backend scripting language processing dynamic code.

Step 0: AWS Infrastructure Setup & Connection

1. Instance Configuration

Our server is provisioned with the following specifications in the AWS Console:

Instance ID: i-09f22a985afd7a447

Instance Type: t2.micro

AMI: Ubuntu 24.04 LTS (HVM)

Region: us-east-2 (Ohio)

Public IPv4: 3.143.110.31

Private IPv4: 172.31.16.168

IMDSv2: Required (Session Token Authentication enforced)

2. Network Security Group Configuration

To allow inbound traffic, we configured our Security Group with the following firewall rules:

Port 22 (SSH): Accessible from our administrative local IP for secure terminal operations.

Port 80 (HTTP): Open to Anywhere (0.0.0.0/0) to allow web traffic.

Port 443 (HTTPS): Open to Anywhere (0.0.0.0/0) for secure web traffic.

Security Group Configuration:

3. Establishing SSH Access

From our local terminal, we set secure permissions on our private SSH key (project2.pem) and connected to the instance:

# Secure the private key permissions
chmod 400 project2.pem

# Securely log into the instance
ssh -i "project2.pem" ubuntu@3.143.110.31


Step 1: Installing and Configuring the Apache Web Server

With a live secure connection to the instance, we ran system package updates and installed Apache:

# Update local repository cache and upgrade outdated packages
sudo apt update && sudo apt upgrade -y

# Install Apache Web Server
sudo apt install apache2 -y

# Enable Apache to run automatically on system boot
sudo systemctl enable apache2
sudo systemctl status apache2


Verification

We verified that the web server is reachable by running curl locally and visiting our public IP (http://3.143.110.31) in a browser:

curl -I http://localhost:80


Apache Web Server Verification:

Step 2: Installing and Securing the MySQL Database

To store relational data for our applications, we deployed a MySQL server instance:

# Install MySQL
sudo apt install mysql-server -y

# Verify running status
sudo systemctl status mysql


Hardening the Installation

To protect the database from external threats and administrative vulnerabilities, we logged into the root command-line interface, defined a strong security password protocol, and ran the built-in interactive hardening utility:

# Enter the MySQL command line as root
sudo mysql


Inside the database shell, we updated the authentication profile for the administrative root account:

ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YourStrongAdminPassword123$';
exit;


We then executed the interactive shell wizard to secure the platform:

sudo mysql_secure_installation


We opted to remove anonymous users, disable remote administrative logins, drop test schema scopes, and flush the privileges index table.

Step 3: Installing PHP and Extensions

To run script-driven backend software, we installed PHP along with two companion packages:

libapache2-mod-php to let Apache process PHP files seamlessly.

php-mysql to facilitate queries between PHP and MySQL.

sudo apt install php libapache2-mod-php php-mysql -y

# Confirm system version and compatibility
php -v


Step 4: Configuring an Isolated Apache Virtual Host

To keep web services organized, we configured a custom Virtual Host representing our project folder instead of using the default system path /var/www/html.

1. File System Organization

# Create a dedicated web directory
sudo mkdir -p /var/www/project_web

# Assign folder ownership to the active shell user
sudo chown -R $USER:$USER /var/www/project_web


2. Creating the Configuration File

We created an Apache configuration profile to instruct the server to process incoming requests using our isolated web directory:

sudo nano /etc/apache2/sites-available/project_web.conf


Inside the file, we defined the server parameters:

<VirtualHost *:80>
    ServerName 3.143.110.31
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/project_web
    
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>


3. Activating the Virtual Host

# Enable our custom site configuration
sudo a2ensite project_web

# Disable the default welcome profile
sudo a2dissite 000-default

# Verify syntax integrity
sudo apache2ctl configtest

# Restart Apache to apply the environment changes
sudo systemctl reload apache2


Step 5: Testing Dynamic PHP & Resolving the IMDSv2 Barrier

1. Dynamic PHP Script Test

To verify that our custom Apache Virtual Host executes PHP files, we wrote a test script:

echo "<?php phpinfo(); ?>" > /var/www/project_web/index.php


Visiting http://3.143.110.31/index.php displays our PHP configuration environment dashboard:

Dynamic PHP Processor Verification:

⚠️ Important Security Note: Immediately after verification, delete this file to prevent exposing structural server details to public scanning bots:
rm /var/www/project_web/index.php

2. Resolving the IMDSv2 Restriction

Because our instance enforces IMDSv2, using a plain curl to request internal instance data yields a 401 Unauthorized block. We resolved this by first requesting an authentication token:

# Request a temporary metadata authorization token
TOKEN=$(curl -s -X PUT "[http://169.254.169.254/latest/api/token](http://169.254.169.254/latest/api/token)" -H "X-aws-ec2-metadata-token-ttl-seconds: 360")

# Access metadata securely using the token
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" [http://169.254.169.254/latest/meta-data/public-ipv4](http://169.254.169.254/latest/meta-data/public-ipv4)
