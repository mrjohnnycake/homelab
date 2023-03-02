You've read it and heard it already but I cannot stress it enough - BACKUP YOUR SERVERS

Now that's all well and good but what you should backup and how to do it can take a very long time to figure out. This guide is meant as a quick go-to to help you narrow down the scope of what you need to do.

This is a layout of how I backup everything on my servers. Your setup will most certainly be different than mine but hopefully this will help you start narrowing down what to do.


# What to Backup #

Let's put in basic English what needs to be backed up on one of my Proxmox servers.

Proxmox is Linux-based so it has the normal Linux file structure that you may already be used to. Then, I have LXC containers and virtual machines installed that run my various services (Plex, Pi-hole, etc.). I also run a NAS on the same machine and use ZFS as a storage system.

Let's start with Proxmox since It's the most top-level OS and then we'll work down from there.

#### Files ####
- Although most editting to the system is done through the Proxmox GUI, I want to make sure I backup all of those files that have go editted so that, in the case of a server failure, I can have that information ready to go instead of looking up how to do it all over again.
	- Examples:
		- Some files in /etc like 
		- CT and VM backup dump
		- Data backup (see "Backup To Crow" below)
		- Fix permissions every once in a while
		- Zpool maintenance
			- Scrubs
			- Snapshots
			- Error and weekly report notifications

3. Establish and implement backup plan for Crow
	- cron jobs
		- CT and VM backup dump
		- Data backup (see "Backup To Tom-Servo" below)
		- Zpool maintenance
			- Scrubs
			- Snapshots
			- Error and weekly report notifications
	
4. Establish and implement backing up to Gypsy
	- Get Restic installed and working
	- Need to run zpool maintenance on Gypsy as well

5. Establish the timing off all of this
	- Need to write down somewhere that mount points and backups will have to be updated whenever adding new datasets to the pool
	- Order
		-Zpool notifications


4pm my time is the same as midnight on UTC

				- check for errors every hour
				- send out weekly rundown every Sunday @ 8am
		
		- Zpool Scrubs

					- Every 2nd Sunday of every month @ about midnight (8 UTC)
					- Check `cat /etc/cron.d/zfsutils-linux` for specifics

		- Server Updates

					- Every Sunday @ 11pm (7am UTC)
					- scripts/update.sh

		- VMs / CTs internal backup cron jobs (can run concurrently on both servers)

				- 15 min.
				- Media VM: 3:05am - 3:15am
				- Websites VM: 3:05am - 3:15am (5 11 UTC)

		- Docker Watchtower updates (can run concurrently on all servers)

				- 10 min.
				- 3:15am - 3:30am

		- Proxmox dumps (can run concurrently on both servers)

				- 30 min.
				- 3:30am - 4a (Sundays only)
		
		- Tom-Servo data backup to Crow cron job (wait before running next)

				- 1 hr. (may need longer, check back later)
				- 4am - 5am (12 UTC)

		- Plex Meta Manager is programmed to run (by default) every day at 5am

				- 15 min.
				- 5am - 5:15am (13 UTC)

		- Crow data backup to Tom-Servo cron job (can run concurrent to PMM, wait before running next)

				- 15 min.
				- 5am - 5:15am (13 UTC)

		- Permissions fix cron job (can run concurrently on both servers)

				- 5 min.
				- 5:15am - 5:20am (15 13 UTC)

		- ZFS Snapshots

				- Instant
				- 5:30am
				- See "ZFS" note for more details
		
		- Crow to Gypsy restic backups (after last Crow job)

					- Depending on how long this takes, adjust this whole backup schedule accordingly. For example, the first backup jobs on this list start at 3am currently and the last end at 5:30a. Since I don't want the restic backups running into the working morning time (let's say after 6 or 7a) adjust the schedule backways by the total time the restic backups take. So if the daily restic backup needs a couple hours, adjust the first backup job from 3a to 1am and adjust everything else off of that




## Automatic Pool Notifications ##
- Summary notification (once a week)
- Notifications for problems in the metrics
	- Long term: use metrics from InfluxDB to create a notification




==============================================================

# VMs and CTs  on Both #
  
  - Each should have their own internal backup of files I need to access.
	  * ie. docker container files, changed OS files, etc. 
	  * rsync from the VM/CT to /mnt/Deep-13/Backups/[MACHINE IT RUNS ON]/appdata/[APP]
  - Watchtower should be set to auto-update containers
  - unattended-upgrades should update everything on the VM/CT operating system


## Instructions ##

1. From the VM:

       `sudo apt install cron rsync`
       `ssh-keygen -t rsa`
	       - /home/administrator/.ssh/machine-keys/websites-to-crow
       `scp /home/administrator/.ssh/websites-to-crow.pub administrator@192.168.10.10:~/.ssh`

*As far as key pairs go, you need to do this:
		- Assuming you want to connect to Tom-Servo FROM Media-VM, you need to create the pairs on Tom-Servo


2. From Tom-Servo:

       `cat ~/.ssh/websites-to-crow.pub >> ~/.ssh/authorized_keys`
       `rm ~/.ssh/websites-to-crow.pub`

3. Create script on VM:

       `mkdir ~/scripts`
       `chmod 775 ~/scripts`
       `touch scripts/backup-to-Deep13.sh`
       `chmod 770 scripts/backup-to-Deep13.sh`
       `vim scripts/backup-to-Deep13.sh`
* Insert your script code (found in the specific VM section)
* Whatever your backup destination is owned by, that's the user you need to ssh into with in your script. See the note on Backups to Crow for more of an explanation.

4. Create cron job

       `sudo crontab -e`
		* needs to use `sudo`

```
see examples below
```

* Don't run backups on Sundays to allow for pool scrubs


Restart cron if necessary
```
sudo service cron restart
```



# ZFS Snapshots w/ cron #

To get automatic snapshots working, you need to install zfs-auto-snapshot. That, along with cron, will allow you to create snapshots based on timeframe AND control how many to keep.

Installation- this is what ended up working without errors where the normal install caused errors
```bash
wget [https://github.com/zfsonlinux/zfs-auto-snapshot/archive/upstream/1.2.4.tar.gz](https://github.com/zfsonlinux/zfs-auto-snapshot/archive/upstream/1.2.4.tar.gz)

tar -xzf 1.2.4.tar.gz

cd zfs-auto-snapshot-upstream-1.2.4

make install
```

Turn on/off snapshots for datasets:
```
sudo zfs set com.sun:auto-snapshot=true Deep-13
sudo zfs set com.sun:auto-snapshot=false Deep-13/Backups
sudo zfs set com.sun:auto-snapshot=false Deep-13/Backups/Crow
sudo zfs set com.sun:auto-snapshot=false Deep-13/Backups/Tom-Servo
sudo zfs set com.sun:auto-snapshot=false Deep-13/Media
sudo zfs set com.sun:auto-snapshot=true Deep-13/Media/Audio
sudo zfs set com.sun:auto-snapshot=true Deep-13/Media/Books
sudo zfs set com.sun:auto-snapshot=true Deep-13/Media/Games
sudo zfs set com.sun:auto-snapshot=true Deep-13/Media/Photos
sudo zfs set com.sun:auto-snapshot=false Deep-13/Media/Video
sudo zfs set com.sun:auto-snapshot=false Deep-13/NAS
sudo zfs set com.sun:auto-snapshot=true Deep-13/NAS/Anna
sudo zfs set com.sun:auto-snapshot=true Deep-13/NAS/Dave
sudo zfs set com.sun:auto-snapshot=true Deep-13/NAS/Emmett
```

sudo vim /etc/cron.d/zfs-auto-snapshot
```bash
PATH="/usr/bin:/bin:/usr/local/sbin:/usr/sbin:/sbin"

*/5 * * * * root which zfs-auto-snapshot > /dev/null || exit 0 ; zfs-auto-snapshot --quiet --syslog --label=frequent --keep=24 //
```

sudo vim /etc/cron.hourly/zfs-auto-snapshot
```bash
#!/bin/sh

# Only call zfs-auto-snapshot if it's available
which zfs-auto-snapshot > /dev/null || exit 0

exec zfs-auto-snapshot --quiet --syslog --label=hourly --keep=24 //
```

sudo vim /etc/cron.daily/zfs-auto-snapshot
```bash
#!/bin/sh

# Only call zfs-auto-snapshot if it's available
which zfs-auto-snapshot > /dev/null || exit 0

exec zfs-auto-snapshot --quiet --syslog --label=daily --keep=14 //
```

sudo vim /etc/cron.weekly/zfs-auto-snapshot
```bash
#!/bin/sh

# Only call zfs-auto-snapshot if it's available
which zfs-auto-snapshot > /dev/null || exit 0

exec zfs-auto-snapshot --quiet --syslog --label=weekly --keep=4 //
```

sudo vim /etc/cron.monthly/zfs-auto-snapshot
```bash
#!/bin/sh

# Only call zfs-auto-snapshot if it's available
which zfs-auto-snapshot > /dev/null || exit 0

exec zfs-auto-snapshot --quiet --syslog --label=monthly --keep=12 //
```


Show all snapshots:
```
zfs list -t snapshot
```

Show system logs for cron jobs (to verify that jobs went thru)
```
grep CRON /var/log/syslog
```



# Local #

## Tom-Servo ##
- Proxmox backs up VMs and CTs to the specified backup folder
- Everything gets backed up to the destination folder(s) on Crow and deleted files on Tom-Servo that were previously synced remain there.
- Every once in a while, I need go to the destination folder(s) and delete things that I know I no longer need.
- Everything runs through cron jobs


### Crontab ###

sudo crontab -e
```
MAILTO="postfixitman@gmail.com"
SHELL=/bin/bash
HOME=/

# All cron times are UTC

# My scripts
0 12 * * * /home/administrator/scripts/backup_to_Crow.sh
15 13 * * * /home/administrator/scripts/permissions.sh

# Updates- every Sunday night at 11pm
0 7 * * 0 /home/administrator/scripts/update.sh

# Snapshots- every five minutes
# */5 * * * * root zfs-auto-snapshot -q -g --syslog --label=frequent --keep=24 //
# Snapshots- every hour, on the hour
# 0 * * * * root zfs-auto-snapshot -q -g --syslog --label=hourly --keep=24 //
# Snapshots- every day @ 4:00am
# 0 4 * * * root zfs-auto-snapshot -q -g --syslog --label=daily --keep=14 //
# Snapshots- every Sunday @ 4:00am
# 0 4 * * 0 root zfs-auto-snapshot -q -g --syslog --label=weekly --keep=4 //
# Snapshots- every first day of the month @ 4:00am
# 0 4 1 * * root zfs-auto-snapshot -q -g --syslog --label=monthly --keep=12 //
```
* Don't run backups on Sundays to allow for scrubs


### Scripts ###

#### Backup to Crow ####

vim scripts/backup_to_Crow.sh
```
#!/bin/bash

sudo rsync -av -e "ssh -i /home/administrator/.ssh/tomservo-to-crow" /mnt/Deep-13/Backups/Tom-Servo/appdata/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Tom-Servo/appdata/

sudo rsync -av -e "ssh -i /root/.ssh/tom_servo-to-crow" /mnt/Deep-13/Backups/Tom-Servo/dump/ root@192.168.10.10:/mnt/Deep-13/Backups/Tom-Servo/dump/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/tomservo-to-crow" /mnt/Deep-13/Backups/Tom-Servo/files/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Tom-Servo/files/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/tomservo-to-crow-videos" /mnt/Deep-13/Media/Audio/ dave@192.168.10.10:/mnt/Deep-13/Media/Audio/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/tomservo-to-crow-videos" /mnt/Deep-13/Media/Books/ dave@192.168.10.10:/mnt/Deep-13/Media/Books/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/tomservo-to-crow-videos" /mnt/Deep-13/Media/Games/ dave@192.168.10.10:/mnt/Deep-13/Media/Games/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/tomservo-to-crow-videos" /mnt/Deep-13/Media/Photos/ dave@192.168.10.10:/mnt/Deep-13/Media/Photos/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/tomservo-to-crow-videos" /mnt/Deep-13/Media/Video/ dave@192.168.10.10:/mnt/Deep-13/Media/Video/
```
* Note that this script calls two different key pairs. The `administrator` user is for the Backups folder as that is owned by `administrator` but the Media files are owned by `dave` so that required a seperate login via `dave` which required a second key pair. Trying to do everything thru `administrator` resulted in permissions issues whether `administrator` was added to all applicable groups, on both servers, or not.


#### Permissions Fixes ####

vim scripts/permissions.sh
```
#!/bin/bash

# Media/Audio/music
sudo chown -R dave:plex-music /mnt/Deep-13/Media/Audio/music
sudo find /mnt/Deep-13/Media/Audio/music -type d -exec chmod 775 {} \;
sudo find /mnt/Deep-13/Media/Audio/music -type f -exec chmod 664 {} \;

# Media/Video/movies
sudo chown -R dave:plex-movies /mnt/Deep-13/Media/Video/movies
sudo find /mnt/Deep-13/Media/Video/movies -type d -exec chmod 775 {} \;
sudo find /mnt/Deep-13/Media/Video/movies -type f -exec chmod 664 {} \;

# Media/Video/shows
sudo chown -R dave:plex-shows /mnt/Deep-13/Media/Video/shows
sudo find /mnt/Deep-13/Media/Video/shows -type d -exec chmod 775 {} \;
sudo find /mnt/Deep-13/Media/Video/shows -type f -exec chmod 664 {} \;
```
	* TIME = ~2 min.
	* This script does not create output and therefore doesn't report anything


#### OS Updates ####

vim scripts/update.sh
```
#!/bin/bash

sudo apt-get update

sudo apt-get full-upgrade -y
```
* make sure to set permissions to 770 afterwards


### VMs ###

#### Media VM ####

sudo crontab -e
```
MAILTO="postfixitman@gmail.com"
SHELL=/bin/bash
HOME=/

# My Scripts
5 11 * * * /home/administrator/scripts/backup-to-Tom_Servo.sh

# For testing purposes (runs pretty soon after saving)
# * * * * * /root/scripts/backup-to-Deep13.sh
```


vim scripts/backup-to-Tom_Servo.sh
```
#!/bin/bash

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/media_vm-to-tomservo" /docker/appdata/bazarr/backup/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Tom-Servo/appdata/Bazarr/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/media_vm-to-tomservo" /docker/appdata/lidarr/Backups/scheduled/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Tom-Servo/appdata/Lidarr/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/media_vm-to-tomservo" /docker/appdata/nzbget/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Tom-Servo/appdata/NZBGet/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/media_vm-to-tomservo" /docker/appdata/plex-meta-manager/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Tom-Servo/appdata/Plex-Meta-Manager/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/media_vm-to-tomservo" /docker/appdata/prowlarr/Backups/scheduled/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Tom-Servo/appdata/Prowlarr/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/media_vm-to-tomservo" /docker/appdata/radarr/Backups/scheduled/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Tom-Servo/appdata/Radarr/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/media_vm-to-tomservo" /docker/appdata/readarr/Backups/scheduled/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Tom-Servo/appdata/Readarr/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/media_vm-to-tomservo" /docker/appdata/sonarr/Backups/scheduled/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Tom-Servo/appdata/Sonarr/
```



## Crow ##
  - All necessary internal datasets as well as backups from Tom-Servo will be backed up to Gypsy with rsync or restic
	  - Everything gets backed up to the destination folder(s) on Gypsy and deleted files on Crow that were previously synced remain there.
	  - Every once in a while, I need go to the destination folder(s) and delete things that I know I no longer need.
  - Everything runs through cron jobs
  - Restic should have full access to the pool
  - Wait until after rsync tasks complete


### Crontab ###

sudo crontab -e
```
MAILTO="postfixitman@gmail.com"
SHELL=/bin/bash
HOME=/

# All cron times are UTC

# My scripts
0 13 * * * /home/administrator/scripts/backup_to_Tom-Servo.sh
15 13 * * * /home/administrator/scripts/permissions.sh

# Updates- every Sunday night at 11pm
0 7 * * 0 /home/administrator/scripts/update.sh

# For testing purposes (runs pretty soon after saving)
# * * * * * /root/scripts/backup_to_Tom-Servo.sh
# * * * * * /root/scripts/permissions.sh
```
* Don't run backups on Sundays to allow for scrubs


### Scripts ###

#### Backup to Tom-Servo ####

vim scripts/backup_to_Tom-Servo.sh
```
#!/bin/bash

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/crow-to-tomservo" /mnt/Deep-13/Backups/Crow/appdata/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Crow/appdata/

sudo rsync -av -e "ssh -i /root/.ssh/crow-to-tom_servo" /mnt/Deep-13/Backups/Crow/dump/ root@192.168.10.5:/mnt/Deep-13/Backups/Crow/dump/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/crow-to-tomservo" /mnt/Deep-13/Backups/Crow/files/ administrator@192.168.10.5:/mnt/Deep-13/Backups/Crow/files/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/crow-to-tomservo-a" /mnt/Deep-13/NAS/Anna/ anna@192.168.10.5:/mnt/Deep-13/NAS/Anna/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/crow-to-tomservo-d" /mnt/Deep-13/NAS/Dave/ dave@192.168.10.5:/mnt/Deep-13/NAS/Dave/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/crow-to-tomservo-e" /mnt/Deep-13/NAS/Emmett/ emmett@192.168.10.5:/mnt/Deep-13/NAS/Emmett/
```
* Note that this requires dave, anna, and emmett to all create their own key pairs


#### Permissions Fixes ####

vim scripts/permissions.sh
```
#!/bin/bash

# ANNA NAS
chown -R anna:anna /mnt/Deep-13/NAS/Anna
find /mnt/Deep-13/NAS/Anna -type d -exec chmod 775 {} \;
find /mnt/Deep-13/NAS/Anna -type f -exec chmod 664 {} \;

# DAVE NAS
chown -R dave:nas /mnt/Deep-13/NAS/Dave
find /mnt/Deep-13/NAS/Dave -type d -exec chmod 775 {} \;
find /mnt/Deep-13/NAS/Dave -type f -exec chmod 664 {} \;

# EMMETT NAS
chown -R emmett:emmett /mnt/Deep-13/NAS/Emmett
find /mnt/Deep-13/NAS/Emmett -type d -exec chmod 775 {} \;
find /mnt/Deep-13/NAS/Emmett -type f -exec chmod 664 {} \;
```
	* TIME = ~2 min.
	* This script does not create output and therefore doesn't report anything


#### OS Updates ####

vim scripts/update.sh
```
#!/bin/bash

sudo apt-get update

sudo apt-get full-upgrade -y
```
* make sure to set permissions to 770 afterwards



### VMs ###

#### Websites VM ####

sudo crontab -e
```
MAILTO="postfixitman@gmail.com"
SHELL=/bin/bash
HOME=/

# My scripts
5 11 * * * /home/administrator/scripts/backup_to_Crow.sh > /dev/null

# For testing purposes (runs pretty soon after saving)
# * * * * * /home/administrator/scripts/backup_to_Crow.sh
# * * * * * /root/scripts/permissions.sh
```

backup-to-Crow.sh
```
#!/bin/bash

# ChangeDetection.io
sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/websites-to-crow" /docker/appdata/change-detection/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Crow/appdata/Change-Detection/

# Daycare Website
sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/websites-to-crow" /mnt/Daycare/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Crow/files/Daycare/

# FoundryVTT
sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/websites-to-crow" /mnt/FoundryVTT/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Crow/files/FoundryVTT/

# Ghost

# Gotify
sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/websites-to-crow" /docker/appdata/gotify/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Crow/appdata/Gotify/

# Homepage
sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/websites-to-crow" /docker/appdata/homepage/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Crow/appdata/Homepage/

# Paperless
sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/websites-to-crow" /mnt/Paperless/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Crow/files/Paperless/

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/websites-to-crow" /docker/appdata/paperless/ administrator@192.168.10.10:/mnt/Deep-13/Backups/Crow/appdata/Paperless/

# Wiki.js
```

prune-db-backups.sh
```
#!/bin/bash

find /home/administrator/WPdbBackups/*.sql -mtime +14 -exec rm -rf {} \;
```



# Offsite #

## Gypsy ##
- Offsite backups will go from Crow to Gypsy only as everything local will be on Crow
- Each dataset will have it's own bucket
- VM/CT dump backups older than (2) months are automatically deleted
	- This needs to happen because the source file names contain the creation date and so Duplicati would see each Proxmox backup as it's own file so backups would functionally be kept forever with the "Keep the last number of backups" setting that the other Duplicati backups use


==============================================================


