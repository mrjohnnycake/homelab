Create VM as per usual w/ Ubuntu Live Server

* during the install process, select minimal install


# Environment Setup #

```
sudo apt update
```

In Unifi, set the correct static IP

Reboot

- Confirm correct IP address with `ip r`
- Setup SSH access as per note

```
sudo apt install locales

sudo dpkg-reconfigure locales

sudo dpkg-reconfigure --priority=low unattended-upgrades
```

```
sudo timedatectl set-timezone America/Los_Angeles

timedatectl
```

```
sudo apt upgrade
```

If some packages say that they "have been kept back", run this:
```
sudo apt-get install [copy list packages from "sudo apt upgrade" output]
```

```
sudo apt install cifs-utils docker.io docker-compose fail2ban less nano smbclient
```

```
sudo systemctl enable fail2ban --now

sudo systemctl enable docker

sudo systemctl start docker
```

```
sudo nano /etc/ssh/sshd_config
```

Change this line to look like this (need to uncomment)
```
# Authentication:
PermitRootLogin no
```

Deploy Watchtower to auto-update container images @ 4:30a every day
```
docker run --name watchtower -e TZ="America/Los_Angeles" -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped containrrr/watchtower --debug --cleanup --schedule "0 30 4 * * *"
```

Connect to Portainer via Agent per Docker note



## Storage Permissions ##

### Create Users ###

* Create users WITHOUT home directories (capital M)
```
sudo useradd -u 1010 -M dave
sudo useradd -u 1061 -M plex
```

* Create groups
```
sudo groupadd -g 1021 plex-movies
sudo groupadd -g 1022 plex-music
sudo groupadd -g 1023 plex-shows
sudo groupadd -g 1024 books
sudo groupadd -g 1025 photos
sudo groupadd -g 1050 torrents
sudo groupadd -g 1051 usenet
sudo groupadd -g 1052 yt-music
sudo groupadd -g 1053 yt-video
```

```
sudo usermod -aG administrator dave
sudo usermod -aG administrator plex
sudo usermod -aG books dave
sudo usermod -aG dave plex
sudo usermod -aG docker plex
sudo usermod -aG photos dave
sudo usermod -aG plex dave
sudo usermod -aG plex plex
sudo usermod -aG plex-movies dave
sudo usermod -aG plex-movies plex
sudo usermod -aG plex-music dave
sudo usermod -aG plex-music plex
sudo usermod -aG plex-shows dave
sudo usermod -aG plex-shows plex
sudo usermod -aG torrents dave
sudo usermod -aG usenet dave
sudo usermod -aG yt-music dave
sudo usermod -aG yt-video dave
```

```
sudo mkdir /mnt/Media
sudo chown -R dave:administrator /mnt/Media
sudo chmod 775 /mnt/Media
```



## Mount SMB Share ##

Automount at startup
```
sudo nano /root/.smb
```

```
user=dave
password=enter-password-here
```
* Save and exit

```
sudo nano /etc/fstab
```

Add this to the bottom:
```
# SMB Mounts
//192.168.10.5/Deep_13_Media_Audio /mnt/audio cifs credentials=/root/.smb,uid=1010,gid=1000 0 0
//192.168.10.5/Deep_13_Media_Video /mnt/video cifs credentials=/root/.smb,uid=1010,gid=1000 0 0
```

Restart


# Services #


```
version: "2.1"
services:
  plex:
    image: lscr.io/linuxserver/plex:latest
    container_name: plex
    network_mode: host
    environment:
      - PUID=1061
      - PGID=1061
      - VERSION=docker
      - PLEX_CLAIM=claim-tUcFSx9QP-76ggRELExz
    volumes:
      - /docker/appdata/plex/config:/config
      - /mnt/video/movies:/movies
      - /mnt/audio/music:/music
	  - /mnt/video/shows:/tv
    ports:
      - 32400:32400
    restart: unless-stopped
```



