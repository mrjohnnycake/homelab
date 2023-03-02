
# Gypsy#

## Networking ##

Gypsy uses 10.9-10.14 in the Servers VLAN


Gateway: 192.168.10.1
Broadcast: 192.168.10.15
IP Address:
	management: 192.168.10.10/28
	network docker: 192.168.10.11
DNS: 192.168.10.1
         1.1.1.1


vmbr0: enp4s0
vmbr1: eno1
vmbr2: ens2f0 (10Gbe, switch cable to other port if not working)



# Server Security #

## Unattended Upgrades ##

```
sudo apt install unattended-upgrades

sudo dpkg-reconfigure --priority=low unattended-upgrades
```

* Confirm it worked

```
ls /etc/apt/apt.conf.d/
```

* This will show if the proper files were created

* If you get a locale error when doing apt update or upgrade, do this:

```
sudo locale-gen

sudo dpkg-reconfigure locales
```


* Turn off root user access

```
sudo nano /etc/ssh/sshd_config
```

* Change these lines

```
# Authentication:
PermitRootLogin no

# To disable tunneled clear text passwords, change to no here!
PasswordAuthentication no
```

Save and restart SSH

```
sudo systemctl restart ssh
```


## Don't expose things ##

```
ss -lptn
```

Go thru the list and see if I need that service/app exposed or not



## Firewall ##

```
ufw enable
```

Only allow ports I need


## Fail2Ban ##

```
sudo apt install fail2ban

sudo systemctl enable fail2ban --now
```


## AppArmor ##

```
sudo apparmor_status
```
    


## Watchtower ##

Deploy Watchtower to auto-update container images @ 4:30a every day

```
docker run --name watchtower -e TZ="America/Los_Angeles" -v /var/run/docker.sock:/var/run/docker.sock --restart unless-stopped containrrr/watchtower --debug --cleanup --schedule "0 30 4 * * *"
```

