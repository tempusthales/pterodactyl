# Steps to get Wings set up

Follow these steps and you should be able to get Wings up and running on a fresh Debian installation.

## Install Docker

Everything from **Step 1** on https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04, but with "debian" substituted for "ubuntu" in the various commands:

1. `apt update`
2. `apt upgrade`
3. `apt install apt-transport-https ca-certificates curl software-properties-common`
4. `curl -fsSL https://download.docker.com/linux/debian/gpg | gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg`
5. `echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null`
6. `apt update`
7. `apt install docker-ce`
8. `systemctl status docker`

## Install Wings

Following the instructions on [https://pterodactyl.io/wings/1.0/installing.html#installing-wings-2](https://pterodactyl.io/wings/1.0/installing.html#installing-wings-2):

1. `mkdir -p /etc/pterodactyl`
2. `curl -L -o /usr/local/bin/wings "https://github.com/pterodactyl/wings/releases/latest/download/wings_linux_$([[ "$(uname -m)" == "x86_64" ]] && echo "amd64" || echo "arm64")"`
3. `chmod u+x /usr/local/bin/wings`

## Set up CF Tunnel

1. Log into https://dash.cloudflare.com **> Zero Trust > Access > Tunnels**
2. Click **Create a tunnel**
3. Give it a name (e.g. `my-wings-server`)
4. Select Debian
5. Copy first option on left to install + run connector
6. Specify subdomain + select domain (e.g. `my-wings-server.example.com`)
7. Specify http://127.0.0.1:8443
   1. Idea here is to choose one of the ports compatible with CF proxies
   2. See https://developers.cloudflare.com/fundamentals/reference/network-ports/

## Add new Wings server to Pterodactyl

1. https://mhguild.cloud/admin/nodes
2. Click **Create New**
3. Specify name (e.g. `my-wings-server`)
4. **Node Visibility:** Public
5. **Deployable via Jexactyl Store:** Allow
6. **FQDN:** (the tunnel created above, e.g. `my-wings-server.example.com`)
7. **Communicate Over SSL:** Use SSL Connection
8. **Behind Proxy:** Behind Proxy
9. **Total Memory:** (Whatever)
10. **Total Disk Space:** (Whatever)
11. **Daemon Port:** 8443 (same as the port you specified in your CF Tunnel above)
12. **Daemon SFTP Port:** 2052 (same idea here, pick a CF-proxy-friendly port)
13. Submit

## Deploy Wings configuration

1. Click **Configuration** tab
2. Copy file contents
3. Go to Wings server and `nano /etc/pterodactyl/config.yml`
4. `wings --debug` and confirm green heart
5. `Ctrl + C` to stop Wings in debug mode
6. Daemonize as per https://pterodactyl.io/wings/1.0/installing.html#daemonizing-using-systemd
7. Set up allocations as per https://pterodactyl.io/wings/1.0/installing.html#node-allocations

## Create a server

1. Do your Pterodactyl thing and deploy a server to your Wings instance
