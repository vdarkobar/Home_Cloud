<p align="center">
  <b>Home Lab, centered arround Traefik Proxy: The Cloud Native Application Proxy, using Proxmox, Debian and Docker.</b><br>
</p>  

# ProxMox
  
<p align="left">
  <a href="https://github.com/vdarkobar/Bastion">Bastion</a> |
  <a href="https://github.com/vdarkobar/Ansible">Ansible</a>  
</p>
<p align="center">
  <img src="https://github.com/vdarkobar/shared/blob/main/bastion.webp">
</p>
  
<p align="left">
  <a href="https://github.com/vdarkobar/Debian">Debian Template</a>  
</p>
<p align="center">
  <img src="https://github.com/vdarkobar/misc/blob/main/infrastructure.webp">
</p>
  
# Traefik  
<p align="center">
  <img src="https://github.com/vdarkobar/misc/blob/main/reverse-proxy.png">
  <br><br>
</p>
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

### CloudFlare:  
  
Point your root domain (example.com) to your WAN IP using an A record.  
```
    A | example.com | YOUR WAN IP
```
<p align="center">
  <img src="https://github.com/vdarkobar/misc/blob/main/A-record.webp">
</p>
  
Add individual subdomains, for all services, pointing to your root domain (@ for the host).  
```
    CNAME | * | @ (or example.com)
```
<p align="center">
  <img src="https://github.com/vdarkobar/misc/blob/main/sub-domain.webp">
</p>
  
Add for non-WWW to WWW redirect.  
```
    CNAME | www | YOUR WAN IP
```
<p align="center">
  <img src="https://github.com/vdarkobar/misc/blob/main/www.webp">
</p>
  
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
  
<p align="center">
  <b> Wait for DNS entries to propagate. Optionally, edit CloudFlare Firewall rules. </b><br>
  <b> In order to see if Let’s Encrypt is working pause CloudFlare on selected website before running docker-compose. </b><br>
  <b> (Advanced Actions > Pause Cloudflare on Site) </b><br>
</p>
  
---
#### *>> Enable port forwarding (80, 443) from your Router, or Gateway, to your Traefik instance (VM).*
--- 
  
<!--- Commented out
### Server:  
  
```
sudo apt update && \
sudo apt-get install -y \
  apt-transport-https \
  ca-certificates \
  curl \
  wget \
  gnupg-agent \
  apache2-utils \
  software-properties-common
  ```
    --->
### Docker:  
*swich shell (bash) if command not working*  
  
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
#### Create necessary Docker networks:  
```
sudo docker network create traefik
```
<!--- Commented out
*option: custom Docker networks (specify the gateway and subnet to use).*
```
sudo docker network create --gateway 192.168.90.1 --subnet 192.168.90.0/24 traefik  
```
*option: set static ip to your service(s).*
```
# ...
    networks:
      traefik:
        ipv4_address: 192.168.90.254
```
--->
#### Securing Docker:  

<p align="center">
  <b>Do no add user to docker group (sudo usermod -aG docker $USER && logout).</b><br>
  <b>Do not mess with the ownership of Docker Socket (/var/run/docker.sock in Linux)</b><br>
  <b>Change DOCKER_OPTS to Respect IP Table Firewall. Edit /etc/default/docker and add the following line:</b><br>
</p>

```
DOCKER_OPTS="--iptables=false"  
```
  
### Docker Compose:  
  
[Check the current Docker Compose release here](https://github.com/docker/compose/releases), update '/1.27.4/' in the command:
```
sudo curl -L https://github.com/docker/compose/releases/download/1.27.4/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose  
sudo chmod +x /usr/local/bin/docker-compose
sudo docker-compose --version
```
--- 

#### Folder/File structure:  

<pre>
traefik2
├── data
│   ├── acme.json
│   └── configurations
│       ├── middlewares-chains.yml
│       ├── middlewares.yml
│       └── tls.yml
├── secrets
│   ├── cloudflare_api_key.secret
│   └── cloudflare_email.secret
├── shared
│   └── .htpasswd
├── tmp
├── docker-compose.yml
├── .env
├── access.log
└── traefik.log
</pre>

### Clone this git repository:
```
RED='\033[0;31m'; echo -ne "${RED}Enter directory name: "; read DIR; \
mkdir -p "$DIR"; cd "$DIR" && git clone https://github.com/vdarkobar/Traefik2.git .
```
  
#### *Decide what you will use for*:
```
Traefik username and password, 
CloudFlare email and API Key, 
Time Zone, 
Let's Encrypt Email, 
Domain name, 
Subdomain for Traefik,
Portainer Port.
```
  

### Copy <a href="https://dash.cloudflare.com/profile/api-tokens">CloudFlare Global API Key</a> to memory (*clipboard*):
  
### Select and run all at once. Enter required data:
```
RED='\033[0;31m'
echo -ne "${RED}Enter Traefik username: "; read UNAME; \
echo -ne "${RED}Enter Traefik password: "; read PASS; \
echo -ne "${RED}Enter CloudFlare email: "; read CFEMAIL; \
echo -ne "${RED}Paste CloudFlare API Key: "; read CFAPI; \
echo -ne "${RED}Enter Time Zone: "; read TZONE; \
echo -ne "${RED}Enter Let's Encrypt Email: "; read LEEMAIL; \
echo -ne "${RED}Enter Traefik Subdomain: "; read SUB; \
echo -ne "${RED}Enter Domain name: "; read DNAME; \
echo -ne "${RED}Enter Portainer Port: "; read PP; \
echo $(htpasswd -nbB "$UNAME" "$PASS") > ~/shared/.htpasswd && \
echo ${CFEMAIL} > secrets/cloudflare_email.secret && \
echo ${CFAPI} > secrets/cloudflare_api_key.secret && \
sed -i "s|01|${TZONE}|" .env && \
sed -i "s|02|${LEEMAIL}|" .env && \
sed -i "s|03|${SUB}|" .env && \
sed -i "s|04|${DNAME}|" .env && \
sed -i "s|05|${PP}|" .env && \
rm README.md && \
sudo chmod 600 data/acme.json && \
sudo chown -R root:root secrets/ && \
sudo chmod -R 600 secrets/ && \
sudo docker-compose up -d
```
### Log:
```
sudo docker-compose logs traefik
sudo docker logs -tf --tail="50" traefik
```
<!--- Commented out
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
--->
