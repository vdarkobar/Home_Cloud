# Traefik
## Traefik Reverse Proxy

### Create custum network 
sudo docker network create --gateway 192.168.90.1 --subnet 192.168.90.0/24 proxy

### Set variables in .env


### Clone this git repository:
```
git clone https://vdarkobar:2211620c9da5dab0c7bb77e9aeb02087d293b293@github.com/vdarkobar/proxy.git
# set permissions
sudo chmod 600 ~/proxy/data/acme.json
```
##### Start and check:
```
sudo docker-compose up -d
sudo docker logs -tf --tail="50" traefik
```
