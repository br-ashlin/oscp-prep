# backdoor-writeup-mar-22



## Backdoor

![](<../../.gitbook/assets/image (6).png>)

### Reconnaissance <a href="#6b46" id="6b46"></a>

First thing.

I will echo the box name into my /etc/hosts file for DNS resolution.

```
echo "backdoor.htb  10.10.11.125" >> /etc/hosts
```

Using '>>' ensures the existing data is added to and not overwriten such as when using '>'.

I then make a directory of the box for all files using mkdir.

```
cd ~/HTB
mkdir Backdoor
```

I run a quick initial nmap scan to see which ports are open and which services are running on those ports. I generally export this to a file for later viewing

```
nmap -Pn -p- -sC -sV -A backdoor.htb > nmap-initial.txt
```

* **-Pn:** Disable host discovery, port scan only
* **-p-:** Scan all ports (TCP/UDP)
* **-sC**: run default nmap scripts
* **-sV**: detect service version
* **-A:** Enable OS detection, versions

![](<../../.gitbook/assets/image (2).png>)

![](<../../.gitbook/assets/image (26) (1).png>)

We get back the following result showing that 4 ports are open:

* **Port 22:** ssh
* **Port 80:** http
* **Port 1337:** waste?
* **Port 444**4: krb524



Using **Whatweb**, I can confirm what web application is running, in comparison to **nmap**.

![](<../../.gitbook/assets/image (10).png>)

Looking for any vunerabilities for Wordpress 5.8.1 using **Searchsploit**.&#x20;

![](<../../.gitbook/assets/image (5).png>)



### Enumeration <a href="#64a0" id="64a0"></a>

With the knowledge that the CMS is Wordpress, I use **WPscan** to enumerate for Themes, Plugins, Users etc..

![](<../../.gitbook/assets/image (23).png>)

![](<../../.gitbook/assets/image (16) (1).png>)

I've made a note of the Plugin 'akismet' and the location at /wp-content/plugins/akismet and the version. **Searchsploit** doesn't have anything available for this version.

![](<../../.gitbook/assets/image (35) (1).png>)

**GoBuster** for futher enumeration of directories, as well as manual verification.

![](<../../.gitbook/assets/image (4) (1).png>)

The only directories I can traverse at this stage is /wp-content. I run **GoBuster** again and use a bigger wordlist.

![](<../../.gitbook/assets/image (28).png>)

After manually verifying each directory, I enumerate the plugins folder.

![](<../../.gitbook/assets/image (38).png>)



There doesn't appear to be much available for this plugin, but after going back a level I find ebook-download.

![](<../../.gitbook/assets/image (20).png>)



Review the readme, version is 1.1

![](<../../.gitbook/assets/image (18).png>)

**Searchsploit** has me covered here,

![](<../../.gitbook/assets/image (19).png>)

![](<../../.gitbook/assets/image (39).png>)

Info suggests that this plugin is vulnerable for Directory traversal by using /filedownload.php?ebookdownloadurl=\<Input>

For this, I check what users are present on the box, Cron jobs & Processes.

![](<../../.gitbook/assets/image (9).png>)

/etc/passwd

![/](<../../.gitbook/assets/image (11).png>)

etc/crontab

![](<../../.gitbook/assets/image (22).png>)

/Processes 1-1000 Fuzzed

![](<../../.gitbook/assets/image (31).png>)

Ref: [https://www.netspi.com/blog/technical/web-application-penetration-testing/directory-traversal-file-inclusion-proc-file-system/](https://www.netspi.com/blog/technical/web-application-penetration-testing/directory-traversal-file-inclusion-proc-file-system/)

**View Individual Processes**

![](<../../.gitbook/assets/image (3) (1).png>)

**gdbserver**

****

After discovering GDB Server running on port 1337, I'm back at Searchsploit looking for any vulnerabilities.

![](<../../.gitbook/assets/image (27).png>)

Based on the result, I am going for RCE.

![](<../../.gitbook/assets/image (14).png>)

![Usage](<../../.gitbook/assets/image (34) (1).png>)

Cat out the commandline and edit based on IP address and local port.

![](<../../.gitbook/assets/image (1).png>)

Setting up my listener and running the script.

![](<../../.gitbook/assets/image (37).png>)

Netcat listener and User Flag

![](<../../.gitbook/assets/image (32).png>)

Output of script.

### Gain an Initial Foothold (SSH) <a href="#4e59" id="4e59"></a>

Now that I have a way in, I will generate my SSH key-pair and drop the Public cert content into the 'authorized Keys' folder on Backdoor under User.

![](<../../.gitbook/assets/image (17).png>)

Copying contents of .pub to 'Authorized Keys' using echo.

![](<../../.gitbook/assets/image (15) (1).png>)

Connecting to Backdoor via SSH using my private key, authenticating as 'user'

![](<../../.gitbook/assets/image (30).png>)

![](<../../.gitbook/assets/image (24).png>)



### Privilage Escalation

using htop and combination of + u to select processes running under root.

![](<../../.gitbook/assets/image (29).png>)

After reviewing each process individually, Process 849 looks like a potenial target (Screen)

![](<../../.gitbook/assets/image (25).png>)

Checking out the Screen manual page

![](<../../.gitbook/assets/image (36) (1).png>)

Viewing current screens

![](<../../.gitbook/assets/image (12).png>)

Switching screens to Root

![](<../../.gitbook/assets/image (8).png>)

![](<../../.gitbook/assets/image (33) (1).png>)

Root flag found
