# Proxmox Nodes#

Email Alerts Setup (Gmail)

1. SSH into a Proxmox node and become root user. Run the following commands to download extra software dependencies we'll need.

    ```
    apt update
    apt install -y libsasl2-modules
    ```

2. Enable 2FA for the gmail account that will be used by going to [security settings](https://myaccount.google.com/security)

3. Create app password for the account.
    1. Go to [App Passwords](https://security.google.com/settings/security/apppasswords)
    2. Select app: Mail
    3. Select device: Other
    4. Type in: Proxmox
  
4. Write gmail credentials to file and hash it. Again, make sure you are root.

    ```bash
    echo "smtp.gmail.com youremail@gmail.com:yourpassword" > /etc/postfix/sasl_passwd
    
    # chmod u=rw
    chmod 600 /etc/postfix/sasl_passwd
    
    # generate /etc/postfix/sasl_passwd.db
    postmap hash:/etc/postfix/sasl_passwd
    ```


5. Open the Postfix configuration file with editor of your choice.

    ```
    nano /etc/postfix/main.cf
    ```

6. Append the following to the end of the file:
    ```
    relayhost = smtp.gmail.com:587
    smtp_use_tls = yes
    smtp_sasl_auth_enable = yes
    smtp_sasl_security_options =
    smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
    smtp_tls_CAfile = /etc/ssl/certs/Entrust_Root_Certification_Authority.pem
    smtp_tls_session_cache_database = btree:/var/lib/postfix/smtp_tls_session_cache
    smtp_tls_session_cache_timeout = 3600s
    ```

    **IMPORTANT**: Comment out the existing line containing just `relayhost=` since we are using this key in our configuration we just pasted in.

7. Reload postfix
    ```
    postfix reload
    ```

8. Test to make sure everything is hunky-dory.
```
echo "sample message" | mail -s "sample subject" anotheremail@gmail.com
```



# VMs #

Email Alerts Setup (Gmail)

1. SSH into proxmox node and become root user. Run the following commands to download extra software dependencies we'll need.

```
sudo apt update

sudo apt install libsasl2-modules mailutils postfix -y
```

* When the postfix install dialogue comes up, enter `1` for `No configuration`

Reboot to get all of the new services working
```
sudo reboot
```

2. Setup Gmail 2FA (if you haven't already)

- Enable 2FA for the gmail account that will be used (if you haven't already) by going to [Gmails's security settings](https://myaccount.google.com/security)

- Create app password for the account (if you haven't already).
    - Go to [App Passwords](https://security.google.com/settings/security/apppasswords)
    - Select app: Mail
    - Select device: Other
    - Type in: VMs or whatever describes your needs
  
3. Configure Postfix

```
sudo vim /etc/postfix/sasl_passwd
```

Add the following line making sure to set the correct email and 2FA secure password from the step above:
```
smtp.gmail.com postfixitman@gmail.com:yourpassword" > /etc/postfix/sasl_passwd
```

```
sudo chmod 600 /etc/postfix/sasl_passwd

sudo postmap hash:/etc/postfix/sasl_passwd
```

* That last line will generate /etc/postfix/sasl_passwd.db so that sasl_passwd is secure

Create the Postfix configuration file:
```
sudo vim /etc/postfix/main.cf
```

Append the following to the end of the file:
```sh
# See /usr/share/postfix/main.cf.dist for a commented, more complete version

# Debian specific:  Specifying a file name will cause the first
# line of that file to be used as the name.  The Debian default
# is /etc/mailname.
#myorigin = /etc/mailname

smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)
biff = no

# appending .domain is the MUA's job.
append_dot_mydomain = no

# Uncomment the next line to generate "delayed mail" warnings
#delay_warning_time = 4h

readme_directory = no

# See http://www.postfix.org/COMPATIBILITY_README.html -- default to 3.6 on
# fresh installs.
compatibility_level = 3.6

# TLS parameters
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
smtpd_tls_security_level=may
smtp_tls_CApath=/etc/ssl/certs
smtp_tls_security_level=may
# smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination
myhostname = websites.tailf3fc5.ts.net
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
mydestination = $myhostname, asd, websites, localhost.localdomain, localhost
# relayhost =
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
inet_protocols = all

relayhost = smtp.gmail.com:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_security_options =
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_tls_CAfile = /etc/ssl/certs/Entrust_Root_Certification_Authority.pem
smtp_tls_session_cache_database = btree:/var/lib/postfix/smtp_tls_session_cache
smtp_tls_session_cache_timeout = 3600s
```

*IMPORTANT*
With updates to Postfix and SMTP and the rest, this configuration will likely need to be updated in the future

Reload postfix and then send a test message
```
sudo postfix reload
```

4. Test to make sure everything is working
```
sudo echo "sample message" | mail -s "sample subject" mrjohnnycake@gmail.com
```
