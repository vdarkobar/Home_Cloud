# Traefik2
## Traefik Reverse Proxy

##### Create docker network
```
sudo docker network create proxy
```

### Clone this git repository:
```
git clone https://vdarkobar:2211620c9da5dab0c7bb77e9aeb02087d293b293@github.com/vdarkobar/Traefik2.git
# set permissions
sudo chmod 600 ~/Traefik2/data/acme.json
```

##### Change domain name
```
sudo nano Traefik2/docker-compose.yml
```
##### Start
```
sudo docker-compose -f Traefik2/docker-compose.yml up -d
```
##### Log
```
sudo docker logs -tf --tail="50" traefik
```
