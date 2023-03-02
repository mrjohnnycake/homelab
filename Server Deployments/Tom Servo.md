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

Enter the 8TB drives paths into the command below to build the zpool (use all five disks)
```
zpool create -o ashift=12 -m /mnt/Deep-13 Deep-13 raidz2 /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD4AL2Z /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD6YG1K /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD7101N /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD7ZQ7W /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD868K6
```

* Set pool options
```
zfs set compression=lz4 xattr=sa dnodesize=auto Deep-13
```

* Create the ZIL
```
zpool add -f Deep-13 log mirror /dev/disk/by-id/ata-KINGSTON_SA400S37240G_50026B7685576DE8 /dev/disk/by-id/ata-KINGSTON_SA400S37240G_50026B7685576FAA
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
mkdir /mnt/Deep-13/Anna/Desktop
mkdir /mnt/Deep-13/Anna/Documents
mkdir /mnt/Deep-13/Anna/Downloads
mkdir /mnt/Deep-13/Anna/Pictures

mkdir /mnt/Deep-13/Media/Audio/music
mkdir /mnt/Deep-13/Media/Audio/other

mkdir /mnt/Deep-13/Media/Books/audiobooks
mkdir /mnt/Deep-13/Media/Books/ebooks

mkdir /mnt/Deep-13/Media/Downloads/yt-music
mkdir /mnt/Deep-13/Media/Downloads/yt-video
mkdir /mnt/Deep-13/Media/Downloads/torrents
mkdir /mnt/Deep-13/Media/Downloads/usenet

mkdir /mnt/Deep-13/Media/Downloads/usenet/downloads
mkdir /mnt/Deep-13/Media/Downloads/usenet/incoming
mkdir /mnt/Deep-13/Media/Downloads/usenet/intermediate
mkdir /mnt/Deep-13/Media/Downloads/usenet/movies
mkdir /mnt/Deep-13/Media/Downloads/usenet/music
mkdir /mnt/Deep-13/Media/Downloads/usenet/tmp
mkdir /mnt/Deep-13/Media/Downloads/usenet/shows

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
zfs set sharesmb=on Deep-13/Media
```

Create SMB users
```
useradd -u 1010 -m -s /bin/bash dave
passwd dave
```

Set user as an SMB user
```
usermod -aG sambashare dave
smbpasswd -a dave
```



# Set Permissions #

* Create users WITH home directories (lowercase m)
```
useradd -u 1000 -m -s /bin/bash administrator
passwd administrator
```

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
usermod -aG sudo administrator
usermod -aG administrator dave
usermod -aG administrator root
usermod -aG nas dave
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

chown -R anna:anna /mnt/Deep-13/NAS/Anna
chown -R emmett:emmett /mnt/Deep-13/NAS/Emmett
chown -R dave:nas /mnt/Deep-13/NAS/Dave
```

```
find /mnt/Deep-13/ -type d -exec chmod 775 {} \;
```


# Transfer Files #

From Crow
```
rsync -ruv --info=progress /mnt/deep-ape/backups/nas/ root@192.168.10.5:/mnt/Deep-13/NAS/

chown -R dave:nas /mnt/Deep-13/NAS
find /mnt/Deep-13/NAS -type d -exec chmod 775 {} \;
find /mnt/Deep-13/NAS -type f -exec chmod 664 {} \;
```

```
rsync -ruv --info=progress /mnt/deep-ape/backups/media/audio/music/ root@192.168.10.5:/mnt/Deep-13/Media/Audio/music/

chown -R dave:plex-music /mnt/Deep-13/Media/Audio/music
find /mnt/Deep-13/Media/Audio/music -type d -exec chmod 775 {} \;
find /mnt/Deep-13/Media/Audio/music -type f -exec chmod 664 {} \;
```

```
rsync -ruv --info=progress /mnt/deep-ape/backups/media/audio/other/ root@192.168.10.5:/mnt/Deep-13/Media/Audio/other/

chown -R dave:administrator /mnt/Deep-13/Media/Audio/other
find /mnt/Deep-13/Media/Audio/other -type d -exec chmod 775 {} \;
find /mnt/Deep-13/Media/Audio/other -type f -exec chmod 664 {} \;
```

```
rsync -ruv --info=progress /mnt/deep-ape/backups/media/books/audiobooks/ root@192.168.10.5:/mnt/Deep-13/Media/Books/audiobooks/

chown -R dave:books /mnt/Deep-13/Media/Books/audiobooks/
find /mnt/Deep-13/Media/Books/audiobooks -type d -exec chmod 775 {} \;
find /mnt/Deep-13/Media/Books/audiobooks -type f -exec chmod 664 {} \;
```

```
rsync -ruv --info=progress /mnt/deep-ape/backups/media/books/ebooks/ root@192.168.10.5:/mnt/Deep-13/Media/Books/ebooks/

chown -R dave:books /mnt/Deep-13/Media/Books/ebooks/
find /mnt/Deep-13/Media/Books/ebooks -type d -exec chmod 775 {} \;
find /mnt/Deep-13/Media/Books/ebooks -type f -exec chmod 664 {} \;
```

```
rsync -ruv --info=progress /mnt/deep-ape/websites/eurekadaycare/ root@192.168.10.5:/mnt/Deep-13/Websites/Daycare/

chown -R dave:administrator /mnt/Deep-13/Websites/Daycare
find /mnt/Deep-13/Websites/Daycare -type d -exec chmod 775 {} \;
find /mnt/Deep-13/Websites/Daycare -type f -exec chmod 664 {} \;
```

```
rsync -ruv --info=progress /mnt/deep-ape/backups/media/video/movies/ root@192.168.10.5:/mnt/Deep-13/Media/Video/movies/

chown -R dave:plex-movies /mnt/Deep-13/Media/Video/movies
find /mnt/Deep-13/Media/Video/movies -type d -exec chmod 775 {} \;
find /mnt/Deep-13/Media/Video/movies -type f -exec chmod 664 {} \;
```

```
rsync -ruv --info=progress /mnt/deep-ape/backups/media/video/shows/ root@192.168.10.5:/mnt/Deep-13/Media/Video/shows/

chown -R dave:plex-shows /mnt/Deep-13/Media/Video/shows
find /mnt/Deep-13/Media/Video/shows -type d -exec chmod 775 {} \;
find /mnt/Deep-13/Media/Video/shows -type f -exec chmod 664 {} \;
```

===========================================



zpool replace Deep-13 /dev/disk/by-id/ata-WDC_WD40EFZX-68AWUN0_WD-WX22D80J2X9L /dev/disk/by-id/ata-WDC_WD40EFZX-68AWUN0_WD-WX42AB1NPF15



 8644450603271339867    UNAVAIL   0   0   0  was /dev/disk/by-id/ata-WDC_WD40EFZX-68AWUN0_WD-WX22D80J2X9L-part1
 
16398014302528060571    UNAVAIL   0   0   0  was /dev/disk/by-id/ata-WDC_WD40EFZX-68AWUN0_WD-WX52D710HC12-part1