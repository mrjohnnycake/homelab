# Initial Setup #

Follow the [[Default Ubuntu VM Setup]] instructions but use the following specifics:

- During VM creation:
```
Name: Backups
VM ID: 100
Cores: 4
Memory: 4096
Disk Size: 32
Bridge: vmbr1
VLAN Tag: 40
```

- During Ubuntu installation
```
Your name: Administrator
Your server's name: backups
Pick a username: administrator
```

- In Unifi Controller, set fixed IP address to 192.168.40.10

- Upon startup, follow the [[SSH Setup]] note to setup connection via Windows Terminal



# Setup Environment #

```
sudo apt install cifs-utils less nano net-tools smbclient vim -y
```



# SMB Setup #

```
sudo su

nano /root/.smb
```

Add this to the file (replacing the temp password with the real one, of course):
```
user=administrator
password=gypsy-administrator-smb-password
```

```
nano /etc/fstab
```

```
# SMB Mounts
//192.168.10.8/Backups_Everything_Else /mnt/Backups/Everything_Else cifs credentials=/root/.smb,uid=1000,gid=1000 0 0
//192.168.10.8/Backups_Video /mnt/Backups/Video cifs credentials=/root/.smb,uid=1000,gid=1000 0 0
```

Make sure you exit out of being `root`
```
exit
```

* Reboot


# Restic #

Create new backup directories for each necessary dataset that exists on Crow (break up Deep-13/Media/Video movies and shows into their own directories):
```
mkdir /mnt/Backups/backups-crow
mkdir /mnt/Backups/backups-tomservo
mkdir /mnt/Backups/media-audio
mkdir /mnt/Backups/media-books
mkdir /mnt/Backups/media-games
mkdir /mnt/Backups/media-photos
mkdir /mnt/Backups/media-video-movies
mkdir /mnt/Backups/media-video-shows
mkdir /mnt/Backups/nas-anna
mkdir /mnt/Backups/nas-dave
mkdir /mnt/Backups/nas-emmett
```

As long as you didn't use `sudo` you shouldn't need to run these:
```
sudo chown -R administrator:administrator /mnt/Backups/backups-crow
sudo chown -R administrator:administrator /mnt/Backups/backups-tomservo
sudo chown -R administrator:administrator /mnt/Backups/media-audio
sudo chown -R administrator:administrator /mnt/Backups/media-books
sudo chown -R administrator:administrator /mnt/Backups/media-games
sudo chown -R administrator:administrator /mnt/Backups/media-photos
sudo chown -R administrator:administrator /mnt/Backups/media-video-movies
sudo chown -R administrator:administrator /mnt/Backups/media-video-shows
sudo chown -R administrator:administrator /mnt/Backups/nas-anna
sudo chown -R administrator:administrator /mnt/Backups/nas-dave
sudo chown -R administrator:administrator /mnt/Backups/nas-emmett
```


Only run these if the Restic VM is in place
```
ssh-keygen -t rsa
```

Save the key to:
```
/home/administrator/.ssh/machine-keys/backups_vm-to-restic_vm
```
* DO NOT USE A PASSPHRASE

```
cat ~/.ssh/machine-keys/backups_vm-to-restic_vm.pub >> ~/.ssh/authorized_keys

rm ~/.ssh/machine-keys/backups_vm-to-restic_vm.pub
```






