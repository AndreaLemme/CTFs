

## Enumeration

1. Starting off with our initial nmap scan we 3 open ports with a web server on port 80

```
┌──(kali㉿kali)-[~/Documents/labs/allinone]
└─$ nmap -sV -sC 10.114.164.78 -o nmap.txt
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-24 09:13 SAST
Nmap scan report for 10.114.164.78
Host is up (0.20s latency).
Not shown: 997 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.5
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:192.168.144.74
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.5 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.13 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 1c:50:8d:68:82:70:c0:fb:54:98:f5:52:1d:3c:e5:25 (RSA)
|   256 1d:48:75:19:49:c3:21:80:84:3e:e5:03:46:a4:2f:67 (ECDSA)
|_  256 cd:44:f9:80:e3:2b:63:fc:ea:a9:46:6a:01:8e:d3:82 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.41 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 41.21 seconds

```

2. Visiting the web server shows a default Apache page so we will have to try some other enumeration techniques to find out more about the server. However, our nmap script scan also revealed FTP allows anonymous login so lets explore this first before moving on.
```
┌──(kali㉿kali)-[~/Documents/labs/allinone]
└─$ ftp 10.114.164.78 
Connected to 10.114.164.78.
220 (vsFTPd 3.0.5)
Name (10.114.164.78:kali): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||5373|)
150 Here comes the directory listing.
226 Directory send OK.
ftp> ls -la
229 Entering Extended Passive Mode (|||37190|)
150 Here comes the directory listing.
drwxr-xr-x    2 0        115          4096 Oct 06  2020 .
drwxr-xr-x    2 0        115          4096 Oct 06  2020 ..
226 Directory send OK.

```

3. Okay so we get nothing in our FTP directory with anonymous account which means we need to explore the web server for more details but remember we have FTP anonymous access as it may come in handy later on. I also want to run a quick online and command line searchsploit search on the open ports service version just to see if maybe we have some available exploits at our disposal
4. As expected, none of the services were running vulnerable versions so we continued with a directory scan on the web server using `ffuf` 
```
┌──(kali㉿kali)-[~/Documents/labs/allinone]
└─$ ffuf -u http://10.114.164.78/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.114.164.78/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

# directory-list-2.3-medium.txt [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 336ms]
# or send a letter to Creative Commons, 171 Second Street, [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 386ms]
# Attribution-Share Alike 3.0 License. To view a copy of this [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 331ms]
#                       [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 240ms]
# license, visit http://creativecommons.org/licenses/by-sa/3.0/ [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 1989ms]
# Copyright 2007 James Fisher [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 2995ms]
# Suite 300, San Francisco, California, 94105, USA. [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 2921ms]
#                       [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 5103ms]
# Priority ordered case-sensitive list, where entries were found [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 5104ms]
#                       [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 5154ms]
# on at least 2 different hosts [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 5173ms]
                        [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 5174ms]
# This work is licensed under the Creative Commons [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 5348ms]
#                       [Status: 200, Size: 10918, Words: 3499, Lines: 376, Duration: 5353ms]
wordpress               [Status: 301, Size: 318, Words: 20, Lines: 10, Duration: 213ms]
hackathons              [Status: 200, Size: 197, Words: 19, Lines: 64, Duration: 157ms]
[WARN] Caught keyboard interrupt (Ctrl-C)


```

5. Our scan revealed the wordpress and hackathons directory and I stopped it here just for time sack as navigating to these directories gives us a lot of info we can use to leverage against the application.
6. The hackathons page reveals just a text snippet, saying "Damn I hate the smell of Vinegar!", but expecting the page source code we find these in the developer comments of the page and could potentially be related to login credentials somewhere in the lab
```
<!-- Dvc W@iyur@123 -->
<!-- KeepGoing -->
```

7. So on the WordPress page we run our wordpress scanner to enumerate plugins and users with `wpscan --url http://10.114.164.78/wordpress/` and found the user elyana, the WordPress version and its plugins. The plugins included a plugin called Mail Rasta which seems to be an exploitable with a searchsploit module available with the details on how to exploit this. 