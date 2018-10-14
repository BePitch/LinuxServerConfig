# LinuxServerConfig

## Basic Info

* Public IP Address: 18.234.109.4
* Port: 2200
* Website: 
    * http://18.234.109.4.xip.io/login
    * http://ec2-18-234-109-4.compute-1.amazonaws.com/login
* SSH Key: submitted in reviewer notes
* Steps [] are on Lightsail Linux Config
* Steps [] are on local Vagrant machine

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
## Step 6 - Add User and create ssh login keys
Created user Grader
 ```ssh
$ sudo adduser grader
```
Edit sudoers file and add `grader ALL=(ALL:ALL) ALL`
 ```ssh
$ sudo nano /etc/sudoers.d/grader
```
## Step 7

## Step 8