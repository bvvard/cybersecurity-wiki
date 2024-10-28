# cybersecurity-wiki
I designed this platform to centralize and streamline the management of all IT and cybersecurity knowledge within an organization. This instance of the BookStack WebApp is hosted on a Debian 12 Linux server running the Apache2 HTTP Server, with databases managed using MariaDB, PHP, and MySQL support.

![Status](https://img.shields.io/badge/status-active-brightgreen) ![License](https://img.shields.io/badge/license-MIT-blue) ![Platform](https://img.shields.io/badge/platform-Debian%2012-informational)

## Overview
This project is a centralized IT and cybersecurity knowledge platform built on the open-source BookStack, hosted on Debian 12 Linux with Apache2, MariaDB, and PHP. This setup provides secure, centralized documentation accessible to authorized personnel, with user-friendly management and customizable permissions.

## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Installation Guide](#installation-guide)
- [HTTPS and Security Hardening](#https-and-security-hardening)
- [Automation](#automation)
- [Backup and Restore](#backup-and-restore)
- [Usage](#usage)
- [Screenshots](#screenshots)
- [License](#license)

## Features
- Centralized IT and cybersecurity knowledge management
- Secure hosting on Debian 12
- Web hosting and database managed with Apache2, MariaDB, and PHP
- Customizable user permissions for secure access

## Architecture
![Architecture Diagram](./images/architecture-diagram.png)

## Installation Guide
### Prerequisites
- Debian 12 server (Bookworm)
- Apache2 HTTP Server
- MariaDB Database
- PHP with MySQL support
- [BookStack Official Documentation](https://www.bookstackapp.com/docs/)

### Steps
Follow these steps to install BookStack and configure the server for optimal performance and security.

#### 1. Update System
Start by updating the server to ensure all packages are up-to-date:
```bash
sudo apt update && sudo apt upgrade -y
```
#### 2. Install Apache2

Apache2 will serve the BookStack application on your server.

```bash
sudo apt install apache2 -y
```
Enable Apache2 to start on boot:

```bash
sudo systemctl enable apache2
sudo systemctl start apache2
```
#### 3. Install MariaDB

MariaDB will manage the BookStack database.

```bash
sudo apt install mariadb-server -y
```
Secure the MariaDB installation:

```bash
sudo mysql_secure_installation
```
#### 4. Install PHP and Required Extensions

Install PHP along with required extensions for compatibility with BookStack.

```bash
sudo apt install php libapache2-mod-php php-mysql php-xml php-gd php-mbstring -y
```
#### 5. Download and Configure BookStack

    Clone BookStack from GitHub or download it from BookStack’s releases page.

    Set the application directory:

```bash
sudo mkdir -p /var/www/bookstack
sudo chown -R www-data:www-data /var/www/bookstack
```
Configure Apache2 to serve BookStack: Create a virtual host file for BookStack:

```bash
sudo nano /etc/apache2/sites-available/bookstack.conf
```
Add the following:
```bash
apache

<VirtualHost *:80>
    ServerAdmin admin@example.com
    DocumentRoot /var/www/bookstack/public
    ServerName your-server-ip

    <Directory /var/www/bookstack/>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Enable the BookStack site and rewrite module:

```bash
    sudo a2ensite bookstack
    sudo a2enmod rewrite
    sudo systemctl restart apache2
```

#### 6. Configure the Database for BookStack

    Log in to MariaDB:

```bash
sudo mysql -u root -p
```
Create the database and user:

```sql

CREATE DATABASE bookstack;
CREATE USER 'bookstackuser'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON bookstack.* TO 'bookstackuser'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```
Configure .env file: In the BookStack installation directory, update the .env file with the database settings:

dotenv

    DB_HOST=localhost
    DB_DATABASE=bookstack
    DB_USERNAME=bookstackuser
    DB_PASSWORD=secure_password

Refer to the official BookStack documentation for detailed configuration and troubleshooting.
HTTPS and Security Hardening
#### 1. Enable HTTPS with OpenSSL

    Install OpenSSL:

```bash
sudo apt install openssl
```
Generate an SSL certificate:

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
```
Configure Apache to use SSL: Edit your virtual host configuration:
```bash
apache

<VirtualHost *:443>
    ServerAdmin admin@example.com
    DocumentRoot /var/www/bookstack/public
    ServerName your-server-ip

    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
    SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key

    <Directory /var/www/bookstack/>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
    </Directory>

    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
Enable SSL and necessary modules:

```bash
    sudo a2enmod ssl
    sudo systemctl restart apache2
```
#### 2. Security Hardening

    Firewall Setup: Configure UFW to allow only essential ports (80, 443).

```bash
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw enable
```
File Permissions: Limit access to sensitive directories.

```bash
sudo chmod -R 750 /var/www/bookstack
```
Disable Root Login: Edit /etc/ssh/sshd_config to disable root login.

```bash
PermitRootLogin no
```
Automatic Updates:

```bash
    sudo apt install unattended-upgrades
    sudo dpkg-reconfigure -plow unattended-upgrades
```
Automation
#### 1. Update and Upgrade Automation

Automate system updates and upgrades with a cron job:

```bash
sudo crontab -e
```
Add the following line to update and upgrade every Sunday at 2 AM:

```bash
cron

0 2 * * 7 apt update && apt upgrade -y
```
#### 2. Automate Apache2 and Database Backups

Automate backups by creating a daily cron job for the database and Apache2 configuration.

    Database Backup:

```bash
mysqldump -u bookstackuser -p bookstack > /backups/bookstack_$(date +\%F).sql
```
Apache2 Configuration Backup:

```bash
    tar -czvf /backups/apache2_config_$(date +\%F).tar.gz /etc/apache2
```
Add these commands to a script and schedule it with cron:
```bash
cron

0 3 * * * /usr/local/bin/backup_script.sh
```
Backup and Restore

This backup script automates backup tasks using mysqldump, rsync, md5sum, and bash scripting for file integrity.
Backup Script Instructions

Create the backup script at /usr/local/bin/backup_script.sh:

```bash
#!/bin/bash
# Database Backup Script for BookStack

DATE=$(date +%F)
BACKUP_DIR="/backups"
DB_USER="bookstackuser"
DB_PASS="secure_password"
DB_NAME="bookstack"

# BookStack Database Backup
mysqldump -u $DB_USER -p$DB_PASS $DB_NAME > $BACKUP_DIR/bookstack_$DATE.sql

# Directory Backups
rsync -av /var/www/bookstack/uploads $BACKUP_DIR/uploads
rsync -av /var/www/bookstack/.env $BACKUP_DIR/env_backup
rsync -av /var/www/bookstack $BACKUP_DIR/bookstack_app

# Configuration File Backup
if [ "$(md5sum /etc/apache2/apache2.conf | awk '{print $1}')" != "$(md5sum $BACKUP_DIR/apache2_conf.md5 | awk '{print $1}')" ]; then
    cp /etc/apache2/apache2.conf $BACKUP_DIR/apache2_conf
```
