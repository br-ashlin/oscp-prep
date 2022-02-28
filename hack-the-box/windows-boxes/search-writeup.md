# Search

#### **Overview**

![](https://cdn-images-1.medium.com/max/800/1\*sBxy-nLlXYXMaA6J9TJd\_w.png)

My first box for ’22.

Since the latest release from Offensive Security on the [OSCP Exam Structure](https://www.offensive-security.com/offsec/oscp-exam-structure/), I have shifted my focus to doing more of Windows boxes with an emphasis on gaining more offensive experience within Active Directory.

Search was one of the more challening boxes I’ve rooted with alot of back and forth enumerating with different privileges, identifying different paths and vulnernabilities.

Search is a relevant example of how compromising a standard user can lead to escalating privileges through known security weaknesses within Active Directory Authentication.

**Something to note**

In the spirit of Hack The Box, I have obfuscated any Usersnames / Passwords / Hashes or Flags. This article is to discuss how I came to rooting this box without giving away the the answers

**Tools and Scripts**

LDAP3, Ad-ldap-enum.py, SMBClient, SMBmap, Kerbrute, CrackMapExec, Ssconvert, Crackpkcs12, Powershell, Bloodhound-python, Bloodhound, GMSAdumper.py, Pth-Toolkit and of course, Nmap & Gobuster.

#### Enumeration

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

![](https://cdn-images-1.medium.com/max/800/1\*hzoQhNrcQcQuIGLggmInaQ.png)

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

![](https://cdn-images-1.medium.com/max/800/1\*MGZ-DyZsc3Cllplsq4ccNQ.png)

Team members noted were Keely and Sierra. I don’t know what part of Security Keely could work in. Sierra works in SecOps which means she could have some elevated privileges.

I tested functionality of contacts page, the message from the contacts page gets loaded to the URL after submitting. I kept a note of this in case it’s something to revert back to later.

![](https://cdn-images-1.medium.com/max/800/1\*pe601dQ1HNBmRRPURzWVFA.png)

I also checked the images to see where they were stored and enumerated.

![](https://cdn-images-1.medium.com/max/800/1\*Av5hNpKjOlSIEoV8eYb1ig.png)

The basic images seemed normal, but within the slides I found a duplicate with very slight difference. These were slide\_2.jpg and slide\_5.jpg

![Slide2.jpg — Filled Diary](https://cdn-images-1.medium.com/max/800/1\*WmtL5I\_LYUQHMsdhuQYdjw.png)

![Slide5.jpg — Blank Diary](https://cdn-images-1.medium.com/max/800/1\*iVPXKAEeOIU-isXGEsMvpg.png)

Slide2 contained a note;

“Send Password to Hope Sharp”, “IsolationIsKey?”

This was the first break with enumeration.This may have been a user and a password — Made a note.

I proceeded to check out the other web pages from the Gobuster results.

![/Images](https://cdn-images-1.medium.com/max/800/1\*Wg3BN5MgFwfVFN7fp7lwzg.png)

![/certsev](https://cdn-images-1.medium.com/max/800/1\*l942cpXgQxfvq7EgELo0FQ.png)

Requesting authentication, I hadn’t got any credentials yet but I tried the usual admin/admin, no credentials etc..

![/staff](https://cdn-images-1.medium.com/max/800/1\*ydIc7BnefS3pJK4kavJsGw.png)

/Staff was prompting for Certificate based authentication.

![/JS](https://cdn-images-1.medium.com/max/800/1\*65mzTf65hTXq2tnaMd5ydQ.png)

From Web enumeration so far, I had a potential User and Password.

**LDAP Enumeration**

I didn’t have much luck with interrogating LDAP, intially.

I used LDAP3 and got some data that wasn’t anything I didn’t get from the Certificate; Server name, Domain etc..

![](https://cdn-images-1.medium.com/max/800/1\*v0ln6Cvwz\_dSm0jJpVl3qQ.png)

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

![](https://cdn-images-1.medium.com/max/800/1\*WW0ofBnDmlnVVKYWVfJEVw.png)

I resorted back to my initial findings. A potential user and a password but I also had a list of potential users from ‘Our Team’.

From my experience, the most common username format is firstname.lastname. I wrote a list of all the users I could find in a text file with the intention of enumerating each user and then Password Spraying. For this I used Kerbrute.

![User Enumeration](https://cdn-images-1.medium.com/max/800/1\*FV0sqLsy1mWUUxUFbFV9xg.png)

![Password Spray](https://cdn-images-1.medium.com/max/800/1\*saQrQEeI6BfDInlivnjoIg.png)

Finally, a hit. Hope.Sharp’s password has been confirmed.

I attempted to user Hope’s credentials for LDAP, SMB and Web enumeration but this didn’t get me much further. The last thing I could think of was Kerberoasting.

By using Crackmapexec, I was able to kerberoast the box to find the Service Account ‘web\_svc’ and proceeded to crack the hash using hashcat with Rockyou.txt

![](https://cdn-images-1.medium.com/max/800/1\*Gfxqizi-49NO60EIRVdiFQ.png)

Once the web\_svc password had been cracked, I attempted to enumerate the web site again with the credentials. This got me nowhere. Next was LDAP.

I had been previously trying to use LDAP3 but was failing to get a bind each time, I ended up moving to ad-ldap-enum.py

![](https://cdn-images-1.medium.com/max/800/1\*08wcVYsEqv0bL\_YVJGD03g.png)

with ad-ldap-enum.py I was able to grab the Users & Groups of the Domains.\
I paid particular attention to those within privilegd groups, assuming ITSec was privileged.

![Domain Admins](https://cdn-images-1.medium.com/max/800/1\*bW9xDCK9n4fG7pHgsyD2kA.png)

![ITSec Group](https://cdn-images-1.medium.com/max/800/1\*D5Y82RCzJnaMNCM5vQMWfg.png)

With all the usernames now listed, I added them to my original usernames.txt to attempt a password spray using the two passwords that I had collected.

![](https://cdn-images-1.medium.com/max/800/1\*\_68w-gsrB90IxgRld67heA.png)

At this stage, I now have 3 Users & their passwords. Back to enumerating.

**SMB (Again)**

Using smbmap, I used all three credentials to see what was available.

![Edgar](https://cdn-images-1.medium.com/max/800/1\*dnH79lJmDxzUncViSaiyWw.png)

![web\_svc](https://cdn-images-1.medium.com/max/800/1\*YnDz-Sa-H6dkrWydcIJBJQ.png)

![Hope](https://cdn-images-1.medium.com/max/800/1\*qEYHgzPs8VoB4B0akdEqEw.png)

using smbmap with the -R recursively returned all directories and files. There were some intersting finds from these results.

![Phishing\_Attempt.xlsx](https://cdn-images-1.medium.com/max/800/1\*lfOmEjPug4plR5SXyrbBEw.png)

![User Flag](https://cdn-images-1.medium.com/max/800/1\*1pqL0LYeHJP7MyU4Or42gg.png)

I was interested in Phishing\_Attempt and retreived it using smbget but did not have anyway of viewing .xlsx’s.\
For this I used ssconvert to convert the file from .xlsx to .csv. I then used Cat to output the contents of the file, From that came a list of users and passwords.

![](https://cdn-images-1.medium.com/max/800/1\*ddMlOU-jfG3qC778LGRvJg.png)

At this stage, I had multiple users and passwords. I also had the credentials to access to user.txt flag. I used smbget to retrevive the flag.

![smbget to download user.txt](https://cdn-images-1.medium.com/max/800/1\*jYhIQ2ld\_SCBlXBz-7DxdQ.png)

Next step was to now enumerate the smb shares with the latest credentials.

![Another interesting find, a staff.pfx](https://cdn-images-1.medium.com/max/800/1\*9ek31UY-9bCz8qxGA1t0sg.png)

smbget was used again to retreive the .pfx. I then used crackpkcs12 and Rockyou.txt to crack the password (This took some time)

![](https://cdn-images-1.medium.com/max/800/1\*I6Ha3x28mfrI8TR\_GatLxg.png)

This certificate could then be added to the browser to authenticate back to the /staff webpage.

![Importing into browser](https://cdn-images-1.medium.com/max/800/1\*jTVbgiczFwAjYK2e5mEcVQ.png)

![Selecting Certificate to authenticate](https://cdn-images-1.medium.com/max/800/1\*itMwxVKC8lBX8EsDSzq8YA.png)

![Powershell Access](https://cdn-images-1.medium.com/max/800/1\*0kdsYUe7aW0lHUHiF\_Tw7Q.png)

![Authenticated](https://cdn-images-1.medium.com/max/800/1\*vyITvOGPZGZncYE0rofI4w.png)

At this point it had been a few hours and my brain was starting to get foggy. I attempted to do some enumeration again of Users / Computers & Service Accounts

![](https://cdn-images-1.medium.com/max/800/1\*CPsrtmttQ4S9BCVl58tGzg.png)

It was only brought to my attention here that a Group Managed Service Account existed. I queried all properties to find where this could be vulnerable.

> _PrincipalsAllowedToRetrieveManagedPassword_ (Who can retrieve the password)

This may have needed to be done earlier but nontheless, Bloodhound.

![](https://cdn-images-1.medium.com/max/800/1\*bOul9ow0GRRLqI0dAB6ZZQ.png)

![](https://cdn-images-1.medium.com/max/800/1\*1cOIJLMuck\_sPir72H9T8g.png)

After reviewing shortest path to Domain Admins from different users I had credentials of, I was able to find an attack path that leveraged the ITSec Group membership’s ability to retreive the GMSA password, which had GenericAll writes on the Domain Admin.

![](https://cdn-images-1.medium.com/max/800/1\*8HBv2dmybkxapTHK\_22n2w.png)

After dumping the GMSA hash, I Passed the Hash with PTH-Toolkit, specifically pth-rpcclient.

![](https://cdn-images-1.medium.com/max/800/1\*sAJgeM7rY8VSJgvc90eAVQ.png)

Now that I had connected over rpc as BIR-ADFS-GMSA$, I could change the password of the Domain Admin.

![](https://cdn-images-1.medium.com/max/800/1\*ioKeQIT1NBj-QeQzROOP6A.png)

This is pretty much it, from here I knew the Domain Admins credentials.

I finished off this box by using smbget to retreive the root.txt file

![](https://cdn-images-1.medium.com/max/800/1\*n\_Rnqrlm23TPKe-ZpeifhA.png)

\\
