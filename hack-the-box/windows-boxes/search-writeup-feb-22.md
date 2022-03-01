# search-writeup-feb-22

### Search

**Overview**

[![](https://camo.githubusercontent.com/7b14d063c825526c46e04701c574430c336f429699e25637011e01152ae47d8b/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a734278792d6e4c6c5859584d6141364a39544a645f772e706e67)](https://camo.githubusercontent.com/7b14d063c825526c46e04701c574430c336f429699e25637011e01152ae47d8b/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a734278792d6e4c6c5859584d6141364a39544a645f772e706e67)

My first box for ’22.

Since the latest release from Offensive Security on the [OSCP Exam Structure](https://www.offensive-security.com/offsec/oscp-exam-structure/), I have shifted my focus to doing more of Windows boxes with an emphasis on gaining more offensive experience within Active Directory.

Search was one of the more challening boxes I’ve rooted with alot of back and forth enumerating with different privileges, identifying different paths and vulnernabilities.

Search is a relevant example of how compromising a standard user can lead to escalating privileges through known security weaknesses within Active Directory Authentication.

**Something to note**

In the spirit of Hack The Box, I have obfuscated any Usersnames / Passwords / Hashes or Flags. This article is to discuss how I came to rooting this box without giving away the the answers

**Tools and Scripts**

LDAP3, Ad-ldap-enum.py, SMBClient, SMBmap, Kerbrute, CrackMapExec, Ssconvert, Crackpkcs12, Powershell, Bloodhound-python, Bloodhound, GMSAdumper.py, Pth-Toolkit and of course, Nmap & Gobuster.

**Enumeration**

Starting with Nmap.

\#Nmap

> sudo nmap -sV -O 10.10.11.129

> PORT STATE SERVICE VERSION\
> 53/tcp open domain Simple DNS Plus\
> 80/tcp open http Microsoft IIS httpd 10.0\
> 88/tcp open kerberos-sec Microsoft Windows Kerberos (server time: 2022–02–15 22:13:22Z)\
> 135/tcp open msrpc Microsoft Windows RPC\
> 139/tcp open netbios-ssn Microsoft Windows netbios-ssn\
> 389/tcp open ldap Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)\
> 443/tcp open ssl/http Microsoft IIS httpd 10.0\
> 445/tcp open microsoft-ds?\
> 464/tcp open kpasswd5?\
> 593/tcp open ncacn\_http Microsoft Windows RPC over HTTP 1.0\
> 636/tcp open ssl/ldap Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)\
> 3268/tcp open ldap Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)\
> 3269/tcp open ssl/ldap Microsoft Windows Active Directory LDAP (Domain: search.htb0., Site: Default-First-Site-Name)

**Ports & Services of interest**

Ports 53, 88, 389 & 636 indicate to us that this is a Domain Controller. The services operating on these ports are DNS, Kerberos, LDAP, LDAP(S) respectively. The other determining factor here is port 3268 and 3269 which are LDAP queries of the Global Catalog in clear text and over SSL respectively.

Port 139 and 445 are used for SMB which is enabled by default on Windows machines.

Port 80 and 443 were also open, which is unusual to see for a Domain Controller but this is a lab at the end of the day.

My thought process when identifying these ports is that we can start with the web service. It’s not common to have a web service running on a Domain Controller unless you’re using it as a PKI Enrollment server. Next will be enumerating services such as LDAP and SMB without credentials. This is known as “null bind” or “null session”, Unless we find some credentials when enumerating 80/443.

**Web Services**

First I check out the web pages on 443 and review the Certificate.

[![](https://camo.githubusercontent.com/8c2d392f1a480fcbda0f27f3b8e1c3ee484f4695bcde7329389ee6e577a39b92/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a687a6f51684e72635163517549474c67676d496e61512e706e67)](https://camo.githubusercontent.com/8c2d392f1a480fcbda0f27f3b8e1c3ee484f4695bcde7329389ee6e577a39b92/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a687a6f51684e72635163517549474c67676d496e61512e706e67)

The common name tells us the box is named reserch.search.htb so I add this entry into my /etc/hosts file.

Gobuster is my prefered tool to enumerate web applications.

> gobuster dir -u [https://](http://35.227.24.107/11ecfee6d0/)research.search.htb -w /usr/share/wordlists/dirb/common.txt

> _/certenroll (Status: 301) \[Size: 161] \[ →_ [_http://research.search.htb/certenroll/_](http://research.search.htb/certenroll/)_]_\
> _/certsrv (Status: 401) \[Size: 1293]_\
> _/css (Status: 301) \[Size: 154] \[ →_ [_http://research.search.htb/css/_](http://research.search.htb/css/)_]_\
> _/fonts (Status: 301) \[Size: 156] \[ →_ [_http://research.search.htb/fonts/_](http://research.search.htb/fonts/)_]_\
> _/images (Status: 301) \[Size: 157] \[ →_ [_http://research.search.htb/images/_](http://research.search.htb/images/)_]_\
> _/Images (Status: 301) \[Size: 157] \[ →_ [_http://research.search.htb/Images/_](http://research.search.htb/Images/)_]_\
> _/index.html (Status: 200) \[Size: 44982]_\
> _/js (Status: 301) \[Size: 153] \[ →_ [_http://research.search.htb/js/_](http://research.search.htb/js/)_]_\
> _/staff (Status: 403) \[Size: 1233]_

Next thing is a manual browse of the site.

Things of Interests I look for are contact pages, About the Team (Particularly IT Staff), blogs etc.. These may be hints or items of interest to be found here.

[![](https://camo.githubusercontent.com/33c3c356d8e180b44be0ea95d4591b9ebb948ca0f2d777c5296a25de8797a448/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a4d475a2d44795a736333436c6c706c73713463634e512e706e67)](https://camo.githubusercontent.com/33c3c356d8e180b44be0ea95d4591b9ebb948ca0f2d777c5296a25de8797a448/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a4d475a2d44795a736333436c6c706c73713463634e512e706e67)

Team members noted were Keely and Sierra. I don’t know what part of Security Keely could work in. Sierra works in SecOps which means she could have some elevated privileges.

I tested functionality of contacts page, the message from the contacts page gets loaded to the URL after submitting. I kept a note of this in case it’s something to revert back to later.

[![](https://camo.githubusercontent.com/6edf7413d1b1fe76bf312908f866918f7131f7ef62bd9e8aa2739feb5038223c/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a7065363031645131484e426d52525055527a575646412e706e67)](https://camo.githubusercontent.com/6edf7413d1b1fe76bf312908f866918f7131f7ef62bd9e8aa2739feb5038223c/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a7065363031645131484e426d52525055527a575646412e706e67)

I also checked the images to see where they were stored and enumerated.

[![](https://camo.githubusercontent.com/f0518d0e808ce419d542f79b1246c7c506e05bdcd5ecbdea86deb38a128164ca/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a417635684e704b6a4f6c5349456f56386559623169672e706e67)](https://camo.githubusercontent.com/f0518d0e808ce419d542f79b1246c7c506e05bdcd5ecbdea86deb38a128164ca/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a417635684e704b6a4f6c5349456f56386559623169672e706e67)

The basic images seemed normal, but within the slides I found a duplicate with very slight difference. These were slide\_2.jpg and slide\_5.jpg

[![Slide2.jpg — Filled Diary](https://camo.githubusercontent.com/6ebdd2c175753707e8dbe6ae0978813c9c6a75134326c9f13d0c47a10612c163/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a576d744c35495f4c595551484d736468755159646a772e706e67)](https://camo.githubusercontent.com/6ebdd2c175753707e8dbe6ae0978813c9c6a75134326c9f13d0c47a10612c163/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a576d744c35495f4c595551484d736468755159646a772e706e67)

[![Slide5.jpg — Blank Diary](https://camo.githubusercontent.com/aaf136ca9c1a86318a3d8bd1a402a774a6e8e7a7f0ad41ee1ec264c7e9a4bf65/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a695650584b4145654f49552d6973584745734d7670672e706e67)](https://camo.githubusercontent.com/aaf136ca9c1a86318a3d8bd1a402a774a6e8e7a7f0ad41ee1ec264c7e9a4bf65/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a695650584b4145654f49552d6973584745734d7670672e706e67)

Slide2 contained a note;

“Send Password to Hope Sharp”, “IsolationIsKey?”

This was the first break with enumeration.This may have been a user and a password — Made a note.

I proceeded to check out the other web pages from the Gobuster results.

[![/Images](https://camo.githubusercontent.com/9c520898c4f926b9429ee93628f9495fd4b7da40bd5831ba981918c4af4eea88/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a576733424e354d6746776656464e376670376c777a672e706e67)](https://camo.githubusercontent.com/9c520898c4f926b9429ee93628f9495fd4b7da40bd5831ba981918c4af4eea88/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a576733424e354d6746776656464e376670376c777a672e706e67)

[![/certsev](https://camo.githubusercontent.com/523c957ff7217c4e4a65bfabb430d467d7864096f86e1b8ccf0c778174cf142c/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a6c393432637058675178667671374567454c6f3046512e706e67)](https://camo.githubusercontent.com/523c957ff7217c4e4a65bfabb430d467d7864096f86e1b8ccf0c778174cf142c/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a6c393432637058675178667671374567454c6f3046512e706e67)

Requesting authentication, I hadn’t got any credentials yet but I tried the usual admin/admin, no credentials etc..

[![/staff](https://camo.githubusercontent.com/149db2419efe7cfe566382383ad25b37e9a7d86beb89bbeed98d687d203a9c23/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a7964496337426e65665333704a4b346b61764a7347772e706e67)](https://camo.githubusercontent.com/149db2419efe7cfe566382383ad25b37e9a7d86beb89bbeed98d687d203a9c23/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a7964496337426e65665333704a4b346b61764a7347772e706e67)

/Staff was prompting for Certificate based authentication.

[![/JS](https://camo.githubusercontent.com/2412f1f7be824109795de5dad42000ea0b0d56e42d7b9c6fdee65baab9207ff8/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a36356d7a546636356854587132746e614d64357964512e706e67)](https://camo.githubusercontent.com/2412f1f7be824109795de5dad42000ea0b0d56e42d7b9c6fdee65baab9207ff8/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a36356d7a546636356854587132746e614d64357964512e706e67)

From Web enumeration so far, I had a potential User and Password.

**LDAP Enumeration**

I didn’t have much luck with interrogating LDAP, intially.

I used LDAP3 and got some data that wasn’t anything I didn’t get from the Certificate; Server name, Domain etc..

[![](https://camo.githubusercontent.com/a77ceb6f7d9adf14c7db3ddc432654eff38f466437a82f363736afa63b110b6c/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a76306c6e364376777a5f64536d306a4a70566c3371512e706e67)](https://camo.githubusercontent.com/a77ceb6f7d9adf14c7db3ddc432654eff38f466437a82f363736afa63b110b6c/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a76306c6e364376777a5f64536d306a4a70566c3371512e706e67)

> DSA info (from DSE):\
> Supported LDAP versions: 3, 2\
> Naming contexts:\
> DC=search,DC=htb\
> CN=Configuration,DC=search,DC=htb\
> CN=Schema,CN=Configuration,DC=search,DC=htb\
> DC=DomainDnsZones,DC=search,DC=htb\
> DC=ForestDnsZones,DC=search,DC=htb

> rootDomainNamingContext:\
> DC=search,DC=htb\
> ldapServiceName:\
> search.htb:research$@SEARCH.HTB

> serverName:\
> CN=RESEARCH,CN=Servers,CN=Default-First-Site-Name,CN=Sites,CN=Configuration,DC=search,DC=htb

> dnsHostName:\
> Research.search.htb\
> defaultNamingContext:\
> DC=search,DC=htb

**SMB Enumeration**

SMB enumeration proved worthless initially. I wasn’t able to get anything without credentials. ‘Null Session’ and ‘Guest’ were no good. I also attempted to use Nmap scripts to find any vulnerabilities but this was a dead end.

[![](https://camo.githubusercontent.com/a746ce857bdb3fcd1a7173cb70faec3100edd9db99c7fc5f42efdc23d1393b0e/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a5757306f66426e446d6c6e56564b595756664a4556772e706e67)](https://camo.githubusercontent.com/a746ce857bdb3fcd1a7173cb70faec3100edd9db99c7fc5f42efdc23d1393b0e/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a5757306f66426e446d6c6e56564b595756664a4556772e706e67)

I resorted back to my initial findings. A potential user and a password but I also had a list of potential users from ‘Our Team’.

From my experience, the most common username format is firstname.lastname. I wrote a list of all the users I could find in a text file with the intention of enumerating each user and then Password Spraying. For this I used Kerbrute.

[![User Enumeration](https://camo.githubusercontent.com/f56c074cd440d46ef1d30a9c4e1b2e9bec8e9f51cb47cd081ceaccebe4e8c771/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a46563073714c7379316d5755557855466246563978672e706e67)](https://camo.githubusercontent.com/f56c074cd440d46ef1d30a9c4e1b2e9bec8e9f51cb47cd081ceaccebe4e8c771/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a46563073714c7379316d5755557855466246563978672e706e67)

[![Password Spray](https://camo.githubusercontent.com/0a6cc9ce5527e65b9f61b1488c254140571fb77b7a48c8f9b4aef48aff6a0dcc/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a736151725145654936426644496e6c69766e6a6f49672e706e67)](https://camo.githubusercontent.com/0a6cc9ce5527e65b9f61b1488c254140571fb77b7a48c8f9b4aef48aff6a0dcc/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a736151725145654936426644496e6c69766e6a6f49672e706e67)

Finally, a hit. Hope.Sharp’s password has been confirmed.

I attempted to user Hope’s credentials for LDAP, SMB and Web enumeration but this didn’t get me much further. The last thing I could think of was Kerberoasting.

By using Crackmapexec, I was able to kerberoast the box to find the Service Account ‘web\_svc’ and proceeded to crack the hash using hashcat with Rockyou.txt

[![](https://camo.githubusercontent.com/ecf371eca7bfcd4422e2246582e2689c57313b53e6ac7430f87298aabf670f5b/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a47667871697a692d34394e4f363045495256646946512e706e67)](https://camo.githubusercontent.com/ecf371eca7bfcd4422e2246582e2689c57313b53e6ac7430f87298aabf670f5b/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a47667871697a692d34394e4f363045495256646946512e706e67)

Once the web\_svc password had been cracked, I attempted to enumerate the web site again with the credentials. This got me nowhere. Next was LDAP.

I had been previously trying to use LDAP3 but was failing to get a bind each time, I ended up moving to ad-ldap-enum.py

[![](https://camo.githubusercontent.com/9661e1554556b438081367025d2f4dcbe397e55f985e01267539e800fa4a0edb/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a3038776356597345717630624c5f59564a47443033672e706e67)](https://camo.githubusercontent.com/9661e1554556b438081367025d2f4dcbe397e55f985e01267539e800fa4a0edb/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a3038776356597345717630624c5f59564a47443033672e706e67)

with ad-ldap-enum.py I was able to grab the Users & Groups of the Domains.\
I paid particular attention to those within privilegd groups, assuming ITSec was privileged.

[![Domain Admins](https://camo.githubusercontent.com/36bd195ff7aefe242d7944778a2bdfdfc44524635f6c96830f467bac6aeb19af/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a6257397844434b396e34664737704867737944326b412e706e67)](https://camo.githubusercontent.com/36bd195ff7aefe242d7944778a2bdfdfc44524635f6c96830f467bac6aeb19af/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a6257397844434b396e34664737704867737944326b412e706e67)

[![ITSec Group](https://camo.githubusercontent.com/170499244f0123ddbdcb7cba24afc0295a248b11aad38b31ba60e9aaecfcf45a/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a443559383252437a4a6e614d4e434d3576514d5766672e706e67)](https://camo.githubusercontent.com/170499244f0123ddbdcb7cba24afc0295a248b11aad38b31ba60e9aaecfcf45a/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a443559383252437a4a6e614d4e434d3576514d5766672e706e67)

With all the usernames now listed, I added them to my original usernames.txt to attempt a password spray using the two passwords that I had collected.

[![](https://camo.githubusercontent.com/d638f53360ec22c473d8d7cd39c4e08034a0660c09a23d5ca3faa2dc65fde8f4/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a5f3638772d677372423930497867526c6436376865412e706e67)](https://camo.githubusercontent.com/d638f53360ec22c473d8d7cd39c4e08034a0660c09a23d5ca3faa2dc65fde8f4/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a5f3638772d677372423930497867526c6436376865412e706e67)

At this stage, I now have 3 Users & their passwords. Back to enumerating.

**SMB (Again)**

Using smbmap, I used all three credentials to see what was available.

[![Edgar](https://camo.githubusercontent.com/b4ba5dc99df542bd472fceb5c110d2ce7359a2788c58fc7db7c2d8c91ac55c45/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a646e4837396c4a6d44787a556e6356695361697957772e706e67)](https://camo.githubusercontent.com/b4ba5dc99df542bd472fceb5c110d2ce7359a2788c58fc7db7c2d8c91ac55c45/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a646e4837396c4a6d44787a556e6356695361697957772e706e67)

[![web\_svc](https://camo.githubusercontent.com/56fed0812fda28e54cdf8ee3a30b0d0fb7cf01a88965726267afdac582241836/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a596e447a2d53612d4836646b7257796463494a424a512e706e67)](https://camo.githubusercontent.com/56fed0812fda28e54cdf8ee3a30b0d0fb7cf01a88965726267afdac582241836/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a596e447a2d53612d4836646b7257796463494a424a512e706e67)

[![Hope](https://camo.githubusercontent.com/44e3fa635a49181305ca8e59b4803567f44575c6a69bef7e460df47bd889d963/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a71455948677a507338566f42344230616b64457145772e706e67)](https://camo.githubusercontent.com/44e3fa635a49181305ca8e59b4803567f44575c6a69bef7e460df47bd889d963/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a71455948677a507338566f42344230616b64457145772e706e67)

using smbmap with the -R recursively returned all directories and files. There were some intersting finds from these results.

[![Phishing\_Attempt.xlsx](https://camo.githubusercontent.com/3052cc6662f569c3a85e4b7f60e3bd33a67446b69e5b91f963e4e62f82a190bc/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a6c664f6d456a50756734706c523553587972624245772e706e67)](https://camo.githubusercontent.com/3052cc6662f569c3a85e4b7f60e3bd33a67446b69e5b91f963e4e62f82a190bc/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a6c664f6d456a50756734706c523553587972624245772e706e67)

[![User Flag](https://camo.githubusercontent.com/0e202a8e2198b81c646e84af57fecde7b9d4b95b29a1a05d896d4cc0e9966b81/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a3170714c304c5965484a50374d7955344f72343267672e706e67)](https://camo.githubusercontent.com/0e202a8e2198b81c646e84af57fecde7b9d4b95b29a1a05d896d4cc0e9966b81/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a3170714c304c5965484a50374d7955344f72343267672e706e67)

I was interested in Phishing\_Attempt and retreived it using smbget but did not have anyway of viewing .xlsx’s.\
For this I used ssconvert to convert the file from .xlsx to .csv. I then used Cat to output the contents of the file, From that came a list of users and passwords.

[![](https://camo.githubusercontent.com/676defd419a9c1816dfc55e329c434a8b6db2e6f69ea835b4c2da9da67be2041/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a64644d6c4f552d6a66473371433737384c4752764a672e706e67)](https://camo.githubusercontent.com/676defd419a9c1816dfc55e329c434a8b6db2e6f69ea835b4c2da9da67be2041/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a64644d6c4f552d6a66473371433737384c4752764a672e706e67)

At this stage, I had multiple users and passwords. I also had the credentials to access to user.txt flag. I used smbget to retrevive the flag.

[![smbget to download user.txt](https://camo.githubusercontent.com/808ae92367e09ea989e2de58a410c7bf81f929e1dcd964f1da0c308aab3acb35/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a6a59684951326c645f5343426c58427a2d37447864512e706e67)](https://camo.githubusercontent.com/808ae92367e09ea989e2de58a410c7bf81f929e1dcd964f1da0c308aab3acb35/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a6a59684951326c645f5343426c58427a2d37447864512e706e67)

Next step was to now enumerate the smb shares with the latest credentials.

[![Another interesting find, a staff.pfx](https://camo.githubusercontent.com/46682aa9dcac8e40cb2711c28fdbdd76e99a8c4e81b4ec78c9d1e9d0a281309c/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a39656b333155592d3962437a387178474131743073672e706e67)](https://camo.githubusercontent.com/46682aa9dcac8e40cb2711c28fdbdd76e99a8c4e81b4ec78c9d1e9d0a281309c/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a39656b333155592d3962437a387178474131743073672e706e67)

smbget was used again to retreive the .pfx. I then used crackpkcs12 and Rockyou.txt to crack the password (This took some time)

[![](https://camo.githubusercontent.com/0cbc04898c4ade463a3a899e8ea2a17f444732f219a6af4930c0d406f83a107c/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a49364861337832386d6672493854525f4761744c78672e706e67)](https://camo.githubusercontent.com/0cbc04898c4ade463a3a899e8ea2a17f444732f219a6af4930c0d406f83a107c/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a49364861337832386d6672493854525f4761744c78672e706e67)

This certificate could then be added to the browser to authenticate back to the /staff webpage.

[![Importing into browser](https://camo.githubusercontent.com/1e03026c3977b573ddcba5b0e9b96fe498618241d1ee76bcd7cf826a9ba7cb0c/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a6a5456626769637a4677416a594b3265356d456356512e706e67)](https://camo.githubusercontent.com/1e03026c3977b573ddcba5b0e9b96fe498618241d1ee76bcd7cf826a9ba7cb0c/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a6a5456626769637a4677416a594b3265356d456356512e706e67)

[![Selecting Certificate to authenticate](https://camo.githubusercontent.com/266e38453ee3341cb3cf77f97c7b084ce13a51f5a49b122d80bdc3fc4fd6e4a0/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a69744d7778564b43386c425838457344537a713859412e706e67)](https://camo.githubusercontent.com/266e38453ee3341cb3cf77f97c7b084ce13a51f5a49b122d80bdc3fc4fd6e4a0/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a69744d7778564b43386c425838457344537a713859412e706e67)

[![Powershell Access](https://camo.githubusercontent.com/fc1df540a5e90708d5f88fd9991619aa57460727132aa4e8ebb12d1f816f9178/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a306b6473595565376157306c48554869465f547737512e706e67)](https://camo.githubusercontent.com/fc1df540a5e90708d5f88fd9991619aa57460727132aa4e8ebb12d1f816f9178/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a306b6473595565376157306c48554869465f547737512e706e67)

[![Authenticated](https://camo.githubusercontent.com/fc52360c5b874758462b4b3293d0b35c50b0878db5713a64cf23ccb5d84457ca/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a76794954764f47505a475a6e63594530726f664934772e706e67)](https://camo.githubusercontent.com/fc52360c5b874758462b4b3293d0b35c50b0878db5713a64cf23ccb5d84457ca/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a76794954764f47505a475a6e63594530726f664934772e706e67)

At this point it had been a few hours and my brain was starting to get foggy. I attempted to do some enumeration again of Users / Computers & Service Accounts

[![](https://camo.githubusercontent.com/2696fff6dda23e1fcfcda5a9caf1db09d5fddcde83853f4508554217ccc55f53/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a43507372746d7474513453394243566c353874477a672e706e67)](https://camo.githubusercontent.com/2696fff6dda23e1fcfcda5a9caf1db09d5fddcde83853f4508554217ccc55f53/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a43507372746d7474513453394243566c353874477a672e706e67)

It was only brought to my attention here that a Group Managed Service Account existed. I queried all properties to find where this could be vulnerable.

> _PrincipalsAllowedToRetrieveManagedPassword_ (Who can retrieve the password)

This may have needed to be done earlier but nontheless, Bloodhound.

[![](https://camo.githubusercontent.com/2f6c8efaca75f4d0a4da0af2b38c0a5d55d5d1b64d2a8769ba4f30b455c67b87/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a624f756c396f77304752524c714930644142365a5a512e706e67)](https://camo.githubusercontent.com/2f6c8efaca75f4d0a4da0af2b38c0a5d55d5d1b64d2a8769ba4f30b455c67b87/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a624f756c396f77304752524c714930644142365a5a512e706e67)

[![](https://camo.githubusercontent.com/8da95059c431a995e16d86f0418078616ece050e48db1477fcc1e9759bbd2805/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a31634f494a4c4d75636b5f73506972373248395438672e706e67)](https://camo.githubusercontent.com/8da95059c431a995e16d86f0418078616ece050e48db1477fcc1e9759bbd2805/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a31634f494a4c4d75636b5f73506972373248395438672e706e67)

After reviewing shortest path to Domain Admins from different users I had credentials of, I was able to find an attack path that leveraged the ITSec Group membership’s ability to retreive the GMSA password, which had GenericAll writes on the Domain Admin.

[![](https://camo.githubusercontent.com/b7133952bd1f24965c2223325e07b8c5b362a36afa48e1f416eafaf86e2c71f3/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a3848427632646d79626b78617054484b5f32326e32772e706e67)](https://camo.githubusercontent.com/b7133952bd1f24965c2223325e07b8c5b362a36afa48e1f416eafaf86e2c71f3/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a3848427632646d79626b78617054484b5f32326e32772e706e67)

After dumping the GMSA hash, I Passed the Hash with PTH-Toolkit, specifically pth-rpcclient.

[![](https://camo.githubusercontent.com/d9d4d071656331861553d31a00f2479a150928814f12ec2a64977ab2026efc4f/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a73414a67654d3772593856534a6776633930654156512e706e67)](https://camo.githubusercontent.com/d9d4d071656331861553d31a00f2479a150928814f12ec2a64977ab2026efc4f/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a73414a67654d3772593856534a6776633930654156512e706e67)

Now that I had connected over rpc as BIR-ADFS-GMSA$, I could change the password of the Domain Admin.

[![](https://camo.githubusercontent.com/24aa4b1711fa598fa199a7f9f51615a6d7311856d6d51ccebff2e1a924c85d5b/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a696f4b65514954314e426a2d5165517a524f4f5036412e706e67)](https://camo.githubusercontent.com/24aa4b1711fa598fa199a7f9f51615a6d7311856d6d51ccebff2e1a924c85d5b/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a696f4b65514954314e426a2d5165517a524f4f5036412e706e67)

This is pretty much it, from here I knew the Domain Admins credentials.

I finished off this box by using smbget to retreive the root.txt file

[![](https://camo.githubusercontent.com/d580c00bb4209de1a4a29dbf9720f09224087fec91573eb84ac8408add583f4f/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a6e5f526e71726c6d323354504b652d5a7065696668412e706e67)](https://camo.githubusercontent.com/d580c00bb4209de1a4a29dbf9720f09224087fec91573eb84ac8408add583f4f/68747470733a2f2f63646e2d696d616765732d312e6d656469756d2e636f6d2f6d61782f3830302f312a6e5f526e71726c6d323354504b652d5a7065696668412e706e67)

\\
