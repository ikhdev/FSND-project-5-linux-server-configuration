# FSND-project-5-linux-server-configuration
Project 5 (Deploying to linux server)

<br />


Configuring a Linux server to host a web app securely using flask application on to AWS Light Sail. Installation of a Linux distribution on a virtual machine and prepare it to host web application(Item Catalog). It includes installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

IP address: 18.219.96.82

Accessible SSH port: 2200

Application URL: http://ec2-18-219-96-82.us-east-2.compute.amazonaws.com

Login with: ssh -i ~/.ssh/Udacity.pem -p 2200 grader@18.219.96.82

## Configuration Steps:
### Step 1 : Create new user named grader and give it the permission to sudo
- SSH into the server through : ```ssh -i ~/.ssh/udacity_key.rsa unbuntu@13.250.18.177```<br />
- Run ``` sudo adduser grader``` to create a new user named grader<br />
- Create a new file in the sudoers directory with ```sudo nano /etc/sudoers.d/grader```<br />
- Add the following line ```grader ALL=(ALL:ALL) ALL```<br />

### Step 2 : Update all currently installed packages
Download package lists with ```sudo apt-get update```<br />
New versions of packages with ```sudo apt-get upgrade```<br />

### Step 3 : Change SSH port from 22 to 2200
Run ```sudo nano /etc/ssh/sshd_config```<br />
Change the port from ```22 to 2200```<br />

### Step 4 : Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
```sudo ufw allow 2200/tcp```<br />
```sudo ufw allow 80/tcp```<br />
```sudo ufw allow 123/udp```<br />
```sudo ufw allow ssh```<br />
```sudo ufw allow www ```<br />
```sudo ufw allow ftp```<br />
```sudo ufw enable```<br />

### Extra :

BACK TO DEFAULT SETTINGS (UFW) : ```sudo ufw reset``` <br />

DISABLE UFW : ```sudo ufw disable```<br />

```sudo nano /etc/ssh/sshd_config```<br />

add these lines : <br /> <br />
```
ClientAliveInterval 25
ClientAliveCountMax 0
```
.<br />

```sudo service ssh restart``` <br />

### Step 5 : Configure the local timezone to UTC
Run ```sudo dpkg-reconfigure tzdata``` and then choose none of above then UTC<br />

### Step 6 : Configure key-based authentication for grader user
generate key-pair with ```ssh-keygen```<br />
Save keygen file into (/home/user/.ssh/id_rsa).and fill the password . 2 keys will be generated, public key (id_rsa.pub) and identification key(id_rsa).<br />
Login into grader account using ```sudo login grader``` <br />
make a directory in grader account : ```mkdir .ssh```<br />
make a authorized_keys file using ```touch .ssh/authorized_keys```<br />
from your local machine,copy the contents of public key(id_rsa.pub) paste that contents on authorized_keys of grader account using ```nano authorized_keys``` and save it .<br />
give the permissions : ```chmod 700 .ssh``` and ```chmod 644 .ssh/authorized_keys.```<br />
```nano /etc/ssh/sshd_config``` , change PasswordAuthentication to ```no``` <br />
```sudo service ssh restart```<br />

### Step 7 : Disable ssh login for root user
Run ```sudo nano /etc/ssh/sshd_config```<br />
Change ```PermitRootLogin without-password``` to ```PermitRootLogin no```<br />
Restart ssh with ```sudo service ssh restart```<br />

### Step 8 : Install Apache
```sudo apt-get install apache2```<br />

### Step 9 : Install mod_wsgi
Run ```sudo apt-get install libapache2-mod-wsgi python-dev```<br />
Enable mod_wsgi with ```sudo a2enmod wsgi```<br />
Start the web server with ```sudo service apache2 start```<br />

### Step 10 : Clone the Catalog app from Github
Install git using ```sudo apt-get install git```<br />
```cd /var/www```<br />
```sudo mkdir catalog```<br />
Change owner of the newly created catalog folder ```sudo chown -R grader:grader catalog```<br />
```cd /catalog```<br />
Clone your project from github ```git clone <LINK FOR PROJECT 4 REPOSETORY> catalog```<br />
Create a catalog.wsgi file ```nano catalog.wsgi```, then add this inside:<br />
```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")
from catalog import app as application
application.secret_key = 'super_secret_key'
WTF_CSRF_ENABLED = True
```
.<br />

Rename application.py to init.py ```mv application.py __init__.py```<br />

### Step 11 : Install virtual environment
Install pip with ```sudo apt-get install python-pip```<br />
Install the virtual environment ```sudo pip install virtualenv```<br />
Create a new virtual environment with ```sudo virtualenv venv```<br />
Activate the virutal environment ```source venv/bin/activate```<br />
Change permissions ```sudo chmod -R 777 venv```<br />

### Step 12 : Install Flask and other dependencies
Install Flask ```pip install Flask```<br />
Install other project dependencies ```sudo pip2 install httplib2 oauth2client sqlalchemy flask-sqlalchemy psycopg2-binary bleach requests sqlalchemy_utils```<br />


### Step 13 : Update path of client_secrets.json file
```nano __init__.py```<br />
Change client_secrets.json path to ```/var/www/catalog/catalog/client_secrets.json```<br />

### Step 14 : Configure and enable a new virtual host

run ```sudo nano /etc/apache2/sites-enabled/000-default.conf``` <br />


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

.<br />

### Step 15 : Install and configure PostgreSQL
```sudo apt-get install libpq-dev python-dev```<br />
```sudo apt-get install postgresql postgresql-contrib```<br />
```sudo su - postgres```<br />
```psql```<br />
write these lines line-by-line : <br />
```CREATE USER catalog WITH PASSWORD 'password';```<br />
```ALTER USER catalog CREATEDB;```<br />
```CREATE DATABASE catalog WITH OWNER catalog;```<br />
```\c catalog```<br />
```REVOKE ALL ON SCHEMA public FROM public;```<br />
```GRANT ALL ON SCHEMA public TO catalog;```<br />
```\q```<br />
```exit```<br />
Change create engine line in your ```__init__.py``` , ```database_setup.py``` and ```fakedata.py``` to: ```engine = create_engine('postgresql://catalog:password@localhost/catalog')```<br />
Run ```sudo python database_setup.py```<br />

### Step 16 : Restart Apache
```sudo service apache2 restart```<br />


### Step 17 : Visit site at [Catalog App](http://ec2-18-219-96-82.us-east-2.compute.amazonaws.com) <br />

> Oauth not work because the domain not https

<br />
<br />
<br />

## References :

[Udacity FSND](https://www.udacity.com/)<br />
[Deploying a Python](https://www.phusionpassenger.com/library/walkthroughs/deploy/python/)<br />
[stackoverflow](https://stackoverflow.com)<br />
[ask Ubuntu](https://askubuntu.com/)<br />
[Amazon EC2 Linux Instances](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)<br />
[mod_wsgi (Apache)](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/)<br />
[mod_wsgi](https://modwsgi.readthedocs.io/en/develop/)<br />
[project 4 Item Catalog](https://github.com/ikhdev/FSND-Item-Catalog-project-5)<br />

<br />
<br />
<br />

## SOME COMMANDS MAY HELP : 
```sudo tail -100 /var/log/apache2/error.log``` ----> To check if there any error with apatche 
```python <File_name>.py runserver -d``` ----> debug mode
