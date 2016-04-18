# Setup Ubuntu Linux Server with Nginx, PHP and Mysql

## Seting up Components
First, we need to update our local package index to make sure we have a fresh list of the available packages. Then we can install the necessary components:

    sudo apt-get update
    sudo apt-get install nginx php5-fpm php5-cli php5-mcrypt git

This will install Nginx as our web server along with the necessary PHP tools. We also install git because the composer tool, the dependency manager for PHP, will use it to pull down packages.

If you want to configure PHP then you can edit _php.ini_ file and change configuration settings for that execute following statements

    sudo vim /etc/php5/fpm/php.ini
    
## Configuring Nginx Server root directory and default document
The next item that we should address is the web server. This will involve following two steps.

First create the folder where your website will be hosted(root directory)

    sudo mkdir -p /var/www/vhosts/example.com
    
Now we need to set _/var/www/vhosts/example.com_ as root directory of **Nginx** web server for that we need to edit default Nginx Configuration file located at _/etc/nginx/sites-available/default_.
    
    sudo nano /etc/nginx/sites-available/default
    
Default Configuration file
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

Now we need to 
* Change the root directory from _/usr/share/nginx/html_ to root directory of our project Eg. _/var/www/vhosts/example.com_.
* Add index.php to default document list.
* Add static IP or Domain Name instead of Localhost in server Name

For thatc just change the Nginx default configuration file to following file.

    server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;

        root /var/www/vhosts/example.com;
        index index.php index.html index.htm;

        server_name <Static IP Address or Domain Name>;

        location / {
                try_files $uri $uri/ =404;
        }
    }
    
## Setup Nginx to execute PHP scripts

For this again we need to open **Nginx** Default Configuration file and add following block into it right before closing of server block

    location ~ \.php$ {
        try_files $uri /index.php =404;
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/var/run/php5-fpm.sock;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }
    
After saving finally configuration file should look like this

    server {
        listen 80 default_server;
        listen [::]:80 default_server ipv6only=on;
    
        root /var/www/vhosts/example.com;
        index index.php index.html index.htm;
    
        server_name <Static IP Address or Domain Name>;
    
        location / {
            try_files $uri $uri/ /index.php?$query_string;
        }
    
        location ~ \.php$ {
            try_files $uri /index.php =404;
            fastcgi_split_path_info ^(.+\.php)(/.+)$;
            fastcgi_pass unix:/var/run/php5-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }
    }
    
After saving the configuration file we just need to restart Nginx

    sudo service nginx restart
    
## Setting up Composer
Composer is the most popular dependency manager for php projects and to setup composer we just need to execute following commands.

    cd ~
    curl -sS https://getcomposer.org/installer | php
    sudo mv composer.phar /usr/local/bin/composer
    
## Installing up Mysql
Mysql is the most common and popular database server used with PHP and to setup mysql databse you just need to execute

    sudo apt-get update
    sudo apt-get install mysql-server
    
This will prompt you for mysql root(admin) password too.
    
## Updating and verifying Mysql Server

    sudo mysql_secure_installation
    
Above command will prompt you and can be used to remove default created users and test databases that came right out of the box along with mysql installation.

to test mysql installation try executing 
    
    service mysql status
    
If above command gives an error then just start mysql service with command

    sudo service mysql start
    
## Securing Mysql 

Database is the crucial part of a website which needs to be secure for that you may consider following guidelines(optional)

**Renaming root username**

Root is the database administrator account which has all the privileges so it make sense to chage root credentials after installation(especially username).

for that execute following command
    
    mysql -u root -p 
    <Enter mysql Password>
    mysql> rename user 'root'@'localhost' to '<new_username>'@'localhost';
    
**Creating New user with limited Database Access and limited privileges**

    mysql -u root -p
    <Enter mysql password>
    mysql> GRANT SELECT,UPDATE,DELETE ON <database_name>.* TO '<new_user>'@'localhost';
    
After completing all the steps mentioned above your webserver will be ready with Nginx, PHP, Composer and Mysql

But In some cases you might get Mysql Connection Exception saying mysql driver not found for that just execute following command

    sudo apt-get install php5-mysql
    
And you are done...

