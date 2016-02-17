# Linux Server Configuration
Set-up information for Udacity course ud299 on how to configure a Linix Server. The 
Virtual Machine using Vagrant and root login credentials are provided by Udacity. This 
server is linked to an EC2 instance on Amazon.

## 1. Launch VM with your VM
Go to `https://www.udacity.com/account#!/development_environment` to get your AWS IP address
and key. Login to the server as this root user. 

## 2. Add user grader
As root, type `sudo adduser grader`. Then follow prompts for password 
and name for this new account. 
<br> Confirm addition of the new user by typeing `sudo /cat/etc/passwd`, 
the user grader should be listed in the output.

## 3. Give grader sudo permission
As the root user, type `visudo`. 
Documentation: https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps


## 4. Set-up Key-based Authentication/Public Key Encryption
Using ssh-keygen on your local machine, create a key for the use grader. 
<br> Follow the prompts to provide a name for this key and use the default location (~/.ssh). 
This will create two keys on your local machine, the file with extension .pub is the public key to 
be transferred to the server.

## 5. Add Public Key to Server
Create a key on the AWS server using the contents of the .pub key file created above.

## 6. Set-up Permissions on the .ssh file in the Student Terminal 

## 7. Disable Password Based Login and Force Login using Key Pair 
on these Steps

## 8. Update all currently installed packages
`sudo apt-get update` This lists all the packages to update. 
`sudo apt-get upgrade` This actually updates the packages.

## 9. Change the SSH port from 22 to 2200
`nano /etc/ssh/sshd_config`
Edit the file to change the SSH port from 22 to 2200. 
<br> Restart the server `service ssh restart`
!! Do not logout as root yet!!
In a separate Terminal window, try to login as root using the new SSH port as
`ssh -i ~/.ssh/udacity_key.rsa –p 2200 root@AWS_IP_ADDRESS`
<br> Documentation: https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-12-04
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
Confirm that root can SSH and login from local computer, 
`ssh -i ~/.ssh/udacity_key.rsa -p 2200 root@AWS_IP_ADDRESS`
If yes, hooray we can proceed. If not, repeat these steps starting at the beginning since
you are locked out of the server. 
<br>~ There may be a way to use a GUI to still access the server and fix the problem, check notes.

## 11. Configure local Timezone to UTC
`dpkg-reconfigure tzdata`
In the window that appears, use the arrow keys to Scroll to the bottom of 
the Continents list and select Etc or None of the Above and then in the 
second list, select UTC
<br>
Cofirm time change by typing `date` on the command line
<br> Documentation: http://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt/138442 
and https://help.ubuntu.com/community/UbuntuTime 

## 12. Install and configure Apache to serve a Python mod_wsgi application
Install Apache `sudo apt-get install apache2`
<br>Confirm successful installation by visiting http://52.24.160.178/ (URL from 
Udacity Environment information for AWS instance). It should say "It Works" and 
display other Apache information on the page.
<br>
Install Python MOD_WSGI `sudo apt-get install libapache2-mod-wsgi`
<br> Configure WSGI app `sudo nano /etc/apache2/sites-enabled/000-default.conf`
At the end of the <VirtualHost *:80> block, right before the closing </VirtualHost> 
add this line: WSGIScriptAlias / /var/www/html/myapp.wsgi
<br> Restart Apache `sudo service apache2 restart`
NOTE: After restart the Home page will return a 404, we’ll fix that next by configuring Apache to serve WSGI application


## 13. 




