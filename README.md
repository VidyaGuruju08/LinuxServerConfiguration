# LinuxServerConfiguration

## Aim of this project ##
* This project is a part of Udacity Nanodegree Full Stack. Explains how to secure and set up a Linux distribution on a virtual machine. 

* For that install and configure a web and database server to host a web application.
* The virtual private server is Amazon EC2. Open Amazon EC2 and login and click on open console and change server place to N.Virginia.
* The database server is PostgreSQL.
* Choose EC2 and Launch Instance and select Ubuntu Server 18.04 LTS..
* Create a new pair and give your key-pair name with name: [private-key](https://github.com/VidyaGuruju08/LinuxServerConfiguration/blob/master/privateKey) and download it and then launch the instance. Then type this command in git:
    $ ssh -i filename.pem ubuntu@3.91.205.99
* 22 is Port by Default,Later we need to Change 2200.
* Public IP Address : 3.91.205.99
* Accessable Port : 2200

## Software Installation and Dependencies:

    * sudo apt-get install python-pip
    * pip install httplib2
    * pip install requests
    * pip install --upgrade oauth2client
    * pip install sqlalchemy
    * pip install flask
    * sudo apt-get install libpq-dev
    * pip install psycopg2-binary

## Steps to configure Linux server ##

1. Create a new folder and paste the downloaded pem file.
2. Open it by using a command ssh -i filename.pem ubuntu@3.91.205.99 .
3. Now, Update and upgrade installed packages
   * $ sudo apt-get update
   * $ sudo apt-get upgrade
   * $ sudo apt-get dist-upgrade
4. Now change port 22 to port 2200. For that edit File using this command: sudo vi /etc/ssh/sshd_config
5. Change Port number 22 to 2200
   * PubkeyAuthentication yes
   * PasswordAuthentication no
   * Save and exit using esc and confirm with :wq
6. Now restart the service using the command: sudo service ssh restart
7. Change inbound rules in Amazon EC2 for that:
   * click on edit and add 3 rules i.e,add port SSh 2200, Http port 80, NTP port 123 and save.
8. To check port 2200: Working or Not
   * $ ssh -i filename.pem -p 2200 ubuntu@3.91.205.99
9. Configure Firewall (UFW)
   * $ sudo ufw status
   * $ sudo ufw default deny incoming
   * $ sudo ufw default allow outgoing
   * $ sudo ufw allow 2200/tcp
   * $ sudo ufw allow www
   * $ sudo ufw allow 123/udp
   * $ sudo ufw deny 22
   * $ sudo ufw enable *After Proceed Option Y/N - Y

## Creating grader ##
1. Login to ubuntu and add user using the command:
   * $ sudo adduser grader
   * password: grader
2.Then edits the sudoers file:
   * $ sudo visudo
3. In that file add the command: grader ALL=(ALL:ALL) ALL
   * Save and exit using ctrl+x and confirm with Y.
4. Verifiy the grader as sudo permission:
   * $ su - grader
   * password:grader
5. Then run this command:
   * $ sudo cp /home/ubuntu/.ssh/authorized_keys /home/grader/.ssh/authorized_keys
6. After that change Ownership using the command:
   * $ chown grader.grader /home/grader/.ssh
7. Add grader to the sudo Group using command:
   * $ usermod -aG sudo grader
8. Then change permissions for .ssh folder:
   * $ chmod 700 /home/grader/.ssh/
   * $ chmod 644 /home/grader/.ssh/authorized_keys
9. Check in vi /etc/ssh/sshd_config file if PermitRootLogin is set to No
10. Restart SSH: sudo service ssh restart
11. Check whether grader account Working or Not using command :
   * $ ssh -i filename.pem -p 2200 grader@3.91.205.99

## Set the time zone for grader##
1. To set time zone for grader the command as follows
   * $ sudo dpkg-reconfigure tzdata

## Installing of Apache and postgresql software
1. Now istall apache2 software at grader
   * $ sudo apt-get install apache2
2. My project is built with Python3. So, I need to install the Python 3 mod_wsgi package:
   * $ sudo apt-get install libapache2-mod-wsgi-py3
3. Enable mod_wsgi using:
   * $ sudo a2enmod wsgi
4. To check that apache has installed successfully or not, Open web browser and type your IP Address.

## Installing of postgresql software##
1. Enable of wsgi install some libraries of python development:
   * $ sudo apt-get install libpq-dev python-dev
2. Then install postgresql:
   * $ sudo apt-get install postgresql postgresql-contrib
3. After installation of postgresql, change to postgresql from grader.
   * $ sudo su - postgres
   * $ psql
4. Creating user catlog with password catlog using the command:
   * $ CREATE USER catalog WITH PASSWORD 'catalog';
5. Alter the user using the command:
    * $ ALTER USER catalog CREATEDB;
6. Create a database using command:
   * $ CREATE DATABASE catalog WITH OWNER catalog;
7. Now change to catalog database: \c catalog
8. Now revoke all the schemas:
   * $ REVOKE ALL ON SCHEMA public FROM public;
9. Then grant all the public schemas to catlog:
   * $ GRANT ALL ON SCHEMA public TO catalog;
10. Now exit from the database using: $ \q
11. Switch back to the grader user: $ exit

## Installation of git ##
1. Now install git using command:
   * $ sudo apt-get install git
2. Change the directory to www cd /var/www.
3. Then clone and setup the Item Catalog project from sudo git clone [url_link](https://github.com/VidyaGuruju08/catalog.git).
4. Now change the ownership of the catalog directory to grader using:
   * $ sudo chown -R grader:grader catalog
5. Change to the /var/www/catalog/catalog directory. 
6. Then rename the Cproject.py file to init.py using:
   * $ mv Cproject.py init.py.
7. We need to change the engine from sqlite to postgresql:
```
engine = create_engine("sqlite:///catalog.db")
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
```
## Creation of google OAuth credentials
1. Go to your app's page in the Google APIs Console
2. Choose Credentials then Create an OAuth Client ID. Then choose Web application.
3. In that you need to set the authorized JavaScript origins as:
   ```
   http://IPAddress.xip.io/login
   http://IPAddress.xip.io/callback
   http://IPAddress.xip.io/gconnect
   ```
4. You will then be able to get the client ID.
5. In login.html change the old client ID with new client ID and also change the old client_secrets.json file with new                      client_secrets.json
6. Configure and enable new virtual host

```
<VirtualHost *:80>
    ServerName IPAddress.xip.io
    ServerAlias ec2-3-91-205-99.compute-1.amazonaws.com
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
</VirtualHost>
```
7. Now enable virtual host using the command:
   * $ sudo a2ensite catalog

## Set up the Flask application and creation of virtual environment
1. Setup the flask application and create a file at /var/www/catalog/ with name catalog.wsgi. 
2. Then open the file using the command : 
   * $ sudo vi catalog.wsgi 
 In edit the file as:
 ```
     import logging
     logging.basicConfig(stream=sys.stderr)
     sys.path.insert(0, "/var/www/catalog/")
     from catalog import app as application
     application.secret_key = 'supersecretkey'
 ```
3. Now reload and restart apache.
   * $ sudo service apache2 reload
   * $ sudo service apache2 restart
4. From var/www/catalog/catalog create a virtual environment.
   * $ sudo virtualenv -p python venv3
5. Then activate the virtual environment:
   * $ . venv3/bin/activate
6. Install the following dependencies in virtual environment:
```
    $ sudo apt-get install python-pip
    $ pip install httplib2
    $ pip install requests
    $ pip install --upgrade oauth2client
    $ pip install sqlalchemy
    $ pip install flask
    $ sudo apt-get install libpq-dev
    $ pip install psycopg2-binary
```
7. If not installed with this queries or got a error that no module found, install with this command:
   * $ sudo -H pip3 install flask.
8. Now enable site using the command:
    * $ sudo a2ensite catalog
    * Then give the authentication details and again reload apache.
9. Now in init.py file do these following changes.
  * At import statement of database use catalog.database import *
  * At place of xrange use range
10. Then run the database file as: **python database.py**
11. Then run the sample items file : **python Clotsofmenu.py**
12. Now reload and restart the apache.
13. To check errors in the file use the command:
   * $ sudo tail -f /var/log/apache2/error.log
14. Deactivate the virtual environment as : deactivate
15. Disable the default Apache site using the command:
   * $ sudo a2dissite 000-default.conf
16. Security Updates and package updates use these commands.
```
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get dist-upgrade
```
17. Now, you can open web-broser or as (http://3.91.205.99.xip.io)

* I took the help of my mentor instructions.
* Refernces from some github README notes and udacity videos.
* Online Resources:
 - Flask Framework: http://flask.pocoo.org/
 - Deployment: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
 - YouTube: https://www.youtube.com/watch?v=f28nZ2ubYFk
 - Stack Overflow: https://stackoverflow.com/
