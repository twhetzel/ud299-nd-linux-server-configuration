# Linux Server Configuration
Set-up information for Udacity course ud299 on how to configure a Linux Server. Root login 
credentials for an EC2 instance on AWS are provided by Udacity.

## 1. Login as root to server
Go to https://www.udacity.com/account#!/development_environment to get your AWS IP address
and key. Login to the server as this root user using the key provided by Udacity.

## 2. Add a new user
- As root, type `sudo adduser grader`. Then follow prompts to add a password 
and name for this new account. 
- Confirm addition of the new user by typing `sudo /cat/etc/passwd`, 
the user grader should be listed in the output.

## 3. Give grader sudo permission
- As the root user, type `visudo`. 
- Reference Documentation: https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps

## 4. Set-up Key-based Authentication/Public Key Encryption
- Using ssh-keygen on your local machine, create a key for the user grader. <br>
Follow the prompts to provide a name for this key and use the default key location (~/.ssh). <br>
This process will create two keys on your local machine, the file with extension .pub is 
the public key to be transferred to the server.

## 5. Add Public Key to Server (EC2 instance)
- Login to the server as the new grader user.
- Run these commands in your home directory 
```
mkdir .ssh
touch .ssh/authorized_keys
``` 
- Copy-and-paste the contents of the .pub key file created on your local machine 
above to the server as the contents of the authorized_keys file.

## 6. Set-up Permissions on the .ssh file while logged in as grader
Run these commands:
```
chmod 700 .ssh
chmod 664 .ssh/authorized_keys
``` 
- Then from the local computer, login to server using an additional flag to indicate the key file:
`ssh student@127.0.0.1 -p 2222 –i ~/.ssh/linuxCourse`

## 7. Disable Password Based Login and Force Login using Key Pair 
- On the server, logged in as the student user, edit the sshd_config file, `sudo nano /etc/ssh/sshd_config` 
- Change the line with Password Authentication from yes to no
- This is read only when the service starts, so to restart the ssh service, `sudo service ssh restart`

## 8. Update all currently installed packages
- List all the packages to update 
`sudo apt-get update` 
- Update the packages  
`sudo apt-get upgrade` 

## 9. Change the SSH port from 22 to 2200
`nano /etc/ssh/sshd_config`
Edit the file to change the SSH port from 22 to 2200. <br>
Restart the server `service ssh restart`  <br>
** _Do not logout as root yet_ ** <br>
In a separate Terminal window, try to login as root using the new SSH port as:
`ssh -i ~/.ssh/udacity_key.rsa –p 2200 root@AWS_IP_ADDRESS` <br>
Reference Documentation: https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-12-04
and https://wiki.knownhost.com/security/misc/how-can-i-change-my-ssh-port
 
## 10. Configure Firewall to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
`sudo ufw status` → Inactive
`sudo ufw default deny incoming`
`sudo ufw default allow outgoing`
`sudo ufw allow ssh`
`sudo ufw allow 2200/tcp (Note: Changed SSH above to port 2200)`
`sudo ufw allow www`
`sudo ufw allow 123/udp`
`sudo ufw enable`
`sudo ufw status` → active

This will display the following status for the UFW
```
To                         Action      From
--                         ------      ----
22                         ALLOW       Anywhere
2200/tcp                   ALLOW       Anywhere
80/tcp                     ALLOW       Anywhere
123/udp                    ALLOW       Anywhere
22 (v6)                    ALLOW       Anywhere (v6)
2200/tcp (v6)              ALLOW       Anywhere (v6)
80/tcp (v6)                ALLOW       Anywhere (v6)
123/udp (v6)               ALLOW       Anywhere (v6)
```
<br><br> 
- Confirm that root can SSH and login from local computer, 
`ssh -i ~/.ssh/udacity_key.rsa -p 2200 root@AWS_IP_ADDRESS` <br>
If yes, hooray we can proceed. If not, repeat the steps above since you are locked out of the server.

## 11. Configure local Timezone to UTC
`dpkg-reconfigure tzdata`
- In the window that appears, use the arrow keys to Scroll to the bottom of 
the Continents list and select `Etc or None of the Above` and then in the 
second list, select UTC <br>
- Confirm time change by typing `date` on the command line <br>
- Reference Documentation: http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442 
and https://help.ubuntu.com/community/UbuntuTime 

## 12. Install and configure Apache to serve a Python mod_wsgi application
*Install Apache* 
`sudo apt-get install apache2` <br>
- Confirm successful installation by visiting http://52.24.160.178/ (URL from 
Udacity Environment information for AWS instance). It should say "It Works" and 
display other Apache information on the page.
<br><br>
*Install Python mod_wsgi* 
`sudo apt-get install libapache2-mod-wsgi` <br> <br>
*Install and Configure Demo WSGI app* 
`sudo nano /etc/apache2/sites-enabled/000-default.conf` <br>
- At the end of the <VirtualHost *:80> block, right before the closing </VirtualHost> 
add this line: `WSGIScriptAlias / /var/www/html/myapp.wsgi` <br> 
Restart Apache `sudo service apache2 restart`
- NOTE: After restart the Home page will return a 404, we’ll fix that next by 
configuring Apache to serve WSGI application

## 13. Configure Apache to serve basic WSGI application
- Create the /var/www/html/myapp.wsgi file using the command `sudo nano /var/www/html/myapp.wsgi`
- Within this file, write the following application:
```
def application(environ, start_response):     
	status = '200 OK'     
	output = 'Hello World!'      
	response_headers = [('Content-type', 'text/plain'), ('Content-Length', str(len(output)))]     start_response(status, response_headers)      
	return [output] 
```
- Refresh the page and the text in the script above will be displayed

## 14. Install and configure PostgreSQL
- Install PostgreSQL `sudo apt-get install postgresql postgresql-contrib`
- Check that no remote connections are allowed. `sudo less /etc/postgresql/9.3/main/pg_hba.conf`
By default, remote connectsions to the database
are disabled for security reasons when installing PostgreSQL from the Ubuntu repositories.<br> 
https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
- Documentation: https://help.ubuntu.com/community/PostgreSQL, 
https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04
- Basic server set-up `sudo -u postgres psql postgres`
- Set-up password for user postgres by typing `\password postgres` and enter password

## 15. Create a new user named _catalog_ that has limited permissions to the database
- Connect to database: `sudo –u postgres psql postgres` to connect as user postgres
- Type `psql` to generate PostgreSQL prompt
- Create new user `CREATE USER catalog WITH PASSWORD ‘your_passwd’;`
- Confirm that user was created `Check that user was created, type: \du`
- Documentation: : http://www.postgresql.org/docs/8.0/static/sql-createuser.html <br>
http://www.postgresql.org/docs/9.1/static/sql-createrole.html 

## 16. Limit permissions to new user catalog
- Run \du to see what permissions catalog has
- To see possible roles, type: `\h CREATE ROLE`
- Add permissions: <br>
`ALTER ROLE catalog WITH LOGIN;`
`ALTER USER catalog CREATEDB;`
- Create the database `CREATE DATABASE catalog WITH OWNER catalog;`
- Login to the database `\c catalog`
- Revoke all rights:
`# REVOKE ALL ON SCHEMA public FROM public;` 
- Grant only access to the catalog role:
`# GRANT ALL ON SCHEMA public TO catalog;` 
- Exit out of PostgreSQL and the postgres user:
`# \q, then $ exit`
- Restart postgresql: `sudo service postgresql restart`
- Documentation: https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps

## 17. Instal Git
- Install Git as: `sudo apt-get install git`
- Edit Git Configuration: <br>	
`git config --global user.name "Your Name"`
`git config --global user.email youremail@domain.com`
- Confirm by running `git config --list`
- Documentation: https://www.digitalocean.com/community/tutorials/how-to-install-git-on-ubuntu-14-04

## 18. Clone Project 3 App to this server
- Create a folder inside the /var/www folder called "catalog" and 
cd into this folder, see commands below. Remember this is a Python Flask app and not just html. 
```
cd /var/www 
sudo mkdir catalog 
cd catalog 
```

- Clone repo from Project 3 (item-catalog): 
`git clone https://github.com/twhetzel/item-catalog.git catalog`
The project is now at /var/www/catalog/catalog

- Make sure the .git directory is not publicly accessible via a browser
At the root of the web directory, add a .htaccess file and include this line: 
`RedirectMatch 404 /\.git`
- Reference Documentation: http://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible

## 19. Install Flask and create Virtual Environment for Catalog App
```
sudo apt-get install python-pip
sudo pip install virtualenv
```
- Give the following command (where venv is the name you would like to give your temporary environment):
`sudo virtualenv venv`

- Now, install Flask in that environment by activating the virtual environment 
with the following command:
`source venv/bin/activate`

- Give this command to install Flask inside:
`sudo pip install Flask`

- Run the following command to test if the installation is successful and the app is running:
`sudo python __init__.py` renamed project.py to __init__.py  <br>
It should display “Running on http://localhost:5000/” or "Running on http://127.0.0.1:5000/". 
If you see this message, you have successfully configured the app.

- To deactivate the environment, give the following command:
`deactivate`

- Configure and Enable the new Virtual Host
`sudo nano /etc/apache2/sites-available/DemoApp.conf`

- Add file contents for VirtualHost configuration, see Step 4 
here: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps 

- Enable the virtual host with the following command:
`sudo a2ensite DemoApp` followed by `service apache2 reload` following prompts

- Create the .wsgi file
`cd /var/www/DemoApp`
`nano demoapp.wsgi`
Add code per documentation to the file

- Restart Apache
`sudo service apache2 restart`

- Reference documentation: https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps

## 20. Add additional packages for the App
- Go to directory for app
```
cd /var/www/catalog/catalog
sudo apt-get install python-pip
sudo pip install virtualenv
sudo virtualenv venv
```
Activate environment, `source venv /bin/activate`

- Install App dependencies for Flask and Database
```
sudo pip install Flask
sudo pip install sqlalchemy
sudo pip install Flask-SQLAlchemy
sudo pip install psycopg2
sudo apt-get install python-psycopg2 
sudo pip install flask-seasurf
sudo pip install oauth2client
sudo pip install httplib2
sudo pip install requests

```

- Configure Virtual Host in Apache
`sudo nano /etc/apache2/sites-available/catalog.conf`

 Enable this Virtual Host
`sudo a2ensite catalog`
Prompted to run: service apache2 reload to activate the new configuration

- Configure the WSGI file
```
cd /var/www/catalog
sudo nano catalog.wsgi
```
- Add this to the file:
```
#!/usr/bin/python 
import sys 
import logging 

logging.basicConfig(stream=sys.stderr) 
sys.path.insert(0,"/var/www/catalog/")  

from catalog import app as application 
application.secret_key = 'super_secret_key'
```

- Restart Apache
`sudo service apache2 restart`

- Modify the database calls in the catalog app to use PostgreSQL vs. SQLite <br>
Edit these files: database_setup.py, project.py, and lotofevents-users.py
 
Remove:
`engine = create_engine(‘sqlite:///androidevents.db’, echo=True)`
Add: 
`engine = create_engine('postgresql://catalog:catalog_passwd@localhost/catalog')`

- Use the full path to client_secrets.json and fb_client_secrets.json in the project.py file
- Reference Documentation: http://docs.sqlalchemy.org/en/rel_1_0/core/engines.html#postgresql

- Get AWS URL to show the site:
- Add ec2 URL (w/o the http://) to the VirtualHost as the ServerAlias value /etc/apache2/sites-enabled/catalog.conf
`sudo cat /etc/hosts` and add ec2 URL (without http://) to file
- Remove default.conf and DemoApp.conf from being enabled 
```
sudo a2dissite 000-default.conf
sudo a2dissite DemoApp.conf
```
- Check what sites are enabled
`ls -alh /etc/apache2/sites-enabled/`
- Restart Apache `sudo service apache2 restart`
- Restart app: `sudo python __init__.py`

## 21. Update OAuth Information for Google+ and Facebook Logins
- Google+
  - Go to: https://console.developers.google.com/home/dashboard?project=udacity-1065 
  - Click on Enable and Manage APIs, then click on Credentials in the left-hand menu
  - Select Catalog App
  - Add URLs to 
  - Authorized Javascript origins, both local URL and EC2 version, 
  e.g. http://52.24.160.178 and http://ec2-52-24-160-178.us-west-2.compute.amazonaws.com/ 
  - Authorized redirect URIs, http://ec2-52-24-160-178.us-west-2.compute.amazonaws.com/login 
  and http://ec2-52-24-160-178.us-west-2.compute.amazonaws.com/gconnect 
  - NOTE: Needed to restart Apache and Python app to get it all working

  - Downgraded some packages to enable Google+ Login
```
pip install werkzeug==0.8.3 
pip install flask==0.9 
pip install Flask-Login==0.1.3
```
- Reference Documentation: https://discussions.udacity.com/t/oauth-course-google-sign-in-doesnt-work/15444


- Facebook
  - Go to https://developers.facebook.com/apps
  - Click on Android Events app
  - Click on Settings and navigate to Valid OAuth redirect URIs section
  - Add URLs for local and EC2 instance and then Save

=-=-=-=-=-=-=-=-=-=
Exceeds Criteria
## Add support for unattended package updates
`sudo apt-get install unattended-upgrades`
- Edit `/etc/apt/apt.conf.d/10periodic` and add this line: 
`APT::Periodic::Unattended-Upgrade "1";`   Also, change the AutocleanInterval from 0 to 7
- The results of unattended-upgrades will be logged to /var/log/unattended-upgrades. 
- Confirm by looking for file in this directory 
`sudo ls /var/log/unattended-upgrades`
- Reference Documentation: https://help.ubuntu.com/12.04/serverguide/automatic-updates.html

## Configure firewall to monitor for unsuccessful login attempts
- Install Fail2Ban
```
sudo apt-get update
sudo apt-get install fail2ban
- Copy jail.conf to local copy since .conf can be modified by package updates
`sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
- Edit file
`sudo nano /etc/fail2ban/jail.local`
- Start the service
`sudo service fail2ban start`
- Test by trying to ssh as a fake user, e.g. ssh  fakeUser@52.24.160.178 -p 2200 
and then in a different terminal run `sudo iptables –S` and the banned user should 
be displayed in the list
- Reference Documentation: https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04
- NOTE: If setting up email notice, install `sudo apt-get install sendmail iptables-persistent`

## Application status monitoring
- Install Glances 
`sudo pip install glances`
- Run Glances by typing `glances`
- Reference Documentation: https://pypi.python.org/pypi/Glances and http://glances.readthedocs.org/en/latest/glances-doc.html#web-server-mode







