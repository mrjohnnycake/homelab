# ZFS Setup #

* In Proxmox, wipe the disks that will be used for the zpool

* Get the dev/disk names

	* First use this to get the model and serial numbers
	```
	lsblk -o +MODEL,SERIAL
	```
	* Make sure to get the 240G disks path for the ZIL

	* Next, get their full path with:
	```
	ls /dev/disk/by-id
	```

Enter the 8TB drives paths into the command below to build the zpool (use all disks)
```
zpool create -o ashift=12 -m /mnt/Deep-13 Deep-13 raidz2 /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD864HJ /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD8676W /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD867DV /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD868A3 /dev/disk/by-id/ata-ST8000VN004-3CP101_WWZ1MGLD
```

* Set pool options
```
zfs set compression=lz4 xattr=sa dnodesize=auto Deep-13
```

* Create the ZIL
```
zpool add -f Deep-13 log mirror /dev/disk/by-id/ata-KINGSTON_SA400S37240G_50026B7685576566 /dev/disk/by-id/ata-KINGSTON_SA400S37240G_50026B778485E8CC
```


## Create datasets ##

```
zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/Backups Deep-13/Backups

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/Backups/Crow Deep-13/Backups/Crow

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/Backups/Tom-Servo Deep-13/Backups/Tom-Servo

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/Media Deep-13/Media

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/Media/Audio Deep-13/Media/Audio

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/Media/Books Deep-13/Media/Books

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/Media/Games Deep-13/Media/Games

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/Media/Photos Deep-13/Media/Photos

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/Media/Video Deep-13/Media/Video

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/NAS Deep-13/NAS

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/NAS/Anna Deep-13/NAS/Anna

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/NAS/Dave Deep-13/NAS/Dave

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Deep-13/NAS/Emmett Deep-13/NAS/Emmett
```

## Create associated folders ##

```
mkdir /mnt/Deep-13/Backups/Crow/appdata
mkdir /mnt/Deep-13/Backups/Crow/files

mkdir /mnt/Deep-13/Backups/Tom-Servo/appdata
mkdir /mnt/Deep-13/Backups/Tom-Servo/files

mkdir /mnt/Deep-13/Media/Audio/music
mkdir /mnt/Deep-13/Media/Audio/other

mkdir /mnt/Deep-13/Media/Books/audiobooks
mkdir /mnt/Deep-13/Media/Books/ebooks

mkdir /mnt/Deep-13/Media/Photos/backups

mkdir /mnt/Deep-13/Media/Video/movies
mkdir /mnt/Deep-13/Media/Video/shows
```

```
zpool status
zfs list
```

```
apt-get install tree
tree /mnt/Deep-13
```



# SMB Setup #

```
apt install samba
systemctl enable smbd
```

```
zfs set sharesmb=on Deep-13/NAS
```


Create users (lowercase m creates users WITH home directories)
```
useradd -u 1000 -m -s /bin/bash administrator
passwd administrator

useradd -u 1010 -m -s /bin/bash dave
passwd dave

useradd -u 1011 -m -s /bin/bash emmett
passwd emmett

useradd -u 1012 -m -s /bin/bash anna
passwd anna
```

Set user as an SMB user
```
usermod -aG sambashare administrator
smbpasswd -a administrator

usermod -aG sambashare anna
smbpasswd -a anna

usermod -aG sambashare dave
smbpasswd -a dave

usermod -aG sambashare emmett
smbpasswd -a emmett
```
* `administrator` needs SMB access for Restic VM


# Set Permissions #

* Create users WITHOUT home directories (capital M)
```
useradd -u 1050 -M torrents
useradd -u 1051 -M usenet
useradd -u 1052 -M yt-music
useradd -u 1053 -M yt-video
useradd -u 1054 -M radarr
useradd -u 1055 -M sonarr
useradd -u 1056 -M lidarr
useradd -u 1057 -M readarr
useradd -u 1058 -M bazarr
useradd -u 1059 -M pmm
useradd -u 1060 -M archiver
useradd -u 1061 -M plex
useradd -u 1064 -M syncthing
```

* Create groups
```
groupadd -g 1020 nas
groupadd -g 1021 plex-movies
groupadd -g 1022 plex-music
groupadd -g 1023 plex-shows
groupadd -g 1024 books
groupadd -g 1025 photos
```

```
usermod -aG administrator dave
usermod -aG administrator root
usermod -aG nas dave
usermod -aG anna dave
usermod -aG emmett dave
usermod -aG books dave
usermod -aG plex-movies dave
usermod -aG plex-movies plex
usermod -aG plex-music dave
usermod -aG plex-music plex
usermod -aG plex-shows dave
usermod -aG plex-shows plex
usermod -aG photos dave
usermod -aG torrents dave
usermod -aG usenet dave
usermod -aG yt-music dave
usermod -aG yt-video dave
usermod -aG plex dave
```

```
chown -R administrator:administrator /mnt/Deep-13/Backups

chown dave:administrator /mnt/Deep-13/Media

chown dave:administrator /mnt/Deep-13/Media/Audio
chown -R dave:plex-music /mnt/Deep-13/Media/Audio/music
chown -R dave:administrator /mnt/Deep-13/Media/Audio/other

chown dave:administrator /mnt/Deep-13/Media/Books
chown -R dave:books /mnt/Deep-13/Media/Books/audiobooks
chown -R dave:books /mnt/Deep-13/Media/Books/ebooks

chown -R dave:administrator /mnt/Deep-13/Media/Games

chown dave:administrator /mnt/Deep-13/Media/Photos
chown -R dave:photos /mnt/Deep-13/Media/Photos/backups

chown dave:administrator /mnt/Deep-13/Media/Video
chown -R dave:plex-movies /mnt/Deep-13/Media/Video/movies
chown -R dave:plex-shows /mnt/Deep-13/Media/Video/shows

chown -R administrator:administrator /mnt/Deep-13/NAS

chown -R anna:anna /mnt/Deep-13/NAS/Anna

chown -R dave:nas /mnt/Deep-13/NAS/Dave

chown -R emmett:emmett /mnt/Deep-13/NAS/Emmett
```

```
find /mnt/Deep-13/ -type d -exec chmod 775 {} \;
```


# Snapshots #

Turn on/off snapshots for datasets:
```
sudo zfs set com.sun:auto-snapshot=true Deep-13
sudo zfs set com.sun:auto-snapshot=false Deep-13/Backups
sudo zfs set com.sun:auto-snapshot=false Deep-13/Backups/Crow
sudo zfs set com.sun:auto-snapshot=false Deep-13/Backups/Tom-Servo
sudo zfs set com.sun:auto-snapshot=false Deep-13/Media
sudo zfs set com.sun:auto-snapshot=true Deep-13/Media/Audio
sudo zfs set com.sun:auto-snapshot=true Deep-13/Media/Books
sudo zfs set com.sun:auto-snapshot=true Deep-13/Media/Games
sudo zfs set com.sun:auto-snapshot=true Deep-13/Media/Photos
sudo zfs set com.sun:auto-snapshot=true Deep-13/Media/Video
sudo zfs set com.sun:auto-snapshot=false Deep-13/NAS
sudo zfs set com.sun:auto-snapshot=true Deep-13/NAS/Anna
sudo zfs set com.sun:auto-snapshot=true Deep-13/NAS/Dave
sudo zfs set com.sun:auto-snapshot=true Deep-13/NAS/Emmett
```

