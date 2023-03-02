# Initial Setup #

Follow the [[Default Ubuntu VM Setup]] instructions but use the following specifics:

- During VM creation:
```
Name: NAS
VM ID: 405
Cores: 2
Memory: 2048
Disk Size: 32
Bridge: vmbr1
VLAN Tag: 40
```

- During Ubuntu installation
```
Your name: Administrator
Your server's name: nas
Pick a username: administrator
```

- Set fixed IP address to 192.168.70.170


# Additional Installs #

```
sudo apt install cifs-utils docker.io docker-compose net-tools smbclient -y

sudo systemctl enable docker

sudo systemctl start docker
```

- Install Portainer as per [[Docker]] note


# Create Users #

* Create users WITHOUT home directories (capital M)
```
sudo useradd -u 1010 -M dave
```

* Create groups
```
sudo groupadd -g 1020 nas
```

```
sudo usermod -aG administrator dave
sudo usermod -aG nas administrator
sudo usermod -aG nas dave
```

```
sudo mkdir /mnt/NAS
sudo chown -R dave:nas /mnt/NAS
sudo chmod 775 /mnt/NAS
```


# Mount SMB Shares #

Automount at startup
```
sudo vim /root/.smb
```

```
user=dave
password=enter-password-here
```
* Save and exit

```
sudo vim /etc/fstab
```

Add this to the bottom:
```
# SMB Mounts
//192.168.10.10/Deep_13_NAS /mnt/NAS cifs credentials=/root/.smb,uid=1010,gid=1020 0 0
```

- Restart
