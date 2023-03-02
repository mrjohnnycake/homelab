# To Do #



https://adamtheautomator.com/restic-backup/

https://blog.bithive.space/post/automatic-backups-with-restic/



https://forum.restic.net/t/recipes-for-managing-repository-environment-variables/1716

https://www.google.com/search?q=restic+export+persist&rlz=1C1CHBF_enUS996US996&oq=restic+export+persist&aqs=chrome..69i57j33i160.6416j0j4&sourceid=chrome&ie=UTF-8

https://restic.readthedocs.io/en/latest/080_examples.html

https://www.civo.com/learn/back-up-your-data-using-restic-minio-and-civo


# Initial Setup #

Follow the [[Default Ubuntu VM Setup]] instructions but use the following specifics:

- During VM creation:
```
Name: Restic
VM ID: 401
Cores: 2
Memory: 2048
Disk Size: 32
Bridge: vmbr1
VLAN Tag: 10
```

- During Ubuntu installation
```
Your name: Administrator
Your server's name: restic
Pick a username: administrator
```

- Set fixed IP address to 192.168.10.13


# Additional Installs #

```
sudo apt install bash-completion bzip2 cifs-utils docker.io docker-compose net-tools smbclient -y
```
* `bash-completion` and `bzip2` are needed for the 

I'm pretty sure this doesn't need to be ran. Don't run it and if the share works just fine then delete this:
```
sudo systemctl enable smbd
```


# Users & Groups #

* Create users WITHOUT home
```
sudo useradd -u 1011 -M emmett
sudo useradd -u 1012 -M anna
```

* Create groups
```
sudo groupadd -g 1020 nas
sudo groupadd -g 1021 plex-movies
sudo groupadd -g 1022 plex-music
sudo groupadd -g 1023 plex-shows
sudo groupadd -g 1024 books
sudo groupadd -g 1025 photos
```

* add `administrator` to all of the groups
```
sudo usermod -aG anna administrator
sudo usermod -aG emmett administrator
sudo usermod -aG nas administrator
sudo usermod -aG books administrator
sudo usermod -aG plex-movies administrator
sudo usermod -aG plex-music administrator
sudo usermod -aG plex-shows administrator
sudo usermod -aG photos administrator
```


# SMB Setup #

- Create these two files with the necessary passwords

/root/.admin-smb
```
user=administrator
password=crow-administrator-smb-password
```

/root/.dave-smb
```
user=dave
password=crow-dave-smb-password
```

Change the files so they can only be opened by `root`:
```
sudo chmod 600 /root/.admin-smb
sudo chmod 600 /root/.dave-smb
```

sudo vim /etc/fstab
```
# SMB Mounts
//192.168.10.10/Deep_13_Backups /mnt/Deep-13/Backups cifs credentials=/root/.admin-smb,uid=1000,gid=1000 0 0
//192.168.10.10/Deep_13_Media /mnt/Deep-13/Media cifs credentials=/root/.dave-smb,uid=1010,gid=1000 0 0
//192.168.10.10/Deep_13_NAS /mnt/Deep-13/NAS cifs credentials=/root/.admin-smb,uid=1000,gid=1000 0 0
```



# Rsync #

```
sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/backups_vm-to-restic_vm" /mnt/Deep-13/Media/Video/movies/ administrator@192.168.40.101:/mnt/Backups/Video/movies

sudo rsync -av -e "ssh -i /home/administrator/.ssh/machine-keys/backups_vm-to-restic_vm" /mnt/Deep-13/Media/Video/shows/ administrator@192.168.40.101:/mnt/Backups/Video/shows
```



# Restic #

I followed [this](https://adamtheautomator.com/restic-backup/) when setting this up and it works just fine for my needs. A slightly different way of running it with environment variables can be found [here](https://blog.bithive.space/post/automatic-backups-with-restic/)


## Installation ##

Download and install Restic
```
wget https://github.com/restic/restic/releases/download/v0.15.1/restic_0.15.1_linux_amd64.bz2
```
* Change the two parts with the version number as needed to get the newest version

```
bzip2 -dv restic_0.15.1_linux_amd64.bz2

chmod +x restic_0.15.1_linux_amd64

sudo mv restic_0.15.1_linux_amd64 /usr/bin/restic

sudo restic generate --bash-completion /etc/bash_completion.d/restic
```

You need to switch to `root` to do the last script and then exit out of `root`:
```
sudo su

source /etc/profile.d/bash_completion.sh

exit
```


## Backups VM Connection ##

Run this after you create the key pair on Backups VM
```
scp administrator@192.168.40.101:~/.ssh/machine-keys/backups_vm-to-restic_vm /home/administrator/.ssh/machine-keys/
```

Create an SSH config file for easier an easier connection to Backups VM:
```
nano .ssh/config
```

Add the following:
```
Host          backups
HostName      192.168.40.101
Port          22
IdentityFile  ~/.ssh/machine-keys/backups_vm-to-restic_vm
User          administrator
```

- Now when calling the SFTP for restic you will simply call `backups` rather than using the IP address


## Usage ##

Create (initialize) all the needed repositories:
```
restic -r sftp:backups:/mnt/Backups/Everything-Else/backups-crow init
restic -r sftp:backups:/mnt/Backups/Everything-Else/backups-tomservo init

restic -r sftp:backups:/mnt/Backups/Everything-Else/media-audio init
restic -r sftp:backups:/mnt/Backups/Everything-Else/media-books init
restic -r sftp:backups:/mnt/Backups/Everything-Else/media-games init
restic -r sftp:backups:/mnt/Backups/Everything-Else/media-photos init

restic -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna init
restic -r sftp:backups:/mnt/Backups/Everything-Else/nas-dave init
restic -r sftp:backups:/mnt/Backups/Everything-Else/nas-emmett init
```
* Enter passwords from 1P

```
sudo mkdir /home/administrator/.restic

sudo chmod 700 /home/administrator/.restic
```

Setup files containing passwords for each each files so you don't have to continually enter them each time a backup is run (this will become absolutely necessary when running via crontab)
```
sudo nano /home/administrator/.restic/.backups-crow
sudo nano /home/administrator/.restic/.backups-tomservo

sudo nano /home/administrator/.restic/.media-audio
sudo nano /home/administrator/.restic/.media-books
sudo nano /home/administrator/.restic/.media-games
sudo nano /home/administrator/.restic/.media-photos

sudo nano /home/administrator/.restic/.nas-anna
sudo nano /home/administrator/.restic/.nas-dave
sudo nano /home/administrator/.restic/.nas-emmett
```
* For each file, type or paste in the password for that specific repo, save and exit

Set permissions for each password file
```
sudo chmod 600 /home/administrator/.restic/.backups-crow
sudo chmod 600 /home/administrator/.restic/.backups-tomservo

sudo chmod 600 /home/administrator/.restic/.media-audio
sudo chmod 600 /home/administrator/.restic/.media-books
sudo chmod 600 /home/administrator/.restic/.media-games
sudo chmod 600 /home/administrator/.restic/.media-photos

sudo chmod 600 /home/administrator/.restic/.nas-anna
sudo chmod 600 /home/administrator/.restic/.nas-dave
sudo chmod 600 /home/administrator/.restic/.nas-emmett
```

Add this to the backup command and change for each as necessary
```
--password-file /home/administrator/.restic/.backups-crow
```
* Also note that the `--keep-last 3` option has been added below. The explanation for that can be found in the Managing Snapshots section

Backup to the repository:
```
# SERVER BACKUP FILES

restic --password-file /home/administrator/.restic/.backups-crow -r sftp:backups:/mnt/Backups/Everything-Else/backups-crow backup /mnt/Deep-13/Backups/Crow/{appdata,files}

restic --password-file /home/administrator/.restic/.backups-tomservo -r sftp:backups:/mnt/Backups/Everything-Else/backups-tomservo backup /mnt/Deep-13/Backups/Tom-Servo/{appdata,files}

* Note that the Proxmox VM/CT dump directory is not included in these backups


# MEDIA FILES

restic --password-file /home/administrator/.restic/.media-audio -r sftp:backups:/mnt/Backups/Everything-Else/media-audio backup /mnt/Deep-13/Media/Audio

restic --password-file /home/administrator/.restic/.media-books -r sftp:backups:/mnt/Backups/Everything-Else/media-books backup /mnt/Deep-13/Media/Books

restic --password-file /home/administrator/.restic/.media-games -r sftp:backups:/mnt/Backups/Everything-Else/media-games backup /mnt/Deep-13/Media/Games

restic --password-file /home/administrator/.restic/.media-photos -r sftp:backups:/mnt/Backups/Everything-Else/media-photos backup /mnt/Deep-13/Media/Photos


# NAS FILES

restic --password-file /home/administrator/.restic/.nas-anna -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna backup /mnt/Deep-13/NAS/Anna

restic --password-file /home/administrator/.restic/.nas-dave -r sftp:backups:/mnt/Backups/Everything-Else/nas-dave backup /mnt/Deep-13/NAS/Dave

restic --password-file /home/administrator/.restic/.nas-emmett -r sftp:backups:/mnt/Backups/Everything-Else/nas-emmett backup /mnt/Deep-13/NAS/Emmett
```


## Excluding Files ##

Make a directory that will store the config files for exclusions (this doesn't get backed up to or from, it just stores the config file that you'll tell Restic to read to know what to exclude):
```
mkdir /opt/backup (can probably be stored wherever, ex. /home/administrator/restic-configs)
```

```
vim /opt/backup/excludes.txt (could also be nas-dave-excludes.txt)
```

Build the file using examples like this:
```
# excludes files .zip
*.tar.gz

# excludes directory logs
logs

# exclude .txt files on the directory data
data/*.txt
```
* Save and exit

Now you need to change your backup commands to call the exclude file:
```
restic -r sftp:backups:/mnt/Backups/Everything-Else/media-games backup /mnt/Deep-13/Media/Games --exclude-file=/opt/backup/excludes.txt
```


## Checking Repositories ##

List stats for a backup:
```
restic --password-file /home/administrator/.restic/.media-audio -r sftp:backups:/mnt/Backups/Video/media-audio stats
```

Check the health of a repository:
```
restic --password-file /home/administrator/.restic/.media-audio -r sftp:backups:/mnt/Backups/Video/media-audio check
```

List snapshots for a backup:
```
restic --password-file /home/administrator/.restic/.nas-anna -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna snapshots
```

List all of the files in a snapshot along with their path (replace ID with yours)
```
restic --password-file /home/administrator/.restic/.nas-anna -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna ls b74f32ba
```

You can check the directory size of the original directory on Crow and the repository directory on Backups VM, respectively, as needed:
```
du -sh /mnt/Deep-13/NAS/Anna

du -sh /mnt/Backups/nas-anna
```


## Managing Snapshots ##

Restic works like this:
	- Backs up the data you want to the other server --> makes a snapshot of it --> checks newer backups against what's listed in the snapshot --> only backs up new files instead of re-uploading every file again --> creates a new snapshot with all of the files including the new ones

Therefore it is a good idea to limit the number of snapshots Restic is allowed to make and/or to delete old snapshots as desired, from time to time.

To delete snapshots, first list all of the snapshots for a given repository:
```
restic -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna snapshots
```

This will show all of the snapshots and give you their names, which look like `736949a9`, for example

First, delete that snapshot:
```
restic -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna forget 736949a9
```

Next, remove the unneeded data that may exist from the snapshot
	- So for instance, maybe that snapshot has a file in it you don't need anymore and you only want to keep the files that are in your other snapshots
```
restic -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna prune
```

Edit this and run for all necessary back repos.
	- This will delete the snapshot(s) AND delete outdated files, all in one command
	- `--tag ''` says to consider all untagged snapshots. Since I'm not tagging any of my snapshots, this looks at all of the repos snapshots when deciding what to delete and what to keep
```
restic --password-file /home/administrator/.restic/.nas-anna -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna forget --tag '' --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --keep-yearly 1 --prune
```


## Restoring ##

List all snapshots:
```
restic --password-file /home/administrator/.restic/.nas-anna -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna snapshots
```
- Take a note of the ID of the snapshot you want to restore from

Restoring a specific file (change the ID accordingly):
```
restic --password-file /home/administrator/.restic/.nas-anna -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna restore b74f32ba --target /mnt/temp --include /mnt/Deep-13/NAS/Anna/test.pdf
```

Restoring a specific directory:
```
restic --password-file /home/administrator/.restic/.nas-anna -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna restore b74f32ba --target /mnt/temp --include /mnt/Deep-13/NAS/Anna/Pictures
```

Restoring an entire repo
```
restic --password-file /home/administrator/.restic/.nas-anna -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna restore b74f32ba --target /mnt/temp
```

- Note that in practice, files will likely be needed to be restored to their original datasets and/or directories rather than a temp directory


## Running via Cron ##

For running all backups, use this as an example:
```
0 1 * * * restic --password-file /home/administrator/.restic/.nas-anna -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna backup --verbose /mnt/Deep-13/NAS/Anna
```

For pruning all backups, use this as an example:
```
0 2 * * * restic --password-file /home/administrator/.restic/.nas-anna -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna forget --tag '' --keep-daily 7 --keep-weekly 4 --keep-monthly 6 --keep-yearly 1 --prune --verbose
```


## Errors ##

If you get an error about the repo being locked, just run this:
```
restic --password-file /home/administrator/.restic/.nas-anna -r sftp:backups:/mnt/Backups/Everything-Else/nas-anna unlock
```
