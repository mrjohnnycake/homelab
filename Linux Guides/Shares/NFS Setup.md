Sharing -> Unix Shares (NFS) -> Add

```
Path: mnt/Deep-13/Downloads

Description: Downloaders and Managers CT

Check "All dirs" and "Enabled"

Click Advanced Options

Maproot User: admin

Maproot Group: admin
```
Save



```
sudo apt install nfs-common
```

NFS is all about permissions

* You need to have the same users and groups and matching uid and gid on both systems (TrueNAS and the server to mount to)



Create the basic NFS directory structure along with the first share directory
```
sudo mkdir /mnt/NFS

mkdir /mnt/NFS/Downloads

sudo chown admin:admin /mnt/NFS/Downloads

sudo chmod 775 /mnt/NFS/Downloads
```


```
sudo mount -t nfs 192.168.40.50:/mnt/Deep-13/Downloads /mnt/NFS/Downloads
```



showmount -e 192.168.40.50

fstab

192.168.40.50:/mnt/Deep-13/Media/Audio/music    /mnt/music   nfs defaults 0 0