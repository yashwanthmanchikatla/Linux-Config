# udacity-linux-server-configuration

### Project Description

Take a baseline installation of a Linux distribution on a virtual machine and prepare it to host your web applications, to include installing updates, securing it from a number of attack vectors and installing/configuring web and database servers.

- IP address: 52.15.190.60

- Accessible SSH port: 2200

- Application URL: http://ec2-52-15-190-60.us-east-2.compute.amazonaws.com/

### Walkthrough

1. Create new user named grader and give it the permission to sudo
  - SSH into the server through `ssh -i ~/.ssh/udacity_key. root@18.217.12.91`
  - Run `$ sudo adduser grader` to create a new user named grader
  - Create a new file in the sudoers directory with `sudo nano /etc/sudoers.d/grader`
  - Add the following text `grader ALL=(ALL:ALL) ALL`
  - Run `sudo nano /etc/hosts`
  - Prevent the error `sudo: unable to resolve host` by adding this line `127.0.1.1 ip-10-20-52-12`
   
2. Update all currently installed packages
  - Download package lists with `sudo apt-get update`
  - Fetch new versions of packages with `sudo apt-get upgrade`

3. Change SSH port from 22 to 2200
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change the port from 22 to 2200
  - Confirm by running `ssh -i ~/.ssh/udacity_key.rsa -p 2200 root@18.217.12.91`
  
4. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
  - `sudo ufw allow 2200/tcp`
  - `sudo ufw allow 80/tcp`
  - `sudo ufw allow 123/udp`
  - `sudo ufw enable`
  
5. Configure the local timezone to UTC
  - Run `sudo dpkg-reconfigure tzdata` and then choose UTC
 
6. Configure key-based authentication for grader user
  - Run this command `cp /root/.ssh/authorized_keys /home/grader/.ssh/authorized_keys`

7. Disable ssh login for root user
  - Run `sudo nano /etc/ssh/sshd_config`
  - Change `PermitRootLogin without-password` line to `PermitRootLogin no`
  - Restart ssh with `sudo service ssh restart`
  - Now you are only able to login using `ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@18.217.12.91`
 
8. Install Apache
  - `sudo apt-get install apache2`

9. Install mod_wsgi
  - Run `sudo apt-get install libapache2-mod-wsgi python-dev`
  - Enable mod_wsgi with `sudo a2enmod wsgi`
  - Start the web server with `sudo service apache2 start`

  
10. Clone the Catalog app from Github
  - Install git using: `sudo apt-get install git`
  - `cd /var/www/itemcatalog/`
  - Clone your project from github `git clone --single-branch https://github.com/yashwanthmanchikatla/Item-Catalog.git`
  - Create a itemcatalog.wsgi file, then add this inside:
  ```
  
import sys
sys.path.insert(0, '/var/www/itemcatalog/Item-Catalog')

# Activate venv
activate_this = '/var/www/itemcatalog/Item-Catalog/venv/bin/activate_this.py'
with open(activate_this) as file_:
    exec(file_.read(), dict(__file__=activate_this))

from itemcatalog import app as application
  ```
  - Rename application.py to __init__.py `mv project.py __init__.py`
  
11. Install virtual environment
  - Install the virtual environment `sudo pip install virtualenv`
  - Create a new virtual environment with `sudo virtualenv venv`
  - Activate the virutal environment `source venv/bin/activate`
  - Change permissions `sudo chmod -R 777 venv`

12. Install Flask and other dependencies
  - Install pip with `sudo apt-get install python-pip`
  - Install Flask `pip install Flask`
  - Install other project dependencies `sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils`

13. Update path of client_secrets.json file
  - `nano __init__.py`
  - Change client_secrets.json path to `/srv/mkdir fullstack-nanodegree-vm/client_secrets.json`
  
14. Configure and enable a new virtual host
  - Run this: `sudo nano /etc/apache2/sites-available/catalog.conf`
  - Paste this code: 
  ```
 
<VirtualHost *:80>
  # In the context of virtual hosts, the ServerName specifies what hostname 
  # must appear in the request's Host-header to match this virtual host.
  # Change the ServerName to your domain or cloud server's IP address!!
  ServerName ec2-52-15-190-60.us-east-2.compute.amazonaws.com
  # Set the alternate names for this host
  ServerAlias 52.15.190.60
  
  # Set the contact address that the server includes in any error messages 
  # it returns to the client
  ServerAdmin myashu1093@gmail.com
  
  # Mounting The WSGI Application At Root of Site
  # First argument is the URL mount point. Second argument is the pathname 
  # to the wsgi application, which should be mounted against the specific URL.
  WSGIScriptAlias / /var/www/itemcatalog/itemcatalog.wsgi  
  
  # The 'Alias' command maps URLs to filesystem locations.
  # This means a request for http://mywebsite.com/static/foo.jpg would cause the 
  # server to return the file /var/www/itemcatalog/itemcatalog/static/foo.jpg
  Alias /static /var/www/itemcatalog/Item-Catalog/static
  # Current host will be allowed to access this directory 
  <Directory /var/www/itemcatalog/Item-Catalog/static/>
      Order allow,deny
      Allow from all
  </Directory>
  
  # Log server events in the following files:
  # ${APACHE_LOG_DIR} is set to '/var/log/apache2' by default
  ErrorLog ${APACHE_LOG_DIR}/error.log
  CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
  ```
  - Enable the virtual host `sudo a2ensite catalog`

15. Install and configure PostgreSQL
  - `sudo apt-get install libpq-dev python-dev`
  - `sudo apt-get install postgresql postgresql-contrib`
  - `sudo su - postgres`
  - `psql`
  - `CREATE USER catalog WITH PASSWORD 'password';`
  - `ALTER USER catalog CREATEDB;`
  - `CREATE DATABASE catalog WITH OWNER catalog;`
  - `\c catalog`
  - `REVOKE ALL ON SCHEMA public FROM public;`
  - `GRANT ALL ON SCHEMA public TO catalog;`
  - `\q`
  - `exit`
  - Change create engine line in your `__init__.py` and `database_setup.py` to: 
  `engine = create_engine("postgresql://catalog:topsecret@localhost/catalogdb")`
  - Make sure no remote connections to the database are allowed. Check if the contents of this file `sudo nano /etc/postgresql/9.3/main/pg_hba.conf` looks like this:
  ```
  local   all             postgres                                peer
  local   all             all                                     peer
  host    all             all             127.0.0.1/32            md5
  host    all             all             ::1/128                 md5
  ```
  
16. Restart Apache 
  - `sudo service apache2 restart`
  
17. Visit site at [http://52.15.190.60](http://52.15.190.60)