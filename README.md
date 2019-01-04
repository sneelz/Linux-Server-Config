# Introduction

##  App Details

* IP Address: 52.91.58.218
* SSH Port: 2200
* App URL: http://52.91.58.218.xip.io

# Process

## Add New User

*  Create user grader: `sudo adduser grader`
* Confirm user creation: `ls /home`

User: grader
Password: password


## Give User Sudo Permissions

* Create sudoers file for grader user: `sudo nano /etc/sudoers.d/grader`
* Add the following to the file and save: `grader ALL=(ALL) NOPASSWD: ALL`
* Login as grader user to verify sudo permissions: `sudo login grader`


## Generate Key Pairs

* On your local machine, generate a keygen: `ssh-keygen`
* Save the file to ~/.ssh

Saved as: pokedexKey

## Install Public Key

* Login as the grader user: `sudo login grader`
* Make directory in grader home directory: `mkdir .ssh`
* Create auhorized_keys file: `touch .ssh/authorized_keys`
* Copy and paste the contents of the local public key file (.pub) into the authorized_keys file: `nano .ssh/authorized_keys`
* Modify permissions of .ssh directory: `chmod 700 .ssh`
* Modify permissions of authorized_keys file: `chmod 644 .ssh/authorized_keys `
* Login as grader user using the newly created key: `ssh grader@[ AWS IP ] –i ~/.ssh/pokedexKey`


## Force Key-Based Authentication

* Edit sshd_config: `sudo nano /etc/ssh/sshd_config`
* Update PasswordAuthentication: `PasswordAuthentication no`
* Restart SSH service: `sudo service ssh restart`


## Disable Remote Login for Root User

* Edit sshd_config: `sudo nano /etc/ssh/sshd_config`
* Update PermitRootLogin: `PermitRootLogin no`
* Restart SSH service: `sudo service ssh restart`


## Change the SSH port from 22 to 2200

* Edit sshd_config: `sudo nano /etc/ssh/sshd_config`
* Change Port 22 to 2200: `Port 2200`
* Restart SSH service: `sudo service ssh restart`


## Configure Firewall

* Check current UFW status: `sudo ufw status`
* Deny all incoming: `sudo ufw default deny incoming`
* Allow all outgoing: `sudo ufw default allow outgoing`
* Allow SSH: `sudo ufw allow ssh`
* Allow new SSH port: `sudo ufw allow 2200/tcp`
* Allow HTTP: `sudo ufw allow www`
* Allow NTP: `sudo ufw allow 123/udp`
* Deny Port 22: `sudo ufw deny 22`
* Enable updates: `sudo ufw enable`
* Check status to verify updates: `sudo ufw status`

In your Amazon Lightsail instance, go to the "Networking" tab and allow the following ports in the Firewall section:
* Custom - TCP - 2200
* Custom - UDP - 123

In a new terminal, login as grader user: `ssh grader@[ AWS IP ] -p 2200 –i ~/.ssh/pokedexKey`


## Update Packages

* See packages to update: `sudo apt-get update`
* Update packages: `sudo apt-get upgrade`


## Webserver Setup

* Install Apache: `sudo apt-get install apache2`
* Install mod_wsgi: `sudo apt-get install libapache2-mod-wsgi`
* Install PostgreSQL: `sudo apt-get install postgresql`


## Create Database User

* Connect to database: `sudo -u postgres psql`
* Create new database user named grader: `CREATE USER pokemaster WITH PASSWORD 'password';`
* Confirm user was created: `\du`
* Limit user permissions: `ALTER ROLE pokemaster WITH LOGIN; ALTER USER pokemaster WITH CREATEDB;`


## Create Database

* Create database: `CREATE DATABASE pokedexdb WITH OWNER pokemaster;`
* Connect to database: `\c pokedexdb`
* Revoke all rights: `REVOKE ALL ON SCHEMA public FROM public;`
* Only grant access to pokemaster role: `GRANT ALL ON SCHEMA public TO pokemaster;`
* Exit PostgreSQL: `\q`
* Restart PostgreSQL: `sudo service postgresql restart`


## Git

* Install git: `sudo apt-get install git`
* Navigate to /var/www: `cd /var/www`
* Clone github repository: `sudo git clone https://github.com/sneelz/Pokedex.git pokedex`

Note: Project is at /var/www/pokedex/


## Install Dependencies

* Pip: `sudo apt-get install python-pip`
* Flask: `sudo pip install flask`
* Flask-HTTPAuth: `sudo pip install flask-httpauth`
* Flask-SQLAlchemy: `sudo pip install flask-sqlalchemy`
* SQLAlchemy: `sudo pip install sqlalchemy`
* psycopg2: `sudo pip install psycopg2`
* oauth2client: `sudo pip install oauth2client`
* httplib2: `sudo pip install httplib2`
* requests: `sudo pip install requests`
* Werkzeug: `sudo pip install werkzeug`


## Configure Virtual Host

* Create new virtual host file: `sudo nano /etc/apache2/sites-available/pokedex.conf`
* Add the following to the file and save: 

```
<VirtualHost * :80>
    ServerName [AWS IP]
    ServerAdmin admin@[AWS IP]
    WSGIScriptAlias / /var/www/pokedex/vagrant/pokedex.wsgi
    <Directory /var/www/pokedex/vagrant/catalog/>
            Order allow,deny
            Allow from all
    </Directory>
    Alias /static /var/www/pokedex/vagrant/catalog/static
    <Directory /var/www/pokedex/vagrant/catalog/static/>
            Order allow,deny
            Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

* Enable virtual host: `sudo a2ensite pokedex`
* Disable default virtual host: `sudo a2dissite 000-default`


## Create .wsgi File

* Navigate to vagrant folder: `cd /var/www/pokedex/vagrant`
* Create .wsgi file: `sudo nano pokedex.wsgi`
* Add the following to the file and save:

```
#!/usr/bin/python

import sys
import logging

logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/pokedex/vagrant/catalog")

from pokedex import app as application
application.secret_key = 'super_secret_key'
```

## Modify Application Code

Update database calls in database_setup.py and pokedex.py
* Remove: `engine = create_engine('sqlite:///pokedexdb.db?check_same_thread=False')`
* Add: `engine = create_engine('postgresql://pokemaster:password@localhost/pokedexdb')`

Update pokedex.py with full path to client_secrets.json: `/var/www/pokedex/vagrant/catalog/client_secrets.json`

Note: The path to client_secrets.json needs to be updated in two locations in pokedex.py


## Give www-data User Permissions for Image Upload

* Create new group: `sudo groupadd wwwdatagroup`
* Add www-data user to group: `sudo adduser www-data wwwdatagroup`
* Change group for saved images folder: `sudo chgrp wwwdatagroup /var/www/pokedex/vagrant/catalog/static/`
* Change permissions for saved images folder: `sudo chmod 775 /var/www/pokedex/vagrant/catalog/static/`


## Restart Apache

* Restart apache: `sudo service apache2 restart`

Note: If you run into any errors, run the following command to see the error log: `sudo tail -f /var/log/apache2/error.log`

## Resources

* https://hk.saowen.com/a/0a0048ca7141440d0553425e8df46b16cdf4c13f50df4c5888256393d34bb1b9
* https://github.com/twhetzel/ud299-nd-linux-server-configuration
* https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
