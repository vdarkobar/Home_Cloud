# Traefik2  

<p align="center">
  <b>Services:</b><br>
  <a href="https://github.com/vdarkobar/NextCloud">NextCloud</a> |
  <a href="https://github.com/vdarkobar/Bitwarden">Bitwarden</a> |
  <a href="https://github.com/vdarkobar/WordPress">WordPress</a> |
  <a href="https://github.com/vdarkobar/Ghost-blog">Ghost</a> |
  <a href="https://github.com/vdarkobar/Portainer">Portainer</a> |
  <a href="https://github.com/vdarkobar/Portainer">Joomla</a> |
  <a href="https://github.com/vdarkobar/Portainer">iPerf</a>  
  <br><br>
</p>
  
<p align="center">
  <img src="https://github.com/vdarkobar/misc/blob/main/reverse-proxy.png">
  <img src="https://github.com/vdarkobar/misc/blob/main/infrastructure.webp">
  <br><br>
</p>
  
### CloudFlare prep:  

Point your root domain (example.com) to your WAN IP using an A record.
```
    A | example.com | YOUR WAN IP
```
  
Add either a wildcard CNAME (*.example.com) or individual subdomains, all pointing to your root domain (@ for the host).  
```
    CNAME | * | @ (or example.com)
```

For Non-WWW to WWW redirect.  
```
    A | www | YOUR WAN IP
```

DNS Settings:  
  
![alt text](https://github.com/vdarkobar/misc/blob/main/cloudflare-dns-entries-740x226.webp "DNS Management for a domain")  
![alt text](https://github.com/vdarkobar/misc/blob/main/cloudflare-dns-records-for-traefik-2-740x290.webp "DNS Management for a domain")  
![alt text](https://github.com/vdarkobar/misc/blob/main/cloudflare-full-ssl-for-traefik-docker-setup.webp "DNS Management for a domain")  
   
wait for a few minutes for the DNS entries to propagate. 
  
Firewall rules:  
  
![alt text](https://github.com/vdarkobar/misc/blob/main/cloudflare-firewall-rules-740x335.webp "Firewall rules")  
  
#### Site settings:  

<pre>
SSL Mode - Full or Strict  

Edge Certificates:  
  Always Use HTTPS: ON  
  HTTP Strict Transport Security (HSTS): Enable (Be Cautious)  
  Minimum TLS Version: 1.2  
  Opportunistic Encryption: ON  
  TLS 1.3: ON  
  Automatic HTTPS Rewrites: ON  
  Certificate Transparency Monitoring: ON  

Firewall Settings:  
  Security Level: High  
  Bot Fight Mode: ON  
  Challenge Passage: 30 Minutes  
  Browser Integrity Check: ON  
</pre>

---

### Server prep:  

```
sudo apt update
sudo apt-get install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  wget \
  acl \
  tree \
  dnsutils \
  gnupg-agent \
  apache2-utils \
  software-properties-common
  ```
--- 
### *>> Enable port forwarding (80, 443) on your router, or gateway, to your Traefik instance (VM).*
--- 
  
### Docker, docker-compose:  
*(swich from fish to bash if command not working)*  

```
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
```
```
sudo add-apt-repository \
  "deb [arch=amd64] https://download.docker.com/linux/debian \
  $(lsb_release -cs) \
  stable"
```
```
sudo apt update && sudo apt install -y docker-ce docker-ce-cli containerd.io
sudo docker -v
sudo docker ps -a
```
  
<p align="center">
  <b><i> Compose is a tool for creating a list of all the containers (along with their configuration) that you want to run. </b></i>
</p>
  
[Check the current Docker Compose release here](https://github.com/docker/compose/releases), update '/1.27.4/' in the command:
```
sudo curl -L https://github.com/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose  
sudo chmod +x /usr/local/bin/docker-compose
sudo docker-compose --version
```
  
### Create necessary Docker networks:  
```
sudo docker network create traefik
```
*option: custom Docker networks (specify the gateway and subnet to use):*
```
sudo docker network create --gateway 192.168.90.1 --subnet 192.168.90.0/24 traefik  
```
*option: set static ip to your service(s)
```
# Option: Specify static IP for service
    networks:
      traefik:
        ipv4_address: 192.168.90.254
```

### Securing Docker:  

<p align="center">
<b>Do no add user to docker group (sudo usermod -aG docker $USER && logout).</b><br>
<b>Do not mess with the ownership of Docker Socket (/var/run/docker.sock in Linux)</b><br>
<b>Do not run Docker Containers as Root. Add environmental variables PUID, PGID, to .env file.</b><br>
<b>Use Privileged Mode Carefully (- no-new-privileges:true)</b><br>
<b>Change DOCKER_OPTS to Respect IP Table Firewall. Edit /etc/default/docker and add the following line:</b><br>
</p>

```
DOCKER_OPTS="--iptables=false"  
```

--- 

#### Folder/File structure example:  

<pre>
traefik2
├── data
│   ├── acme.json
│   └── configurations
│       ├── middlewares.yml
│       └── tls.yml
├── secrets
│   ├── cloudflare_api_key.secret
│   └── cloudflare_email.secret
├── shared
│   └── .htpasswd
├── docker-compose.yml
├── .env
├── tmp
└── traefik.log
</pre>

### Clone this git repository and give it a name:
```
echo -n "Enter directory name: "; read NAME; mkdir -p "$NAME"; cd "$NAME" \
&& git clone https://vdarkobar:2211620c9da5dab0c7bb77e9aeb02087d293b293@github.com/vdarkobar/Traefik2.git .
```
<!-- This is commented out. 
...
-->
### Set permissions:
```
sudo chmod 600 data/acme.json
sudo chown -R root:root secrets/
```
### Copy <a href="https://dash.cloudflare.com/profile/api-tokens">CloudFlare Global API Key</a> and edit the files:
```
sudo nano secrets/cloudflare_email.secret
sudo nano secrets/cloudflare_api_key.secret
```
### Edit:
```
sudo nano .env
```
### Start:
```
sudo docker-compose up -d
```
### Stop:
```
sudo docker-compose down -v
```
### Log:
```
sudo docker-compose logs traefik
sudo docker logs -tf --tail="50" traefik
```
  
<p align="center">
  <b>Resources:</b><br>
  <a href="https://www.smarthomebeginner.com/traefik-2-docker-tutorial/">Link 1</a> |
  <a href="https://github.com/htpcBeginner/docker-traefik">Link 2</a> |
  <a href="https://github.com/CVJoint/traefik2">Link 3</a> |
  <a href="https://tech.aufomm.com/">Link 4</a> |
  <a href="https://goneuland.de/">Link 5</a> |
  <a href="https://github.com/adam-p/markdown-here/wiki/Markdown-Cheatsheet">Link 6</a>
  <br><br>
</p>
