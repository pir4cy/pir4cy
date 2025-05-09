---
title: 'HTB: Lightweight'
date: '2019-05-11'
excerpt: 'Lightweight is a pretty unique and challenging box which showcases the common mistakes made by system administrators and the need for encryption in any kind protocol used. It deals with the abuse of Linux capabilities which can be harmful in bad hands and how unencrypted protocols like LDAP can be sniffed to gain information and credentials.'
readingTime: 2
tags: ['HTB', 'Writeup', 'HTB', 'Writeup', 'Easy', 'Medium', 'Linux', 'Web']
author: 'pir4cy'
coverImage: '/images/htb/covers/lightweight-cover.png'
---

# LightWeight

## Info
  * IP : 10.10.10.119
  * OS : Linux
  * Diffculty: Easy/Medium

## System Enumeration

### NMAP

![Nmap](/images/htb/machines/Lightweight/nmap.png "Nmap")

### Web App Enumeration

![front page](/images/htb/machines/Lightweight/front.png "Front Page")

#### user.php

![user.php](/images/htb/machines/Lightweight/userphp.png "User.php")

#### status.php

![status.php](/images/htb/machines/Lightweight/statusphp.png "Status.php")


So we see that we can log in to the box using our IP and SSH.  
After logging in, there was nothing special to look for in the box, so I tried a `tcpdump`.  
	   `tcpdump -i lo -vv -nn -w capture.pcap`

![tcpdump](/images/htb/machines/Lightweight/tcpdump.png "TCP Dump")

We see traffic being captured, when we open `http://10.10.10.119/status.php`  
    
Using `scp` to transfer the captured pcap file to my local pc, I opened it up in wiresharkhttps://gist.github.com/bcoles/421cc413d07cd9ba7855

#### WireShark

Since, we know that this machine is related to `LDAP`, I directly checked the `.pcap` file for any LDAP requests and luckily enough

![ldapcheck](/images/htb/machines/Lightweight/ldapcheck.png "LDAP Found")  
Looking further, we find a possible authentication hash for `ldapuser2`  
![ldapuser2](/images/htb/machines/Lightweight/ldapuser2.png "LDAP User2")

## User Exposed

Elevating to `ldapuser2` using `su` and the hash as password, we log in easily. Thus, exposing `user.txt`

![User found](/images/htb/machines/Lightweight/userfound.png "User Found")

### Backup.7z

In the home folder of `ldapuser2`, there's a 7zip file named backup, which means it might contain some information for us.  

Downloading the file and extracting it.  
![Backup.7z Denied](/images/htb/machines/Lightweight/backupdenied.png "Backup.7z Denied")  

So it's locked and we have to decrypt it.  
I found a simple and straightforward script on github to do the job for me.(https://gist.github.com/bcoles/421cc413d07cd9ba7855)  

![Backup PW Found](/images/htb/machines/Lightweight/backuppwfound.png "Password Cracked")

We got new files! Let's check them out.  

### Status.php

So, the status.php file reveals some more information to us 

![status reveal](/images/htb/machines/Lightweight/statusreveal.png "Status.php Reveal")

## Going for Root

We now have both the passwords for ldapuser1 and ldapuser2, but still no root access or `root.txt`

### General Enumeration

![SysInfo](/images/htb/machines/Lightweight/uname.png "uname -a")

![SUID Info](/images/htb/machines/Lightweight/suid.png "SUID info")

### Linux Capabilities

Since, there was nothing peculiar I checked for linux capabilities using `getcap -r / 2>/dev/null`  
It seems openssl has empty capabilities, that means we can use it to access any file. 

![Linux Capabilities](/images/htb/machines/Lightweight/lincap.png "Linux Capabilities") 

So there are 2 ways we can go about this:  
  * Get root.txt
  * Get root shell by changing sudoers

### Getting root.txt

![root.txt](/images/htb/machines/Lightweight/root.png "Root.txt")

### Changing Sudoers

So we create our own sudoers using `./openssl enc -in "/etc/sudoers" -out sudoers`  

Editing this sudoers file to:  
![Sudoers](/images/htb/machines/Lightweight/sudoers.png "Sudoers")

Changing the original sudoers file and getting root shell.  

![Rooted](/images/htb/machines/Lightweight/rooted.png "Rooted")

Thank you for following!
