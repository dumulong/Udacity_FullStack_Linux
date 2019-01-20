# Udacity_FullStack_Linux

repository for Udacity Full Stack - Linux project

## Server's address and port:

Server's IP: 3.89.214.99

SSH port: 2200

## Web application URI:

http://3.89.214.99/catalog

## Summary of software installed:

Linux packages:

- finger (optional)
- apache2
- postgresql
- git
- python2.7
- python-pip
- libapache2-mod-wsgi

Python packages (PIP):

- SQLAlchemy
- psycopg2
- flask
- oauth2client
- requests

## Configuration change:

Adding a new user (pickmeup)
```
sudo adduser grader
```
Copy the vagrant user to use for the student account...
```
sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
```
Now edit and change "ubuntu" to "grader"
```
sudo nano /etc/sudoers.d/grader
```

now, we need to be able to login in the "grader"'s account, so let temporarely set the ssh configuration to accept passwords:
```
sudo nano /etc/ssh/sshd_config
```
And change the line "PasswordAuthentication no" to "PasswordAuthentication yes"

use the console to reboot

now using the command prompt (enter password):
```
ssh grader@3.89.214.99 -p 22
```

Use Lightsail interface to upload the ssh key to AWS.

Then create a .ssh directory (in the home directory of that account)
```
mkdir .ssh
```

Then create a file that will contain all the keys:
```
touch .ssh/authorized_keys
```

Go back to your terminal and copy the content of the xxxx.pub file
The edit/copy the authorized_keys file:
```
nano .ssh/authorized_keys
```

Now set the permission:
```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```


now disable the password based authentication to only allow key based.
sudo nano /etc/ssh/sshd_config

And change the line "PasswordAuthentication yes" to "PasswordAuthentication no"

use the console to reboot

Now use the key to login:
```
ssh grader@3.89.214.99 -p 22 -i c:/users/denis/.ssh/linuxCourse
```

make sure you update the lightsail firewall by creating a custom ssh rule for TCP port 2200 (network tab of the instance).
While you are at it, add the custom UDP 123 (NTP)

Now edit again the sshd config file
```
sudo nano /etc/ssh/sshd_config
```

Uncomment port 22 and change to port 2200

use the console to reboot

You should be now able to ssh with port 2200:

```
ssh grader@3.89.214.99 -p 2200 -i c:/users/denis/.ssh/linuxCourse
```

Deny all incoming trafic, then open some ports
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ntp
sudo ufw allow 2200/tcp
sudo ufw allow www
sudo ufw enable
```

Now, time to stat with the website.  Install the following packages:

```
sudo apt-get install apache2
sudo apt-get install postgresql
sudo apt-get install git
sudo apt install python2.7 python-pip
sudo apt-get install libapache2-mod-wsgi
```

Refresh the packages repository
```
sudo apt update
```

Finally, go through with the updates:
```
sudo apt upgrade
```

Install the python packages needed for the catalog website:
```
pip install SQLAlchemy
pip install psycopg2
pip install flask
pip install oauth2client
pip install requests
```

Uses GIT to get the calalog project and copy it to **/var/www/html/catalog**

Create a new catalog database:
```
sudo -u postgres psql
postgres=# create database catalog;
postgres=# create user udacity with encrypted password 'n0pedy';
postgres=# grant all privileges on database catalog to udacity;
```
Then, create the tables and insert the data.

Also, we need to add our IP to the list of valid **Authorized JavaScript origins** from the google API website:

https://console.developers.google.com

NOTE: Sadly, an IP address can't be entered in the **Authorized redirect URIs**.  So, our login does not work but I believe that this course only request our website to be serving pages...

From there, the website should be able to run:
```
python application.py
```

We then bridge the apache server to our python app by using wsgi.  In order to do so, we've created a file in our directory **application.wsgi** with the following content:
```
import sys
sys.path.insert(0, '/var/www/html/catalog')

from application import app as application

application.secret_key = 'Add your secret key'
```
Then we need to configure apache to pick up that file.  We did that by modifying the file **/etc/apache2/sites-enabled/000-default.conf** to include the following code:
```
WSGIScriptAlias /catalog /var/www/html/catalog/application.wsgi

<Directory /var/www/html/catalog>
    WSGIProcessGroup catalog
    WSGIApplicationGroup %{GLOBAL}
    Order deny,allow
    Allow from all
</Directory>
```
Et Voila!

## Third party resources:

https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/

https://knowledge.udacity.com/questions/12307

https://knowledge.udacity.com/questions/14605

