# FSND-project-5-linux-server-configuration
Project 5 (Deploying to linux server)

</ br>


Configuring a Linux server to host a web app securely using flask application on to AWS Light Sail. Installation of a Linux distribution on a virtual machine and prepare it to host web application(Item Catalog). It includes installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

IP address: 

Accessible SSH port: 2200

Application URL: 

-Login with: ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@13.250.18.177

## Configuration Steps:
### Step 1 : Create new user named grader and give it the permission to sudo
SSH into the server through : ```ssh -i ~/.ssh/udacity_key.rsa unbuntu@13.250.18.177```
Run ``` sudo adduser grader``` to create a new user named grader
Create a new file in the sudoers directory with ```sudo nano /etc/sudoers.d/grader```
Add the following line ```grader ALL=(ALL:ALL) ALL```
</ br>
### Step 2 : Update all currently installed packages
Download package lists with ```sudo apt-get update```
New versions of packages with ```sudo apt-get upgrade```
</ br>
### Step 3 : Change SSH port from 22 to 2200
Run ```sudo nano /etc/ssh/sshd_config```
Change the port from ```22 to 2200```
</ br>
### Step 4 : Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
```sudo ufw allow 2200/tcp```
```sudo ufw allow 80/tcp```
```sudo ufw allow 123/udp```
```sudo ufw enable```
</ br>
### Step 5 : Configure the local timezone to UTC
Run ```sudo dpkg-reconfigure tzdata``` and then choose none of above then UTC
</ br>
### Step 6 : Configure key-based authentication for grader user
generate key-pair with ```ssh-keygen```</ br>
Save keygen file into (/home/user/.ssh/id_rsa).and fill the password . 2 keys will be generated, public key (id_rsa.pub) and identification key(id_rsa).</ br>
Login into grader account using ```sudo login grader``` </ br>
make a directory in grader account : ```mkdir .ssh```</ br>
make a authorized_keys file using ```touch .ssh/authorized_keys```</ br>
from your local machine,copy the contents of public key(id_rsa.pub) paste that contents on authorized_keys of grader account using ```nano authorized_keys``` and save it .</ br>
give the permissions : ```chmod 700 .ssh``` and ```chmod 644 .ssh/authorized_keys.```</ br>
```nano /etc/ssh/sshd_config``` , change PasswordAuthentication to ```no``` .</ br>
```sudo service ssh restart```
</ br>
### Step 7 : Disable ssh login for root user
Run ```sudo nano /etc/ssh/sshd_config```</ br>
Change ```PermitRootLogin without-password``` to ```PermitRootLogin no```</ br>
Restart ```ssh with sudo service ssh restart```</ br>
</ br>
### Step 8 : Install Apache
```sudo apt-get install apache2```
</ br>
### Step 9 : Install mod_wsgi
Run ```sudo apt-get install libapache2-mod-wsgi python-dev```
Enable ```mod_wsgi with sudo a2enmod wsgi```
Start the web server with ```sudo service apache2 start```
</ br>
### Step 10 : Clone the Catalog app from Github
Install git using ```sudo apt-get install git```
```cd /var/www```
```sudo mkdir catalog```
Change owner of the newly created catalog folder ```sudo chown -R grader:grader catalog```
```cd /catalog```
Clone your project from github ```git clone <LINK FOR YOUR REPOSETORY> catalog```
Create a catalog.wsgi file ```nano catalog.wsgi```, then add this inside:
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")
from catalog import app as application
application.secret_key = 'supersecretkey'
```
</ br>
Rename application.py to init.py ```mv application.py __init__.py```</ br>

### Step 11 : Install virtual environment
Install pip with ```sudo apt-get install python-pip```
Install the virtual environment ```sudo pip install virtualenv```</ br>
Create a new virtual environment with ```sudo virtualenv venv```</ br>
Activate the virutal environment ```source venv/bin/activate```</ br>
Change permissions ```sudo chmod -R 777 venv```</ br>

### Step 12 : Install Flask and other dependencies
Install Flask ```pip install Flask```</ br>
Install other project dependencies ```sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils```</ br>

### Step 13 : Update path of client_secrets.json file
```nano __init__.py```
Change client_secrets.json path to ```/var/www/catalog/catalog/client_secrets.json```

### Step 14 : Configure and enable a new virtual host
Run ```sudo nano /etc/apache2/sites-available/catalog.conf```
Paste this code:
```
<VirtualHost *:80>
    ServerName 18.222.207.136
    ServerAlias http://ec2-18-222-207-136.us-east-2.compute.amazonaws.com
    ServerAdmin admin@18.222.207.136
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
    WSGIProcessGroup catalog
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```
Enable the virtual host ```sudo a2ensite catalog```</ br>
### Step 15 : Install and configure PostgreSQL
```sudo apt-get install libpq-dev python-dev```
```sudo apt-get install postgresql postgresql-contrib```
```sudo su - postgres```
```psql```
write these lines line-by-line : 
```CREATE USER catalog WITH PASSWORD 'password';```
```ALTER USER catalog CREATEDB;```
```CREATE DATABASE catalog WITH OWNER catalog;```
```\c catalog```
```REVOKE ALL ON SCHEMA public FROM public;```
```GRANT ALL ON SCHEMA public TO catalog;```
```\q```
```exit```
Change create engine line in your ```__init__.py``` , ```database_setup.py``` and ```fakedata.py``` to: ```engine = create_engine('postgresql://catalog:password@localhost/catalog')```
Run ```sudo python database_setup.py```</ br>

### Step 16 : Restart Apache
```sudo service apache2 restart```

### Step 17 : Visit site at Catalog App()
