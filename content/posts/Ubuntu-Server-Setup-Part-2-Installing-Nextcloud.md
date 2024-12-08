+++
title = "Ubuntu Server Setup Part 2: Installing Nextcloud"
date = "2024-06-22T20:35:26+05:30"
tags = [ "Nextcloud", "Ubuntu", "Server" ]
showtoc = true
+++

I finally setup my Ubuntu server quite well along with wifi drivers.

Next up, I had to install Nextcloud, because I wanted a cloud storage that could backup and sync photos, videos and other stuff from several devices at my home. Thus I looked up a tutorial on it and found one by Jay, which was extremely easy to follow and helped me setup my Nextcloud instance from scratch.

### The steps to setup Nextcloud

1) You should be using a normal user account, and should not be executing the commands as 'root'. In case you are 'root', simply execute the following:

    ```bash
    adduser <Enter your desired username here>
    usermod -aG sudo <Enter the username you set above>
    ```

2) Update the system's cache and also apply any pending updates with the following command:

    ```bash
    sudo apt update && sudo apt dist-upgrade
    ```

3) Now, let us download and install php, mariadb and apache (along with some additional dependencies and required packages):

    ```bash
    sudo apt install libmagickcore-6.q16-6-extra php php-apcu php-bcmath php-cli php-common php-curl php-gd php-gmp php-imagick php-intl php-mbstring php-mysql php-zip php-xml apache2 unzip mariadb-server redis-server php-redis -y
    ```
    _Note:_ You can check if apache is running [Should be active and enabled] with:

    ```bash
    systemctl status apache2
    ```

4) Download and unzip nextcloud with:

    ```bash
    wget https://download.nextcloud.com/server/releases/latest.zip
    unzip latest.zip
    ```

   We will now change the ownership of nextcloud to the group and user 'www-data' and move the nextcloud folder into the '/var/www/' directory as SOP. The 'www-data' is a user (and group) created by default when we installed apache2. The **-R** flag changes ownership of all files and folders within the nextcloud directory recursively.

    ```bash
    sudo chown www-data:www-data -R nextcloud
    sudo mv nextcloud /var/www/
    ```

5) PHP needs some modules to be enabled for it's functioning. Do so with:

    ```bash
    sudo phpenmod imagick intl bcmath gmp  
    ```

6) Let us start configuring database, which is required to base our Nextcloud on. We must first run a setup of mysql to secure our installation. To do so, run:

    ```bash
    sudo mysql_secure_installation
    ``` 

    It will open a bash asking you to fill certain details. 

    - The first prompt will be asking for default root password, just press Enter button to skip this option.
    - Next it will ask for Unix socket, **type 'n'** and proceed.
    - Then it will ask to enter a password, **type 'y'** and then enter a password (store it securely).
    - Just keep pressing Enter button to skip through rest of the options (Capital letter means default option, **'Y'** in all these options).

7) Once done with this, we must now configure mariadb. Open it's shell by:

    ```bash
    sudo mariadb
    ```

    In this shell, type these commands (replace where marked):

    - ```CREATE DATABASE nextcloud;```
    - ```GRANT ALL PRIVILEGES ON nextcloud.* TO 'nextcloud'@'localhost' IDENTIFIED BY <'Enter a secure password here for the db. Do keep the password inside single inverted commas.'>;```
    - ```FLUSH PRIVILEGES;```
    - ```exit;```

    **NOTE:** Store this password very securely, as it can be misused if leaked.

8) We should now generate our own SSL/TLS certificate to use with our server (In case you have a static IP, you should use Let's encrypt's certificate using [Certbot](https://certbot.eff.org/))
    
    **NOTE:** This command will ask you several question. Answer whatever you want to fill in (won't have much effect).
    ```bash
    mkdir $HOME/certs && cd $HOME/certs/

    openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem -days 3650 -nodes
    ```

    **NOTE:** Please store both the key.pem and cert.pem somewhere secured. Also import cert.pem as an Authorities certificate in your browser to avoid the browser from flagging our nextcloud instance as a 'Not Secure' website everytime we visit it.

9) Now we must disable the default webpage of apache, place our config file in the apache directory and enable the config for apache to load the nextcloud page (Along with adding Strict Transport Security).

    ```bash
    sudo a2dissite 000-default.conf

    sudo nano /etc/apache2/sites-available/nextcloud.conf
    ```
    In the text editor which opens, paste the following code:

    ```bash
    <VirtualHost *:80>

        RewriteEngine On
        RewriteCond %{SERVER_PORT} !443
        RewriteRule ^(/(.*))?$ https://%{HTTP_HOST}/$1 [R=301,L]

    </VirtualHost>

    <VirtualHost *:443>               
        DocumentRoot "/var/www/nextcloud"

	    SSLCertificateFile $HOME/certs/cert.pem
	    SSLCertificateKeyFile $HOME/certs/key.pem
	    SSLEngine on


        <Directory "/var/www/nextcloud/">
            Options MultiViews FollowSymlinks
            AllowOverride All
            Order allow,deny
            Allow from all
        </Directory>

        TransferLog /var/log/apache2/nextcloud_access.log
        ErrorLog /var/log/apache2/nextcloud_error.log

        <IfModule mod_headers.c>
            Header always set Strict-Transport-Security "max-age=15552000; includeSubDomains"
        </IfModule>
    </VirtualHost>
    ```
    Save this file by doing ctrl+o and ctrl+x. Then enable this config file using:

    ```bash
    sudo a2ensite nextcloud.conf
    ```

10) Let us tune PHP to make our server run more efficiently:

    **Note**: PHP versions can differ on different distributions, like 8.3 on Ubuntu 24 LTS and 8.1 on 22 LTS, thus press tab as directed below: 

    ```bash
    sudo nano /etc/php/<press tab to autocomplete here>/apache2/php.ini
    ```

    Find and edit the file by change stuff inside it according to this (Replace the areas where you see fit. To remove a comment, remove the ';' semi-colon in front of the line in the file):
    - memory_limit = 2G
    - upload_max_filesize = 20G
    - max_execution_time = 360
    - post_max_size = 20G
    - date.timezone = Asia/Kolkata
    - opcache.enable=1
    - opcache.interned_strings_buffer=16
    - opcache.max_accelerated_files=10000
    - opcache.memory_consumption=128
    - opcache.save_comments=1
    - opcache.revalidate_freq=1

11) Some modules are required by apache2, so enable them for use with nextcloud.

    ```bash
    sudo a2enmod dir env headers mime rewrite ssl
    ```

12) Enable the apcu module in php. This module caches data in memory, so helps in reducing database queries and file system operations. This also allows the occ script to function.

    **Note**: PHP versions can differ on different distributions, like 8.3 on Ubuntu 24 LTS and 8.1 on 22 LTS, thus press tab as directed below:

    ```bash
    echo "apc.enable_cli=1" | sudo tee -a /etc/php/<press tab here>/mods-available/apcu.ini
    ```

13) Finally, let us restart apache2 to get nextcloud working:

    ```bash
    sudo systemctl restart apache2
    ```
***

### Let's configure nextcloud via its webpage

Create an administrator account in the webpage that opens when you visit the IP address / URL that you attached to the server. Then when you scroll down, it will ask for your database details. Fill according to what you entered in the above steps (db account and name should be 'nextcloud' and password will be the one you entered while configuring mariadb, step 7).

Sign in and setup nextcloud as you like. Next, head over to the Admin settings by clicking on your profile on top right side and then on "Administration settings". 

{{< figure align=center src="/img/admin_settings_nextcloud.png">}}

In here, you will see the various issues that nextcloud has automatically detected with the installation.

#### Some of the errors that I had received and their solutions:

1) If you get a very big error which states that the database is missing some indices, simply do as follows:

    ```bash
    sudo chmod +x /var/www/nextcloud/occ                                      

    sudo -u 'www-data' /var/www/nextcloud/occ db:add-missing-indices          

    sudo chmod -x /var/www/nextcloud/occ                                      
    ```

    This adds the missing indices using occ script provided by nextcloud themselves, which helps in correcting issues with nextcloud.

2) Enable memory caching:

    ```bash
    sudo nano /var/www/nextcloud/config/config.php
    ```
    In the text editor which opens, add the following line somewhere in between the other lines (before the closing bracktes atleast):

    - 'memcache.local' => '\\OC\\Memcache\\APCu',
    - 'default_phone_region' => 'IN',

    **NOTE:** Replace phone region code to whatever your region code is. Also, remember to have those commas in the end of each of the above 2 lines.

3) Additional security parameter is to lock down the config.php file to root group, as it contains sensitive information like our database password.

    ```bash
    sudo chmod 660 /var/www/nextcloud/config/config.php

    sudo chown root:www-data /var/www/nextcloud/config/config.php
    ```
***
### Conclusion

After doing this much, you can restart apache and/or even reboot your system once. 

Now I think you will be good to go with your own personal cloud storage. Remember to keep your system updated and files backed up (try to follow 3-2-1 rule) to be safe.