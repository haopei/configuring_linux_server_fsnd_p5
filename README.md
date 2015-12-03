# Configuring a Linux Server
This project configures a new linux server to host a catalog application which was created in Project #3 on Udacity's FSND.

##### Project Links
 * Amazon EC2 Link: ec2-52-33-152-154.us-west-2.compute.amazonaws.com
 * IP Address: http://52.33.152.154

##### Summary of Installed Packages
 * finger
 * apache2
 * libapache2-mod-wsgi
 * git
 * python-dev
 * python-pip
 * virtualenv
 * Flask
 * sqlalchemy
 * requests
 * httplib2
 * python-psycopy2
 * postgresql
 * postgresql-contrib
 * fail2ban
   * sendmail
   * iptables-persistent
 * cron

### Student Configuration Activity Log
These are the steps taken by the student to complete the project.

##### Setting Up
 * Download `udacity_key.rsa` into local machine's `~/.ssh/`. Run `$ chmod 600 ~/.ssh/udacity_key.rsa` to configure its file permission.
 * SSH into development environment as `root` using `$ ssh -i ~/.ssh/udacity_key.rsa root@52.33.152.154`.
 * Create new user **grader** with `$ sudo adduser grader` and select a password.
 * Create a public/private key-pair named **udacity_devenv** with `$ ssh-keygen`, and select a password.
 * Switch to **grader** user: `$ su - grader`
 * As **grader**, create the file `~/.ssh/authorized_keys` on the server and copy the contents of `~/.ssh/udacity_devenv.pub` (from local machine) into it.
 * Configure permissions of the newly created files with `$ chmod 700 ~/.ssh` and `$ chmod 644 ~/.ssh/authorized_keys`.
 * We may now login with **grader** using `$ ssh -i ~/.ssh/udacity_devenv grader@52.33.152.154`.
 * Disable Password Login: Update to new value `PasswordAuthentication no` inside `/etc/ssh/sshd_config`
 * Make **grader** a sudoer: create the `/etc/sudoers.d/grader` file with the value `grader ALL=(ALL) NOPASSWD:ALL`
 * Update and Upgrade Packages:  `$ sudo apt-get update`, then `$ sudo apt-get upgrade`
 * Install `finger`: `$ sudo apt-get install finger`
 * Change from port 22 to 2200 by editing the `Port 22` value to `Port 2200` inside `/etc/ssh/sshd_config`
 * Configure `ufw` firewall:
   * `$ sudo ufw default deny incoming`
   * `$ sudo ufw default allow outgoing`
   * `$ sudo ufw allow 2200/tcp`
   * `$ sudo ufw allow ssh`
   * `$ sudo ufw allow http`
   * `$ sudo ufw allow www`
   * `$ sudo ufw allow 80/tcp`
   * `$ sudo ufw allow ntp`
   * `$ sudo ufw allow 123/tcp`
   * `$ sudo ufw enable`
 * Change timezone to UTC: `$ sudo dpkg-reconfigure tzdata` and follow the GUI.
 * Install apache2: `$ sudo apt-get install apache2`. Visiting `52.33.152.154` to validate that it has been successfully installed.
 * Install and enable `mod-wsgi`: `$ sudo apt-get install llibapache2-mod-wsgi`, then `$ sudo a2enmod wsgi`
 * Install git: `$ sudo apt-get install git`
 * Install `python-dev`: `$ sudo apt-get install python-dev`
 * Install pip: `$ sudo app-get install python-pip`
 * Install python-psycopy2 (PostgreSQL/Python adapter): `$ sudo apt-get install python-psycopy2`

##### Set up Flask app on Ubuntu Server
 * Create this directory to hold the app within the server: `/var/www/catalog/catalog/` then `$ cd /var/www/catalog/catalog/`.
 * Clone the app from its github source into the above directory: `$ git clone [github-project-url].git`
 * Move all project files into `/var/www/catalog/catalog`
 * Rename `app.py` into `__init__.py`
 * Create `/var/www/catalog/app.wsgi` file with the following code:
``` python
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
# define path to the wsgi application
sys.path.insert(0,"/var/www/catalog/")
from catalog import app as application
application.secret_key = 'super_secret_key'
```
 * Set up a virtual environment inside /var/www/catalog/catalog/ which would keep the application's dependencies isolated from the main system. Changes to the virtual environment will not affect the cloud server's system. Install virtual engironment: `$ sudo pip install virtualenv`
 * Set up a virtual environment named **venv**: `$ sudo virtualenv venv`
 * Activate venv: `$ source venv/bin/activate`
 * Change its file permission: `$ sudo chmod -R 777 venv`
 * Install Flask inside venv: `$ sudo pip install Flask`
 * Install `sqlalchemy`, `requests` and `httplib2` dependencies, using target flag `-t`, inside `/var/www/catalog/catalog/venv/lib/python2.7/site-packages`. Example, for sqlalchemy, run: `sudo pip install sqlalchemy -t /var/www/catalog/catalog/venv/lib/python2.7/site-packages`. Without the -t flag, these wwould be installed within the system (`/usr/local/lib/python2.7/site-packages`) instead of the virtual environment.
 * To deactivate venv: `$ deactivate`

##### Enable Virtual Host
 * Create new conf file for the app: `$ sudo nano /etc/apache2/sites-available/catalog.conf` to contain these configurations below:
```
<VirtualHost *:80>
    ServerName http://ec2-52-33-152-154.us-west-2.compute.amazonaws.com/
    ServerAlias 52.33.152.154
    ServerAdmin grader@52.33.152.154
    WSGIScriptAlias / /var/www/catalog/app.wsgi
    <Directory /var/www/catalog/catalog/>
      Order allow,deny
      Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
      Order allow,deny
      Allow from all
    </Directory>
    Alias /uploads /var/www/catalog/catalog/uploads
    <Directory /var/www/catalog/catalog/uploads/>
      Order allow,deny
      Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
 * Reload apache2 server: `$ sudo service apache2 reload`

##### Set Up PostgreSQL
 * Download PostgreSQL: `$ sudo apt-get install postgresql postgresql-contrib`
 * Switch to **postgres** user: `$ sudo su - postgres` (upon installation, PostgreSQL creates a linux user **postgres** which is used to access the system)
 * Check to see if remote connections are disabled to prevent one type of potential attacks. Remote connections are disallowed by default when installing PostgreSQL from Ubuntu repositories — `$ sudo cat /etc/postgresql/9.1/main/pg_hba.conf`
 * Enter psql: `$ psql`
 * Create **catalog** user: `$ CREATE ROLE catalog WITH LOGIN` (To explicitly allow login when a role is created, use CREATE ROLE role_name WITH LOGIN otherwise login is disallowed by default.)
 * Create a database named **catalog** with owner **catalog**: `$ CREATE DATABASE catalog WITH OWNER catalog`

##### App Engine Credentials
 * Add `http://ec2-52-33-154-152.us-west-2.compute.amazonaws.com` and `http://52.33.152.154` to App Engine credentials so that Google Connect API can recognise these urls for oauth logins.

##### Protect SSH with Fail2Ban
Since ssh is exposed to the internet, it is also opened to attacks. Fail2Ban mitigates this by automatically creating rules that alter your iptables firewall configuration based on a predefined number of unsuccessful login attempts.
 * `$ sudo apt-get update`
 * `$ sudo apt-get install fail2ban`
 * Fail2ban's default configuration file is in `etc/fail2ban/jail.conf`. Since this file may be modified with package upgrades, we need to make and use a copy of it: `$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
 * Install iptables-persistent which allows the server to automatically set up the firewall rules at boot: `$ sudo apt-get install nginx sendmail iptables-persistent`
 * Inside jail.local, change to `action = %(action_mwl)s` value for a whois report and relevant log lines to be emailed. Also, change from `port = ssh` to `port = 2200` (under `[ssh]`) to reflect out change in port inside `/etc/ssh/sshd_config`.

##### Using Cron for Automatic Package Updates
 * Install cron: `$ sudo apt-get update` then `$ sudo apt-get install cron`
 * The user’s **crontab** holds the schedule of jobs to run. Each user’s crontab is located in `/var/spool/cron/crontab`. Do not edit this directly; instead, use `$ crontab -e` to edit.
 * Open user's crontab: `$ sudo crontab -e`
 * Save this job inside the crontab: `0 0 * * 0 grader (sudo apt-get update && sudo apt-get -y -d upgrade) > /dev/null`. The `-y` flag tells apt-get to answer ‘yes’ to every question. The `-d` flag tells apt-get to just download the packages but do not install. You can then install by manually running the `$ sudo apt-get dist-upgrade` and look out for any errors which may occur.


##### File Changes
These are changes to the app files to work on the new server environment.
 * (Inside `__init__.py` and `db_setup.py`) To connect to the server's `catalog` database, change from `engine = create_engine('sqlite:///events.db')` to `engine = create_engine('postgresql://catalog:DB_PASSWORD@localhost/catalog')`. The syntax is `create_engine(postgresql://USERNAME:DB_PASSWORD@localhost/DB_NAME)`.
 * Update to absolute path inside **__init__.py** to reflect the file system of the apache server's directory: `CLIENT_ID = json.loads(open('/var/www/catalog/catalog/client_secrets.json', 'r').read())['web']['client_id']`
 * The app's image uploads are stored within `/var/www/catalog/catalog/uploads/`. Change permission of this folder: `$ sudo chmod 777 /var/www/catalog/catalog/uploads`


 ##### Disallow Root Login
 * Update to new value `PermitRootLogin no` inside `/etc/ssh/sshd_config`

### Resources
   - https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-14-04
   - https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04
   - http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html
   - https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
   - http://killtheyak.com/use-postgresql-with-django-flask/
   - https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04
   - https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
   - http://blog.trackets.com/2013/08/19/postgresql-basics-by-example.html
   - https://github.com/stueken/FSND-P5_Linux-Server-Configuration
   - https://code.google.com/p/modwsgi/wiki/ConfigurationDirectives#WSGIScriptAlias
   - http://werkzeug.pocoo.org/docs/0.11/deployment/mod_wsgi/
   - http://flask.pocoo.org/docs/0.10/deploying/

