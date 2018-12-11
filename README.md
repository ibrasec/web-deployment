# web-deployment
An ilustration of how to deploy my item-catalog repository into a linux machine

# Server preparation
i have used DigitalOcean droplet (Virtual mahcine) to host my repository
here are the necessary information to access the droplet web-age:
- IP address: 68.183.70.105
- URL: http://itemcatalog.68.183.70.105.xip.io/
- Linux Distr: Ubuntu-16.04 LTS

# summary of configurations made
## 1 - Securing the Server
At the very begining the server must be secured since it is going to be accessed by anyone in the world, so the below steps has to be considered before moving forward

**1.a Update the Server**

Every system in the world have to be updated periodically because old machines are having vulnerabilities that are now been fixed through security patches,so the first step to secure the server is by making it up to date, this is not going to completly secure the server, but it is going to decrease the number of threats possible on older versions of the current installed distribution:
```
sudo apt-get update
sudo apt-get upgrade
sudo apt autoremove
```
**1.b change the default ssh port**
There are some ports that are delicios to the attackes, or they are considered as the first items in their Attack list, SSH port (port 22) is one of them, the ssh port for the currently working virtual machine was set to 2200 instead of 22.
The below steps was executed:
```
$ sudo vi /etc/ssh/sshd_config
# What ports, IPs and protocols we listen for
 Port 2200
```

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
