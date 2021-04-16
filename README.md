# LAMP_EC2Instance_Installation
Install LAMP for use with AWS EC2 Web Server Instance

## Description
Create an ec2 instance with an Ubuntu server and host a Wordpress site

## Table of Contents
1. Create an ec2 Instance
2. Install LAMP
3. Install Wordpress
4. Host Site Using Route 53

## 1. Create AWS ec2 Instance
1.  Using AWS or AWS Educate account, Launch Instance.

2.  Choose Ubuntu Server 20.04 LTS (HVM) SSD Volume Type 64-bit(x86) as the AMI (Amazon Machine Image).

3.  Choose t2.micro (Free-tier Eligible) 1 GiB memory as the instance type.

4.  Keep defaults for Configure Instance Details, Add Storage and Add Tags.

5.  Choose Add Rule in Configure Security Group.

6.  Add: HTTP Port 80 CIDR block 0.0.0.0/0.

7.  Add: HTTPS Port 443 CIDR block 0.0.0.0/0.

8.  Click Review and Launch, then Launch.

9.  To access server, create a new key pair.

10. Download and save the .pem file.

11. Launch Instance, then choose View Instances.

12. Copy the public IP address of the new ec2 instance.

13. Connect to the Linux instance over SSH using PuTTY Key Generator to create .ppk file.

## 2. LAMP Installation
Use the copied IP address to connect to the Linux server as ubuntu@[ec2 ip address here].

Ensure you are in the home directory and perform an update.

```sh
sudo apt update
```

Install Apache.

```sh
sudo apt install apache2
```

Install MySQL and PHP with extra features to be used in Wordpress

```sh
sudo apt install mysql-server mysql-client php7.4 php7.4-dev php7.4-mysql
sudo mysql_secure_installation
sudo apt install php libapache2-mod-php php-mysql
sudo apt install php-curl php-gd php-mbstring php-xml php-xmlrpc php-soap php-intl php-zip
sudo systemctl restart apache2
```

Set Up Database

```sh
sudo mysql
SHOW DATABASES;
CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_unicode_ci;
CREATE USER 'wordpressuser'@'%' IDENTIFIED WITH mysql_native_password BY 'strongpasswordhere';
GRANT ALL ON wordpress.* TO 'wordpressuser'@'%';
FLUSH PRIVILEGES;
exit
```
## 3. Install Wordpress

Download and Extract Wordpress, Create VHost

```sh
wget https://wordpress.org/latest.tar.gz
sudo mkdir /var/www/html/yoursite.example.com
tar -xvzf latest.tar.gz /var/www/html/yoursite.example.com/
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/yoursite.example.com.conf
sudo nano /etc/apache2/sites-available/yoursite.example.com.conf
```

In VHost file, make the following changes:

```sh
<VirtualHost *:80>
    ServerName yoursite.example.com
    ServerAlias www.yoursite.example.com
    ServerAdmin webmaster@localhost
    DocumentRoot /var/www/html/yoursite.example.com
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
<Directory /var/www/html/yoursite.example.com/>
    AllowOverride All
</Directory>
```
Enable the site.

```sh
sudo a2ensite your_domain
sudo a2dissite 000-default
sudo a2enmod rewrite
sudo apache2ctl configtest
sudo systemctl restart apache2
```

Add Permissions

```sh
mkdir wp-content/upgrade
sudo chown -R www-data:www-data /var/www/html/yoursite.example.com
sudo find /var/www/html/yoursite.example.com/ -type d -exec chmod 750 {} \;
sudo find /var/www/html/yoursite.example.com/ -type f -exec chmod 640 {} \;
```
Create a Public DNS Record.

```sh
dig +short myip.opendns.com @resolver1.opendns.com
sudo apt install certbot python3-certbot-apache
sudo certbot --apache
```
Give Wordpress Access to the Database.

To do this, provide the name and password of the database created earlier in MySql. Change the Table Prefix from the default to enhance security of the site. After creating a Wordpress account, click on Install Wordpress.

## 4. Host Site Using Route 53

1. In AWS Console, choose Route 53.

2. In Hosted Zones, locate option where the site will be hosted.

3. Create Record and give the record a name.

4. For the value, use the public IP address of the ec2 instance.

