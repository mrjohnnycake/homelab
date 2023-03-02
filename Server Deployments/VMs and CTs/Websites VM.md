# Initial Setup #

Follow the [[Default Ubuntu VM Setup]] instructions but use the following specifics:

- During VM creation:
```
Name: Websites
VM ID: 420
Cores: 4
Memory: 4096
Disk Size: 32
Bridge: vmbr1
VLAN Tag: 70
```

- During Ubuntu installation
```
Your name: Administrator
Your server's name: websites
Pick a username: administrator
```

- Set fixed IP address to 192.168.70.170


# Additional Installs #

```
sudo apt install docker.io docker-compose net-tools samba vim -y

sudo systemctl enable smbd

sudo systemctl enable docker

sudo systemctl start docker
```

- Install Portainer as per [[Docker]] note


# Create SMB Shares #

## Create Users #

* Create users WITHOUT home directories (capital M)
```
sudo useradd -u 1010 -M dave

sudo usermod -aG administrator dave

sudo usermod -aG sambashare dave

sudo smbpasswd -a dave
```

- Create some directories to share
```
sudo mkdir /mnt/FoundryVTT

sudo chown -R dave:administrator /mnt/FoundryVTT

sudo chmod 775 /mnt/FoundryVTT
```


## Setup the Share ##

```
sudo vim /etc/samba/smb.conf
```

- Add this to the bottom
```
[Websites]
path = /mnt
valid users = @administrator
browsable = yes
writable = yes
read only = no
read list = dave
write list = dave
```

- Restart and test


# Services #

- Wordpress is installed directly on the host. Check the Wordpress note for how.