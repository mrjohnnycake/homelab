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
zpool create -o ashift=12 -m /mnt/Backups Backups raidz /dev/disk/by-id/ata-WDC_WD40EFZX-68AWUN0_WD-WX12D91JC524 /dev/disk/by-id/ata-WDC_WD40EFZX-68AWUN0_WD-WX22D51DKDUL /dev/disk/by-id/ata-WDC_WD40EFZX-68AWUN0_WD-WX22DC1K2S5X /dev/disk/by-id/ata-WDC_WD40EFZX-68AWUN0_WD-WX42AB1NPF15 /dev/disk/by-id/ata-WDC_WD40EFZX-68AWUN0_WD-WX72D32NCU7L /dev/disk/by-id/ata-WDC_WD40EFZX-68AWUN0_WD-WX92DA0J22F5
```

* Set pool options

```
zfs set compression=lz4 xattr=sa dnodesize=auto Backups
```


## Create datasets ##

```
zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Backups/Everything-Else Backups/Everything-Else

zfs create -o casesensitivity=mixed -o xattr=sa -o dnodesize=auto -o mountpoint=/mnt/Backups/Video Backups/Video
```

```
mkdir /mnt/Backups/Video/movies

mkdir /mnt/Backups/Video/shows
```

```
chown administrator:administrator /mnt/Backups/Everything-Else

chown -R administrator:administrator /mnt/Backups/Video
```



# Set Permissions #

* Create users WITH home directories (lowercase m)
```
useradd -u 1000 -m -s /bin/bash administrator
passwd administrator
```

```
usermod -aG sambashare administrator
usermod -aG sudo administrator
```

```
smbpasswd -a administrator
```
* `administrator` needs samba privileges for the Backups VM

```
apt install sudo
```
* You need to install this to be able to login as `administrator`

- Now switch to using `administrator` instead of `root` by following the [SSH Setup](SSH%20Setup.md)



# Setup Environment #

```
sudo dpkg-reconfigure locales
```

```
sudo apt update

sudo apt upgrade

sudo apt install fail2ban net-tools samba -y

sudo systemctl enable fail2ban --now

sudo systemctl enable smbd
```

```
sudo zfs set sharesmb=on Backups
sudo zfs set sharesmb=on Backups/Everything-Else
sudo zfs set sharesmb=on Backups/Video
```






===============================================================


Deploy Watchtower to auto-update container images @ 4:30a every day
```
docker run --name watchtower -e TZ="America/Los_Angeles" -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped containrrr/watchtower --debug --cleanup --schedule "0 30 4 * * *"
```

Restart

* Create users WITHOUT home directories (capital M)
```
useradd -u 1063 -M minio
```

```
chown -R minio:minio /mnt/Backups/MinIO
```



# MinIO #

[Install Video](https://www.youtube.com/watch?v=2iVhfbrP_-o)


## Setup Container ##

* Download an LTS version of Ubuntu template in Node-->local-->CT Templates

* Node-->Create CT

```
CT ID: 100
Hostname: MinIO
Uncheck "Unprivileged container"
Enter password and confirm
Choose template
Disk size: 32GB (overkill but whatever)
Cores: 6
Memory: 16384
Set IPv4 and IPv6 to DHCP
Finish
```


From Proxmox:
```
nano /etc/pve/lxc/100.conf
```

* Add this to the bottom:
```
mp0: /mnt/Backups/MinIO,mp=/mnt/MinIO
```

* 100 (MinIO) CT-->Options-->Start at boot-->Check box
* Start container

```
lxc-attach --name 100
```

Create user
```
useradd -u 1063 -M --shell /sbin/nologin --system minio
```

Set minio user's group to 997
```
groupmod -g 997 minio
```



## Install MinIO ##

### Install Go ###

In Windows Terminal

```
lxc-attach --name 100

wget -c https://dl.google.com/go/go1.18.linux-amd64.tar.gz

tar xvf go1.18.linux-amd64.tar.gz

chown -R root:root ./go

mv go /usr/local

echo 'export PATH=$PATH:/usr/local/go/bin' >> /etc/profile

source /etc/profile

go version
```

* As long as the version shows up:
```
rm go1.18.linux-amd64.tar.gz
```

### Install MinIO ###

```
wget https://dl.min.io/server/minio/release/linux-amd64/minio

usermod -L minio

chage -E0 minio

mv minio /usr/local/bin

chmod +x /usr/local/bin/minio

chown minio:minio /usr/local/bin/minio

touch /etc/default/minio

echo 'MINIO_ROOT_USER="Mm5bOM3W1C"' >> /etc/default/minio

echo 'MINIO_VOLUMES="/mnt/MinIO"' >> /etc/default/minio

echo 'MINIO_OPTS="-C /etc/minio --address :9000 --console-address :46699"' >> /etc/default/minio

echo 'MINIO_ROOT_PASSWORD="oLIYG}Bafrl=2h^7@78?"' >> /etc/default/minio

mkdir /etc/minio

chown minio:minio /etc/minio
```

### Install MinIO Service ###

```
wget https://raw.githubusercontent.com/minio/minio-service/master/linux-systemd/minio.service

sed -i 's/User=minio-user/User=minio/g' minio.service

sed -i 's/Group=minio-user/Group=minio/g' minio.service

mv minio.service /etc/systemd/system

systemctl daemon-reload

systemctl enable minio

systemctl start minio

systemctl status minio
```

* Confirm it's active and running

* Take note of the IP address


### Firewall Settings ###

```
ufw default deny incoming

ufw default allow outgoing

ufw allow ssh

ufw allow 9000

ufw allow 46699

ufw allow http

ufw allow https

ufw enable

ufw status verbose
```

Confirm everything looks good

* Make sure you forward ports 9000 and 46699 on your router to your server


### Console Setup ###

* Open it by going to http://ip-address:46699 that you wrote down earlier

Set Server Location
* Go to Configurations
* Set the Server Location to whatever you'd like (mine is "gizmonics-institute")
* Save
* Restart the instance by clicking on the popup or by rebooting the CT

* Identity → Users → Create User
    * Enter a User Name (essentially an Access Key)
    * Enter a Password (essentially a Secret Key)
    * Select "readwrite" under Assign Policies
    * Save

That's it in MinIO. Now is a good time to restart your Proxmox server.

* Node → Reboot
