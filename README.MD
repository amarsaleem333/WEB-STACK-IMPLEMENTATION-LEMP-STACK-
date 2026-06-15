Part 1: Step-by-Step Project Execution & Screenshot Checklist

As you run the commands on your EC2 instance (3.143.110.31), stop at the designated points below to capture your screenshots.

Step 0: AWS Infrastructure Setup

Launch EC2: Launch your t2.micro Ubuntu 24.04 instance.

Configure Security Group: Ensure Port 22 (SSH), Port 80 (HTTP), and Port 443 (HTTPS) are open to 0.0.0.0/0.

📸 https://github.com/amarsaleem333/WEB-STACK-IMPLEMENTATION-LEMP-STACK-/commit/b933d4bccfdbaef8e3f532a54f18de8799334152#diff-098d87542d1cefd1cea52fd5abaacee741bb3c5ec38c8d04a42d7b1ea003ec47 (Security Group Rules): * In your AWS Console, navigate to your EC2 Instance -> Security tab -> Click on your Security Group.

Take a screenshot showing the Inbound Rules table (Ports 22, 80, 443 active).

Save this image on your computer as: security-rules.png

Connect via SSH: Open your terminal (Git Bash, Mac Terminal, or Command Prompt) and run:

chmod 400 my-key.pem
ssh -i "my-key.pem" ubuntu@3.143.110.31



Step 1: Install Apache

Run the update and installation commands:

sudo apt update && sudo apt upgrade -y
sudo apt install apache2 -y



Open your web browser and go to http://3.143.110.31.

📸 https://github.com/amarsaleem333/WEB-STACK-IMPLEMENTATION-LEMP-STACK-/commit/b933d4bccfdbaef8e3f532a54f18de8799334152#diff-347d32017b411745cd6331b5cd25ebee13d1db1e2f5a78509bce49653984805b (Apache Default Page):

Take a screenshot of the "Apache2 Ubuntu Default Page" in your browser showing your server's IP address (3.143.110.31) in the URL bar.

Save this image on your computer as: apache-default.png

Step 2: Install and Secure MySQL

Install the database server:

sudo apt install mysql-server -y



Secure your database by running:

sudo mysql



Update the root password inside the SQL command line (replace YourStrongPassword123$ with a password of your choice):

ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'YourStrongPassword123$';
exit;



Run the security hardening script:

sudo mysql_secure_installation



Follow the prompts to remove anonymous users, disable remote root logins, and reload privileges.

Step 3: Install PHP

Install PHP and its Apache/MySQL modules:

sudo apt install php libapache2-mod-php php-mysql -y



Step 4: Configure the Apache Virtual Host

Create your custom web directory and assign permissions:

sudo mkdir -p /var/www/project_web
sudo chown -R $USER:$USER /var/www/project_web



Create your configuration file:

sudo nano /etc/apache2/sites-available/project_web.conf



Paste the virtual host configuration (Press Ctrl+O, Enter, and then Ctrl+X to save and exit in Nano):

<VirtualHost *:80>
    ServerName 3.143.110.31
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/project_web

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>



Enable the virtual host, disable the default layout, test configuration, and reload Apache:

sudo a2ensite project_web
sudo a2dissite 000-default
sudo apache2ctl configtest  # Should return: Syntax OK
sudo systemctl reload apache2



Step 5: Test PHP Execution

Create a dynamic test file in your new web root directory:

echo "<?php phpinfo(); ?>" > /var/www/project_web/index.php



In your browser, navigate to: http://3.143.110.31/index.php.

📸 https://github.com/amarsaleem333/WEB-STACK-IMPLEMENTATION-LEMP-STACK-/commit/b933d4bccfdbaef8e3f532a54f18de8799334152#diff-bdb784c5f31634f59bbb698030b6bbf709bfd3b6e47d24557ceb7a7d889e2d99 (PHP Info Page):

Take a screenshot of the purple PHP info diagnostic page showing your public IP address in the URL bar.

Save this image on your computer as: php-info.png

Remove the test file for safety:

rm /var/www/project_web/index.php
