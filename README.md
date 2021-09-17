# homeServer

## Summary

| Mediaserver contains: | Portwatch contains: | AdGuard contains: |
|---------------------|-----------------------|-------------------|
| Plex                | Portainer             | AdGuard Home      |
| Tautulli            | Watchtower            |
| Gluetun             |
| qBittorrent         |
| Prowlarr            |
| Radarr              |
| Sonarr              |

## Docker Setup
<!-- Debian maintained version, does not support rootless -->
Install Docker
```
sudo apt install -y docker.io docker-compose
```
Add your user to the Docker group
```
sudo usermod -aG docker $(whoami)
```
Reboot the server to complete installation
```
sudo reboot
```

<!-- Install script for Raspian
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
-->


<!-- Docker-ce, supports rootless, buggy
### Install Docker
Install prerequisites
```
sudo apt install -y apt-transport-https ca-certificates curl gnupg lsb-release
```
Install GPG Key
```
curl -fsSL https://download.docker.com/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
Add repo to apt
```
echo \
"deb [arch=$( dpkg --print-architecture ) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/debian \
$(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list
```
Update apt and install Docker
```
sudo apt update && sudo apt install -y docker-ce docker-ce-cli containerd.io docker-compose
```
Add your user to the Docker group
```
sudo groupadd docker
```
```
sudo usermod -aG docker $(whoami)
```
Reboot the server to complete installation
```
sudo reboot
```

### Setup Rootless
Install dependencies
```
sudo apt install -y uidmap dbus-user-session fuse-overlayfs slirp4netns
```
Disable system-wide docker
```
sudo systemctl disable --now docker.service docker.sock
```
Run the setup script
```
/usr/bin/dockerd-rootless-setuptool.sh install
```
Update .bashrc
```
echo 'export PATH=/usr/bin:$PATH' >> ~/.bashrc
echo "export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock" >> ~/.bashrc
source ~/.bashrc
```
-->
<!-- docker socket can by found using:
$XDG_RUNTIME_DIR/docker.sock
export DOCKER_HOST=unix:///run/user/1000/docker.sock
-->
<!--
Configure services
```
systemctl --user start docker
systemctl --user enable docker
```
```
sudo loginctl enable-linger $(whoami)
```
### Finish Setup
-->
Clone the repo
```
cd ~/ && git clone https://github.com/Rockz1152/homeServer.git && cd homeServer
```
Update the `.env` file before bringing up any services
```
nano .env
```

## Portainer and Watchtower
_The default configuration for portainer and watchtower will automatically keep them up-to-date_
### Installation
```
docker-compose -p "PortWatch" -f portwatch.yml up -d
```
Repairing
- _If you accidentally remove Portainer or Watchtower, use these commands to repair them_
```
cd ~/homeServer
docker stop portainer
docker stop watchtower
docker rm portainer
docker rm watchtower
docker-compose -p "PortWatch" -f portwatch.yml up -d
```

<!--
#### Using Docker
Install Portainer
 - _*Running Portainer and Watchtower with docker instead of docker-compose will prevent them from showing up as an unmanged stack inside portainer_
```
docker run -d \
-p 8000:8000 \
-p 9000:9000 \
--name=portainer \
--restart=always \
--privileged \
--label "owner=portainer" \
--label "com.centurylinklabs.watchtower.enable=true" \
-v /var/run/docker.sock:/var/run/docker.sock \
-v $(source .env; echo ${DATADIR})/portainer:/data \
portainer/portainer-ce \
--hide-label owner=portainer
```
Install Watchtower
```
docker run -d \
--name watchtower \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /etc/localtime:/etc/localtime:ro \
--label "owner=portainer" \
--label "com.centurylinklabs.watchtower.enable=true" \
containrrr/watchtower \
--cleanup \
--include-restarting \
--label-enable
```
-->

## Services

### Using Docker-Compose

#### Media Server
_*Make sure you fill in all the directory paths in the .env file or the stack will fail to start_
```
docker-compose -p "Mediaserver" -f mediaserver.yml up -d
```

#### AdGuard
```
docker-compose -p "AdGuard Home" -f adguard.yml up -d
```

### Using Portainer
:warning: **Warning**

**Portainer has issues parsing .env files that contain public and private keys that Wireguard uses. Wait until after you've loaded the .env file and use the web editor to enter the correct values**

- In Portainer goto Stacks and click `+ Add stack`
- Give the stack a name. e.g. `Mediaserver`
- Paste the contents of the .yml file into the web editor
- Scroll down, under "Environment variables" click `Load variables from .env file`
- Select your .env file
- Use the web editor to make sure each variable has the correct value
- When everything looks correct, under "Actions" clikc `Deploy the stack`
