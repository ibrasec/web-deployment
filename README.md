# web-deployment
An ilustration of how i deployed my item-catalog repository into a linux machine

# Server preparation
i have used DigitalOcean droplet (Virtual mahcine) to host my repository
here are the necessary information to access the droplet web-age:
- IP address: 68.183.70.105
- URL: http://itemcatalog.68.183.70.105.xip.io/
- Linux Distr: Ubuntu-16.04 LTS

# summary of configurations made
## 1 - Securing the Server
At the very begining the server must be secured since it is going to be accessed by anyone in the world, so the below steps has been considered before moving forward

**1.a Update the Server**

Every system in the world have to be updated periodically because old machines are having vulnerabilities that are now been fixed through security patches,so the first step to secure the server is by making it up to date, this is not going to completly secure the server, but it is going to decrease the number of threats possible on older versions of the current installed distribution:
```
sudo apt-get update
sudo apt-get upgrade
sudo apt autoremove
```
**1.b change the default ssh port**

There are some ports that are delicious to the attackes, or they are considered as the first items in their Attack list, SSH port (port 22) is one of them, the ssh port for the currently working virtual machine was set to 2200 instead of 22.
The below code was executed:
```
$ sudo vi /etc/ssh/sshd_config
# What ports, IPs and protocols we listen for
 Port 2200
$ sudo service sshd restart
```

**1.c creat grader user**

To create a user, adduser command was executed, adn the password was set accordingly.
```
sudo adduser grader
```

**1.d Add grader to sudoers**

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


**1.d key based authentication **

Since adding a user is not enough to secure the account, thus a key was generated on my local machine and the public key was copied to the server, wherase the private key is intended to be shared with the grader - in a private channel - for him to be able to access the droplet.
No passphrase was configured for simplicty
**NOTE: Done on the local machine not on the Server**
```
$ ssh-keygen -t rsa
        Generating public/private rsa key pair.
        Enter file in which to save the key (/home/ibrahim/.ssh/id_rsa): udacityvm_key
        Enter passphrase (empty for no passphrase): 
        Enter same passphrase again: 

```
on the droplet, the public key was copied inside a file ./ssh/authorized_keys
- The contents of the udacityvm_key.pub file was copied
- Accessed the droplet using the grader username and password
- Then the below was executed
```
$ sudo mkdir .ssh
$ sudo nano authorized_keys
```
Then, the contents of the udacityvm_key.pub was pasted in this file.
After that, file restriction was done using file permision restriction chmod command as follows: 
```
chmod 700 .ssh 
chmod 644 .ssh/authorized_keys
```
Testing ssh was successful using ssh -i command
```
$ ssh grader@68.183.70.105 -p 2200 -i ./udacityvm_key
```



Then the permission level was set
To force all users to use key pair authentication and not username and password authentiation
- edit the sshd_config 
sudo nano /etc/ssh/sshd_config
PasswordAuthentication no


**1.c remote login disabled**
$ sudo vi /etc/ssh/sshd_config
PermitRootLogin no 

**1.d remote login disabled**




- summary of software installed
- list of third-party resources used.



## item-catalog Repository
- copy old FlaskApp/FlaskApp files
  git clone https://github.com/ibrasec/item-catalog-vm
- copy the new repository to the an Empty FlaskApp file
    ```
    grader@udacity-vm:/var/www/FlaskApp$ sudo mkdir FlaskApp
    grader@udacity-vm:/var/www/FlaskApp$ cd FlaskApp
    grader@udacity-vm:/var/www/FlaskApp/FlaskApp$ sudo cp -R /home/grader/updateditemcatalog/item-catalog-vm/vagrant/catalog/* .
    ```
- change the permision of the img file inside the static folder to be accissble by any one
    grader@udacity-vm:/var/www/FlaskApp/FlaskApp$ sudo chmod 777 -R static/img

- change application.py to __init__.py
    grader@udacity-vm:/var/www/FlaskApp/FlaskApp$ sudo mv application.py __init__.py
- comment sqlite , change username , password inside the __init__.py, model.py and load_catagories.py
