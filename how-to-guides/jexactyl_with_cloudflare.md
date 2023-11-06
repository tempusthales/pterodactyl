# Setting up Jexactyl with Cloudflare Full Steps!!

> I made this guide for my own help, since I am constantly optimizing process. However, not all the content in this guide was created solely by me. Portions of this guide were made from the original instructions in [docs.jexactyl.com](docs.jexactyl.com), [developers.cloudflare.com](https://developers.cloudflare.com) and [docs.pterodactyl.io](https://pterodactyl.io/project/introduction.html) in addition to countless hours of troubleshooting and experimentation. --TT

# OPTIONAL FIRST STEPS

## Server VM Configuration

The optimal server config is a VM with 4-8 cores, 128-256 GB of RAM, and 1 TB + Storage running Ubuntu 22.04.

### Install Webmin

```
sudo apt update && curl -fsSL https://download.webmin.com/jcameron-key.asc | sudo gpg --dearmor -o /usr/share/keyrings/webmin.gpg
```

**Next you will add this repository to your /etc/apt/sources.list file while referencing the newly converted file you just acquired in the previous step. Open the file in your preferred editor. Here, you’ll use nano:**

`sudo nano /etc/apt/sources.list`

**Then add this line to the bottom of the file to add the new repository:**

`deb [signed-by=/usr/share/keyrings/webmin.gpg] http://download.webmin.com/download/repository sarge contrib`

**Save the file and exit the editor. If you had used nano to edit, you can exit by pressing CTRL+X, Y, then ENTER.****

### Proceed to install Webmin

`sudo apt update && sudo apt install webmin`

**To log into webmin https://your_server:10000**

# Game Server Manager Installation

#### Dependency Installation

`apt -y install software-properties-common curl apt-transport-https ca-certificates gnupg`

`LC_ALL=C.UTF-8 add-apt-repository -y ppa:ondrej/php && add-apt-repository ppa:redislabs/redis -y`

#### The command below is not needed if you are using Ubuntu 22.04 or higher.
`curl -sS https://downloads.mariadb.com/MariaDB/mariadb_repo_setup | sudo bash`

`apt update &&  apt -y install php8.1 php8.1-{cli,gd,mysql,pdo,mbstring,tokenizer,bcmath,xml,fpm,curl,zip} mariadb-server nginx tar unzip git redis-server` 

`curl -sS https://getcomposer.org/installer | sudo php -- --install-dir=/usr/local/bin --filename=composer`

#### Harden MariaDB Instance
`sudo mysql_secure_installation`


# Download Files

#### Create Directory
`mkdir -p /var/www/jexactyl && cd /var/www/jexactyl`

#### Download Panel

`curl -Lo panel.tar.gz https://github.com/jexactyl/jexactyl/releases/latest/download/panel.tar.gz && tar -xzvf panel.tar.gz && chmod -R 755 storage/* bootstrap/cache/`


# Database Setup

#### Create Database

`mariadb -u root -p`

#### Configure A Password-authenticated Administrative User
```bash
CREATE USER 'mhadmin'@'localhost' IDENTIFIED BY '8tatdiZyzdboSw';
GRANT ALL PRIVILEGES ON *.* TO 'mhadmin'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```



#### Create jexactyl user and password and grant permissions
```bash
CREATE USER 'jexactyl'@'127.0.0.1' IDENTIFIED BY '9XaUtK3AoszMsy';
CREATE DATABASE panel;
GRANT ALL PRIVILEGES ON panel.* TO 'jexactyl'@'127.0.0.1' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```



# Environment Setup

#### Create configuration file

`cp .env.example .env`

#### Install Composer Packages

`composer install --no-dev --optimize-autoloader`

#### Generate a random token which will be the encryption/app key for this project.

#### This encryption key is used to store important data (such as API keys). Do NOT share this key with anyone - protect it like a password. If you lose ### this key, all data is impossible to recover.

`php artisan key:generate --force`

#### Configure Panel Environment
```
php artisan p:environment:setup
php artisan p:environment:database
php artisan p:environment:mail # Not required to run the Panel.
```

#### Database Migration
###### The command below may take some time to run depending on your machine. Please DO NOT exit the process until it is completed!

`php artisan migrate --seed --force`

### Create admin user

`php artisan p:user:make`

### Assign Permissions
#### For using NGINX or Apache (not on CentOS):
`chown -R www-data:www-data /var/www/jexactyl/*`

# Queue Workers

### Crontab

`sudo crontab -e`

`* * * * * php /var/www/jexactyl/artisan schedule:run >> /dev/null 2>&1`

### Systemd Queue Worker

`nano /etc/systemd/system/panel.service`

###### Copy and paste the contents below then CTRL+X to save.

```
# Jexactyl Queue Worker File
# ----------------------------------

[Unit]
Description=Jexactyl Queue Worker

[Service]
User=www-data
Group=www-data
Restart=always
ExecStart=/usr/bin/php /var/www/jexactyl/artisan queue:work --queue=high,standard,low --sleep=3 --tries=3
StartLimitInterval=180
StartLimitBurst=30
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

# Setup SSL with Cloudflare

`[Enable SSL Access Over HTTPS with Cloudflare](https://docs.bitnami.com/aws/faq/administration/enable-ssl-cloudflare/)`

# Nginx with SSL Configuration

### Remove default configuration

`rm /etc/nginx/sites-available/default; rm /etc/nginx/sites-enabled/default`

### Enable Cloudflare SSL in NGINX

1. Go to Cloudflare.com > Choose your domain > SSL/TLS > [Origin Server](https://dash.cloudflare.com/f93f58e13b6767361e921ce67fa127c3/mhguild.cloud/ssl-tls/origin).
2. Click Create Certificate > Whatever
3. Check your hostnames, these are the hostnames protected by your certificate.
4. Choose the Certificate Validity. The default is 15 years.
5. Click the button on the bottom right of the page called CREATE.
6. Keep the Key Format as PEM
7. In the Private Key, copy the contents of your private key by pressing Click to copy then open VS Code (or whatever, just not notepad) paste the contents, then save it to <whatever_domain_com.pem>.
8. In the Private Key, copy the contents of your private key by pressing Click to copy then open VS Code (or whatever, just not notepad) paste the contents, then save it to <whatever_domain_com_key.pem>.
9. Upload the files via SSH to your server and put them in /etc/ssl/certs.

`**It is best if you do this on the same server just using nano and creating each file individually into /etc/ssl/certs the path for the files will be /etc/ssl/certs/whatever_domain_com.pem and /etc/ssl/certs/whatever_domain_com_key.pem**`

### Create the configuration file

###### Make sure to replace <domain> with your own domain in this config file. Please also note that this configuration is for NGINX with SSL enabled. If ###### you want to use Apache as a webserver, or do not want to use SSL, please refer to the other webserver instructions.

`nano /etc/nginx/sites-available/panel.conf`

### Copy the text below and remember to edit ssl_certificate and ssl_certificate_key values to your own!!!

```
server {
    listen 80;
    server_name <domain>;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name <domain>;

    root /var/www/jexactyl/public;
    index index.php;

    access_log /var/log/nginx/jexactyl.app-access.log;
    error_log  /var/log/nginx/jexactyl.app-error.log error;

    # allow larger file uploads and longer script runtimes
    client_max_body_size 100m;
    client_body_timeout 120s;

    sendfile off;

    # SSL Configuration
    ssl_certificate /etc/ssl/certs/whatever_domain_com.pem;
    ssl_certificate_key /etc/ssl/certs/whatever_domain_com_key.pem;
    ssl_session_cache shared:SSL:10m;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384";
    ssl_prefer_server_ciphers on;

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

### Enabling Configuration

`ln -s /etc/nginx/sites-available/panel.conf /etc/nginx/sites-enabled/panel.conf`

### Test the NGINX configuration to make sure its working and valid:

`nginx -t` 

### Restart NGINX server process to make the Panel available on the domain.

`systemctl restart nginx`

##### Congrats! Panel is installed and should be functioning normally.

# Installing Wings

## System Requirements

To run Wings, you will need a Linux system capable of running Docker containers. Most VPS and almost all dedicated servers should be capable of running Docker, but there are edge cases.

When your provider uses `Virtuozzo`, `OpenVZ` (or `OVZ`), or `LXC` virtualization, you will most likely be unable to run Wings. **KVM** is guaranteed to work.

To check type `systemd-detect-virt`. If the result doesn't contain OpenVZ orLXC, it should be fine. The result of none will appear when running dedicated hardware without any virtualization.

You can also try using the `sudo dmidecode -s system-manufacturer`

### Installing Docker

`curl -sSL https://get.docker.com/ | CHANNEL=stable bash`

### Starting Docker on Boot

`sudo systemctl enable --now docker`

### Enabling Swap

On most systems, Docker will be unable to setup swap space by default. You can confirm this by running `docker info` and looking for the output of `WARNING: No swap limit support` near the bottom.

Enabling swap is entirely optional, but we recommended doing it if you will be hosting for others and to prevent OOM errors.

To enable swap, open `/etc/default/grub` as a root user and find the line starting with `GRUB_CMDLINE_LINUX_DEFAULT`. Make sure the line includes `swapaccount=1` somewhere inside the double-quotes.

After that, run `sudo update-grub` followed by `sudo reboot` to restart the server and have swap enabled. Below is an example of what the line should look like, do not copy this line verbatim. It often has additional OS-specific parameters.

`GRUB_CMDLINE_LINUX_DEFAULT="swapaccount=1"`

### Installing Wings

`sudo mkdir -p /etc/pterodactyl && curl -L -o /usr/local/bin/wings "https://github.com/pterodactyl/wings/releases/latest/download/wings_linux_$([[ "$(uname -m)" == "x86_64" ]] && echo "amd64" || echo "arm64")" && sudo chmod u+x /usr/local/bin/wings`

#### Configure

Once you have installed Wings and the required components, the next step is to create a node on your installed Panel. Go to your Panel administrative view, select Nodes from the sidebar, and on the right side click Create New button.

After you have created a node, click on it and there will be a tab called Configuration. Copy the code block content, paste it into a new file called `config.yml` in `/etc/pterodactyl` and save it.

Alternatively, you can click on the **Generate Token** button, copy the bash command and paste it into your terminal.

**WARNING**: When your Panel is using SSL, the Wings must also have one created for its FQDN. See Creating SSL Certificates documentation page for how to create these certificates before continuing.

#### Starting Wings

`sudo wings --debug`

#### Daemonizing (using systemd)

Running Wings in the background is a simple task, just make sure that it runs without errors before doing this. Place the contents below in a file called `wings.service` in the `/etc/systemd/`system directory.

```
[Unit]
Description=Pterodactyl Wings Daemon
After=docker.service
Requires=docker.service
PartOf=docker.service

[Service]
User=root
WorkingDirectory=/etc/pterodactyl
LimitNOFILE=4096
PIDFile=/var/run/wings/daemon.pid
ExecStart=/usr/local/bin/wings
Restart=on-failure
StartLimitInterval=180
StartLimitBurst=30
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

Then, run the commands below to reload systemd and start Wings.

`systemctl enable --now wings`

#### Node Allocations

Allocation is a combination of IP and Port that you can assign to a server. Each created server must have at least one allocation. The allocation would be the IP address of your network interface. In some cases, such as when behind NAT, it would be the internal IP. To create new allocations go to Nodes > your node > Allocation.

Type `hostname -I | awk '{print $1}'` to find the IP to be used for the allocation. Alternatively, you can type `ip addr | grep "inet "` to see all your available interfaces and IP addresses. Do not use `127.0.0.1` for allocations.

# THE END.

# Additional Configuration (Optional)

### Enabling Cloudflare Proxy

Cloudflare proxying of the Wings isn't beneficial since users will be connecting to the machine directly and bypassing any Cloudflare protection. As such, your Node machine IP will still be exposed.

To enable Cloudflare proxy, you must change the Wings port to one of the Cloudflare HTTPS ports with caching enabled (more info [here](https://developers.cloudflare.com/fundamentals/get-started/reference/network-ports/) (opens new window)), such as 8443, because Cloudflare only supports HTTP on port 8080. Select your Node in the Admin Panel, and on the settings tab, change the port. Make sure that you set "Not Behind Proxy" when using Full SSL settings in Cloudflare. Then on Cloudflare dashboard, your FQDN must have an orange cloud enabled beside it.

You are unable to proxy the SFTP port through Cloudflare unless you have their enterprise plan.
