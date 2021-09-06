# Docker

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
Create a directory to store container configs
```
mkdir ~/dockerData
```
Reboot the server to complete installation
```
sudo reboot
```

## Services
Clone the repo
```
git clone https://github.com/Rockz1152/Docker.git && cd Docker
```

### Portainer
```
docker-compose -f portainer.yml up -d
```

### Pi-Hole
```
docker-compose -f pihole.yml up -d
```
