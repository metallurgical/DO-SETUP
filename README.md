#### DO 1st Setup
------------

**1) Create Droplet As Ussual**
 - After creating droplet is finished in DO, the root password with droplet IP address will sent to an email address. For the first time login, the root access required to change the password

**2) Change Root Password**
 - Use SSH client(putty) for windows or use DO console for easy access. Login as root and will asked to change new password. Fill in current password, hit enter and another prompted will showing up to enter new password. Just fill in new password and you're done 
 
#### Install nginx
------------------

**1) Run this command to update our local package index before we begin so that we are using the most up-to-date information**
 - `sudo apt-get update`
 
**2) Afterwards, install nginx**
 - `sudo apt-get install nginx`
 
**3) Find the Ip address of current droplet**
 - `ip addr show eth0 | grep inet | awk '{ print $2; }' | sed 's/\/.*$//'` or
 - type : `curl http://icanhazip.com`
 
**4) Copy the Ip, and paste into browser and see the Nginx Welcome Message**
 - `http://ip_address_step_3`
 - If the installation running well, should see **nginx welcome message**
 
**5) After all of these done, we can use the basic command to play with nginx :**
  - Stop nginx
   - `sudo service nginx stop`
  - Start nginx
   - `sudo service nginx start`
  - Restart nginx
   - `sudo service nginx restart`
  - Make sure that our web server will restart automatically when the server is rebooted by typing:
   - `sudo update-rc.d nginx defaults` hit enter and if see this message `System start/stop links for /etc/init.d/nginx already exist.`, this just means that it was already configured correctly and that no action was necessary. Either way, your Nginx service is now configured to start up at boot time.
  
#### Install Mysql-server
------------------
**1) Run this command to update our local package index before we begin so that we are using the most up-to-date information**
 - `sudo apt-get update`
 
**2) Afterwards, install mysql**
 - `sudo apt-get install mysql-server`
 - During installation, the package will asked for setting up password for mysql user @root. Just type in your password and proceed till it finish.
 
**3) For the short version, above step is enough, but to make our database more secure, we can proceed to no 4)**

**4) Secure the database/mysql, First, you'll want to run the included security script. This changes some of the less secure default options for things like remote root logins and sample users. Run :**
 - `sudo mysql_secure_installation`
 - Just type in the required and suitable answer when prompted during installation
