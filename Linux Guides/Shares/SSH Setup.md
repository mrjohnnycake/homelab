# Create Structure (if needed) #

```
mkdir -p .ssh/machine-keys

touch .ssh/authorized_keys

chmod 700 -R .ssh
```


# Create the Key Pair #

In Windows Terminal CMD, login to the server
```
ssh administrator@192.168.40.93
```
* type "yes" to trust the server
* Enter the administrator password

Sudo isn't needed here because it can be owned by administrator

```
ssh-keygen -t rsa
```

At the prompt, set the filename and location
```
/home/administrator/.ssh/machine-keys/media_vm


/home/administrator/.ssh/machine-keys/playground_vm-to-client

```
* Set a passphrase that is known to you

```
cat ~/.ssh/machine-keys/playground_vm-to-client.pub >> ~/.ssh/authorized_keys
```
* This will add it to the end of the file and not overwrite the exiting key(s)

```
rm ~/.ssh/machine-keys/playground_vm-to-client.pub

exit

cls
```
* It's okay to use the same window to complete the next steps


# Setup Computer Access #

In Windows Terminal on the machine you want to add server access to:

- Download the key from the server
```
*Windows*
scp administrator@192.168.40.206:~/.ssh/machine-keys/playground_vm-to-client C:\Users\mrjoh/.ssh/

*Linux*
scp administrator@192.168.10.10:~/.ssh/machine-keys/websites-to-crow /home/administrator/.ssh/machine-keys/websites-to-crow
```
* Enter the administrator's server password

In File Explorer, open the C:\Users\mrjoh\.ssh\config file (create the file if you haven't already) and add this:
```
Host          proxy-vm
HostName      192.168.70.200
Port          22
IdentityFile  ~/.ssh/id_rsa_proxy-vm
User          administrator
```
* Save

In the Windows Terminal settings, duplicate another profile (or create a new one)

* Rename it to Proxy VM @ Crow

* Put this into the command line:
```
ssh proxy-vm
```

* Set the icon to an Ubuntu icon found online

* Set tab title to Proxy VM

* Save

* Open the new profile and enter the passphrase for the key. You should be good to go!



# Personalizing #

## All Distros ##

You can turn off login messages alltogether if you just create a file called `.hushlogin` in your home folder. There doesn't need to be anything in it and no special permissions.


## Debian ##
- When you login via SSH, Ubuntu will spit out a disclaimer message before the first prompt. To edit or remove that, edit this file
```
nano /etc/motd
```


## Ubuntu ##

- VMs have a slightly different setup and need more files editted
```
System info text:
sudo vim /etc/update-motd.d/00-header
	- I like to add printf "\n" BEFORE the print function to create some space between the system info and the passphrase line
	- I like to add printf "\n" AFTER the print function to create some space between the system info and the last login line

Help text:
sudo vim /etc/update-motd.d/10-help-text
```


Delete this file for the "Unminimize" text
```
sudo rm /etc/update-motd.d/60-unminimize
```
