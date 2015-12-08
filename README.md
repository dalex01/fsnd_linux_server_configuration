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

TBD

## Configuration changes made

##### 1. Create grader user
```
sudo adduser grader
```

##### 2. Update all currently installed packages:
```
sudo apt-get update
sudo apt-get upgrade
```
##### 3. Configure the local timezone to UTC

- Verify that timezone is UTC
- If it is not equal to UTC change it
```
cat /etc/timezone
cd /etc
rm localtime
ln -s /usr/share/zoneinfo/UTC localtime
```
##### 4. Change the SSH port from 22 to 2200
- in file `/etc/ssh/sshd_config` edit the line which states 'Port 22'
- switch over to the new port by restarting SSH
- verify SSH is listening on the new port by connecting to it with option `-p 2200`
```
sudo nano /etc/ssh/sshd_config
sudo restart ssh
ssh -i ~/.ssh/udacity_key.rsa root@52.34.190.50 -p 2200
```
##### 5. Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
```
##### 6. Install and configure Apache to serve a Python mod_wsgi application
##### 7. Install and configure PostgreSQL
##### 8. Install git
##### 9. Clone and set up my Restaurant Menu App project

## Third-party resources used of to complete this project

[How To: 2 Methods To Change TimeZone in Linux](http://www.thegeekstuff.com/2010/09/change-timezone-in-linux/)
[SSH/OpenSSH/Configuring](https://help.ubuntu.com/community/SSH/OpenSSH/Configuring)