-------------------------------------------------
-------------------------------------------------
------PROJECT: LINUX SERVER CONFIGURATION -------
-------------------------------------------------
-------------------------------------------------

THIS README FILE IS FOR THE UDACITY LINUX SERVER CONFIGURATION 
PROJECT, PART OF THE FULL STACK WEB DEVELOPER NANO DEGREE.

PROJECT OVERVIEW: 
You will take a baseline installation of a Linux server and prepare 
it to host your web applications. You will secure your server 
from a number of attack vectors, install and configure a database 
server, and deploy one of your existing web applications onto it.

Your README.md file should include all of the following:
i. The IP address and SSH port so your server can be accessed by the reviewer.
ii. The complete URL to your hosted web application.
iii. A summary of software you installed and configuration changes made.
iv. A list of any third-party resources you made use of to complete this project.


-------------------------------------------------
-------------IP ADDRESS & SSH PORT---------------
-------------------------------------------------
IP: 13.211.203.41
SSH PORT: 2200

-------------------------------------------------
-------------URL OF WEB APPLICATION--------------
-------------------------------------------------
URL: http://13.211.203.41.xip.io

-------------------------------------------------
--SUMMARY OF SOFTWARE INSTALLED & CONFIGURATION--
-------------------------------------------------

INITIATE INSTANCE OF AMAZON LIGHTSAIL & CONNECT
1. Connect as ubuntu user with private key authentication.


UPDATE CURRENTLY INSTALLED PACKAGES
1. Update the package source list.
	$ sudo apt-get update
2. Upgrade all packages to latest version.
	$ sudo apt-get upgrade


CHANGE SSH PORT FROM 22 TO 2200
1. Edit the sshd config file.
	$ sudo nano /etc/ssh/sshd_config

2. Where it says port number change it from 22 to 2200 and save the changes.

3. Restart the sshd service.
	$ sudo service sshd restart

4. Close the current ssh connection.

5. Set LightSail Firewall to allow TCP connections on port 2200.

6. Delete the SSH connections on port 22.

7. Reconnect with ssh on port 2200. 


SET UP THE UNCOMPLICATED FIREWALL (UFW)
1. Set LightSail Firewall to allow UDP connections on port 123.

2. Now configure the UFW.
	a. Confirm firewall is inactive.
		$ sudo ufw status
	b. Deny incoming connections:
		$ sudo ufw default deny incoming
	c. Allow outgoing connections:
		$ sudo ufw default allow outgoing
	d. Allow ports for SSH, HTTP and NTP.
		$ sudo ufw allow 2200/tcp
		$ sudo ufw allow 123/udp
		$ sudo ufw allow 80/tcp
	e. Enable the firewall.
		$ sudo ufw enable


SET TIMEZONE TO UTC
1. Run:
	$ sudo dpkg-reconfigure tzdata

2. Select "none of the above".

3. Select "UTC".


GIVE GRADER ACCESS
1. Create a new user account named grader and note password.
	$ sudo adduser grader

2. A grader to the sudo users group.
	a. Run:
		$ sudo usermod -aG sudo grader
	b. Run:
		$ sudo visudo
	c. Add this to the last line of the file to stop sudo password prompts for grader:
		$ grader ALL=(ALL) NOPASSWD:ALL

3. On my local Ubuntu virtual machine I created SSH key pair for grader.
	a. Execute 'ssh-keygen' from command line.
	b. Choose file path and name e.g. /home/vagrant/.ssh/grader
	c. Leave passphrase empty.
	d. I now have public and private keys for grader.
	e. As I'm connecting on Windows copy the key files to the local ~/.ssh folder.

4. Back on the LightSail instance.
	a. Change to the following directory.
		$ cd /home/grader
	b. Make a new directory for the public key.
		$ sudo mkdir .ssh
	c. Create and edit the public key file.
		$ sudo nano .ssh/authorized_keys
	d. Copy contents of 'grader' public key into this new file and save.
	e. Set file permissions on the directory and file:
		a. RW to owner, R to group/everyone.
			$ sudo chmod 644 .ssh/authorized_keys
		b. RWX to owner only.
			$ sudo chmod 700 .ssh
	f. Change owner and group from root to grader.
		$ sudo chown -R grader.grader /home/grader/.ssh

5. Enforce key pair authentication for all users.
	a. Edit sshd config file.
		$ sudo nano /etc/ssh/sshd_config
	b. If not already, change the line: PasswordAuthentication 'yes' to 'no'.
	c. Change PermitRootLogin 'prohibit-password' to 'no'.
	d. Restart the ssh service.
		$ sudo service ssh restart

6. Log off as 'ubuntu' user and connect as 'grader' with private key on port 2200.
	$ ssh grader@13.211.203.41 -i ~/.ssh/grader -p 2200


PREPARE APACHE WEB SERVER
1. Install Apache web server.
	$ sudo apt-get install apache2

2. Navigate to http://13.211.203.41 to check that the Apache default page is visible.

3. Install mod_wsgi for python3.
	$ sudo apt-get install libapache2-mod-wsgi-py3
	$ sudo service apache2 restart
	$ sudo a2enmod wsgi


PREPARE POSTGRESQL DATABASE
1. Install PostgreSQL:
	$ sudo apt-get install postgresql

2. Configure PostgreSQL to start up upon server boot.
	$ sudo update-rc.d postgresql enable

3. Create databases and user.
	$ sudo su -
	$ service postgresql start
	$ su - postgres
	$ psql
	postgres=# CREATE DATABASE itemcategory;
	postgres=# CREATE USER itemcategory;
	postgres=# ALTER ROLE itemcategory WITH PASSWORD 'password';
	postgres=# GRANT ALL PRIVILEGES ON DATABASE itemcategory TO itemcategory;
	\q
	$ exit
	$ exit

4. Check that remote connections are not allowed.
	$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf


DEPLOY THE ITEM CATALOG WEB APPLICATION
1. If not installed already, install git.
	$ sudo apt-get install git

2. Create directory to hold Item Catalog app.
	$ cd /var/www/
	$ sudo mkdir catalog

3. Clone git repository.
	$ cd /var/www/catalog
	$ sudo git clone https://github.com/milesperry81/catalog.git

4. Change application.py to __init__.py
	$ cd catalog/
	$ sudo mv application.py __init__.py

5. Install pip3.
	$ sudo apt-get install python3-pip
	$ sudo -H pip3 install --upgrade pip

6. Install virtual environment.
	$ sudo pip3 install virtualenv

7. Install psycopg2.
	$ sudo apt-get install python-psycopg2
	$ sudo apt-get install libpq-dev python-dev
	$ pip3 install psycopg2 --user

8. Create virtual environment and install required python packages.
	a. Create.
		$ sudo virtualenv -p python3 venv
	b. Give grader owner and group privileges on the 'venv' folder.
		$ sudo chown -R grader.grader /var/www/catalog/catalog/venv
	c. Activate.
		$ source venv/bin/activate
	d. Install python packages.
		$ pip3 install Flask 
		$ pip3 install sqlalchemy
		$ pip3 install oauth2client
		$ pip3 install flask_httpauth
		$ pip3 install psycopg2
	e. Deactivate.
		$ deactivate

9. Install python packages with sudo outside the virtual environment.
	$ sudo pip3 install Flask 
	$ sudo pip3 install sqlalchemy
	$ sudo pip3 install oauth2client
	$ sudo pip3 install flask_httpauth
	$ sudo pip3 install psycopg2

10. Create virtual host.
	a. Create and edit conf file:
		$ sudo nano /etc/apache2/sites-available/catalog.conf
	b. Copy and paste:
	----------------------------------------------
	<VirtualHost *:80>
			ServerName 13.211.203.41
			ServerAdmin admin@13.211.203.41
			WSGIDaemonProcess catalog home=/var/www/catalog/catalog python-path=/var/www/catalog:/var/www/catalog/catalog/venv/lib/python3.5/site-packages
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
	----------------------------------------------
	c. Enable virtual host:
		$ sudo a2ensite
	d. Choose catalog.

11. Create wsgi file.
	a. Create and edit file.
		$ sudo nano /var/www/catalog/catalog.wsgi 
	b. Copy and paste:
	----------------------------------------------
	#!/usr/bin/python3
	import sys
	import logging
	logging.basicConfig(stream=sys.stderr)
	sys.path.insert(0,"/var/www/catalog/")

	from catalog import app as application
	application.secret_key = 'Add your secret key'
	----------------------------------------------

12. Edit the python files for the web application.
	a. cd /var/www/catalog/catalog
	b. sudo nano __init__.py
	c. Replace: create_engine('sqlite:///itemCategory.db')
		With: create_engine('postgresql://itemcategory:password@localhost/itemcategory')
	d. Change app.run(host='0.0.0.0', port=8000), to app.run(host='0.0.0.0')
	e. Replace create_engine as above in step c) for database_setup.py and createCategoriesAndItems.py.

13. Run database_setup.py then createCategoriesAndItems.py.
	a. python3 database_setup.py
	b. python3 createCategoriesAndItems.py

14. Restart apache:
	$ sudo service apache2 restart 


MAKE THE REQUIRED CHANGES FOR THE GOOGLE OAUTH
1. Change client_secrets.json file:
	a. $ sudo nano client_secrets.json
	a. Set javascript origins to http://13.211.203.41.xip.io
	b. Set redirect URI's to http://13.211.203.41.xip.io/storeauthcode

2. Update the authorised domain for the project in Google Cloud Platform to:
	xip.io

3. Change the client ID for your project in the Google Cloud Platform:
	a. $ sudo nano client_secrets.json
	a. Set javascript origins to http://13.211.203.41.xip.io
	b. Set redirect URI's to http://13.211.203.41.xip.io/storeauthcode

4. Edit the virtual host .conf file:
	a. $ sudo nano /etc/apache2/sites-available/catalog.conf
	b. Add new line under the server name: ServerAlias 13.211.203.41.xip.io

4. Restart apache service.
	$ sudo service apache2 restart


-------------------------------------------------
-------------THIRD PARTY RESOURCES---------------
-------------------------------------------------

https://github.com/jungleBadger/-nanodegree-linux-server-troubleshoot

https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps#step-two-%E2%80%93-creating-a-flask-app




