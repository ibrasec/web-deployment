# web-deployment
web-deployment is the final project in the Full Stack udacity nanodegree program, 
which is an ilustration of how i deployed my item-catalog repository into a linux machine

Note: The Droplet has been destroyed/Deleted and it is not available. 
thus the ip address and the link is no more accessable.

## Server preparation
i have used DigitalOcean droplet (Virtual mahcine) to host my repository,
here are the necessary information to access the droplet web-age:
- IP address: 68.183.70.105 ()
- URL: http://itemcatalog.68.183.70.105.xip.io/
- Linux Distr: Ubuntu-16.04 LTS

**Note: It is recommended to access the page using the given URL instead of directly using the Public IP address, this is due to the fact that the URL is added as a verified domain to use google oauth2, wherease the public ip address is not**

## summary of configurations made
## 1 - Server security and preparation
At the very begining the server must be secured since it is going to be accessed by anyone in the world, 
so the below steps has been considered before moving forward:

**1.1 Update the Server**

Every system in the world have to be updated periodically because old machines are having vulnerabilities that are now been fixed through security patches,so the first step to secure the server is by making it up to date, this is not going to completly secure the server, but it is going to decrease the number of threats possible on older versions of the current installed distribution:
```
sudo apt-get update
sudo apt-get upgrade
sudo apt autoremove
```
**1.2 change the default ssh port**

There are some ports that are delicious to the attackes, or they are considered as the first items in their Attack list, SSH port (port 22) is one of them, the ssh port for the currently working virtual machine was set to 2200 instead of 22.
The below code was executed:
```
$ sudo vi /etc/ssh/sshd_config
# What ports, IPs and protocols we listen for
 Port 2200
``` 
Then the the ssh deamon was restarted
```
$ sudo service sshd restart
```



**1.3 creat grader user**

To create a user, adduser command was executed, and the password was set accordingly.
```
sudo adduser grader
```

**1.4 Add grader to sudoers**

Because in the /etc/sudoers file there are the below two comments:
```
Please consider adding local content in /etc/sudoers.d/ instead of
directly modifying this file.
```
A grader file was created for udacity grader to be able to login and access the system, here are the steps used to do it:
```
$ sudo touch /etc/sudoers.d/grader
```
Then the following was added to the grader file using vi tool
```
$ sudo vi /etc/sudoers.d/grader
grader ALL=(ALL) NOPASSWD:ALL
```


**1.5 key based authentication**

Adding a user is not enough to secure the account, thus a key was generated on my local machine and the public key was copied to the server, wherase the private key is intended to be shared with the grader - in a private channel - for him to be able to access the droplet.
No passphrase was configured for simplicty

**NOTE: Done on the local machine not on the Server**
```
$ ssh-keygen -t rsa
        Generating public/private rsa key pair.
        Enter file in which to save the key (/home/ibrahim/.ssh/id_rsa): udacityvm_key
        Enter passphrase (empty for no passphrase): 
        Enter same passphrase again: 

```
on the droplet, the public key was copied inside the grader .ssh file /home/grader/.ssh/authorized_keys through the following steps:
- The contents of the udacityvm_key.pub file was copied
- Accessed the droplet using the grader username and password through 
```
ssh grader@68.183.70.105
```
- Then the below was executed
```
$ sudo mkdir /home/grader/.ssh
$ sudo nano /home/grader/.ssh/authorized_keys
```
Then, the contents of the udacityvm_key.pub was pasted in this file.
- After that, file restriction was done using file permision restriction chmod command as follows: 
```
$ sudo chmod 700 .ssh 
$ sudo chmod 644 .ssh/authorized_keys
```
Testing ssh was successful using the private key through ssh -i command
```
$ ssh grader@68.183.70.105 -p 2200 -i ./udacityvm_key
```

**1.6 Restrict key pair authentication**

Note: if you are following through the steps, make sure you were able to access the server using the below command before proceeding, otherwise you might find yourself unable to login to your server.
```
$ ssh grader@<you ip address > -p 2200 -i ./udacityvm_key
before proceeding, otherwise you might find yourself unable to login to your server.
```
The sshd_config file was updated to disable Password Authentication
```
$ sudo nano /etc/ssh/sshd_config
PasswordAuthentication no
```
Then the the ssh deamon was restarted
```
$ sudo service sshd restart
```

**1.7 Disable root login**

The sshd_config file was updated to disable root login
```
$ sudo nano /etc/ssh/sshd_config
PermitRootLogin no 
```
Then the the ssh deamon was restarted
```
$ sudo service sshd restart
```

**1.8 Enable Firewall UFW**

To prevent access to Services other than SSH,HTTP and NTP, the below code for enabling the firewall was executed.
**Note: Double check that you can access the server through SSH port 2200 before proceeding with the below code.**
```
$ sudo ufw default deny incoming
$ sudo ufw default allow outgoing
$ sudo ufw allow 2200/tcp
$ sudo ufw allow ntp
$ sudo ufw allow www
$ sudo ufw enable
```
checking the firewall status using sudo ufw status was OK
```
grader@udacity-vm:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
2200/tcp                   ALLOW       Anywhere                  
123                        ALLOW       Anywhere                  
80/tcp                     ALLOW       Anywhere                  
2200/tcp (v6)              ALLOW       Anywhere (v6)             
123 (v6)                   ALLOW       Anywhere (v6)             
80/tcp (v6)                ALLOW       Anywhere (v6)  
```

## 2- Apache2 Server Installation
For the Server to handle Websites, apache2 was the deamon service selected to host item-catalog repository,
to install apache2 service, the below code was executed:

```
grader@udacity-vm:~$ sudo apt-get install python-minimal
```
Since the item-catalog Repository was built using Flask, an interface with apache is essential for apache server to support such framework, thus wsgi (Web Server Gateway Interface) was installed using the below command
```
grader@udacity-vm:~$ sudo apt-get install libapache2-mod-wsgi
```
For this step, it is just enough to create a new directory and name it as FlaskApp
```
grader@udacity-vm:~$ sudo mkdir /var/www/FlaskApp
```
This directory was intended to have 3 main files for our website
- FlaskApp: which will contain our item-catalog repository files (python, html, images..etc)
- flaskapp.wsgi: which is a python code to call our main repository application
- flasnv: which is the virutalenv directory that will host our virtual environment didicated for this site
More information will be described below


## 3- python installation

Since my item-catalog repository was written in python 2.7,
python 2.7 was installed on the server
```
grader@udacity-vm:~$ sudo apt-get install python-minimal
```
To install python packages using pip,
pip2 was also installed on the server
```
grader@udacity-vm:~$ sudo apt-get install python-pip`

```

## 4- postgreSQL Installation

The server was intended to also work as a local database server, 
and it was recommended to install PostgreSQL. hence postgresql was installed
and a database named as 'catalog' with the following username and password was created:
```
grader@udacity-vm:~$ sudo apt-get install postgresql
grader@udacity-vm:~$ sudo -u postgres psql
postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER codeuser WITH ENCRYPTED PASSWORD 'code_udacity';
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO codeuser;
postgres=# \q
grader@udacity-vm:~$
```


## 5- virutalenv installation
virtualenv provides the ability to use more than one environment to install different versions of the same python package, i used virtualenv to provide future support of multiple sites each with his own independent python packages
The installation of virutalenv was simply done by using pip as follows:
```
grader@udacity-vm:~$ sudo pip install virtualenv 
```
From the /var/www/FlaskApp directory that we have created, 
we created our new virtual environment "flasknv".
```
grader@udacity-vm:/var/www/FlaskApp$ sudo virtualenv flasknv

```
After getting inside this virtualenv "flasknv", the permission was set to 755 in order be able to install the following python packages.

**Note: If you following up with the below steps make sure not to use``` sudo pip ```command to do python installation, otherwise python packages will be installed out of the virutalenv**
```
grader@udacity-vm:/var/www/FlaskApp$ source flaskenv/bin/activate
(flasknv) grader@udacity-vm:/var/www/FlaskApp$ sudo chmod 755 /var/www/FlaskApp/flasknv/

(flasknv) grader@udacity-vm:/var/www/FlaskApp$ pip install flask packaging oauth2client redis passlib flask-httpauth
(flasknv) grader@udacity-vm:/var/www/FlaskApp$ pip install sqlalchemy flask-sqlalchemy psycopg2-binary bleach requests

```
Up to this step it was possible to call flask library from python interpretter inside the virtualenv "flasknv" but not outside of it, hwich is what we want.
```
(flasknv) grader@udacity-vm:/var/www/FlaskApp$ python
Python 2.7.12 (default, Nov 12 2018, 14:36:49) 
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import flask
>>>
>>> exit()
(flasknv) grader@udacity-vm:/var/www/FlaskApp$ deactivate
grader@udacity-vm:/var/www/FlaskApp$ python
Python 2.7.12 (default, Nov 12 2018, 14:36:49) 
[GCC 5.4.0 20160609] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import flask
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ImportError: No module named flask
>>> 
```

## 6- Clonning and preparing repository 
Now comes the great part, from outside of the virutalenv;
the repository was cloned using the already installed git command
```
grader@udacity-vm:~$ git clone https://github.com/ibrasec/item-catalog-vm
```
Due to the nature of this repositroy - it was mainly designed to be a ready vagrant machine - only the catalog directory and its contents were copied to the new location where the fun is going to run ( inside the /var/www/FlaskApp ).
So we go to the /var/www/FlaskApp and make a new directory called FlaskApp Just for simplicity
```
grader@udacity-vm:~$ cd /var/www/FlaskApp
grader@udacity-vm:/var/www/FlaskApp$ mdkdir FlaskApp
grader@udacity-vm:/var/www/FlaskApp$ cd FlaskApp
grader@udacity-vm:/var/www/FlaskApp/FlaskApp$ 
```
from the FlaskApp directroy we copy all the repository's catalog contents into the current directory as follows
```
grader@udacity-vm:/var/www/FlaskApp/FlaskApp$ sudo cp -R /home/grader/item-catalog-vm/vagrant/catalog/* .
```
Now, in order for the web users to Upload images to the site through web forms,
the img directory inside the static folder has to be set to 777
```
 grader@udacity-vm:/var/www/FlaskApp/FlaskApp$ sudo chmod 777 -R static/img
```
Also the main application that runs item-catalog was set to be named as application.py,
for sake of compatibility with flaskapp.wsgi it was changed it to be named as __init__.py 
```
 grader@udacity-vm:/var/www/FlaskApp/FlaskApp$ mv application.py __init__.py
```
it was also necessasry to change the code in 3 main python files to allow using postgresql to access the local database instead of sqlite, so the following was also changed
- \__init\__.py
```
#engine = create_engine('sqlite:///catalog.db',
#                       connect_args={'check_same_thread': False})
# un-comment the below to use postgres instead of sqlite
# update the username, password and the database name accordingly
engine = create_engine('postgresql://codeuser:code_udacity@localhost:5432/catalog')
```
- models.py
```
#engine = create_engine('sqlite:///catalog.db',
#                       connect_args={'check_same_thread': False})
# un-comment the below code if You are using postgresql and not sqlite,
# be aware of the username and password
engine = create_engine('postgresql://codeuser:code_udacity@localhost:5432/catalog')
```
- load_catagories.py
```
# comment the below to use postgresql instead of sqlite
#engine = create_engine('sqlite:///catalog.db',
#                       connect_args={'check_same_thread': False})
# un-comment the below to use postgres instead of sqlite
# update the username, password and the database name accordingly
engine = create_engine('postgresql://codeuser:code_udacity@localhost:5432/catalog')
```
## 7- wsgi preparation
a file named as flaskapp.wsgi was created inside the /var/www/FlaskApp directory,
```
grader@udacity-vm:/var/www/FlaskApp$ sudo touch flaskapp.wsgi  
```
and in order for this file to call our item-catalog main code "\__init\__.py";
the following code was added
```
grader@udacity-vm:/var/www/FlaskApp$ cat flaskapp.wsgi 
#!/usr/bin/python
import sys
import random, string
from FlaskApp import app as application
application.secret_key = ''.join(random.choice(
                string.ascii_uppercase + string.digits) for x in xrange(32))
```
After Adding the flaskapp.wsgi file, apache2ctrl restart command was executed
```
grader@udacity-vm:~$ sudo apache2ctl restart
```

## 8- apache2 site configuration 
Inside the /etc/apache2/sites-available directory, a file named as FlaskApp.conf was created,
```
grader@udacity-vm:/var/wwww/FlaskApp$ cd /etc/apache2/sites-available
grader@udacity-vm:/etc/apache2/sites-available$ sudo touch FlaskApp.conf
```
this file includes the following code to call and execute flaskapp.wsgi script that was previously added.
```
grader@udacity-vm:/etc/apache2/sites-available$ cat FlaskApp.conf 
<VirtualHost *:80>
		    ServerName 68.183.70.105
		    ServerAdmin admin@mywebsite.com
		    WSGIDaemonProcess FlaskApp python-path=/var/www/FlaskApp/:/var/www/FlaskApp/flasknv/local/lib/python2.7/site-packages/
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
		    Alias /templates /var/www/FlaskApp/FlaskApp/templates
		    <Directory /var/www/FlaskApp/FlaskApp/templates/>
                         Order allow,deny
                         Allow from all
                    </Directory>
		    ErrorLog ${APACHE_LOG_DIR}/error.log
		    LogLevel warn
		    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>

```
After the previouse file was created, we have to enable that site using the "a2ensite" command as follows:
```
grader@udacity-vm:/etc/apache2/sites-available$ sudo a2ensite FlaskApp.conf
```
Then restart apache service as follows:
```
grader@udacity-vm:/etc/apache2/sites-available$ sudo apache2 restart
```

Now Every thing is properly setup and you can login simply by accessing the below link:
http://itemcatalog.68.183.70.105.xip.io/

## summary of software/packages installed
- finger
- python-minimal
- apache2
- python-pip
- libapache2-mod-wsgi
- postgresql
- virtualenv
The below were instaled from within the flasknv virtualenv
- flask
- packaging
- oauth2client
- redis
- passlib
- flask-httpauth
- sqlalchemy
- flask-sqlalchemy
- psycopg2-binary
- bleach
- requests
