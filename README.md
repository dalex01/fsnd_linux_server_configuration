# Restaurant Menu App on Configured Web Server

This app was created as [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299) course final project development.
For more information about Restaurant Menu App you can see this [repo](https://github.com/dalex01/fsnd_restaurants)

## Requirements

Project was developed and reviewed according to this [rubric](http://i.imgur.com/XbQyHSL.png).

## How to use

- Server IP: 52.34.190.50
- SSH port: 2200
- URL to application:

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
```

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
sudo mkdir app
cd app
git clone https://github.com/dalex01/fsnd_restaurants.git
cd fsnd_restaurants
sudo rm README.md
sudo mv * ..
cd ..
sudo rm -rf fsnd_restaurants
```

## Third-party resources used of to complete this project

- [How To: 2 Methods To Change TimeZone in Linux](http://www.thegeekstuff.com/2010/09/change-timezone-in-linux/)
- [SSH/OpenSSH/Configuring](https://help.ubuntu.com/community/SSH/OpenSSH/Configuring)
- [Error message when I run sudo: unable to resolve host (none)](http://askubuntu.com/questions/59458/error-message-when-i-run-sudo-unable-to-resolve-host-none)
- [Problem with restarting Apache2](http://askubuntu.com/questions/329323/problem-with-restarting-apache2)
- [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)
- [psql: FATAL: Peer authentication failed for user “dev”](http://stackoverflow.com/questions/17443379/psql-fatal-peer-authentication-failed-for-user-dev)
- [Connect to PostgreSQL from the command line](https://www.a2hosting.com/kb/developer-corner/postgresql/connect-to-postgresql-from-the-command-line)
- [How To Install and Use PostgreSQL on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04)

All other instructions was taken from [Configuring Linux Web Servers](https://www.udacity.com/course/configuring-linux-web-servers--ud299) Udacity course.