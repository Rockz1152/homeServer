# homeServer

## Installation
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

## Services
Clone the repo
```
git clone https://github.com/Rockz1152/homeServer.git && cd homeServer
```
Update the `.env` file before bringing up any services
```
nano .env
```

### Portainer
```
docker run -d -p 8000:8000 -p 9000:9000 --name=portainer --restart=always --privileged -v /var/run/docker.sock:/var/run/docker.sock -v `source .env; echo ${DATADIR}`/portainer:/data portainer/portainer-ce
```

### Pi-Hole
```
docker-compose -f pihole.yml up -d
```
