# LinuxServerConfiguration

This project is a part of Udacity Nanodegree Full Stack. Explains how to secure and set up a Linux distribution on a virtual machine. For that install and configure a web and database server to host a web application.

The virtual private server is Amazon EC2. Open Amazon EC2 and login and click on open console and change server place to N.Virginia.

The database server is PostgreSQL.

Choose EC2 and Launch Instance and select Ubuntu Server 18.04 LTS..

Create a new pair and give your key-pair name with name:pj1429.pem and download it and then launch the instance. Then type this command in git:

$ ssh -i pj1429.pem ubuntu@3.91.205.99
22 is Port by Default,Later we need to Change 2200.

Public IP Address : 3.91.205.99

Accessable Port : 2200

Steps to configure Linux server
Create a new folder and paste the downloaded pem file.
Open it by using a command ssh -i pj1429.pem ubuntu@3.91.205.99 .
Now, Update and upgrade installed packages
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get dist-upgrade
Now change port 22 to port 2200. For that edit File using this command: sudo vi /etc/ssh/sshd_config
Change Port number 22 to 2200
PubkeyAuthentication yes
PasswordAuthentication no Save and exit using esc and confirm with :wq
Now restart the service using the command: sudo service ssh restart
Change inbound rules in Amazon EC2 for that:
click on edit and add 3 rules i.e,add port SSh 2200, Http port 80, NTP port 123 and save.
To check port 2200: Working or Not
$ ssh -i pj1429.pem -p 2200 ubuntu@3.91.205.99
Configure Firewall (UFW)
$ sudo ufw status
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2200/tcp
$ sudo ufw allow www
$ sudo ufw allow 123/udp
$ sudo ufw deny 22
$ sudo ufw enable *After Proceed Option Y/N - Y
Creating grader
Login to ubuntu and add user using the command:
$ sudo adduser grader
password: grader 2.Then edits the sudoers file:
$ sudo visudo
In that file add the command: grader ALL=(ALL:ALL) ALL
Save and exit using ctrl+x and confirm with Y.
Verifiy the grader as sudo permission:
$ su - grader
password:grader
Then run this command:
$ sudo cp /home/ubuntu/.ssh/authorized_keys /home/grader/.ssh/authorized_keys
After that change Ownership using the command:
$ chown grader.grader /home/grader/.ssh
Add grader to the sudo Group using command:
$ usermod -aG sudo grader
Then change permissions for .ssh folder:
$ chmod 700 /home/grader/.ssh/
$ chmod 644 /home/grader/.ssh/authorized_keys
Check in vi /etc/ssh/sshd_config file if PermitRootLogin is set to No
Restart SSH: sudo service ssh restart
Check whether grader account Working or Not using command :
$ ssh -i pj1429.pem -p 2200 grader@3.91.205.99
Set the time zone for grader
To set time zone for grader the command as follows
$ sudo dpkg-reconfigure tzdata
Installing of Apache and postgresql software
Now istall apache2 software at grader
$ sudo apt-get install apache2
My project is built with Python3. So, I need to install the Python 3 mod_wsgi package:
$ sudo apt-get install libapache2-mod-wsgi-py3
Enable mod_wsgi using:
$ sudo a2enmod wsgi
To check that apache has installed successfully or not, Open web browser and type your IP Address.
Installing of postgresql software
Enable of wsgi install some libraries of python development:
$ sudo apt-get install libpq-dev python-dev
Then install postgresql:
$ sudo apt-get install postgresql postgresql-contrib
After installation of postgresql, change to postgresql from grader.
$ sudo su - postgres
$ psql
Creating user catlog with password catlog using the command:
$ CREATE USER catalog WITH PASSWORD 'catalog';
Alter the user using the command:
$ ALTER USER catalog CREATEDB;
Create a database using command:
$ CREATE DATABASE catalog WITH OWNER catalog;
Now change to catalog database: \c catalog
Now revoke all the schemas:
$ REVOKE ALL ON SCHEMA public FROM public;
Then grant all the public schemas to catlog:
$ GRANT ALL ON SCHEMA public TO catalog;
Now exit from the database using: $ \q
Switch back to the grader user: $ exit
Installation of git
Now install git using command:
$ sudo apt-get install git
Change the directory to www cd /var/www.
Then clone and setup the Item Catalog project from sudo git clone url_link(https://github.com/VidyaGuruju08/catalog.git).
Now change the ownership of the catalog directory to grader using:
$ sudo chown -R grader:grader catalog
Change to the /var/www/catalog/catalog directory. Then rename the Cproject.py file to init.py using:
$ mv Cproject.py init.py.
We need to change the engine from sqlite to postgresql:
engine = create_engine("sqlite:///catalog.db")
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
Creation of google OAuth credentials
Go to your app's page in the Google APIs Console
Choose Credentials then Create an OAuth Client ID. Then choose Web application.
In that you need to set the authorized JavaScript origins as:
http://IPAddress.xip.io/login
http://IPAddress.xip.io/callback
http://IPAddress.xip.io/gconnect You will then be able to get the client ID.
In login.html change the old client ID with new client ID and also change the old client_secrets.json file with new client_secrets.json
Configure and enable new virtual host
    ServerName ---------------.xip.io
    ServerAlias ec2---------------.compute-1.amazonaws.com
    ServerAdmin ubuntu@54.210.140.47
    WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/catalog/venv/lib/python3.6/site-packages
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
</VirtualHost>```
Now enable virtual host using the command:
$ sudo a2ensite catalog
Set up the Flask application
Setup the flask application and create a file at /var/www/catalog/ with name catalog.wsgi. Then open the file using the command : $ sudo vi catalog.wsgi In edit the file as:
     import logging
     logging.basicConfig(stream=sys.stderr)
     sys.path.insert(0, "/var/www/catalog/")
     from catalog import app as application
     application.secret_key = 'supersecretkey'
Now reload and restart apache.
sudo service apache2 reload
sudo service apache2 restart
From var/www/catalog/catalog create a virtual environment.
$ sudo virtualenv -p python venv Then activate the virtual environment:
$ . venv3/bin/activate
Install the following dependencies in virtual environment:
$ pip install httplib2
$ pip install requests
$ pip install --upgrade oauth2client
$ pip install sqlalchemy
$ pip install flask
$ sudo apt-get install libpq-dev
$ pip install psycopg2-binary
If not installed with this queries or got a error that no module found, install with this command:
$ sudo -H pip3 install flask.
Now enable site using the command:
$ sudo a2ensite catalog *Then give the authentication details and again reload apache.
Now in init.py file do these following changes.
At import statement of database use catalog.database import *
At place of xrange use range
Then run the database file as: python database.py
Then run the sample items file : python Clotsofmenu.py
Now reload and restart the apache.
To check errors in the file use the command:
$ sudo tail -f /var/log/apache2/error.log
Deactivate the virtual environment as : deactivate
Disable the default Apache site using the command:
$ sudo a2dissite 000-default.conf
Security Updates and package updates use these commands.
sudo apt-get update
sudo apt-get upgrade
sudo apt-get dist-upgrade
Now, you can open web-broser or as (http://3.91.205.99.xip.io)
