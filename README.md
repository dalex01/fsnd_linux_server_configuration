# Restaurant Menu App on Configured Web Server

This app was created as [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299) course final project development.
For more information about Restaurant Menu App you can see this [repo](https://github.com/dalex01/fsnd_restaurants)

## Requirements

Project was developed and reviewed according to this [rubric](http://i.imgur.com/XbQyHSL.png).

## How to use

- Server IP: 52.34.190.50
- SSH port: 2200
- URL to application: [http://ec2-52-34-190-50.us-west-2.compute.amazonaws.com/](http://ec2-52-34-190-50.us-west-2.compute.amazonaws.com/)

## Software installed on server

- apache2
- libapache2-mod-wsgi
- postgresql
- postgresql-server-dev-9.3
- git
- python
	- pip
	- flask
	- sqlalchemy
	- oauth2client
	- psycopg2
	- Flask-SeaSurf
	- glances
- fail2ban

## Configuration changes made

As we work under root in items 1-3 we don't use sudo before commands.

##### 1. Create `grader` user
```
adduser grader
nano /etc/sudoers.d/grader
```

##### 2. Give the `grader` the permission to `sudo`
```
nano /etc/sudoers.d/grader
nano /etc/hosts
```

Line `grader ALL=(ALL) NOPASSWD:ALL` should be added into `/etc/sudoers.d/grader` file.
To resolve problem `unable to resolve host ip-10-20-7-186` line `127.0.1.1 ip-10-20-7-186` should be added in `/etc/hosts` file.

##### 3. Give grader access via ssh and remove access from root user
```
mkdir /home/grader/.ssh
cp ~/.ssh/authorized_keys /home/grader/.ssh/authorized_keys
chgrp grader /home/grader/.ssh/authorized_keys
chown grader /home/grader/.ssh/authorized_keys
rm ~/.ssh/authorized_keys
rmdir .ssh
nano /etc/ssh/sshd_config
```

During editing `/etc/ssh/sshd_config` line `PermitRootLogin without-password` should be changed on `PermitRootLogin no`

Now we can access server by grader user via ssh with command (later, when we change ssh port, we will add `-p 2200` parameter to this command):
```
ssh -i ~/.ssh/udacity_key.rsa grader@52.34.190.50
```
All further commands we will execute under grader user (using sudo before command)

##### 4. Update all currently installed packages:
```
sudo apt-get update
sudo apt-get upgrade
```

Also information from this link [AutomaticSecurityUpdates](https://help.ubuntu.com/community/AutomaticSecurityUpdates) can be (but was not) applyed.

##### 5. Change the SSH port from 22 to 2200
- in file `/etc/ssh/sshd_config` edit the line which states 'Port 22'
- switch over to the new port by restarting SSH
- verify SSH is listening on the new port by connecting to it with option `-p 2200`
```
sudo nano /etc/ssh/sshd_config
sudo restart ssh
ssh -i ~/.ssh/udacity_key.rsa grader@52.34.190.50 -p 2200
```

##### 6. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw allow ntp
sudo ufw enable
```

##### 7. Configure the local timezone to UTC

- Verify that timezone is UTC
- If it is not equal to UTC change it
```
sudo cat /etc/timezone
cd /etc
sudo rm localtime
sudo ln -s /usr/share/zoneinfo/UTC localtime
```


##### 8. Install and configure Apache to serve a Python mod_wsgi application
```
sudo apt-get install apache2
sudo apt-get install libapache2-mod-wsgi
sudo nano /etc/apache2/sites-enabled/000-default.conf
sudo nano /etc/apache2/apache2.conf
sudo apache2ctl restart
```
During editing `000-default.conf` file we should add the following line at the end of the `<VirtualHost *:80>` block, right before the closing `</VirtualHost>` line: `WSGIScriptAlias / /var/www/html/myapp.wsgi`

To resolve problem `AH00558: apache2: Could not reliably determine the server's fully qualified domain name, using 127.0.1.1` we should add line `ServerName localhost` into file `/etc/apache2/apache2.conf`

##### 9. Install and configure PostgreSQL
```
sudo apt-get install postgresql
sudo apt-get install postgresql-server-dev-9.3
sudo nano /etc/postgresql/9.3/main/pg_hba.conf
sudo service postgresql restart
sudo -u postgres -i
psql
```

During modification of `/etc/postgresql/9.3/main/pg_hba.conf` we should change line:
```
# TYPE DATABASE USER ADDRESS METHOD
local  all      all          peer
```
to line:
```
# TYPE DATABASE USER ADDRESS METHOD
local  all      all          md5
```

After we enter in psql we should execute the following commands:
```
=> CREATE ROLE catalog PASSWORD 'catalog';
=> ALTER ROLE "catalog" WITH LOGIN;
=> CREATE DATABASE restaurantsdb OWNER catalog;
```

##### 10. Install git
```
sudo apt-get install git
```

##### 11. Install all required python modules
```
sudo apt-get install python-dev
sudo apt-get install python-pip
sudo pip install flask==0.9
sudo pip install Werkzeug==0.8.3
sudo pip install sqlalchemy
sudo pip install oauth2client
sudo pip install psycopg2
sudo pip install Flask-SeaSurf
```

##### 12. Clone and set up my Restaurant Menu App project
```
cd /var/www/html
sudo mkdir app
cd app
sudo touch .htaccess | echo "$RedirectMatch 404 /\.git" >> .htaccess
git clone https://github.com/dalex01/fsnd_restaurants.git
cd fsnd_restaurants
sudo rm README.md
sudo mv * ..
cd ..
sudo rm -rf fsnd_restaurants
sudo nano /etc/apache2/sites-available/app.conf
sudo service apache2 reload
sudo a2ensite app
sudo a2dissite 000-default.conf
sudo mv run.py app.wsgi
sudo nano app.wsgi
sudo nano restaurants/models.py
sudo nano restaurants/helpers.py
sudo service apache2 restart
```

The follwoing text should be entered into `/etc/apache2/sites-available/app.conf` file:
```
<VirtualHost *:80>
                ServerName 52.34.190.50
                ServerAdmin alexey.drozdov@gmail.com
                WSGIScriptAlias / /var/www/app/app.wsgi
                <Directory /var/www/app/restaurants/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /static /var/www/app/restaurants/static
                <Directory /var/www/app/restaurants/static/>
                        Order allow,deny
                        Allow from all
                </Directory>
                Alias /images /var/www/app/restaurants/static/images
                <Directory /var/www/catalog/catalog/uploads/>
                       Order allow,deny
                       Allow from all
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

The follwoing code should be added into `app.wsgi` file (previous content can be removed):
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/app/")

from restaurants import app as application

application.secret_key = 'super_super_secret_key'
```

In `restaurants/models.py` and `restaurants/helpers.py` files path to db should be changed on `engine = create_engine("postgresql+psycopg2://catalog:catalog@localhost/restaurantsdb")`

##### 13. Configure firewall to monitor for repeat unsuccessful login attempts
```
sudo apt-get install fail2ban
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```

We will use default configuration.

##### 14. Application monitoring
```
sudo pip install glances
```

Glances package is installed to monitor server health. To see monitoring dashboard just run `glances` from console
Also the following tools can be (but was not) used:
[Munin](https://www.digitalocean.com/community/tutorials/how-to-install-munin-on-an-ubuntu-vps)
[Nagios](https://www.digitalocean.com/community/tutorials/how-to-install-nagios-4-and-monitor-your-servers-on-ubuntu-14-04)
[Apache mod_status](https://www.digitalocean.com/community/tutorials/how-to-install-configure-and-use-modules-in-the-apache-web-server) (check the Status Module section)
[Monitorix](http://www.monitorix.org)

## Third-party resources used of to complete this project

- [How To: 2 Methods To Change TimeZone in Linux](http://www.thegeekstuff.com/2010/09/change-timezone-in-linux/)
- [SSH/OpenSSH/Configuring](https://help.ubuntu.com/community/SSH/OpenSSH/Configuring)
- [Error message when I run sudo: unable to resolve host (none)](http://askubuntu.com/questions/59458/error-message-when-i-run-sudo-unable-to-resolve-host-none)
- [Problem with restarting Apache2](http://askubuntu.com/questions/329323/problem-with-restarting-apache2)
- [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
- [How To Install and Use PostgreSQL on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04)
- [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- [psql: FATAL: Peer authentication failed for user “dev”](http://stackoverflow.com/questions/17443379/psql-fatal-peer-authentication-failed-for-user-dev)
- [Connect to PostgreSQL from the command line](https://www.a2hosting.com/kb/developer-corner/postgresql/connect-to-postgresql-from-the-command-line)
- [How do I change the root directory of an apache server?](http://stackoverflow.com/questions/5891802/how-do-i-change-the-root-directory-of-an-apache-server/23175981#23175981)
- [Glances 2.5.1](https://pypi.python.org/pypi/Glances)
- [How To Protect SSH with Fail2Ban on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04)
- [AutomaticSecurityUpdates](https://help.ubuntu.com/community/AutomaticSecurityUpdates)
- [Uptime](https://en.wikipedia.org/wiki/Uptime)
- [System_monitor](https://en.wikipedia.org/wiki/System_monitor)

All other instructions was taken from [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299) Udacity course.