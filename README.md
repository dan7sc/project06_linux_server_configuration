# Linux Server Configuration Project


## Project Overview

For this project, we will take a baseline installation of a bare-bones Linux server and prepare it to host web applications. We will secure the server from a number of attack vectors, install and configure a database server, and deploy web applications onto it.

## PreRequisites

  * [Amazon Lightsail](https://lightsail.aws.amazon.com/)
  * [Ubuntu](https://www.ubuntu.com/)
  * [Apache HTTP Server](http://httpd.apache.org/)
  * [PostgreSQL](https://www.postgresql.org/)
  * [Git](https://git-scm.com/)
  * [Python](https://www.python.org/)
  * [Fail2ban](https://www.fail2ban.org/)

## Setup Project

### 1. Get started on Lightsail

* Create a publicly accessible Ubuntu Linux server instance using Amazon Lightsail.
* Configure the Lightsail firewall.
* The public IP address of the instance is 3.80.185.159.
* SSH port is 2200
* URL to hosted web app: www.3.80.185.159.xip.io/ (google login works).

                           http://ec2-3-80-185-159.compute-1.amazonaws.com/ (google login doesn't work).


### 2. Get started on Server

* Download SSH key pairs `LightsailDefaultKey-us-east-1.pem` in the local machine, open a terminal and enter

  `sudo cp LightsailDefaultKey-us-east-1.pem ~/.ssh/LightsailDefaultKey.pem`

  `sudo chmod 644 ~/.ssh/LightsailDefaultKey.pem`
* Login in the remote server through ssh

  `ssh ubuntu@3.80.185.159 -i ~/.ssh/LightsailDefaultKey.pem`


### 3. Configure SSH

* Change SSH port to 2200 and disable remote login for root

  `sudo vi /etc/ssh/sshd_config`

  Change `Port 22` to `Port 2200`

  Check if `PasswordAuthentication` is set as `no`

  Change `PermitRootLogin prohibit-password` to `PermitRootLogin no`
* Restart SSH service

  `sudo service ssh restart`


### 4. Configure firewall

* Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

  `sudo ufw status`

  `sudo ufw default deny incoming`

  `sudo ufw default allow outgoing`

  `sudo ufw allow 2200/tcp`

  `sudo ufw allow 80/tcp`

  `sudo ufw allow 123/udp`

  `sudo ufw enable`


### 5. Add new user

* In terminal enter

  `sudo adduser grader`
* Set a password:

  `sudo passwd grader`
* Give `grader` the permission to `sudo`

  `sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader`

  `su -c 'nano /etc/sudoers.d/grader'`

  Change `ubuntu ALL=(ALL) NOPASSWD:ALL` to `grader ALL=(ALL) NOPASSWD:ALL`


### 6. Generate public/private rsa key pair

* Logout server and in the local machine create an SSH key pair for grader

  `ssh-keygen`

  (Save the key as `id_rsa_key`)
* Login in the server
* Create hidden directory `.ssh`

  `mkdir /home/grader/.ssh`
* Create the file `authorized_keys`

  `touch /home/grader/.ssh/authorized_keys`
* Change permissions:

  `chmod 700 .ssh`

  `chmod 644 .ssh/authorized_keys`
* Copy `id_rsa_key.pub` content in the local machine to `authorized_keys` in the server
* Restart SSH

  `sudo service ssh restart`
* Logout and login entering

  `ssh -p 2200 -i ~/.ssh/id_rsa_key grader@3.80.185.159`


### 7. Configure the local timezone to UTC

* In the server enter

  `sudo timedatectl set-timezone UTC`


### 8. Install and configure Apache to serve a Python mod_wsgi application

* Install apache

  `sudo apt-get install apache2`
* Install WSGI module to python 3

  `sudo apt-get install libapache2-mod-wsgi-py3`
* Configure Apache to handle requests using the WSGI module

  `sudo vi /etc/apache2/sites-enabled/000-default.conf`

  Add `WSGIScriptAlias / /var/www/html/myapp.wsgi` just before the closing tag `</VirtualHost>`
* Create wsgi file

  `sudo vi /var/www/html/myapp.wsgi`
* Create Apache virtual host

  ` sudo vi /etc/apache2/sites-available/myapp.conf`
* Enable virtual host

  `sudo a2ensite myapp.conf`
* Restart apache

  `sudo apache2ctl restart`


### 9. Install and configure PostgreSQL

* Install PostgreSQL

  `sudo apt-get install postgresql`
* Set passwd to postgres user

  `sudo -u postgres psql`

  Into PostgreSQL shell

      \password postgres
* Get into PostgreSQL shell with postgres user

  `psql -h localhost -U postgres`

  Create a user named catalog and set a password

      CREATE USER catalog with PASSWORD 'catalog123';

  Create a database named catalogdb

      CREATE DATABASE catalogdb;

  Set permissions

      GRANT ALL PRIVILEGES ON DATABASE catalogdb TO catalog;

  Edit the file `/etc/postgresql/9.5/main/pg_hba.conf` changing

      host all all 127.0.0.1/32 md5 to host catalogdb catalog 127.0.0.1/32 md5

      host all all ::1/128 md5 to host catalogdb catalog ::1/128 md5
* Restart PostegreSQL
  
  `sudo systemctl status postgresql`


### 10. Deploy the Item Catalog project

* Install git

  `sudo apt-get install git`
* Clone the Item Catalog project

  `git clone https://github.com/dan7sc/project04_build_an_item_catalog.git`
* Install and upgrade pip for python 3

  `sudo apt-get install python3-pip`

  `sudo pip3 install --upgrade pip`
* Install modules for the project

  `sudo pip3 install flask oauth2client passlib flask-httpauth`

  `sudo pip3 install sqlalchemy flask-sqlalchemy psycopg2-binary requests`

  `sudo pip3 install flask-seasurf`


### 11. Block access after unsuccessful login attempts

* Install fail2ban

  `sudo apt-get install fail2ban`
* Generate the local files for configuration

  `sudo cp /etc/fail2ban/fail2ban.conf /etc/fail2ban/fail2ban.local`

  `sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
* Configure fail2ban file sto block ssh access after unsuccessful login attempts 

  `sudo vi /etc/fail2ban/jail.local`

  ```
  [sshd]
  enabled = true
  port    = 2200
  logencoding = utf-8
  logpath = %(sshd_log)s
  bantime = 600
  findtime= 600
  maxretry = 3
  ```
* Restart fail2ban

  `sudo systemctl restart fail2ban.service`

* Restart fail2ban

  `sudo systemctl restart fail2ban.service`


### 12. Final

* Upgrade the system

  `sudo apt-get update`

  `sudo apt-get upgrade`
* Restart Apache

  `sudo apache2ctl restart`
* See the app running on

  www.3.80.185.159.xip.io/
