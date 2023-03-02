# Next Up #
*Look at headings further down the page to pick new projects*


https://code.visualstudio.com/docs/editor/settings-sync

- Can PMM config.yaml secrets be put in their own file?

- Set up the already synced Restic repos up in cron

- Need to use Wiki.js with GitHub repo
- Setup a GitHub repo to use for different apps and figure out how to use git by only cloning certain folders
	- PMM (able to edit in VS Code and then push)
- Need to create another repo for Windows dotfiles






- Update README.md with necessary YADM instructions




https://www.ruipro.com/contact-us/
https://www.amazon.com/gp/product/B092ZP7X7P






Christian's home folder
https://youtu.be/oF6gLyhQDdw?t=343

https://dev.to/abdfnx/oh-my-zsh-powerlevel10k-cool-terminal-1no0

https://github.com/abdfnx/tran








What about installing and assigning an SSD to the Backups VM that I can sync to and then using a script to move those files over after?
	- Which would be faster and would it be worth it?
	- I don't know if that would even work since rsync couldn't read what was on the zpool first to know what to upload


- Figure out how to sync Wiki.js to Github



```
sudo rsync -av --info=progress -e "ssh -i /home/administrator/.ssh/machine-keys/backups_vm-to-restic_vm" /mnt/Deep-13/Media/Video/movies/ administrator@192.168.40.101:/mnt/Backups/Video/movies

sudo rsync -av --info=progress -e "ssh -i /home/administrator/.ssh/tomservo-to-crow-videos" /mnt/Deep-13/Media/Video/ dave@192.168.10.10:/mnt/Deep-13/Media/Video/
```




Find out how to prioritize local traffic so I can share Plex again



Pay vehicle registrations
	- motorcycle
	- van
	- trailer


[Restic scripts](https://gist.github.com/MatthewVance/19fd0d9e3ff05c06ad01c5a2d6ddfa56)


https://www.juicedbikes.com/products/juiced-bikes-current-series-torque-sensor?variant=18177155205

- Put a change detection alert on the brace and bolt site

- Setup new Plex VM




Setup 1Password for Emmett
	- Let him create his own accounts but we have his master password until he's 18






- Run rsync backups (delete each line here as they're run)

```
sudo rsync -av --info=progress -e "ssh -i /home/administrator/.ssh/machine-keys/backups_vm-to-restic_vm" /mnt/Deep-13/Media/Video/shows/ administrator@192.168.40.101:/mnt/Backups/Video/shows
```

- Implement cron jobs for running Restic backups
	- Document in [Backup and Maintenance](Backup%20and%20Maintenance.md)
	- If cron has issues, consider the .ssh/config file since it's in the `administrator` directory rather than the `root` folder, since `root` will be running the crontab


---------------------------------------------


Finish Pi-KVM install with this [video](https://www.youtube.com/watch?v=aOgcqVcY4Yg)
	- Enable WOL in BIOS



Start using Debian instead of Ubuntu for server OS



https://linuxhint.com/xargs_linux/

iftop
https://www.ssh.com/academy/ssh/copy-id
htop

Install [Tiny11](https://pureinfotech.com/tiny11-iso-install-windows-11/) on Anna's laptop







Why does the office computer keep turning on the monitors randomly all day and night


- Ask Reddit to Photoshop out the extra text on "Countdown to Looking Glass" poster





- Get receiver connected to the network
- Get Google working with the receiver



Rack Work
- Connect PiKVM to Tom Servo when moving drives around
- Check if I plugged in all of Crow's fans



# General #


- Sync new Windows Terminal profiles to Vader as needed


- Setup Glances so I can monitor VMs on Homepage


- GitLens for VScode


[Proxmox Automation Scripts](https://tteck.github.io/Proxmox/)

Automatic updating of Offline .zim files
```
wget https://dumps.wikimedia.org/kiwix/zim/wikipedia/wikipedia_en_all_maxi_2022-05.zim
```

DNS Server
- Christian Lempa's bind9 video
- Cloudflare Family DNS


InfluxDb / Grafana / Glances



Link Aggregation between Modem and Router
https://tongfamily.com/2022/03/12/getting-1-4gbe-xfinity-working-with-arris-sb8200-via-unifi-dream-machine-pro-and-unifi-48-vlan/




https://blog.gurucomputing.com.au/homelabbing-with-proxmox/docker-offsite-backups/

https://blog.gurucomputing.com.au/from-docker-to-kubernetes/

https://www.redhat.com/sysadmin/bound-dns


https://github.com/Casvt/Plex-scripts
https://github.com/jkirkcaldy/plex-utills
https://github.com/ccjensen/PlexMediaTagger
https://github.com/WebTools-NG/WebTools-NG/wiki
https://github.com/kolsys/YouTubeTV.bundle


- Can I do any network bonding on Proxmox to speed up the ZFS I/O?
	- https://icesquare.com/wordpress/how-to-improve-zfs-performance/#section6


# Documentation #

- Sync certain Obsidian notes to GitHub
	- Options
		- Sync GitHub repositories to a specific folder but have those folders be subfolders of the vault. From there you can create other subfolders that wouldn't sync to GitHub
	- Upload finished notes to GitHub
	- Move notes from Google Drive to Obsidian?


- Notifications in Proxmox
	- Need to document this process
		- https://youtu.be/85ME8i4Ry6A



# Shell #

- Look into using ZSH instead of bash

~/.ssh/config
```
Host *
    ServerAliveInterval 300
    ServerAliveCountMax 2
```



# Hardware #




# Maintenance & Backups #

- Backups
	- Setup these Websites VM backups
		- Ghost
			- https://ghost.org/docs/faq/manual-backup/
		- Wiki.js
			- https://docs.requarks.io/install/transfer
	- Update the "Backup and Maintenance" note


[Docker Db Backup Script](https://github.com/ChristianLempa/scripts/tree/main/db-container-backup)



# Notifications #

- Notification Apps Options
	- https://github.com/methatronc/checker
	- https://soketi.app/
	- https://github.com/muety/telepush
	- https://ntfy.sh/
	- https://github.com/notifo-io/notifo
	- https://top.gg/tag/notifications
	- 

- Configure Netdata items that send notifications
	- https://learn.netdata.cloud/docs/monitor/configure-alarms#edit-health-configuration-files
	- How to edit Netdata notifications
```
/etc/netdata/edit-config health.d/net.conf
```

-   https://containrrr.dev/watchtower/notifications/

- Setup error notification script
	- Use one of the two options below to start out
		- https://gist.github.com/norsemanGrey/531626b98ca56e88b64019d013bb2cea
			- this option can double as the weekly notification
		- https://gist.github.com/petervanderdoes/bd6660302404ed5b094d
- Setup weekly notification email script
	- build my own using commands and grep and email?
	- InfluxDB notifications
	- https://github.com/bobbygecko/zfstat
	- https://github.com/IzakMarais/reporter
	- http://zfswatcher.damicon.fi/



# Moving From the Cloud #

- Photos
	- How would taking photos work?
		- They save to the phone
		- Some app (syncthing?) gets them to the server(s)
	- How would Anna (and the rest of us) access them?
	- I could only transition to this setup if...
		- Gypsy is fully functional
		- The family knows how to get all of our photos (and other data) if I were gone
		- 



# New Apps To Try Out #

Miniflux
	- https://miniflux.app/docs/installation.html#docker

Tdarr

Torrent App

Tube Archivist

Watchtower

PiHole

3D Printer
- OctoPrint
- Moonraker
- Klipper

Loki (logs in the same place)


Look into syncing cloud services like contacts, photos, etc.

Setup InfluxDB & Grafana
	- ***ZFS Metrics Needed***
			State (whole pool only, individual disks not needed)
			Available space - normally found with zfs list
			Capacity (total percentage used) - normally found with zpool list
			Last scrub date and time it took
			Snapshots
	- [zpool_influxdb](https://github.com/richardelling/zpool_influxdb)
	- https://andrewdoering.org/blog/2020/10/11/monitoring-zfs-and-docker-with-tig/
	- Uninstall Netdata afterwards (both servers)
	- https://github.com/prometheus-pve/prometheus-pve-exporter

Plex VM (turn into GPU VM?)

IPTV
- [IPTV Guide](https://www.smarthomebeginner.com/plex-iptv-guide/)
- xteve?
- [Providers](https://iptvwire.com/best-iptv-services/#Sportz_TV_JC_Media)
- [Provider example](https://getsportztv.com/pricing/)

Readarr
  -worked on Unmapped Files

Game Server
- Parsec
- [Web-based Emulation](https://www.linuxserver.io/blog/self-hosted-web-based-emulation)
- [Pterodactyl eggs](https://github.com/parkervcp/eggs)
- [Proxmox vGPU Gaming Tutorial](https://www.youtube.com/watch?v=cPrOoeMxzu0)

Grist

Still need to figure out how to automate 1P backups
- Don't send 1p backups to off-site server, only keep on local servers


Portmaster

[FileFlows](https://fileflows.com/)

Netshoot

nagios



# Security #

Hook up laptop to SSDC and check if I can access servers on Trusted LAN from Guest LAN
	-   If so then that needs to get fixed

[Linux Hardening Guide](https://github.com/trimstray/the-practical-linux-hardening-guide)

[Router security](https://www.tomsguide.com/us/home-router-security,news-19245.html)

[Supposedly NSA’s recommendations for network security](https://www.zdnet.com/article/nsa-report-this-is-how-you-should-be-securing-your-network/)


Docker
- [Dockersocket](https://github.com/Tecnativa/docker-socket-proxy)
- Check into each docker image I have installed to see if there are any security issues with the image or developer or if the image is no longer being developed
- As far as security with docker goes, just use different networks based on use cases (internal, external, whether or not they need to talk to each other, etc.) ???


Look into Yubikey for the servers
- How could it be used to help if it simply lives in the server all the time
- [Proxmox & YubiKey](https://pve.proxmox.com/wiki/YubiKey)

[Self-Signed SSL Certificates](https://youtu.be/VH4gXcvkmOY)



# Troubleshooting #



# Network #

Setup UFW / Docker fix
	- https://www.howtogeek.com/devops/how-to-use-docker-with-a-ufw-firewall/

