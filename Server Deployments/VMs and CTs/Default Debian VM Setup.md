This is meant as a general installation and setup documentation to show all of the common settings for all of my Debian based VMs. Consult the specific VM (Playground, etc.) for specifics to do AFTER following this manual.

All of these instructions are for Debian. Any options not highlighted are meant to be kept as the default.

These sections are meant to be followed in order.


# Initial Creation #

* Confirm that you have a Debian ISO downloaded onto your Proxmox node's "local" drive before continuing

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
ISO Image: Select your Debian ISO

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
TO SPEED UP THE INSTALL, MAXIMIZE THE CONSOLE WINDOW AND USE TAB, ENTER, AND THE ARROW KEYS

-Choose Graphical Install

-Select your language

-Select your country

-Select keyboard language

-Name the system

-Leave the domain name blank

-Type in a root password known to you to start (you'll change it to a more secure password later)

-Type in "Administrator" as the new user's actual name

-Stick with "administrator" for the username

-Type in an administrator password known to you to start (you'll change it to a more secure password later)

-Select your time zone

-Select "Guided - use entire disk"

-Select the QEMU disk listed

-All files in one partition

-Finish partitioning and write changes to disk

-Yes

-After it's finished with the initial install, select "No" for Scan extra installation media

-Select your country

-Select an archive mirror that's close you where your server is

-Leave proxy information blank

-Select No for Participate in the package usage survey

-Only check "SSH server"

-Select Yes for installing GRUB

-Choose the primary QEMU disk

-Hit Continue to reboot

-After it reboots, STOP the VM (choosing Shutdown will cause it to hang)
```

- Choose the VM you just created from the left hand menu and go to the Hardware submenu. Double click on CD/DVD Drive and select "Do not use any media" and click OK. The reasoning is because you already have Debian installed so you need to remove the installation media.

- Start the VM again

- Login via the Console window using "administrator" and the password you set

- Type `ip r` to get the IP address and then type `exit` and the close the Proxmox window


# Set a Fixed IP #
*do this step if needed*

The assigned IP is likely not the one you want to use so we need to change that in the router

- In Unifi Controller, find the VM and double click it. In the Settings tab, give your VM a name and turn on Fixed IP. Change the IP to one you want to use and click Apply Changes
- Reboot the VM


192.168.40.93


# Sudo Fix #

```
su -

passwd
```
* Set a stronger password for `root`

Still as root, install `sudo` along with other packages you'll need and then set `sudo` priveledges:
```
apt install curl fail2ban git net-tools sudo vim wget yadm zsh -y

usermod -aG sudo administrator

reboot
```

Log back in and test the `administrator` user's `sudo` access


# SSH Setup #

The next step is to setup your SSH client on your computer so you can connect to the server.  Go to [[SSH Setup]] and follow the instructions there to complete this step.


# Environment Setup #

Now that SSH is setup and you don't need to enter your password so often, let's change the administrator passwd to something more secure.

- Create a secure password in 1P and copy it to your clipboard

```
passwd
```
- Enter the current password and then paste in the new password twice


```
sudo systemctl enable fail2ban --now

sudo vim /etc/ssh/sshd_config
```

* Change this line to look like this (need to uncomment)
```
# Authentication:
PermitRootLogin no
```


# GitHub Authentication #

*You need to do this to be able to use YADM*

GitHub CLI (all one command)
```
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
&& sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg \
&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
&& sudo apt update \
&& sudo apt install gh -y
```

Authenticate GitHub
```
gh auth login
```

```
Select GitHub.com

Select SSH

Yes

No passphrase

Name: [VM Name]

Login with a web browser
	- Hit enter
	- Click the link
	- Copy the one-time code over
```

```
rm ~/.ssh/id_ed25519.pub
```

Set ZSH as the default shell:
```
chsh -s /usr/bin/zsh
```


# Upon Cloning #

- If the Powerlevel10k dialogue pops up, enter `q`

- Clone into the dotfiles repo to download it to your machine
```
yadm clone git@github.com:mrjohnnycake/dotfiles.git
```

When you get an error about overwriting files, run this:
```
yadm fetch --all
```

- Run the `~/.scripts/step-one.sh` 

- Close the window and open again