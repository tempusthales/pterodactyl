# Install Pterodactyl in Debian 11 w/NGINX

## Prerequisites

* Debian 11
* Tabby - https://github.com/Eugeny/tabby (works on windows and mac) if you are working directly on linux then you dont need this.

## Install Debian 11

If you need help follow this guide: https://www.linuxtechi.com/how-to-install-debian-11-bullseye/

* Note: DO NOT install UI. Don't be a Noob. =)

## Prepping Linux

* Log in as root
* `apt install sudo` ("If you get an error that you cannot install sudo, then skip ahead")

* `apt update`

## Updating `sources.list`

* `rm -rf /etc/apt/sources.list`

* Copy the contents below into `sources.list`
* * `sudo nano /etc/apt/sources.list`

```none
deb http://deb.debian.org/debian/ bullseye main contrib non-free
deb-src http://deb.debian.org/debian/ bullseye main contrib non-free

deb http://deb.debian.org/debian/ bullseye-updates main contrib non-free
deb-src http://deb.debian.org/debian/ bullseye-updates main contrib non-free

deb http://security.debian.org/debian-security bullseye-security main contrib
deb-src http://security.debian.org/debian-security bullseye-security main contrib

deb http://deb.debian.org/debian bullseye-proposed-updates main contrib non-free

deb http://ftp.debian.org/debian bullseye-backports main
deb-src http://ftp.debian.org/debian bullseye-backports main
```

* Ctrl+X to save

## Updating 89_backports_default

* `cd /etc/apt/preferences.d`
* Upload the file `89_backports_default` from: `https://cdn.discordapp.com/attachments/967031398693208164/981576918270894110/89_backports_default`

**Note:** ("If you are using Tabby, then you can sftp with Tabby")

* `apt install sudo`
* `sudo apt install net-tools`
* `sudo apt install ufw`
* `sudo apt install curl`

## Install and Configure Open SSH Server

* `sudo apt install openssh-server`

  * `sudo systemctl status ssh` There will be a line named *Active*, where it should read: **active (running)**

  * If Open SSH Server is not running type: `sudo systemctl start sshd`

  * To enable ssh service on system boot, type: `sudo systemctl enable --now sshd`

  * `sudo nano /etc/ssh/sshd_config`

    * Find a line that reads `Include /etc/ssh/sshd_config.d/*.conf` then below it remove the `#` from `#Port 22` so it reads `Port 22`.  Change the port from `22` to `2201` or whichever port you want to use.  Just don't forget it!!
    * About 19 lines below `Port 2201` you will find an entry for `PermitRootLogin` change it so it reads `PermitRootLogin yes` then press Ctrl+X to save the file.

  * Note: to start, restart or stop the service:

    * ```none
      sudo systemctl start sshd
      sudo systemctl restart sshd
      sudo systemctl stop
      ```

  * `sudo ufw allow ssh`

## Upgrading Debian

* `sudo apt upgrade` - This will upgrade all components in your server.  it might be 500mb's and it will take a couple of minutes.
* type: `sudo shutdown -r now` 
* log back into root

## Installing Dependencies

Go to this link: `https://pterodactyl.io/panel/1.0/getting_started.html#dependencies`

* `curl -sSL https://packages.sury.org/php/README.txt | sudo bash -x`
* `sudo wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg`
* `apt install redis`
* `apt -y install software-properties-common curl apt-transport-https ca-certificates gnupg`
* `apt -y install php8.1 php8.1-{common,cli,gd,mysql,mbstring,bcmath,xml,fpm,curl,zip} nginx tar unzip git`
* `systemctl enable --now redis-server`

## Install and configure Maria DB on Debian

* `sudo apt install mariadb-server`

* `sudo mysql_secure_installation `

  * Answer the following questions:
    * *You already have your root account protected, so you can safely answer: `n`*
      * Switch to unix_socket authentication [Y/n]: `n`
    * *You already have your root account protected, so you can safely answer: `n`*
      * Change the root password? [Y/n]: `n`
    * *By default MariaDB installation has an anonymous user, allowing anyone to log into MariaDB without having to have a user account created for them.  This is intended only for testing, and to make the installation go a bit smoother.  You should remove them before moving into a production environment.*
      * Remove anonymous users? [Y/n]: `Y`
    * *Normally, root should only be allowed to connect from 'localhost'.  This ensures that someone cannot guess at the root password from the network.*
      * Disallow root login remotely? [Y/n]: `Y`
    * *By default MariaDB comes with a database named 'test' that anyone can access.  This is also intended only for testing, and should be removed before moving into a production environment.*
      * Remove test database and access to it: `Y`
    * *Reloading the privilege tables will ensure that all changes made so far will take effect immediately.*
      * Reload privilege tables now: `Y`

  **Note**: *The options are self-explanatory, for the first two options choose “n” and for the next sequence of options press “y” for yes.*

   

  ## Create Privileges User with Authentication

  * `sudo mysql` then you will be at a prompt that will read: `MariaDB [(none)]>` enter the following commands there:
    * `CREATE USER 'pterodactyl'@'127.0.0.1' IDENTIFIED BY 'yourPassword';` Note: change yourPassword to whatever password you want. 
      * For example: `CREATE USER 'pterodactyl'@'127.0.0.1' IDENTIFIED BY 'CWa4gDk5ZmzfXm';`
      * **Note**: Don't use that password, its an example. Generate your own password!
    * `CREATE DATABASE panel; GRANT ALL PRIVILEGES ON panel.* TO 'pterodactyl'@'127.0.0.1' WITH GRANT OPTION; exit`
  * Apply the new changes, execute: `FLUSH PRIVILEGES;`
  * And to quit, type "exit": `EXIT`

  ## Connect MariaDB Server

One can manage MariaDB service using the `Systemd`. To test the status of MariaDB use the following command:

* `sudo systemctl status mariadb`

If for some reasons MariaDB is not running then use the below-mentioned command to start it:

* `sudo systemctl start mariadb`

For one more check you can try to connect to the database using:

* `sudo mysqladmin version`

```
mysqladmin  Ver 9.1 Distrib 10.5.11-MariaDB, for debian-linux-gnu on x86_64
Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Server version          10.5.11-MariaDB-1
Protocol version        10
Connection              Localhost via UNIX socket
UNIX socket             /run/mysqld/mysqld.sock
Uptime:                 3 hours 45 min 24 sec

Threads: 1  Questions: 497  Slow queries: 0  Opens: 171  Open tables: 28  Queries per second avg: 0.036
```

Next, connect to the MySQL shell by using the credentials created in the above step:

* `mysql -u pterodactyl -p`

The output of the above command asks for the password; use the password you set in the above steps. On successful authentication, you will get the MariaDB shell as below:

```
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 60
Server version: 10.5.11-MariaDB-1 Debian 11

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]>
```

## Installing Composer

Composer is a dependency manager for PHP that allows us to ship everything you'll need code wise to operate the Panel. You'll need composer installed before continuing in this process.

* `curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer`

## Download Files

The first step in this process is to create the folder where the panel will live and then move ourselves into that newly created folder. Below is an example of how to perform this operation.

```shell
mkdir -p /var/www/pterodactyl
cd /var/www/pterodactyl
```

Once you have created a new directory for the Panel and moved into it you'll need to download the Panel files. This is as simple as using `curl` to download our pre-packaged content. Once it is downloaded you'll need to unpack the archive and then set the correct permissions on the `storage/` and `bootstrap/cache/` directories. These directories allow us to store files as well as keep a speedy cache available to reduce load times.

```shell
curl -Lo panel.tar.gz https://github.com/pterodactyl/panel/releases/latest/download/panel.tar.gz
tar -xzvf panel.tar.gz
chmod -R 755 storage/* bootstrap/cache/
```

## Panel Installation

First we will copy over our default environment settings file, install core dependencies, and then generate a new application encryption key.

```shell
cp .env.example .env
composer install --no-dev --optimize-autoloader

# Only run the command below if you are installing this Panel for
# the first time and do not have any Pterodactyl Panel data in the database.
php artisan key:generate --force
```

## Environment Configuration

Pterodactyl's core environment is easily configured using a few different CLI commands built into the app. This step will cover setting up things such as sessions, caching, database credentials, and email sending.

```shell
php artisan p:environment:setup
php artisan p:environment:database

# To use PHP's internal mail sending (not recommended), select "mail". To use a
# custom SMTP server, select "smtp".
php artisan p:environment:mail
```

## DB Setup

* `php artisan migrate --seed --force`

## Add the first user

You'll then need to create an administrative user so that you can log into the panel. To do so, run the command below. At this time passwords **must** meet the following requirements: 8 characters, mixed case, at least one number.

* `php artisan p:user:make` to make the first user

## Set Permissions

The last step in the installation process is to set the correct permissions on the Panel files so that the webserver can use them correctly.

```shell
# If using NGINX or Apache (not on CentOS):
chown -R www-data:www-data /var/www/pterodactyl/*

# If using NGINX on CentOS:
chown -R nginx:nginx /var/www/pterodactyl/*

# If using Apache on CentOS
chown -R apache:apache /var/www/pterodactyl/*
```

## Queue Listeners

We make use of queues to make the application faster and handle sending emails and other actions in the background. You will need to setup the queue worker for these actions to be processed.

## Create Crontab Configuration

The first thing we need to do is create a new cronjob that runs every minute to process specific Pterodactyl tasks, such as session cleanup and sending scheduled tasks to daemons. You'll want to open your crontab using `sudo crontab -e` and then paste the line below.

```shell
* * * * * php /var/www/pterodactyl/artisan schedule:run >> /dev/null 2>&1
```

## Create Queue Worker

Next you need to create a new systemd worker to keep our queue process running in the background. This queue is responsible for sending emails and handling many other background tasks for Pterodactyl.

Create a file called `pteroq.service` in `/etc/systemd/system` with the contents below.

```
# Pterodactyl Queue Worker File
# ----------------------------------

[Unit]
Description=Pterodactyl Queue Worker
After=redis-server.service

[Service]
# On some systems the user and group might be different.
# Some systems use `apache` or `nginx` as the user and group.
User=www-data
Group=www-data
Restart=always
ExecStart=/usr/bin/php /var/www/pterodactyl/artisan queue:work --queue=high,standard,low --sleep=3 --tries=3
StartLimitInterval=180
StartLimitBurst=30
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

If you are using redis for your system, you will want to make sure to enable that it will start on boot. You can do that by running the following command:

* `sudo systemctl enable --now redis-server`

Finally, enable the service and set it to boot on machine start.

* `sudo systemctl enable --now pteroq.service`

## Webserver Configuration

**WARNING**

*When using the SSL configuration you MUST create SSL certificates, otherwise your webserver will fail to start. See the [Creating SSL Certificates](https://pterodactyl.io/tutorials/creating_ssl_certificates.html) documentation page to learn how to create these certificates before continuing.*

Remove default NGINX configuration: `rm -rf /etc/nginx/sites-enabled/default`

Paste the contents below, replacing `<domain>` with your domain name being used in a file called `pterodactyl.conf` and place the file in `/etc/nginx/sites-available/`.

### Nginx with SSL

```conf
server_tokens off;

server {
    listen 80;
    server_name <domain>;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name <domain>;

    root /var/www/pterodactyl/public;
    index index.php;

    access_log /var/log/nginx/pterodactyl.app-access.log;
    error_log  /var/log/nginx/pterodactyl.app-error.log error;

    # allow larger file uploads and longer script runtimes
    client_max_body_size 100m;
    client_body_timeout 120s;

    sendfile off;

    # SSL Configuration - Replace the example <domain> with your domain
    ssl_certificate /etc/letsencrypt/live/<domain>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<domain>/privkey.pem;
    ssl_session_cache shared:SSL:10m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";
    ssl_prefer_server_ciphers on;

    # See https://hstspreload.org/ before uncommenting the line below.
    # add_header Strict-Transport-Security "max-age=15768000; preload;";
    add_header X-Content-Type-Options nosniff;
    add_header X-XSS-Protection "1; mode=block";
    add_header X-Robots-Tag none;
    add_header Content-Security-Policy "frame-ancestors 'self'";
    add_header X-Frame-Options DENY;
    add_header Referrer-Policy same-origin;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param PHP_VALUE "upload_max_filesize = 100M \n post_max_size=100M";
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTP_PROXY "";
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
        include /etc/nginx/fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

###Nginx without SSL

```
server {
    # Replace the example <domain> with your domain name or IP address
    listen 80;
    server_name <domain>;


    root /var/www/pterodactyl/public;
    index index.html index.htm index.php;
    charset utf-8;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location = /favicon.ico { access_log off; log_not_found off; }
    location = /robots.txt  { access_log off; log_not_found off; }

    access_log off;
    error_log  /var/log/nginx/pterodactyl.app-error.log error;

    # allow larger file uploads and longer script runtimes
    client_max_body_size 100m;
    client_body_timeout 120s;

    sendfile off;

    location ~ \.php$ {
        fastcgi_split_path_info ^(.+\.php)(/.+)$;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_index index.php;
        include fastcgi_params;
        fastcgi_param PHP_VALUE "upload_max_filesize = 100M \n post_max_size=100M";
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param HTTP_PROXY "";
        fastcgi_intercept_errors off;
        fastcgi_buffer_size 16k;
        fastcgi_buffers 4 16k;
        fastcgi_connect_timeout 300;
        fastcgi_send_timeout 300;
        fastcgi_read_timeout 300;
    }

    location ~ /\.ht {
        deny all;
    }
}
```

## Enabling Configuration

The final step is to enable your NGINX configuration and restart it.

```shell
# You do not need to symlink this file if you are using CentOS.
sudo ln -s /etc/nginx/sites-available/pterodactyl.conf /etc/nginx/sites-enabled/pterodactyl.conf

# You need to restart nginx regardless of OS.
sudo systemctl restart nginx
```

## Wings Installation

Wings is the next generation server control plane from Pterodactyl. It has been rebuilt from the ground up using Go and lessons learned from our first Nodejs Daemon.

to be continued...
