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
   
**6) Reference**
-  https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-14-04-lts
  
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
 
**5) If the installation process running well, type in :**
 - `mysql --version`
 - and should see something like this 
  - `mysql  Ver 14.14 Distrib 5.5.47, for debian-linux-gnu (x86_64) using readline 6.3`
  
**6) If you're using a version of MySQL earlier than 5.7.6, you should initialize the data directory by running**
 - `sudo mysql_install_db`
 - In MySQL 5.6, you might get an error that says **FATAL ERROR: Could not find my-default.cnf.** If you do, copy the **/usr/share/my.cnf** configuration file into the location that **mysql_install_db** expects, then rerun it. :
  - `sudo cp /etc/mysql/my.cnf /usr/share/mysql/my-default.cnf` copy
  - `sudo mysql_install_db` and rerun
 - The **mysql_install_db** command is deprecated as of MySQL 5.7.6. If you're using version 5.7.6 or later, you should use:
  - `mysqld --initialize instead`
  - However, if you installed version 5.7 from the Debian distribution, like in step one, the data directory was initialized automatically, so you don't have to do anything. If you try running the command anyway, you'll see the following error:
   - `[ERROR] --initialize specified but the data directory has files in it. Aborting.`
  
**7) Reference** :
- https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-14-04


#### Install PHP5-fpm(skip this if you did't use PHP as a server side)
------------------
**1) If we want to use PHP as a server side, we need to install this. We still need something to connect the nginx and mysql. Since Nginx does not contain native PHP processing like some other web servers, we will need to install php5-fpm, which stands for "fastCGI process manager". We will tell Nginx to pass PHP requests to this software for processing. We can install this module and will also grab an additional helper package that will allow PHP to communicate with our database backend. The installation will pull in the necessary PHP core files. Do this by typing:**
 - `sudo apt-get install php5-fpm php5-mysql`
 
**2) Configure the PHP Processor, We now have our PHP components installed, but we need to make a slight configuration change to make our setup more secure. Open the main php5-fpm configuration file with root privileges:**
 - `sudo nano /etc/php5/fpm/php.ini`
 - What we are looking for in this file is the parameter that sets **cgi.fix_pathinfo**. This will be commented out with a semi-colon (;) and set to "1" by default. This is an extremely insecure setting because it tells PHP to attempt to execute the closest file it can find if a PHP file does not match exactly. This basically would allow users to craft PHP requests in a way that would allow them to execute scripts that they shouldn't be allowed to execute. We will change both of these conditions by uncommenting the line and setting it to "0" like this:
  - `cgi.fix_pathinfo=0`
  - Save and close the file when you are finished.

**3) Now, we just need to restart our PHP processor by typing:**
- `sudo service php5-fpm restart`

**4) Configure Nginx to Use our PHP Processor, Now, we have all of the required components installed. The only configuration change we still need to do is tell Nginx to use our PHP processor for dynamic content, We do this on the server block level (server blocks are similar to Apache's virtual hosts). Open the default Nginx server block configuration file by typing:**
- `sudo nano /etc/nginx/sites-available/default` , this will open virtual host block set by nginx on default state. 
- During this time, Currently, there is a lot of example with comment, and with the comments removed, the Nginx default server block file looks like this:

```php 
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /usr/share/nginx/html;
    index index.html index.htm;

    server_name localhost;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
change to :

``` php
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    root /usr/share/nginx/html;
    index index.php index.html index.htm;

    server_name server_domain_name_or_IP;

    location / {
        try_files $uri $uri/ =404;
    }

    error_page 404 /404.html;
    error_page 500 502 503 504 /50x.html;
    location = /50x.html {
        root /usr/share/nginx/html;
    }

    location ~ \.php$ {
        try_files $uri =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
}
```
- if you doesn't want to use nano as editor, just use FTP software like winscp or filezilla to access into server root, and find the file `/etc/nginx/sites-available/default` and edit to be like above state.

**5) After that restart nginx to ensure the nginx take effect :**
 - `sudo service nginx restart`
 
**6) Testing PHP working or not, We still should test to make sure that Nginx can correctly hand .php files off to our PHP processor. We can do this by creating a test PHP file in our document root. Open a new file called info.php within your document root in your text editor:**
- `sudo nano /usr/share/nginx/html/info.php`
- Put this :
``` php
   <?php
   phpinfo();
   ?>
```
- and visit our page :
 - `http://server_domain_name_or_IP/info.php`
- You should see a web page that has been generated by PHP with information about your server.
- If you see a page that looks like this, you've set up PHP processing with Nginx successfully, After you test this, it's probably best to remove the file you created as it can actually give unauthorized users some hints about your configuration that may help them try to break in. You can always regenerate this file if you need it later, For now, remove the file by typing:
 - `sudo rm /usr/share/nginx/html/info.php`
