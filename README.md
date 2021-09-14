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

## Portainer and Watchtower

### Installation
Clone the repo
```
cd ~/ && git clone https://github.com/Rockz1152/homeServer.git && cd homeServer
```
Update the `.env` file before bringing up any services
```
nano .env
```

#### Using docker-compose (Recommended)
```
docker-compose -p "portwatch" -f portwatch.yml up -d
```

#### Manually
<sup>_*Running Portainer and Watchtower with docker instead of docker-compose will prevent them from showing up as an unmanged stack inside portainer_</sup>

Install Portainer
```
docker run -d \
-p 8000:8000 \
-p 9000:9000 \
--name=portainer \
--restart=always \
--privileged \
--label "com.centurylinklabs.watchtower.enable=true" \
-v /var/run/docker.sock:/var/run/docker.sock \
-v $(source .env; echo ${DATADIR})/portainer:/data \
portainer/portainer-ce
```
Install Watchtower
```
docker run -d \
--name watchtower \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /etc/localtime:/etc/localtime:ro \
--cleanup \
--include-restarting \
--label-enable \
--label "com.centurylinklabs.watchtower.enable=true" \
containrrr/watchtower
```

### Repairing
_*The default configuration for portainer and watchtower will automatically keep them up-to-date_
If you accidentally remove Portainer or Watchtower, use these commands to repair them
```
cd ~/homeServer
docker stop portainer
docker stop watcher
docker rm portainer
docker rm watchtower
docker-compose -p "portwatch" -f portwatch.yml up -d
```

<!-- old docker run cmd
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always --privileged -v /var/run/docker.sock:/var/run/docker.sock -v $(source .env; echo ${DATADIR})/portainer:/data portainer/portainer-ce
-->

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
