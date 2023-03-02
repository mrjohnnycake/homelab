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

sudo apt update (repeat as necessary)
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

Install Portainer as per Docker note



## Storage Permissions ##

### Create Users ###

* Create users WITHOUT home directories (capital M)
```
sudo useradd -u 1010 -M dave
sudo useradd -u 1050 -M torrents
sudo useradd -u 1051 -M usenet
sudo useradd -u 1052 -M yt-music
sudo useradd -u 1053 -M yt-video
sudo useradd -u 1054 -M radarr
sudo useradd -u 1055 -M sonarr
sudo useradd -u 1056 -M lidarr
sudo useradd -u 1057 -M readarr
sudo useradd -u 1058 -M bazarr
sudo useradd -u 1059 -M pmm
sudo useradd -u 1067 -M prowlarr
sudo useradd -u 1068 -M vpn
sudo useradd -u 1070 -M requests
```

* Create groups
```
sudo groupadd -g 1021 plex-movies
sudo groupadd -g 1022 plex-music
sudo groupadd -g 1023 plex-shows
sudo groupadd -g 1024 books
sudo groupadd -g 1025 photos
```

```
sudo usermod -aG administrator dave
sudo usermod -aG administrator radarr
sudo usermod -aG books dave
sudo usermod -aG books readarr
sudo usermod -aG plex-movies dave
sudo usermod -aG plex-movies radarr
sudo usermod -aG plex-movies bazarr
sudo usermod -aG plex-movies pmm
sudo usermod -aG plex-music dave
sudo usermod -aG plex-music lidarr
sudo usermod -aG plex-shows dave
sudo usermod -aG plex-shows sonarr
sudo usermod -aG plex-shows bazarr
sudo usermod -aG plex-shows pmm
sudo usermod -aG photos dave
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
//192.168.10.5/Deep_13_Media /mnt/Media cifs credentials=/root/.smb,uid=1010,gid=1000 0 0
```

Restart


# Services #



