---
layout: default
title: OpenAdmin
parent: HackTheBox
nav_order: 4
---

![](pictures/logo-openadmin.png)

## Table of contents
{: .no_toc .text-delta }


- Summary
- Recon
- Intrusion
- Lateral Movement
- Privilege Escalation
{:toc}

## [](#header-2)Summary:

1. Use login function in hidden directory to enumerate vulnerable OpenNetAdmin version 
2. Exploit OpenNetAdmin to gain low privilege shell
3. SSH back in with newly found credentials 
4. Edit Ibpage to execute shell with escalated privileges
5. Break into root shell with nano binary 

## [](#header-2)Recon:


Doing our regular nmap scan for safe scripts and running services-versions I see that there is a ssh port 22 as and an apache server running on port 80.

![](pictures/nmap-oa.png)


From gobuster, I can see a few directories that look interesting.

![](pictures/gobuster-oa.png)


Once I go to to the website on port 80, I see the default apache config doc.


![](pictures/apache-oa.png)

After navigating through the newly found directories some more, I find that /music has a login/create account functions which always peek my interest.



![](pictures/music-oa.png)



After clicking on the login function, I am immediately directed into a OpenNetAdmin portal.

![](pictures/login-oa.png)

When I try logging with admin/admin credentials I am able to successfully log in.

![](pictures/login1-oa.png)

This does not give us any different information unfortunately.

After researching the OpenNetAdmin version with searchsploit I see that the version is vulnerable.

![](pictures/sp-oa.png)

TIP: use searchsploit -m flag to put the exploit in same dir.

Once I edit the exploit code a bit and run I am able to get a shell within the OpenNetAdmin directory.

![](pictures/open-oa.png)

## [](#header-2)Intrusion:


 
Get pop a shell but it is restricting.

From here I wanted to get a more interactive shell so after confirming python was on the system with.

```bash
locate python

```

I tried spawning a shell with 
```bash
python3 -c "import pty; pty.spawn('/bin/bash')"

```

 and I also tried many other shell methods (perl,sh,ruby,etc..) but none worked, so I just searched through the dirs with ls and cat commands to see if I could find anything useful.

Once I go through a couple of files I am able to find one with database credentials.



![](pictures/dbset-oa.png)



In order to access the mysql database with these creds, I had to get a better shell.

I started by downloading a php reverse shell from a python server I set up on my host computer to the ona web folder and executing it.

![](pictures/shell-oa.png)


setting up nc to listen for the connection
![](pictures/nc-oa.png)

Command to execute shell
```bash
php php-reverse-shell.php
```
![](pictures/shell1-oa.png)

Once I'm in with the new shell, I had to use spawn a python shell because the mysql command was still not working...

![](pictures/py-oa.png)

After that I was able to get into the mysql database and have a look around.

The only thing I get is encrypted creds for the guest and admin which come out to be:

guest pass test
admin pass admin 

But these end up being a dead end.


After trying a couple of things I ended getting ssh access with one of the user accounts "jimmy" who used the same password as the database user "ona_sys".


![](pictures/jm-oa.png)


Looking at possible way to escalate privileges or find a flag I see that their is a high port open, which is worth a little investigation.

Going to the IP on that port does not get me anything but I am able to find another web directory /var/www/internal.

One of the files "main.php" caught my eye since it used shell_exec to spit out the contents of Joanna's ssh key.

Once I curled main.php I was able to receive the encrypted ssh key
```bash
jimmy@openadmin:/var/www/internal$ curl localhost:52846/main.php
<pre>-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,2AF25344B8391A25A9B318F3FD767D6D

kG0UYIcGyaxupjQqaS2e1HqbhwRLlNctW2HfJeaKUjWZH4usiD9AtTnIKVUOpZN8
ad/StMWJ+MkQ5MnAMJglQeUbRxcBP6++Hh251jMcg8ygYcx1UMD03ZjaRuwcf0YO
ShNbbx8Euvr2agjbF+ytimDyWhoJXU+UpTD58L+SIsZzal9U8f+Txhgq9K2KQHBE
6xaubNKhDJKs/6YJVEHtYyFbYSbtYt4lsoAyM8w+pTPVa3LRWnGykVR5g79b7lsJ
ZnEPK07fJk8JCdb0wPnLNy9LsyNxXRfV3tX4MRcjOXYZnG2Gv8KEIeIXzNiD5/Du
y8byJ/3I3/EsqHphIHgD3UfvHy9naXc/nLUup7s0+WAZ4AUx/MJnJV2nN8o69JyI
9z7V9E4q/aKCh/xpJmYLj7AmdVd4DlO0ByVdy0SJkRXFaAiSVNQJY8hRHzSS7+k4
piC96HnJU+Z8+1XbvzR93Wd3klRMO7EesIQ5KKNNU8PpT+0lv/dEVEppvIDE/8h/
/U1cPvX9Aci0EUys3naB6pVW8i/IY9B6Dx6W4JnnSUFsyhR63WNusk9QgvkiTikH
40ZNca5xHPij8hvUR2v5jGM/8bvr/7QtJFRCmMkYp7FMUB0sQ1NLhCjTTVAFN/AZ
fnWkJ5u+To0qzuPBWGpZsoZx5AbA4Xi00pqqekeLAli95mKKPecjUgpm+wsx8epb
9FtpP4aNR8LYlpKSDiiYzNiXEMQiJ9MSk9na10B5FFPsjr+yYEfMylPgogDpES80
X1VZ+N7S8ZP+7djB22vQ+/pUQap3PdXEpg3v6S4bfXkYKvFkcocqs8IivdK1+UFg
S33lgrCM4/ZjXYP2bpuE5v6dPq+hZvnmKkzcmT1C7YwK1XEyBan8flvIey/ur/4F
FnonsEl16TZvolSt9RH/19B7wfUHXXCyp9sG8iJGklZvteiJDG45A4eHhz8hxSzh
Th5w5guPynFv610HJ6wcNVz2MyJsmTyi8WuVxZs8wxrH9kEzXYD/GtPmcviGCexa
RTKYbgVn4WkJQYncyC0R1Gv3O8bEigX4SYKqIitMDnixjM6xU0URbnT1+8VdQH7Z
uhJVn1fzdRKZhWWlT+d+oqIiSrvd6nWhttoJrjrAQ7YWGAm2MBdGA/MxlYJ9FNDr
1kxuSODQNGtGnWZPieLvDkwotqZKzdOg7fimGRWiRv6yXo5ps3EJFuSU1fSCv2q2
XGdfc8ObLC7s3KZwkYjG82tjMZU+P5PifJh6N0PqpxUCxDqAfY+RzcTcM/SLhS79
yPzCZH8uWIrjaNaZmDSPC/z+bWWJKuu4Y1GCXCqkWvwuaGmYeEnXDOxGupUchkrM
+4R21WQ+eSaULd2PDzLClmYrplnpmbD7C7/ee6KDTl7JMdV25DM9a16JYOneRtMt
qlNgzj0Na4ZNMyRAHEl1SF8a72umGO2xLWebDoYf5VSSSZYtCNJdwt3lF7I8+adt
z0glMMmjR2L5c2HdlTUt5MgiY8+qkHlsL6M91c4diJoEXVh+8YpblAoogOHHBlQe
K1I1cqiDbVE/bmiERK+G4rqa0t7VQN6t2VWetWrGb+Ahw/iMKhpITWLWApA3k9EN
-----END RSA PRIVATE KEY-----
</pre><html>
<h3>Don't forget your "ninja" password</h3>
Click here to logout <a href="logout.php" tite = "Logout">Session
</html>
```

By playing around with main.php some more I was able to get the page to reveal out the user flag with the commands below.

![](pictures/play-oa.png)



Unfortunately, I could not access the root folder via this method.
## [](#header-2)Lateral Movement:

From here I knew that the webpage could execute commands with Joanna's permissions so I decided to download another reverse php shell to the web directory and execute it within main.php.

Once I edit the main.php file to include the command.

```php
shell_exec(php php-reverse-shell.php)

```

Al I have to do is curl the main.php page and set up a netcat listener for the reverse php shell and......


![](pictures/nc1-oa.png)


From here I am able to find the user flag, but still not able to access the root folder.
In order to even get a suitable shell, I had to make a new set of ssh keys and put them in the ~/.ssh/authorized_folder so that I could ssh back in without a passphrase/password for joanna.
![](pictures/ssh-oa.png)
## [](#header-2)Privilege Escalation:


Using sudo -l I find that the the command /bin/nano /opt/priv can be used with sudo privileges and without providing a password.

Looking at GTFObins seeing 

![](pictures/nano-oa.png)


Using the commands provided by GTFObins I am able to get a root shell.
![](pictures/root-oa.png)

![](pictures/root1-oa.png)

That's it!

