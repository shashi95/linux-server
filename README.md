# Project: linux-server 

Live at 52.66.214.34.xip.io

What is this project all about?

Baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

Why this project?
-------------------
A deep understanding of exactly what your web applications are doing, how they are hosted, and the interactions between multiple systems are what define you as a Full Stack Web Developer. In this project, you’ll be responsible for turning a brand-new, bare bones, Linux server into the secure and efficient web application host your applications need.

System Environment System
-----------------------------------
1. Start a new Ubuntu Linux server instance on Amazon Lightsail (https://lightsail.aws.amazon.com/).
2. Follow the instructions provided to SSH into your server.

Secure your server.
----------------------
3. Update all currently installed packages.
4. Change the SSH port from 22 to 2200. Make sure to configure the Lightsail firewall to allow it.
5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

##Warning: When changing the SSH port, make sure that the firewall is open for port 2200 first, so that you don't lock yourself out of the server. When you change the SSH port, the Lightsail instance will no longer be accessible through the web app 'Connect using SSH' button. The button assumes the default port is being used. There are instructions on the same page for connecting from your terminal to the instance. Connect using those instructions and then follow the rest of the steps.

Give grader access.
---------------------

6. Create a new user account named grader.
7. Give grader the permission to sudo.
8. Create an SSH key pair for grader using the ssh-keygen tool.

Prepare to deploy your project.
---------------------------------
9. Configure the local timezone to UTC.
10. Install and configure Apache to serve a Python mod_wsgi application.
11. Install and configure PostgreSQL:

Do not allow remote connections
----------------------------------
Create a new database user named catalog that has limited permissions to your catalog application database.
12. Install git.

Deploy the Item Catalog project.
------------------------------------
13. Clone and setup your Item Catalog project from the Github repository.
14. Set it up in your server so that it functions correctly when visiting your server’s IP address in a browser.


How to ssh into remote instance
------------------------------------

Generate private public using ssh-keygen: $ssh-keygen
Its asks for file to save keys, by default it will save in id_rsa and id_rsa.pub
Download .pem file from lightsail account dashboard and save it in local machine.

Now you can ssh to remote machine using : $ ssh -i path/to/pem/file ubuntu@ip-addr

Create a new user named grader
------------------------------------

sudo adduser grader
sudo vi /etc/sudoers.d/grader and add grader ALL=(ALL:ALL) ALL into this file and save it.

generate ssh key on local and follow below steps
* $ su - grader
* $ mkdir .ssh
* $ sudo vi .ssh/authorized_keys
* Copy the public key generated on your local machine to this file and save
We can now ssh as grader user using private key generated in above step.

$ chmod 600 .ssh/authorized_keys
$service ssh restart

ssh -i private_key grader@52.66.214.34/

Update packages
------------------------------------

sudo apt-get update
sudo apt-get upgrade

Change the SSH port from 22 to 2200
------------------------------------

Use sudo vi /etc/ssh/sshd_config and then change Port 22 to Port 2200.
$sudo service ssh restart

Lets configure the firewall now
------------------------------------

sudo ufw allow 2200/tcp
sudo ufw allow 80/tcp
sudo ufw allow 123/udp
sudo ufw enable 

Configure the local timezone to UTC
------------------------------------

$sudo dpkg-reconfigure tzdata
select UTC from popout list


Install and configure Apache to serve a Python mod_wsgi application
------------------------------------

$sudo apt-get install apache2
$sudo apt-get install python-setuptools libapache2-mod-wsgi
$sudo service apache2 restart


Install and configure PostgreSQL
------------------------------------

$sudo apt-get install postgresql

In order to check for remote connections, do the following 
$sudo vim /etc/postgresql/9.3/main/pg_hba.conf

Login to postgres now
------------------------------------

$sudo su - postgres
In postgres shell run $psql

Create database and user using this command

postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;

Set password for user catalog

postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
Grant catalog user permission to catalog databse

postgres=# GRANT ALL PRIVILEGES ON DATABASE public TO catalog;
postgres=# \q

get back to home user now.

Git
------------------------------------

$sudo apt-get install git
$cd /var/www
$sudo mkdir CatalogFlaskApp
$cd CatalogFlaskApp
$git clone https://github.com/shashi95/item-catalog
$sudo mv item-catalog itemcatalog

Since we are using python 2.7 we must need to change name of our application file 
$sudo mv itemcatalog/views.py __init__.py

Change few lines in database_setup.py, __init__.py and populate_db.py files
from engine = create_engine('sqlite:///sport.db') to engine = create_engine('postgresql://catalog:password@localhost/catalog')


$sudo apt-get install python-pip
$sudo pip install -r requirements.txt
$sudo apt-get -qqy install postgresql python-psycopg2
$sudo python database_setup.py


New vitual host setup
$sudo vi /etc/apache2/sites-available/catalogflaskApp.conf

Add the following lines of code here:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
<VirtualHost *:80>
	ServerName 52.66.214.34.xip.io
	ServerAdmin youremail@gmail.com
	WSGIScriptAlias / /var/www/CatalogFlaskApp/catalogflaskapp.wsgi
	<Directory /var/www/CatalogFlaskApp/itemcatalog/>
		Order allow,deny
		Allow from all
	</Directory>
	Alias /static /var/www/CatalogFlaskApp/itemcatalog/static
	<Directory /var/www/CatalogFlaskApp/itemcatalog/static/>
		Order allow,deny
		Allow from all
	</Directory>
	ErrorLog ${APACHE_LOG_DIR}/error.log
	LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

$sudo a2ensite FlaskApp
$sudo vi /var/www/CatalogFlaskApp/catalogflaskapp.wsgi 

add below code to flaskapp.wsgi file:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/CatalogFlaskApp/")
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
from FlaskApp import app as application
application.secret_key = 'Add your secret key'

$sudo service apache2 restart

