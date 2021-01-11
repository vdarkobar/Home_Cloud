---

<p align="center">
Small <i><a href="https://github.com/vdarkobar/Home_Cloud#services-to-be-used-behind-traefik">Home-Cloud</a>, behind <a href="https://doc.traefik.io/traefik/getting-started/quick-start/">Traefik Proxy</a>, using <a href="https://pve.proxmox.com/pve-docs/pve-admin-guide.html">Proxmox</a>, <a href="https://www.debian.org/doc/">Debian</a> and <a href="https://docs.docker.com/">Docker</a> to run:</i> <b></br>
<b>NextCloud | Bitwarden | WordPress | Ghost | Joomla</b>
</p>  

---

Install and configure:
# <a href="https://github.com/vdarkobar/shared/blob/main/Proxmox.md#proxmox">Proxmox</a>
  
<p align="left">
  Create <a href="https://github.com/vdarkobar/shared/blob/main/Bastion.md#bastion">Bastion VM</a> and 
  install <a href="https://github.com/vdarkobar/Playbooks#ansible">Ansible</a>. 
  Create <a href="https://github.com/vdarkobar/shared/blob/main/Debian.md#debian">Debian Template</a>.
</p>
  
<p align="center">
  <img src="https://github.com/vdarkobar/shared/blob/main/bastion.webp">
</p>
  
<p align="center">
  <img src="https://github.com/vdarkobar/misc/blob/main/infrastructure.webp">
</p>
  
--- 
  
# <a href="https://github.com/vdarkobar/Home_Cloud/blob/main/README.md#cloudflare">CloudFlare</a>  
  
Login to <a href="https://dash.cloudflare.com/">CloudFlare</a> and point your root domain (example.com) to your WAN IP using an A record.  
```
    A | example.com | YOUR WAN IP
```
<p align="center">
  <img src="https://github.com/vdarkobar/misc/blob/main/A-record.webp">
</p>
  
Add individual subdomains*, for all <a href="https://github.com/vdarkobar/Home_Cloud#services-to-be-used-behind-traefik">Services</a>, pointing to your root domain (@ for the host).  
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
  <b><i> Advanced Actions > Pause Cloudflare on Site </i></b><br>
</p>
  
---
#### *>> Enable port forwarding (80, 443) from your Router, or Gateway, to your Traefik instance (VM). <<*
--- 
  
# <a href="https://github.com/vdarkobar/Home_Cloud#traefik-proxy">Traefik Proxy</a>  

   Clone new VM from <a href="https://github.com/vdarkobar/shared/blob/main/Debian.md#debian">Template</a>, 
   <a href="https://github.com/vdarkobar/shared/blob/main/Bastion.md#bastion">SSH</a> in and install 
   <a href="https://github.com/vdarkobar/shared/blob/main/Docker.md#docker">Docker</a>.
  <br><br>
  
<p align="center">
  <img src="https://github.com/vdarkobar/misc/blob/main/reverse-proxy.png">
  <br><br>
</p>
  

### Clone Traefik Git Repository:
```
RED='\033[0;31m'; echo -ne "${RED}Enter directory name: "; read DIR; \
mkdir -p "$DIR"; cd "$DIR" && git clone https://github.com/vdarkobar/Home_Cloud.git .
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

    networks:
      traefik:
        ipv4_address: 192.168.90.254
```
--->
  
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
  
### Prepare <a href="https://dash.cloudflare.com/profile/api-tokens">CloudFlare Global API Key</a>:
  
### Select and run all at once. Enter required data:
*Only works once, on wrong data input delete folder and clone again*.
```
clear
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
echo $(htpasswd -nbB "$UNAME" "$PASS") > shared/.htpasswd && \
echo ${CFEMAIL} > secrets/cloudflare_email.secret && \
echo ${CFAPI} > secrets/cloudflare_api_key.secret && \
sed -i "s|01|${TZONE}|" .env && \
sed -i "s|02|${LEEMAIL}|" .env && \
sed -i "s|03|${SUB}|" .env && \
sed -i "s|04|${DNAME}|" .env && \
sed -i "s|05|${PP}|" .env && \
rm README.md && \
touch access.log && \
touch traefik.log && \
touch data/acme.json && \
sudo chown root:root data/acme.json && \
sudo chmod 600 data/acme.json && \
sudo chown -R root:root secrets/ && \
sudo chmod -R 600 secrets/
```
  
### Start Traefik:
```
sudo docker-compose up -d
```
  
--- 
  
## Services to be used behind Traefik:  
  
<p align="center">
   In order to install Services, clone new VM from a <a href="https://github.com/vdarkobar/shared/blob/main/Debian.md#debian">Template</a>, 
   <a href="https://github.com/vdarkobar/shared/blob/main/Bastion.md#bastion">SSH</a> in and install 
   <a href="https://github.com/vdarkobar/shared/blob/main/Docker.md#docker">Docker</a>.
  <br><br>
</p>
  
<p align="center">
  <a href="https://github.com/vdarkobar/NextCloud#nextcloud">NextCloud</a> |
  <a href="https://github.com/vdarkobar/Bitwarden#bitwarden">Bitwarden</a> |
  <a href="https://github.com/vdarkobar/WordPress#wordpress">WordPress</a> |
  <a href="https://github.com/vdarkobar/Ghost-blog">Ghost</a> |
  <a href="https://github.com/vdarkobar/Portainer">Joomla</a>  
  <br><br>
</p>  
  
---  
  
### Log:
```
sudo docker-compose logs traefik
sudo docker logs -tf --tail="50" traefik
```
  
#### Example:  

<pre>
proxmox                                         server1
└── server1                                     └── docker 
    │   └── docker                                  └── traefik
    │       └── traefik                                 ├── data
    │                                                   │   ├── acme.json
    server2                                             │   └── configurations
    │   └── docker                                      │       ├── middlewares-chains.yml
    │       ├── Bitwarden                               │       ├── middlewares.yml
    │       └── NextCloud                               │       └── tls.yml
    │                                                   ├── secrets
    server3                                             │   ├── cloudflare_api_key.secret
    │   └── docker                                      │   └── cloudflare_email.secret
    │       ├── Bitwarden                               ├── shared
    │       ├── NextCloud                               │   └── .htpasswd
    │       └── WordPress                               ├── tmp
    │                                                   │   ├── example1.yml
    server4                                             │   └── example2.yml
    │   └── docker                                      ├── .env  
    │       ├── Joomla                                  ├── access.log  
    │       └── Ghost                                   ├── traefik.log  
    │                                                   └── docker-compose.yml 
    └─bastion                                                    
        └── ansible                                            
                                                    
</pre>

<a href="https://github.com/vdarkobar/Home_Cloud#proxmox">top of the page</a>

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
