




## Task 7

1. I just ran `strings hello_1577977122465.hello` and the flag appeared on one of the lines in the output. You could also filter your strings search to find "THM{" but the output for this file was short enough for me to easily find it

## Task 8

1. I identified that this was an encoded bit of text, intended to be decoded but wasn't sure on the encoding method which was being used.  I tried `hashid` and `hash-identifier` but got no results. The text looked similar to base64 so I opened up [CyberChef](https://gchq.github.io/CyberChef/) and basically manually brute force all the different base encoding methods
2. From this process base58 was being used and I simply pasted the hash in the input, selected from base58, baked and hacked!

## Task 9

1. So this was pretty easy because I am a bit of an encoding nerd and knew right away it was a Caesar Cipher right away but didn't know the shift count. I could tell this just by looking at the pattern of the letters used in relation to their indexes on the alphabet.
2. To solve the flag I just went to [Cryptii](https://cryptii.com/pipes/caesar-cipher) and add the text, left the shift on default which was 7 and boom cracked! I would've just brute forced shifts had the default not worked.

## Task 10

1. This was pretty straightforward, just inspected the page in my browser and found the flag in a hidden developer comment

## Task 11

1. so basically just read the file in a hex editor to find the header is set to "3D" instead of PNG like the extension suggests.
2. Use hexeditor to change the first characters to match the pattern for PNG headers which is "**89 50 4E 47**"
## Task 12

1. Well it seems the OP made for this challenge on Reddit is archived so there's no way to do it but basically just use google dorking to find flags like the one in this challenge

## Task 13

1. This take we got some very strange looking encoding which I had no idea on how to approach. I did some google searches on "weird" encoding formats and even used some AI searches to help me identify the pattern used a "Brainfuck" encoding method.
2.  I went to [Dcode](https://www.dcode.fr/brainfuck-language) online and selected "Brainfuck", pasted the message and boom we get the flag.

## Task 14

1. This task stumped at first but after reading the hint and doing some googling on XOR and how it works it was as simple as going [XOR Calculator](https://xor.pw/#)and simply inputting s1, s2 and selecting ASCII as the output gives you the flag

## Task 15

1. the task name "binary walk" provided a hint to use `binwalk` to find the flag
2. I ran `binwalk -e hell_1578018688127.jpg` to examine the file contents and found a zip archive containing `hello_there.txt`
3. I simply ran `unzip hell_1578018688127.jpg` to extract the txt file and catted out its contents to retrieve the flag

## Task 16

1. So this task involves using a tool called stegsolver to extract the flag.
2.  I ran this command to get the tool`wget http://www.caesum.com/handbook/Stegsolve.jar -O stegsolve.jar` and then we run the tool, open the file and analyze it through the different pane screens till we get a readable version of the flag

## Task 17

1. this was straight forward, scan the QR code, listen to the soundcloud song and obtain the flag from the song.

## Task 18

1. With this task I knew to use the wayback machine as it is a tool which provides snapshots of websites in the past and the task provides you with the url and specific date to find the flag at.
2. Simple inputting the url and selecting the specific on Wayback showed me the flag 

## Task 19

1. So I started by making a text file with all the alphabet letters and what they translate to in the encrypted text by manually mapping these using that we are given the first part of the text as TRYHACKME and equating them to what each letter is represented by in the cipher text
2. This and the tasks mention of the key revealed it was a Vigenère cipher and proceeded to use Dcode from earlier.
3. We input the encrypted text and select "Knowing a plain text word" in the decryption options section and add TRYHACKME. We than click decrypt and retrieve the flag.

## Task 20

1. So I used the hint for this one which tells you to convert the decimal to hex and then from hex to ASCII or plain text
2. I went to [RapidTables](https://www.rapidtables.com/convert/number/decimal-to-hex.html) , converting decimal to hex in the calculator and than from hex to text to retrieve the flag

## Task 21

1. So in this task we get a pcap file, which immediately points towards using wireshark to analyze the file and extract the flag
2. So we know we are looking for a GET request as the lab specifies the scenario as finding out their neighbour was up to no good via hacking their wifi. We can filter for these requests by adding `http.request.method == GET` to the filter search bar where only one result is returned
3. We right click on the packet -> follow HTTP stream and find the flag in this output