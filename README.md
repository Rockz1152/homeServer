# homeServer

## Docker Setup
Install docker
```
sudo apt install -y docker.io docker-compose
```
Enable docker service
```
sudo systemctl enable docker && sudo systemctl start docker && systemctl status --no-pager docker
```
Add your user to the Docker group
```
sudo usermod -aG docker $(whoami)
```
Reboot the server to complete installation
```
sudo reboot
```

----

## Portainer

### Starting Portainer
_*Running portainer with docker instead of docker-compose will prevent portainer from showing up as an unmanged stack inside itself_
Clone the repo
```
cd ~/ && git clone https://github.com/Rockz1152/homeServer.git && cd homeServer
```
Update the `.env` file before bringing up any services
```
nano .env
```
Install Portainer
```
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always --privileged -v /var/run/docker.sock:/var/run/docker.sock -v $(source .env; echo ${DATADIR})/portainer:/data portainer/portainer-ce
```

### Updating
```
cd ~/homeServer
docker stop Portainer
docker rm Portainer
docker pull portainer/portainer-ce
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always --privileged -v /var/run/docker.sock:/var/run/docker.sock -v $(source .env; echo ${DATADIR})/portainer:/data portainer/portainer-ce
```

----

## Services

### AdGuard
```
docker-compose -f adguard.yml up -d
```

### Media Server

...

- Plex
- Tautulli
- qBittorrent-VPN 

- Radarr
- Sonarr

...

----

### Using Portainer
- "Load variables from .env file"
