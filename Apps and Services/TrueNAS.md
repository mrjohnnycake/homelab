su - dave-smb
chmod go-w ~/  
chmod 700 ~/.ssh  
chmod 600 ~/.ssh/authorized_keys



# TrueNAS #

## Create a VM ##

* Set the TrueNAS ISO, a couple cores, 32GB ram (32768mb) and the rest is default
	- 



## HD Passthrough ##

From the Proxmox profile in WIndows Terminal

* Get the model and serial numbers for the disks you want to passthrough to the VM
```
lsblk -o +MODEL,SERIAL
ls /dev/disk/by-id
```
* Use that info to fill out this next command that will passthrough the drives. Change the first number to the VM number on Proxmox
```
qm set 200 -scsi1 /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD4AL2Z
qm set 200 -scsi2 /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD6YG1K
qm set 200 -scsi3 /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD7101N
qm set 200 -scsi4 /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD7ZQ7W
qm set 200 -scsi5 /dev/disk/by-id/ata-ST8000VN004-2M2101_WSD868K6
qm set 200 -scsi6 /dev/disk/by-id/ata-KINGSTON_SA400S37240G_50026B7685576DE8
qm set 200 -scsi7 /dev/disk/by-id/ata-KINGSTON_SA400S37240G_50026B7685576FAA
```
* Do that for the remaining disks remembering to change the -scsi1 to -scsi2 and sequentially for all of the remaining disks
* 
* Check the VM-->Hardware tab to see that they were passed through correctly



## Initial CLI Setup ##

Not sure what goes in this section

zpool import -R /

## Users and Permissions ##

Create a samba share user
	* Accounts-->Users-->Add
	* Fill out the required fields and leave the rest default

Add a user to a dataset
	* Storage-->Pools
	* Click on the three dots menu on the right side of a dataset and click Edit Permissions
	* On the left side, select the user you created and check Apply User
	* For the group, select the group named after your user and check Apply User
	* In the group area of the Access Control List, set Permissions to Full Control
	* If needed, check "Apply permissions recursively" under Advanced and confirm
	* Save

Create groups
	* Accounts-->Groups-->Add
		* Set the name and click Submit
	* Click the arrow on the right side of the group and select Members
	* Add whichever members you need to the group
	* Save




