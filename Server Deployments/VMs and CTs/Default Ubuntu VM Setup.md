This is meant as a general installation and setup documentation to show all of the common settings for all of my VMs. Consult the specific VM (Media, Plex, etc.) for specifics to do AFTER following this manual.

All of these instructions are for Ubuntu Live Server. Any options not highlighted are meant to be kept as the default.

These sections are meant to be followed in order.


# Initial Creation #

* Confirm that you have an Ubuntu Live Server ISO downloaded onto your Proxmox node's "local" drive before continuing

Create the VM
- From the Proxmox node you want to install it to, click on Create VM
	* For settings for specific VMs, use the names and numbers from their specific document (Proxy VM, etc.)

```
GENERAL TAB
VM ID: Set to whatever number you would like based on your numbering convention
Name: Set to whatever name you would like based on your naming convention
Check "Start at boot"

- click Next

OS TAB
ISO Image: Select your ISO

- click Next

- click Next again on the System Tab

DISKS TAB
Disk size (GiB): Whatever suits your needs

- click Next

CPU TAB
Cores: Whatever suits your needs (minimum of 2)

- click Next

MEMORY TAB
Memory (MiB): Whatever suits your needs (minimum of 2048)

- click Next

NETWORK TAB
Bridge: choose the one you'll need
VLAN Tag: Enter the number of your needed VLAN, if used (192.168.VLAN##.1)

- click Next

CONFIRM TAB
Check "Start after created" unless you need to make some other change first

- click Finish
```



# OS Setup #

With the VM running, select the new VM in the sidebar and click Console

```
-Try or Install Ubuntu Server

-Select English and click Enter

-Update to the new installer

-Use the default English keyboard layouts and select Done and hit Enter

-Select "Ubuntu Server (minimized)" and select Done and hit Enter

-Go with the assigned IP for now and select Done and hit Enter

-Leave "Proxy address" empty and select Done and hit Enter

-Use the default "Mirror address" and select Done and hit Enter

-Select "Use an entire disk" and "Set up this disk as an LVM group" and select Done and hit Enter

-On the summary page, select Done and hit Enter AND THEN select Continue on the popup by hitting Enter

Your name: Administrator
Your server's name: something similar to the name given during initial VM creation
Pick a username: administrator
Passwords: use a basic one for now that you can remember as you'll be changing it later
- select Done and hit Enter

-Don't install OpenSSH server as, long story short, it's installed by default and this particular package is likely a snap install which could be sandboxed. Select Done and hit enter

-Don't install any additional packages for the same reasons stated above

-On the summary page, select Done and hit enter

-Let the install do it's initial work but when it gives you the option to "Cancel update and reboot" select that and hit Enter

-The shutdown dialogue will eventually get stuck on "Failed unmounting cdrom" so you need to turn off the VM. If you select Shutdown it won't work due to the cdrom error so instead choose Stop
```

- Choose the VM you just created from the left hand menu and go to the Hardware submenu. Double click on CD/DVD Drive and select "Do not use any menu" and click OK. The reasoning is because you already have Ubuntu installed so you need to remove the installation media.

- Start the VM again


# Set a Fixed IP #

The assigned IP is likely not the one you want to use so we need to change that in the router

- In Unifi Controller, find the VM and double click it. In the Settings tab, give your VM a name and turn on Fixed IP. Change the IP to one you want to use and click Apply Changes
- Reboot the VM


# SSH Setup #

You'll need this to continue:
```
sudo apt install vim
```

The next step is to setup your SSH client on your computer so you can connect to the server.  Go to [[SSH Setup]] and follow the instructions there to complete this step.


# Environment Setup #

Now that SSH is setup and you don't need to enter your password so often, let's change the administrator passwd to something more secure than one we can remember.

- Create a secure password in 1P and copy it to your clipboard

```
passwd
```
- Enter the current password and then paste in the new password twice

```
sudo apt update

sudo apt install locales

sudo dpkg-reconfigure locales
```

- Hit Enter to show more locales until you get to "en_US.UTF-8 UTF-8" and remember the corresponding number.
- Hit Enter again until the dialogue says "Locales to be generated" and enter the number of your locale and hit Enter
- For "Default locale for the system environment", enter the number corresponding to "en_US.UTF-8" and hit Enter

```
sudo apt upgrade -y
```

- At the "Which services should be restarted" prompt, enter the number corresponding to "none of the above" and hit Enter

- Reboot the VM

```
sudo apt install fail2ban less

sudo systemctl enable fail2ban --now

sudo vim /etc/ssh/sshd_config
```

* Change this line to look like this (need to uncomment)
```
# Authentication:
PermitRootLogin no
```
