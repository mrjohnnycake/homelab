https://wiki.debian.org/ZFS

## Pool and Datasets ##
* In the Proxmox GUI, wipe the disks that are to be used

* Create the pool:

```
zpool create -o ashift=12 -m /mnt/deep-ape Deep-Ape raidz /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD4AL2Z /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD7101N /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD6YG1K
```

* Set pool options

```
zfs set compression=lz4 xattr=sa dnodesize=auto Deep-Ape
```

* Create Media datasets and all associated datasets

```
zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/deep-ape/media Deep-Ape/Media

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/deep-ape/media/audio Deep-Ape/Media/Audio

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/deep-ape/media/books Deep-Ape/Media/Books

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/deep-ape/media/downloaders Deep-Ape/Media/Downloaders

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/deep-ape/media/video Deep-Ape/Media/Video
```

* Create associated folders

```
mkdir /mnt/deep-ape/media/audio/music
mkdir /mnt/deep-ape/media/audio/other
mkdir /mnt/deep-ape/media/books/audiobooks
mkdir /mnt/deep-ape/media/books/ebooks
mkdir /mnt/deep-ape/media/downloaders/metatube
mkdir /mnt/deep-ape/media/downloaders/torrents
mkdir /mnt/deep-ape/media/downloaders/usenet
mkdir /mnt/deep-ape/media/video/movies
mkdir /mnt/deep-ape/media/video/tv-shows
```

## SMB/CIFS ##

```
apt install samba
systemctl enable smbd
```

```
zfs set sharesmb=on Deep-Ape/Media
zfs set sharesmb=on Deep-Ape/Media/Audio
zfs set sharesmb=on Deep-Ape/Media/Books
zfs set sharesmb=on Deep-Ape/Media/Downloaders
zfs set sharesmb=on Deep-Ape/Media/Video
```

```
zfs share Deep-Ape/Media
zfs share Deep-Ape/Media/Audio
zfs share Deep-Ape/Media/Books
zfs share Deep-Ape/Media/Downloaders
zfs share Deep-Ape/Media/Video
```


Create a SMB user
```
adduser dave-deepape
usermod -aG sambashare dave-deepape
smbpasswd -a dave-deepape
```

## Permissions ##

Set permissions for the share

```
chown -R dave-deepape:media /mnt/deep-ape/media
```


## Connection ##

Restart samba to get everything working

```
service smbd restart
```

* Open File Explorer and go to the Network tab
* In the location bar where it says Network, paste this:

```
\\192.168.10.10\Deep_13_NAS_Dave
```


## NFS ##

- Not currently using NFS


# Maintenance #

## Replacing Failed Drives ##

- Check which disk is unavailable
```
zpool status -v
```

- In this last instance, the faulted disk was WX52D32D5L5D

- Take the faulted disk offine
```
zpool offline Deep-13 /dev/disk/by-id/ata-WDC_WD40EFZX-68AWUN0_WD-WX52D32D5L5D-part1
```

- Shutdown the server and replace the faulted drive with the new one
	- Take note of the new disk's serial number. In this case, the new disk is WX72D32NCU7L

- Start the server and log back into the CLI

- Run this to replace the failed drive (the first disk listed in the command) with the new disk (the second disk listed in the command)
```
zpool replace Deep-13 /dev/disk/by-id/ata-WDC_WD40EFZX-68AWUN0_WD-WX52D32D5L5D-part1 /dev/disk/by-id/ata-WDC_WD40EFZX-68AWUN0_WD-WX72D32NCU7L
```

- Let it go thru the resilvering process (about the same amount of time as a scrub)


## Managing Snapshots ##

- List all snapshots
```
zfs list -t snapshot
```

Delete a snapshot (edit end of command with exact snapshot name)
```
zfs destroy Deep-13/Backups@zfs...
```

Destroy all snapshots from a particular dataset
```
sudo -i

zfs list -H -o name -t snapshot Deep-13/Media/Video | xargs -n1 zfs destroy
```


Destroy all snapshots from entire pool
```
sudo -i

zfs list -H -o name -t snapshot | xargs -n1 zfs destroy
```