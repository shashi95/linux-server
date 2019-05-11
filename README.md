# Project: linux-server 

* ip addess - 52.66.192.4
* ssh port  - 2200
* Hosted web link -  http://52.66.192.4.xip.io/
* App directory - /var/www/CatalogFlaskApp/
* grader user password - 12345

What is this project all about?
----------------------------------------

Baseline installation of a Linux server and prepare it to host your web applications. You will secure your server from a number of attack vectors, install and configure a database server, and deploy one of your existing web applications onto it.

Why this project?
-------------------
A deep understanding of exactly what your web applications are doing, how they are hosted, and the interactions between multiple systems are what define you as a Full Stack Web Developer. In this project, you’ll be responsible for turning a brand-new, bare bones, Linux server into the secure and efficient web application host your applications need.

Required software to be installed on the system
----------------------------------------------------
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Cheetah==2.4.4
Flask==0.9
Jinja2==2.6
PAM==0.4.2
PyYAML==3.10
SQLAlchemy==0.7.4
Twisted-Core==13.2.0
Twisted-Names==13.2.0
Twisted-Web==13.2.0
Werkzeug==0.10.4
apt-xapian-index==0.45
argparse==1.2.1
chardet==2.0.1
cloud-init==0.7.5
colorama==0.2.5
configobj==4.7.2
html5lib==0.999
httplib2==0.9
jsonpatch==1.3
jsonpointer==1.0
oauth==1.0.1
oauth2client==1.4
prettytable==0.7.2
psycopg2==2.4.5
pyOpenSSL==0.13
pyasn1==0.1.7
pyasn1-modules==0.0.5
pycurl==7.19.3
pyserial==2.6
python-apt==0.9.3.5ubuntu1
python-debian==0.1.21-nmu2ubuntu2
requests==2.6.0
rsa==3.1.4
six==1.9.0
ssh-import-id==3.21
urllib3==1.7.1
wheel==0.24.0
wsgiref==0.1.2
zope.interface==4.0.5

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

System Environment settings
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

* $sudo adduser grader
* $sudo vi /etc/sudoers.d/grader 
* add grader ALL=(ALL:ALL) ALL into this file and save it.
* $id grader: uid=1001(grader) gid=1001(grader) groups=1001(grader),27(sudo)


generate ssh key on local and follow below steps
* $ su - grader
* $ mkdir .ssh
* $ sudo vi .ssh/authorized_keys
* copy /home/grader/.ssh/id_rsa.pub to /home/grader/.ssh/authorized_keys
* $ chmod 600 .ssh/authorized_keys
* $service ssh restart
* Now using private key, we can access grader user.
* ssh -i private_key grader@52.66.214.34/

Disbale root login remotely
----------------------------------
* sudo vi /etc/ssh/sshd_config
* Set PermitRootLogin no, save and quit.

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
* sudo apt-get install ufw
* sudo ufw default allow outgoing
* sudo ufw default deny incoming
* sudo ufw allow 2200/tcp
* sudo ufw allow 80/tcp
* sudo ufw allow 123/udp
* sudo ufw enable 

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

Third party resources
------------------------------
* https://www.tecmint.com/disable-root-login-in-linux/
* https://www.linode.com/docs/security/firewalls/configure-firewall-with-ufw/
* https://askubuntu.com/questions/94102/what-is-the-difference-between-apt-get-update-and-upgrade
* https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
