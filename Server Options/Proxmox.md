
[https://github.com/Tontonjo/proxmox_toolbox](https://github.com/Tontonjo/proxmox_toolbox)

# Updating #

-   From the Proxmox profile in Windows Terminal

```
apt update

apt upgrade or apt dist-upgrade or apt full-upgrade
```

-   Reboot via the Proxmox GUI



# CT Numbering #

Servers (host): 100-109
	* Network Docker


Internal: 110-119
	* Internal Docker
	* NPM


Backups: 120-129
	* Backups LXC
		* MinIO (on bare metal)
		* Duplicacy (on Docker)


Media: 130-139
	-Plex Docker
	-Arrs Docker
	-Downloaders Docker


Family: 140-149
	-Pi-hole
	-Home Assistant


Personal: 150-159
	-Paperless


Exposed: 160-169
	-Exposed Docker
	-Wordpress


Templates: 190-199
	-


# VM Numbering #

Internal: 200-209
	-TrueNAS
	-Internal VM


Exposed: 210+
	-Game Server




# Install Steps #

* Download ISO and burn to USB with Rufus (use the DD function)
* Install from the USB drive


## Network settings ##

Node -> System -> DNS

```
local
192.168.10.1
1.1.1.1
1.0.0.1
```


## Setup ##

Run the Proxmox VE 7 Post Install and Proxmox Dark Theme scripts found here
```
https://tteck.github.io/Proxmox/
```

Those scripts will negate the commands below until the ===== break



Node → Shell
* Install [Dark Theme](https://github.com/Weilbyte/PVEDiscordDark)

```
bash <(curl -s https://raw.githubusercontent.com/Weilbyte/PVEDiscordDark/master/PVEDiscordDark.sh ) install
```

* Reboot and it should be changed


```
nano /etc/apt/sources.list
```

* Add this to the bottom:

```
# not for production use
deb http://download.proxmox.com/debian/pve bullseye pve-no-subscription
```

```
nano /etc/apt/sources.list.d/pve-enterprise.list
```

* Comment out the “enterprise” line

```
apt-get update

apt dist-upgrade
```

* Reboot

==========================================


```
nano /etc/default/grub
```

* Comment out:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet”
```

* Add beneath that:

* If Xeon machine:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on"
```

* If Threadripper machine:
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet amd_iommu=on"
```

```
update-grub
```

```
nano /etc/modules
```

* Add these lines:

```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```

* Reboot



Datacenter → Backup → Add…

* Select the node
* Select the storage
* Select Every day 21:00 but after change to 01:00
* Selection Mode = All
* Leave the rest default
* On the Retention tab set Keep Last to 3
* Click OK


## Storage settings ##

Datacenter → Storage → Add → Directory

* Any ID (I named it Backups)
* Same directory as mount point from above
* Select desired content
* Add


Node → local → CT Templates → Templates

* Download whatever OS you want to use for the CT



# Mount Points Setup #

## Mount Points ##

```
nano /etc/pve/lxc/100.conf
```

* Add this to the bottom:

```
mp0: /mnt/deep-ape/websites,mp=/mnt/websites
```


## Permissions ##

```
nano /etc/pve/lxc/100.conf
```

* Add this to the bottom:

```
# uid map: from uid 0 map 1005 uids (in the ct) to the range starting 100000 (on the host), so 0..1004 (ct) → 100000..101004 (host)

lxc.idmap = u 0 100000 1005

lxc.idmap = g 0 100000 1005

# we map 1 uid starting from uid 1005 onto 1005, so 1005 → 1005

lxc.idmap = u 1005 1005 1

lxc.idmap = g 1005 1005 1

# we map the rest of 65535 from 1006 upto 101006, so 1006..65535 → 101006..165535

lxc.idmap = u 1006 101006 64530

lxc.idmap = g 1006 101006 64530
```

Open this:

```
nano /etc/subuid
```

* Add this to the bottom:

	```
	root:1005:1
	```

Open this:

```
nano /etc/subgid
```

* Add this to the bottom:

	```
	root:1005:1
	```

Change the directory permissions for the mounted folder

```
chown -R 1005:1005 /mnt/Deep13
```


More in depth explanation here https://www.itsembedded.com/sysadmin/proxmox_bind_unprivileged_lxc/




# Networking #

To get the network device names:
```
apt install lshw

lshw -C network
```


## Bonded Interfaces w/ VLANs ##

First, remove the static IP in Unifi for the server interface

For VLAN networking, set up a bonded interface:

* Node-->System-->Network-->Create-->Linux Bond

```
Name: bond0

Slaves: eno1 enp4s0 (or whichever need to be in it)

Mode: LACP (802.3ad)

Hash policy: layer2+3

Autostart checked
```

* Everything else can be left empty

* For all of the physical interfaces, they can stay empty other than autostart for the used ones. I also like to comment on which one is which (ex. "built in" for eno1)

Now we need to create a bridge pointing to the bond:

```
Name: vmbr0

Autostart checked

VLAN aware checked

Bridge ports: bond0
```

Next create a Linux VLAN for one of the VLANs you'll be using. This one will double as the static IP of the Proxmox interface:

```
Name: vmbr0.10 (where 10 is the VLAN ID/Tag)

IPv4/CIDR: 192.168.10.10/28 (change as needed)

Gateway: 192.168.10.1

Autostart checked

VLAN raw device: vmbr0

VLAN Tag: 10

Comment: SERVERS VLAN (or whatever)
```

* Click OK

* Apply Configuration

* Reboot Proxmox

You probably won't be able to get back into the interface. That's because you need to tell your switch to Aggregate these two separate connections into one

* Go to Unifi Controller-->Devices-->24 Port Switch-Ports
* The ports used need to be sequential (ie. 5-6, 20-21, etc.)
* Go to the lower number port to be used
* Set the Port Profile to All since you'll need access to more than one VLAN with this connection
* Click on Port Profile Override and change the Operation to Aggregate
* For the Aggregate Ports, select the next port up from this one
* Apply Changes

Now log back into your Proxmox interface. If it comes up, it worked.

Now create another Linux VLAN for any other VLANs you need on the server. These 2nd+ Linux VLANs will follow the same format and are slightly different than the first VLAN you set up

```
Name: vmbr0.40 (where 40 is the VLAN ID/Tag)

IPv4/CIDR: 192.168.40.10/28 (change as needed) - this IP is necessary for switching though you can't access anything with it yourself - change it to .20 for Gypsy

Gateway: leave empty (Proxmox is supposed to only use one default gateway)

Autostart checked

VLAN raw device: vmbr0

VLAN Tag: 40

Comment: INTERNAL VLAN (or whatever)
```

* Click OK

* Apply Configuration

## 10Gbe Direct Connection ##

### Initial Setup ###

In Node-->Network, find the 10Gbe card ports
* They were called ens2f0 and ens2f1 before

Edit port 1

```
Autostart checked
Comment: 10Gbe port 1
MTU:9000
```

* Do the same for port 2 and just adjust the comment

Create Linux Bridge

```
Name: vmbr1 is fine
IPv4/CIDR: 10.10.10.10/24 (or whatever unused network address you can think of)
Autostart checked
Bridge ports: ens2f0 ens2f1 (you actually only need the one you're using, if you want)
Comment: 10Gbe direct connection
MTU: 9000
```


### CT Config ###

For any CTs that need to use the 10Gbe connection, just change CT-->Network settings to:

```
Bridge: vmbr1
IPv4: Static
IPv4/CIDR: 10.10.10.30/24 (or whatever IP that in the range of the vmbr1 settings)
```

* Leave any fields not referred to above as whatever their default is




## Container Networking ##

For setting up containers to use the bonded, VLAN aware interface correctly:

* Container-->Network-->Edit the default setup (or Add if it doesn't exist)

```
Name: eth0

MAC address: this should already be populated

Bridge: vmbr0

VLAN Tag: 10

IPv4: DHCP

IPv6: DHCP
```

* You can copy this same setup for other containers. Just change the VLAN Tag as needed.



# Clustering #

## Initial Cluster Creation ##

The easiest way to create a cluster is thru the GUI

* You need to stop all running CTs and VMs. If you get an error you may need to delete the second node's CTs and VMs entirely

Go to the main server -> Datacenter -> Cluster -> Create Cluster
* Name it
* Leave the IP the same
* Create

Copy the Join Information

If you get a known_hosts issue when trying to use the shell, you have to run the command it tells you to on both of the servers BUT you have to run the command in the server from that server's IP. So to run the command on server 1 you'd run it from the Shell in the GUI @ 192.168.10.5 and to run the command on server 2 you'd run it from the Shell in the GUI @ 192.168.10.10



## RPi QDevice ##

### Create the OS ###

* Insert the MicroSD card
* Open Raspberry Pi Imager
* Choose OS -> Raspberry Pi OS (other) -> Raspberry Pi OS Lite (32-bit)
* Choose the Storage (MicroSD)
* Gear Icon ->
	* Set hostname to "pveqdevice"
	* Enable SSH
		* Use password authentication
	* Set username and password
		* pi
		* whatever password you want (make secure)
	* Set locale settings
	* Save

Take out the card, install it into the RPi and start it up


### Set the IP ###

Go into Unifi Controller and set a static IP for the RPi and name it something you like Proxmox QDevice
* Make sure that the IP is in the same VLAN as the Proxmox servers


### Setup the OS ###
Log in to Raspberry Pi
* First, confirm that the IP assigned in Unifi Controller is correct and then continue:
```
ip a
```

```
sudo passwd root

sudo systemctl restart sshd

sudo apt update && sudo apt upgrade -y

sudo apt install corosync-qnetd corosync-qdevice -y

sudo systemctl start corosync-qnetd.service

sudo systemctl enable corosync-qnetd.service
```

Before being able to add the QDevice to the cluster, you need to enable the Pi to accept root logins
```
sudo nano /etc/ssh/sshd_config
```

- Uncomment the line `PermitRootLogin` and change the "prohibited-password" to "yes"
- Save and exit

```
/etc/init.d/ssh restart
```

My understanding at the time of this writing is that you should leave the login as root enabled after adding it to the cluster.


### Add to Cluster ###
From Tom-Servo:
```
pvecm qdevice setup [qdevice IP]

example:
pvecm qdevice setup 192.168.10.6
```

Run `pvecm status` on both nodes to confirm there are now 3 votes



# Resizing VM Drives #

https://blog.dgprasetya.com/promox-extend-lvm-partition-ofly/

First- Resize the VM Disk in the Proxmox GUI

	- Node --> VM --> Hardware
	- Select Hard Disk (scsi0) and then click Disk Action --> Resize
	- Adjust size by adding how many more GBs you need

Second- You need to resize the VM Disk partition to use that exist disk space.  The steps below all take place in the VM CLI:
```
df -h -T | grep vg
```
* Take note of the size of the LV

```
sudo apt install parted -y

sudo parted /dev/sda

print

Fix (if needed)

resizepart 3 100%

print

quit
```

Third- Resize the VM LV
```
sudo su - root

pvdisplay

pvresize /dev/sda3

pvdisplay

lvresize -t -v -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv
```

- If the last command worked in test mode, remove the test option and run again:
```
lvresize -v -l +100%FREE /dev/mapper/ubuntu--vg-ubuntu--lv

resize2fs -p /dev/mapper/ubuntu--vg-ubuntu--lv

df -h -T | grep vg
```

Compare the last command results with what they originally were. That should be it.

* Note-I found that I had to update at least one docker image (NZBGet) and then update the stack for that container before the container would recognize the change in disk size.
