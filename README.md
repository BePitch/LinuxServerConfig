# LinuxServerConfig

## Basic Info

* Public IP Address: 18.234.109.4
* Port: 2200
* Website: 
    * http://18.234.109.4.xip.io
    
* SSH Key: submitted in reviewer notes
* Steps [3,4,5,6,7,8] are on Lightsail Linux Config
* Steps [7] are on local Vagrant machine

## Step 1 & 2- Create Lightsail Server Instance and login via browser console
Created amazon lightsail instance via AWS using Ubuntu.

1. Create an account at https://lightsail.aws.amazon.com/ls/webapp/home
2. Follow instructions in Udacity Notes

Logged into the browser console and booted up in ssh via browser

## Step 3 - Update and Upgrade server
I updated all the packages on the server 
```ssh
$ sudo apt-get update
```
Next, I upgraded the packages
```ssh
$ sudo apt-get updgrade
```
## Step 4 - Change Port to 2200
 Per project notes, I changed the port from 22 to 2200 using the sudo command 'nano'. I changed it by opening the /etc/ssh/sshd_config file and scrolling down to the prot line and making the necessary changes then I Control-X and save to back out the file.
 ```ssh
$ sudo nano /etc/ssh/sshd_config
```
Note: Remember to add and save port 2200 with as TCP in your Amazon Lightsail instance console.

## Step 5 - Update Firewall
 Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH, HTTP, and NTP.
 ```ssh
$ sudo ufw allow ssh
$ sudo ufw allow www
$ sudo ufw allow ntp
$ sudo ufw allow 2200/tcp
$ sudo ufw allow 80/tcp
$ sudo ufw allow 123/udp
$ sudo ufw enable 
$ sudo ufw status
```
## Step 6 - Add User 
Created user Grader on lightsail instance
 ```ssh
$ sudo adduser grader
```
Edit sudoers file and add `grader ALL=(ALL:ALL) ALL`
 ```ssh
$ sudo nano /etc/sudoers.d/grader
```

## Step 7 - Create SSH login keys
#### contains tasks on both environments
Switching to my vagrant instance to create an SSH key
```ssh
$ mkdir .ssh
$ chown vagrant:vagrant /home/vagrant/.ssh
$ cd .ssh
/.ssh $ ssh-keygen
```
when prompted I named the key grader. This would generate two files grader and grader.pub

On lightsail instance
```ssh
$ mkdir .ssh
$ chown grader:grader /home/grader/.ssh
$ chmod 700 /home/grader/.ssh
```
Created a file called authorized_keys and copied over the ssh keygen contents from grader.pub file on vargant machine
```ssh
$ sudo nano /.ssh
$ sudo chmod 400 /.ssh/authorized_keys
```

I can now login to my server from the vagrant machine using 
```ssh
ssh -i [privatekeyfilename] grader@18.234.109.4 -p 2200
```
## Step 8 - Deploy project
Confirm timezone is UTC with `$ sudo dpkg-reconfigure tzdata`

### Install Apache, WSGI, Python
```ssh
$ sudo apt-get install apache2 libapache2-mod-wsgi python-setuptools
$ sudo service apache2 restart
```
### Install and Confirgue PostgreSQL
* Install PostgreSQL `sudo apt-get install postgresql`
* Login as user `sudo su - postgres`
* Go to shell `psql`
* Run `CREATE DATABASE catalog;`
* Run `CREATE USER catalog;`
* Set password `ALTER ROLE catalog WITH PASSWORD 'password';`
* Assign access to user `GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;`
* Quit and Exit `\q` then `exit`

### Install git, packages, clone and  modify files
1. Install Git `sudo apt-get install git`
2. move into var/www dir `cd /var/www`
3. create directory for repo clone `sudo mkdir FlaskApp` and cd into it `cd FlaskApp`
4. clone `git clone https://github.com/BePitch/OAuth2.0.git` and rename `sudo mv ./OAuth2.0 ./FlaskApp` then `cd FlaskApp`
5. rename  app python file `sudo mv project.py __init__.py` 
6. edit database_setup file with Postgres changes `engine = create_engine('postgresql://catalog:password@localhost/catalog')`
7. Install pip `sudo apt-get install python-pip`
8. install packages `sudo pip install sqlalchemy flask oauth2client passlib requests psycopg2`
9. Create database `sudo python database_setup.py`
10. Insert data `sudo python lotsofsoftware.py`

### Configure Virtual Host
1. Create File: `sudo nano /etc/apache2/sites-available/FlaskApp.conf`
2. Add this to the file
```ssh
<VirtualHost *:80>
	ServerName 18.234.109.4
	ServerAdmin kingpitch@gmail.com
    ServerAlias 18.234.109.4.xip.io
    ServerAlias ec2-18-234-109-4.compute-1.amazonaws.com
	WSGIDaemonProcess FlaskApp python-path=/var/www/FlaskApp:/var/www/Flask$
    WSGIProcessGroup FlaskApp
    WSGIScriptAlias / /var/www/FlaskApp/flaskapp.wsgi
    <Directory /var/www/FlaskApp/FlaskApp/>
            Order allow,deny
            Allow from all
    </Directory>
    Alias /static /var/www/FlaskApp/FlaskApp/static
    <Directory /var/www/FlaskApp/FlaskApp/static/>
            Order allow,deny
            Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
3. Enable `sudo a2ensite FlaskApp`

### Create WSGI file
1. verify your directory is `cd /var/www/FlaskApp`
2. sudo nano flaskapp.wsgi
```ssh
#!/usr/bin/python
import sys
import logging

logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/FlaskApp/")

from FlaskApp import app as application
application.secret_key = 'super_secret_key'
```
3. `sudo service apache2 restart`

## Notes:
* Checking your error logs will be helpful using the tail 20 command.
* I was getting a No such file or directory: 'client_secrets.json' error. I fixed using a raw path to the init file `open(r'/var/www/catalog/catalog/client_secrets.json', 'r').read())...`


## References
1. https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
2. https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
3. Helped fix error client_secrets.json unknown https://github.com/sivcan/Linux-Server-Configuration
4. Udacity's FSND Forum
5. https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet#code
6. https://aws.amazon.com/documentation/lightsail/ 

