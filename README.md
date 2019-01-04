# Linux Server Configuration

Third project in Udacity Fullstack Nanodegree.
 baseline installation of a Linux server and prepare it to host your web applications, secure the server from a number of attack vectors, and configuring firewall, dealing with encrypted key as login secure method, install and configure a database server, and deploy one of existing web applications onto it.
 
# Server Information
  - Host: http://ec2-18-184-59-194.eu-central-1.compute.amazonaws.com
  - Public IP address: 18.184.59.194
  - Accessible SSH port: 2200

# How did I complete this project?
There are a few things you need to do when you create your server instance in [Amazon Lightsail](https://aws.amazon.com/lightsail/).
1. Sign up or log in if you already have an account.

2. Create an instance.

3. Choose "OS Only" (rather than "Apps + OS"). then, choose Ubuntu as the operating system. Pick your instance image (Ubuntu)

4. Choose your instance plan. Choose $5/month with first month free.

5. Give your instance a hostname.

6. Wait for it to start up. It may take a few minutes for your instance to start up.


7. Once your instance has started up, you can log into it with SSH from your browser.
8. Download the private SSH key by navigating to the Account Page in the connect tab.
9. As required only allow connections for SSH (port 2200), HTTP (port 80), and NTP (port 123) from the networking tab.
10. go to connect tab and press on Connect Using SSH. you'll be logged as the ubuntu user.
11. first you need to add new user and name it grader by Run `$ sudo adduser grader`
12. Create a new file in the sudoers directory with `sudo nano /etc/sudoers.d/grader`
13. Add this `grader ALL=(ALL:ALL) NOPASSWD:ALL`
14. Run `sudo nano /etc/hosts`
15. To solve this error`sudo: unable to resolve host` add this line `127.0.1.1 ip-10-20-52-12`
16. Update all currently installed packages 
- `sudo apt-get update`
- `sudo apt-get upgrade`
17. Change SSH port from 22 to 2200

- Run `sudo nano /etc/ssh/sshd_config`
- Change the port from 22 to 2200
- then run `sudo service ssh restart`
- Confirm it

18. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)

  -  `sudo ufw allow 2200/tcp`
  -  `sudo ufw allow 80/tcp`
  -  `sudo ufw allow 123/udp`
  -  `sudo ufw enable`

19. Change local timezone to UTC. Run `sudo dpkg-reconfigure tzdata` and then none of the above and will show UTC now choose it.

20. Configure key-based authentication for grader user. 
- on your local machine run `ssh-keygen -f ~/.ssh/key_rsa` and configure a password for it. then, run `cat ~/.ssh/key_rsa.pub` and copy the generated key.
- Run this command `sudo mkdir /home/grader/.ssh` then run this `sudo nano /home/grader/.ssh/authorized_keys` and paste your public key and save.
21. Disable ssh login for root user.

   - Run `sudo nano /etc/ssh/sshd_config`
   - Change `PermitRootLogin without-password` line to `PermitRootLogin no`
   - Restart ssh with `sudo service ssh restart`
   - Now you are only able to login using `ssh -i ~/.ssh/udacity_key.rsa -p 2200 grader@35.234.117.82`

22. Install Apache `sudo apt-get install apache2`
23. Install mod_wsgi `sudo apt-get install libapache2-mod-wsgi python-dev` then, Enable mod_wsgi with `sudo a2enmod wsgi` then, Start the web server with `sudo service apache2 start`
24. Clone from Github
   - Install git using: `sudo apt-get install git`
   - `cd /var/www`
   - `sudo mkdir catalog`
   - Change owner of the newly created catalog folder `sudo chown -R grader:grader catalog`
   - `cd /catalog`
   - Clone your project from github `git clone https://github.com/FahadAlsubaie/Item-catalog.git catalog`
   - Create a catalog.wsgi file, with this content. first run `sudo nano catalog.wsgi`
   ```sh 
   import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = 'supersecretkey'
   ```
   Note: your project path should be `var/www/catalog/catalog`
   - Rename `application.py` to `init.py` by this command `mv application.py __init__.py`
25. Install virtual environment

   - First install pip with this command `sudo apt-get install python-pip`
   - Install the virtual environment `sudo pip install virtualenv`
   - Create a new virtual environment with `sudo virtualenv venv`
   - Activate it with `source venv/bin/activate`
   - Change permissions `sudo chmod -R 777 venv`

26. Install Flask and other dependencies

   - Install Flask `pip install Flask`
   - Install all others project dependencies `sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils`

27. Update path of client_secrets.json file

   - `nano __init__.py`
   - Change client_secrets.json path to `/var/www/catalog/catalog/client_secrets.json`
   
28. Configure a new virtual host
- Create a new file with this :` sudo nano /etc/apache2/sites-available/catalog.conf`
- Put this code:
```
<VirtualHost *:80>
    ServerName 18.184.59.194
    ServerAlias http://ec2-18-184-59-194.eu-central-1.compute.amazonaws.com
    ServerAdmin admin@18.184.59.194
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
- Enable the virtual host `sudo a2ensite catalog`

29. Install and setup PostgreSQL

   - `sudo apt-get install libpq-dev python-dev`
   - `sudo apt-get install postgresql postgresql-contrib`
   - `sudo su - postgres`
   - `psql`
   - `CREATE USER catalog WITH PASSWORD 'password';`
 - `ALTER USER catalog CREATEDB;`
 -   `CREATE DATABASE catalog WITH OWNER catalog;`
 -   `\c catalog`
 -   `REVOKE ALL ON SCHEMA public FROM public;`
 -   `GRANT ALL ON SCHEMA public TO catalog;`
 -   `\q`
 -   `exit`
 -   now you need to change the create engine line in your `__init__.py`, `database_setup.py` and `lotsofitem.py` to: `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
  - ` python /var/www/catalog/catalog/database_setup.py`

30. Restart Apache `sudo service apache2 restart`
31. Visit site at `http://[your public ip]/`
# References
- https://github.com/stueken/FSND-P5_Linux-Server-Configuration
- https://github.com/FahadAlsubaie/linux_server_configuration
- https://askubuntu.com
- https://ubuntuforums.org
- https://httpd.apache.org/
