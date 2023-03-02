
# Initial Setup #

Follow the [[Default Ubuntu VM Setup]] instructions but use the following specifics:

- During VM creation:
```
Name: Proxy
VM ID: 410
Cores: 2
Memory: 4096
Disk Size: 20
Bridge: vmbr1
VLAN Tag: 70
```

- During Ubuntu installation
```
Your name: Administrator
Your server's name: proxy
Pick a username: administrator
```

- Set fixed IP address to 192.168.70.200


# Additional Installs #

```
sudo apt install docker.io docker-compose net-tools vim -y

sudo systemctl enable docker

sudo systemctl start docker
```

- Install Portainer as per [[Docker]] note

