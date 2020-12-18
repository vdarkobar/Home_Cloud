# Traefik
## Traefik Reverse Proxy

##### Create docker network
```
sudo docker network create proxy
```

### Clone this git repository:
```
git clone https://vdarkobar:2211620c9da5dab0c7bb77e9aeb02087d293b293@github.com/vdarkobar/proxy.git
# set permissions
sudo chmod 600 ~/proxy/data/acme.json
```

##### Change domain name
```
sudo nano proxy/docker-compose.yml
```
##### Start
```
sudo docker-compose -f proxy/docker-compose.yml up -d
```
##### Log
```
sudo docker logs -tf --tail="50" traefik
```
