# Traefik2
## Traefik Reverse Proxy

##### Create docker network
```
sudo docker network create traefik
```
### Clone this git repository:
```
echo -n "Enter directory name: "; read NAME; mkdir -p "$NAME"; cd "$NAME" \
&& git clone https://vdarkobar:2211620c9da5dab0c7bb77e9aeb02087d293b293@github.com/vdarkobar/Traefik2.git .
```
#### Set permissions
```
sudo chmod 600 data/acme.json
sudo chown -R root:root secrets/
```
##### Change domain name
```
sudo nano docker-compose.yml
```
##### Start
```
sudo docker-compose up -d        # If needed: sudo docker-compose up -d --force-recreate
```
##### Log
```
sudo docker-compose logs traefik
sudo docker logs -tf --tail="50" traefik
```
