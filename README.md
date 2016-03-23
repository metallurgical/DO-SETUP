[logo_main]: https://encrypted-tbn2.gstatic.com/images?q=tbn:ANd9GcST9v8A3x54BSoD9ipEB2i_QLTkh7OCY6VK_sGI_THbAH2IY0G1 "Logo Title Text 1"
[logo_sub]: https://www.cannastaff.com/include/themes/nasthon1001-restyle/images/delicious.gif "Logo Title Text 2"

![alt text][logo_main]  DO Inital Setup
------------

**1) Create Droplet As Ussual**
 - After creating droplet is finished in DO, the root password with droplet IP address will sent to an email address. For the first time login, the root access required to change the password

**2) Change Root Password**
 - Use SSH client(putty) for windows or use DO console for easy access. Login as root and will asked to change new password. Fill in current password, hit enter and another prompted will showing up to enter new password. Just fill in new password and you're done 
 

![alt text][logo_main] Install nginx
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
  

![alt text][logo_main] Install Mysql-server
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

![alt text][logo_main] Install PHP5-fpm(skip this if you did't use PHP as a server side)
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

**7) Reference**
- https://www.digitalocean.com/community/tutorials/how-to-install-linux-nginx-mysql-php-lemp-stack-on-ubuntu-14-04


![alt text][logo_main] Install PhpMyadmin(skip this if not related)
------------
**1) Run this command to update our local package index before we begin so that we are using the most up-to-date information**
 - `sudo apt-get update`
 
**2) Afterwards, install phpmyadmin**
- `sudo apt-get install phpmyadmin`
- during this time, will asked to configure for appache or etc server, since we are using nginx as web server, just skip this process.
- We need to provide password for phpmyadmin root access, type in your mysql root user password

**3) Create symbolink(shortcut) from original location to be able to access it from nginx html/www folder :**
- `sudo ln -s /usr/share/phpmyadmin /usr/share/nginx/html` or 
- `sudo ln -s /usr/share/phpmyadmin/ /usr/share/nginx/www`
- it depend on where you serve html content either inside **html or www** folder

**4) Testing, try to access :**
- Restart nginx before use : `sudo service nginx restart`
- Type in browser url : `http://server_ip_address/phpmyadmin`
- If everything goes well, then the **Basic Setup for phpmyadmin is DONE**, if want to secure the phpmyadmin, can proceed to number **5) and forth..**

**5) Since the phpmyadmin folder can be accessible from the public internet, we need to secure our database. A final item that we need to address is enabling the mcrypt PHP module, which phpMyAdmin relies on. This was installed with phpMyAdmin so we just need to toggle it on and restart our PHP processor:**
- `sudo php5enmod mcrypt`
- `sudo service php5-fpm restart`
- We are done enabling mcrypt for phpmyadmin

**6) Secure your phpMyAdmin Instance, The phpMyAdmin instance installed on our server should be completely usable at this point. However, by installing a web interface, we have exposed our MySQL system to the outside world., Even with the included authentication screen, this is quite a problem. Because of phpMyAdmin's popularity combined with the large amount of data it provides access to, installations like these are common targets for attackers..We will implement two simple strategies to lessen the chances of our installation being targeted and compromised. We will change the location of the interface from /phpmyadmin to something else to sidestep some of the automated bot brute-force attempts. We will also create an additional, web server-level authentication gateway that must be passed before even getting to the phpMyAdmin login screen.**
- ![alt text][logo_sub] By Changing the Application's Access Location
 - In order for our Nginx web server to find and serve our phpMyAdmin files, we created a symbolic link from the phpMyAdmin directory to our document root in an earlier step. To change the URL where our phpMyAdmin interface can be accessed, we simply need to rename the symbolic link. Move into the Nginx document root directory to get a better idea of what we are doing:
  - Go to our site dir : `cd /usr/share/nginx/html`
  - Listing all the file with privilege shown : `ls -l ` and we can see below output from console :
  
   ```php
    total 8
    -rw-r--r-- 1 root root 537 Mar  4 06:46 50x.html
    -rw-r--r-- 1 root root 612 Mar  4 06:46 index.html
    lrwxrwxrwx 1 root root  21 Aug  6 10:50 phpmyadmin -> /usr/share/phpmyadmin
   ```

   - As you can see, we have a symbolic link called phpmyadmin in this directory. We can change this link name to whatever we would like. This will change the location where phpMyAdmin can be accessed from a browser, which can help obscure the access point from hard-coded bots, Choose a name that does not indicate the purpose of the location. In this guide, we will name our access location `/you_name_it`. To accomplish this, we will just rename the link:
    - Rename the folder name : `sudo mv phpmyadmin you_name_it`
    - And again try to list all dir to see if the actual name really renamed or not by typing : `ls -l`, and we can see the below output :
  
   ```php
   total 8
   -rw-r--r-- 1 root root 537 Mar  4 06:46 50x.html
   -rw-r--r-- 1 root root 612 Mar  4 06:46 index.html
   lrwxrwxrwx 1 root root  21 Aug  6 10:50 the_name_that_you_use_to_rename_it -> /usr/share/phpmyadmin
   ```

   - Done. Now we can access the phpmyadmin by using the new name : `http://server_ip_address/the_name_that_you_use_to_rename_it`
 
- ![alt text][logo_sub] Second, by Setting up a Web Server Authentication Gate
 - The next feature we wanted for our installation was an authentication prompt that a user would be required to pass before ever seeing the phpMyAdmin login screen, Fortunately, most web servers, including Nginx, provide this capability natively. We will just need to modify our Nginx configuration file with the details, Before we do this, we will create a password file that will store our the authentication credentials. Nginx requires that passwords be encrypted using the crypt() function. The OpenSSL suite, which should already be installed on your server, includes this functionality,To create an encrypted password, type :
  - `openssl passwd`
  - Type password for auth
  - after finish, the new encrypted password will created like `artbdweckuhsdf34de` and copy those and paste into any editor as you will need to paste it into the authentication file we will be creating, Now, create an authentication file. We will call this file pma_pass and place it in the Nginx configuration directory:
   - `sudo nano /etc/nginx/pma_pass` use nano or
   - directly open file using any ftp to modified it.
   - Within this file, you simply need to specify the username you would like to use, followed by a colon (:), followed by the encrypted version of your password you received from the openssl passwd utility, We are going to name our user `you_name_it`. The file for this guide looks like this: 
    - `you_name_it:encrypted_password_we_copy_earlier` ---> example : `admin:artbdweckuhsdf34de`
  - Save and close the file when you are finished., Now, we are ready to modify our Nginx configuration file. Open this file in your text editor to get started:
    - `sudo nano /etc/nginx/sites-available/default` or open by any ftp client to modified it.
    - Within this file, we need to add a new location section. This will target the location we chose for our phpMyAdmin interface (we selected `/the_phpmyadmin_new_folder_name_after_rename` ., Create this section within the server block, but outside of any other blocks. We will put our new location block below the location / block in our example:
 
     ```php
      server {
        . . .

        location / {
          try_files $uri $uri/ =404;
        }

        location /the_phpmyadmin_new_folder_name_after_rename {
        }

        . . .
     }
    ```
    - Within this block, we need to set the value of a directive called auth_basic to an authentication message that our prompt will display to users. We do not want to indicate to unauthenticated users what we are protecting, so do not give specific details. We will just use "Admin Login" in our example., We then need to use a directive called auth_basic_user_file to point our web server to the authentication file that we created. Nginx will prompt the user for authentication details and check that the inputted values match what it finds in the specified file., After we are finished, the file should look like this:
 
    ```php
      server {
        . . .

        location / {
           try_files $uri $uri/ =404;
        }

        location /the_phpmyadmin_new_folder_name_after_rename {
           auth_basic "Admin Login";
           auth_basic_user_file /etc/nginx/pma_pass;
        }
   

      . . .
     }
   ```
    - Save and close the file when you are finished., To implement our new authentication gate, we must restart the web server:
     - `sudo service nginx restart`
    - Finally we open the page in our browser `http://server_ip_address/the_phpmyadmin_new_folder_name_after_rename` and here we can see the Auth promp will showing up. DONE!!
    
**7) Reference**
 - https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-with-nginx-on-an-ubuntu-14-04-server
 



