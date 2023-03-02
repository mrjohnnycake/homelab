
# NFS Setup #

* Settings → NFS, set to Yes
* Go to each share you want to connect with NFS and set NFS Security Settings to:
    * Export: Yes
    * Security: Private
    * Rule: 192.168.110.102(sec=sys,rw) 192.168.110.110(sec=sys,rw)
    * Change IP addresses to any computer you want to connect to the share from (ie. Office, laptop, etc.)



# Backup Scheme #

Media files

* Onsite: Proxmox ZFS pool (managed by MinIO) via Duplicacy on unRAID
* Offsite: None currently (use HD stored in safe for now)


NAS and other unRAID files

* Onsite- Proxmox ZFS pool (managed by MinIO) via Duplicacy on unRAID
* Offsite- Duplicacy-->B2


Internal Backups

* Appdata is backed up to Backblaze B2 via Duplicacy


unRAID USB

* The Unraid USB is backed up through the My Servers plugin to unraid.net




## External drive ##

* It's in the safe

* Plug it in

* Mount it
	* No need to stop the array

* In the Unassigned Devices section of Main, click on the settings gear icon next to the drive
	* Last time it was used it was called backup1 and the mount point was /mnt/disks/unRAID_Backup
	* Make sure that pass through is turned off and auto mount is turned on and click save or done
	* Confirm that those settings took effect
	* On the Main screen, you should be able to click a new Mount button next to the drive

* It’s already been formatted properly and can be found in Krusader as /backup/unRAID_Backup (if needed in that app)
    
In the unRAID console:
```
rsync -auv /mnt/user/data/media/* /mnt/disks/unRAID_Backup/data/media
```
* That will sync the drive with what’s on the array and any files in the cache
    * Insert an "n" into the arguments to do a dry run first