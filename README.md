# Linux-server-config--Udacity-FSND
The 3rd project for Udacity's FSND (Full Stack Nano Degree). the purpose is to setup a linux server installation too host a flask app. The rubric includes setting up a user and security as well as basic firewall settings.

This is the configuration log for the Project 3 for Udacity’s Full Stack Nano Degree (FSND).


# Server details for user:

Static Ip address: 18.139.119.48
Web address: http://18.139.119.48
Ssh port: 2200
User: grader
Password: grader (if needed) 
Cloud provider: Amazon AWS Lightsail
OS on server: Ubuntu 16.04

To connect to server, use a terminal app on your local machine.
    sudo nano ~/.ssh/grader
(First, copy the key provided by email and store it as ~/.ssh/grader in your local machine)

    ssh grader@18.139.119.48 -p 2200 -i ~/.ssh/grader


# Software Installed:

ntp:
sudo apt-get install ntp

apache2:
    sudo apt-get install apache2

libapache2-mod-wsgi:
    sudo apt-get install libapache2-mod-wsgi python-dev

sqlite3:
    sudo apt-get sqlite3

postgresql:
     sudo apt-get postgresql postgresql-contrib

Python-pip:
    sudo apt-get install python-pip

virtualenv:
    sudo pip install virtualenv

unattended-upgrades:
    sudo apt install unattended-upgrades    


# Server setup :

After making a Lightsail account, the ssh key can be copied from the user account page.

This key can be saved in a file called Lightsail.pem in ~/.ssh on the local machine.

Log into the server:
    ssh ubuntu@18.139.119.48 -p 2200 -i ~/.ssh/Lightsail.pem

# Updated Packages:

    sudo apt-get update
    sudo apt-get upgrade

# Enabled automation upgrades:
    sudo dpkg-reconfigure --priority low unattended-upgrades

# Created new user:
Added a new user called ‘grader'
    adduser grader
Set password to ‘grader'

# Grant sudo access to new user:

    touch /etc/sudoers.d/grader
    nano /etc/sudoers.d/grader


Pasted this line to give sudo access  to grader
    grader ALL=(ALL) NOPASSWD:ALL

# Configure SSH for New User :
On the local machine crate a new key pair of RSA key using:
    ssh-keygen -t rsa
Reference: https://www.digitalocean.com/community/tutorials/how-to-set-up-ssh-keys--2

Rename the keys to grader and grader.pub respectively

Temporarily log in as the newuser:
    sudo su - grader

# Create the .ssh directory and the authorized_keys file:
    cd /home/grader
    mkdir .ssh
    touch .ssh/authorized_keys
    nano .ssh/authorized_keys

Pasted the contents of grader.pub from the local machine
# Set the ssh file permissions:
    chmod 700 .ssh
    chmod 644 .ssh/authorized_keys
test the connection from the local machine in a new terminal window:
    ssh grader@18.139.119.48 -p 2200 -i ~/.ssh/grader

# SSH Config :
change the ssh default port from 22 to 2200 edit the following file :
    sudo nano /etc/ssh/sshd_config
change  the Port value for 2200

# Prevent Root Login :
Changed the following to no:
    #Authentication:
    PermitRootLogin no

# Force ssh login :
Changed the following to:

    PasswordAuthentication no

# Restarted the service:
    sudo service ssh restart

# Configure Firewall :
Configured (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123):    

    sudo ufw default deny incoming
    sudo ufw allow 2200
    sudo ufw allow 80
    sudo ufw allow 123
    sudo ufw enable
    sudo ufw status


# Set Local Timezone :
Updated timezone to UTC:
    sudo dpkg-reconfigure tzdata

# Configure mod_wsgi :

# Enabled mod_wsgi :
    sudo a2enmod wsgi

# Started the apache web server :
    sudo service apache2 start

# Deploying the App :
Use the following command to move to the /var/www directory:
    cd /var/www
    mkdir catalog
    cd catalog
    mkdir catalog
    cd catalog

# Created the init.py to have the flask application logic:
    sudo nano __init__.py

Added following logic to the file:
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Hello, I love Udacity!"
if __name__ == "__main__":
    app.run()

# Setup Flask :

setup virtual env :
    sudo virtualenv venv

activated the virtual environment with the following command:
    source venv/bin/activate

install Flask :
      pip install flask packaging oauth2client redis passlib flask-httpauth
      pip install sqlalchemy flask-sqlalchemy psycopg2 bleach requests 

set up new Virtual Host :
Ran the following commands to configure the apache sites
    sudo nano /etc/apache2/sites-available/catalog.conf

Added the code :
<VirtualHost *:80>
       ServerName 18.139.119.48
       ServerAdmin admin@18.139.119.48
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

Enable the virtual host :
    sudo a2ensite catalog

Create the .wsgi File :
Apache uses the .wsgi file to serve the Flask app.
    cd /var/www/catalog
    sudo nano catalog.wsgi

Added this code:

#!/usr/bin/python
activate_this = '/var/www/catalog/catalog/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")
from catalog import app as application
application.secret_key = 'Add your secret key'


Restarted apache to reflect changes:
    sudo service apache2 reload
    sudo service apache2 restart

# Confirm app is served :
    View the app : http://18.139.119.48


# Resources :
https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-ubuntu-14-04-lts
https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
