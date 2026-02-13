# Raspberry Pi

## Installing Raspian
raspberrypi.com offers a tool called "Raspberry Pi Imager" that will download and setup an SD card for your pi

- It can be downloaded from here https://www.raspberrypi.com/software/
- It gives you the choice of picking an OS during the setup process
  - For a minimal install, select `Raspberry Pi OS (other)` and then `Raspberry Pi OS Lite (64-bit)`
  - The imager allows you to set a hostname, username, password, connect wifi, and enable SSH
- If you are running Linux, you can install it using `sudo apt install rpi-imager`

## Packages

Install Base packages
```
sudo apt update; sudo apt install -y curl nano htop ncdu wget
```

Install updates
```
sudo apt upgrade -y
```

### Update Script
Make a script that can be launched as `~/update-system.sh` to install system updates
```
echo 'sudo sh -c "apt update;apt upgrade -y;apt autoremove --purge -y;"' > $HOME/update-system.sh; chmod u+x $HOME/update-system.sh
```

## Networking

### Disable IPv6
This is optional
```
echo 'net.ipv6.conf.all.disable_ipv6 = 1' | sudo tee -a /etc/sysctl.d/99-disableipv6.conf && sudo sysctl --system
```
```
sudo reboot now
```

### Static IP

Since Raspian 12 the pi uses NetworkManager. You can use `nmtui` to set addresses interactively
```
sudo nmtui
```

Manually configure Netplan
```
sudo nano /etc/netplan/90-NM-******.yaml
```
```
network:
  version: 2
  ethernets:
    eth0:
      renderer: NetworkManager
      match: {}
      addresses:
        - "192.168.0.XX/24"
      nameservers:
        addresses:
        - 1.1.1.1
        - 8.8.8.8
      routes:
        - to: "0.0.0.0/0"
          via: 192.168.0.1
      networkmanager:
        uuid: "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
        name: "netplan-eth0"
        passthrough:
          proxy._: ""
```
  - _*When editing Yaml files, make sure you follow the YAML code indent standards._
  - _*If the syntax is not correct, the changes will not be applied._

Be sure to update the following:

- Under "ethernets" update `eth0` with the name of your network interface
- Under "addresses" update `192.168.0.XX` with your preferred host IP address
- Under "routes" update `192.168.0.1` to your default gateway
- Leave all the settings under `networkmanager:` as is on your system

Apply and verify the changes by running:
```
sudo netplan apply; sudo reboot
```

## Mount a USB Drive

### Use an existing filesystem
Determine filesystem of drive
```
lsblk; sudo blkid /dev/sdX#
```

- Example output
  - `/dev/sda1: LABEL="My Passport" UUID="8A2CF4F62CF4DE5F" TYPE="ntfs" PTTYPE="atari" PARTUUID="00042ada-01"`
  - Make note of the values for `UUID` and `Type`

Install appropriate files to read drive

- NTFS `sudo apt install ntfs-3g`
- exFAT `sudo apt install exfat-fuse`

### Format the external drive instead
Install exFAT driver
```
sudo apt install exfat-fuse
```

Partition the drive
```
sudo cfdisk /dev/sdX
```

- Just remove the current partition and create a new one, don't worry about the filesystem yet

Format with exFAT
```
sudo mkfs.exfat /dev/sdX#
```

Find the `UUID` of the partition and make note of it
```
sudo blkid /dev/sdX#
```

### Mount the partition

Create a mount point
```
sudo mkdir -p /mnt/usb1
```

Edit the `fstab` and create an entry for the drive
```
sudo nano /etc/fstab
```

Add to the bottom:

- NTFS or ext4 
```
UUID=[UUID] /mnt/usb1 [filesystem] defaults,nofail,noatime 0 0
```
- exFAT
```
UUID=[UUID] /mnt/usb1 exfat defaults,nofail,noatime,uid=1000,gid=1000,umask=007 0 0
```
-
  - Substitute the `[UUID]`value for the one from `sudo blkid /dev/sdXX`, and `[filesystem]` as well
  - For exFAT, substitute `1000` in `uid` and `gid` with your user's from `id`

Reload systemd
```
sudo systemctl daemon-reload
```

Mount the drive and check the contents
```
sudo mount -av
```

Verify the mount
```
mount
```

Reboot the system
```
sudo reboot
```
