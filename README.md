# Fullstack Nanodegree
## Linux Server Configuration
For this project, we are required to configure a Ubuntu server on Amazon Lightsail
and host our previous Item Catalog project on it.
### The IP address, SSH port and the complete URL to your hosted web application
 - 54.175.155.136
 - SSH port is 2200
 - http://54.175.155.136.xip.io/shop/

### A summary of software you installed and configuration changes made
#### Packages installed
- flask
- sqlalchemy
- oauth2client
- httplib2
- requests
- apache2
- psycopg2
- git
- pip
- virtualenv
- libpq-dev

### Configuration changes made
#### Get your server
For this project we followed the course instructions to set up an Amazon webservices instance.  
We have to `ssh` into our server from our terminal, so we have to download our private key.  
There will be a permissions error if we try to login straight away, we need to run `chmod 600 PATH_TO_PRIVATE_KEY`, or 400. [(source)](https://stackoverflow.com/questions/9270734/ssh-permissions-are-too-open-error)

#### Secure your server
After connecting, we run the commands shown in the course to upgrade our packages.  
`sudo apt-get update`  
`sudo apt-get upgrade`  
Next, we change the `ssh` port, which can be found in the `/etc/ssh/sshd_config`  
`nano` into it, change the `22` port to `2200` and restart ssh. [(source)](https://ie.godaddy.com/help/changing-the-ssh-port-for-your-linux-server-7306)  
Finally we configure our `UFW`, following the instructions in the course videos.  
Make sure it looks like this :  

`Status: active`  

|To                     |Action      |From
|---                    |------      |----
| 22                    |DENY        |Anywhere                  
| 2200/tcp              |ALLOW       |Anywhere                  
|80/tcp                 |ALLOW       |Anywhere                  
|123                    |ALLOW       |Anywhere                  
|22 (v6)                |DENY        |Anywhere (v6)             
|2200/tcp (v6)          |ALLOW       |Anywhere (v6)             
|80/tcp (v6)            |ALLOW       |Anywhere (v6)             
|123 (v6)               |ALLOW       |Anywhere (v6)

#### Give grader access

As per the course instructions, we create a user with username `grader`, password `grader`.  
We give the user `sudo` access by running `sudo visudo`, find the lines  

```
# User privilege specification
root    ALL=(ALL:ALL) ALL
```  
then we add  
`grader  ALL=(ALL:ALL) ALL`  
save and exit.

Then, we must make and `ssh` key using `ssh-keygen`.  
We copy the public key ( ending in `.pub`) and log in to our server.  
Make an `.ssh` directory in the grader user (`mkdir /home/grader/.ssh`).  
In there we create a file called `authorized_keys`, paste the public key in it and change the permissions like above (`chmod 600 PATH_TO_PRIVATE_KEY`).  

After that, the grader user can login using `ssh`, the `port number` and the path to the `private key`.  

#### Prepare to deploy your project

Time was UTC by default.  

Install and configure Apache to serve a Python mod_wsgi application:  

`sudo apt-get install apache2` and test by visiting our public IP, http://54.175.155.136/.  
The default apache html will appear. [(source)](https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-16-04)  
`sudo apt-get install libapache2-mod-wsgi` like explained on the course.  

Install and configure PostgreSQL:  

`sudo apt-get install postgresql` like explained on the course.  
Further configuration from this [source](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps).  
We need a database called `furniture` for our project.  
Install `git`:  
`sudo apt-get install git`

#### Deploy the Item Catalog project
There are a quite a few changes that needed to be made, found them with a lot of errors and googling.  
Error logs were seen from `/var/log/apache2/error.log`.  
After cloning the original project, i made a new branch called `linux` and commited the changes there.  
- Change the engine lines to `engine = create_engine('postgresql://catalog:catalog@localhost/furniture')`
- `app.run(host='0.0.0.0', port=5000)` change to `app.run()`
- Change the authorised URIs on the developers google and facebook pages
- Change the `path` of the `client_secrets.json` and `fb_client_secrets.json` where needed.
- rename `project.py` to `__init__.py`  

`sudo apt-get install python-virtualenv` and make a virtual environment called `env`  
`. env/bin/activate` and install all our dependencies named above.  
`deactivate`  
After that, we need to create a `.conf` file to serve our app. At `/etc/apache2/sites-available` we create `Udacity_Catalog.conf`.  
[(source)](http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/#working-with-virtual-environments)  
```
<VirtualHost *:80>
                ServerName 54.175.155.136
                ServerAdmin antonisdee@gmail.com
                WSGIScriptAlias / /var/www/Udacity_Catalog/Udacity_Catalog.wsgi
                <Directory /var/www/Udacity_Catalog/Udacity_Catalog/>
                        Order allow,deny
                        Allow from all
                        Options -Indexes
                </Directory>
                Alias /static /var/www/Udacity_Catalog/Udacity_Catalog/static
                <Directory /var/www/Udacity_Catalog/Udacity_Catalog/static/>
                        Order allow,deny
                        Allow from all
                        Options -Indexes
                </Directory>
                ErrorLog ${APACHE_LOG_DIR}/error.log
                LogLevel warn
                CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

Next is the .wsgi file:  
`/var/www/Udacity_Catalog` create `Udacity_Catalog.wsgi` and fill in:  
```
activate_this = '/var/www/Udacity_Catalog/Udacity_Catalog/env/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))

#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/Udacity_Catalog/")

from Udacity_Catalog import app as application
application.secret_key = 'super_secret_key'
```  

After that we disable the default Apache page, `sudo a2dissite 000-default.conf`, and our app is deployed.

#### lotsofproducts.py

The way the app is setup, we need to create one user first before loading the products.  
So, first sign in with a new account, to be assigned user id 1 .  
After that, go into the server, activate `env`, run `lotsofproducts.py` and `deactivate` the environment.  
The database should be populated and the rooms and products showing at the app.

#### Final Notes
I went ahead and made the project without documenting my every error encountered, google search and solutions.  
Sources included official documentation, stackoverflow, github and many more tutorials, even if i just rushed through them to the line required to fix one error at the time.  
Only later I saw the steps should be written here, and I did my best to replicate them.
There were many errors, countless google tabs, apache reloads/restarts, server reboots, database dropping, name changing etc.
