# Agent Sudo CTF
A very fun and challenging CTF. 

## Enumerate
Starting with scanning our target with nmap: ``` nmap -sV -sC target-ip ```
### How many open ports?
3

### How you redirect yourself to a secret page?
This was very hard to find. First, I thought FTP server contains some information regarding this. But nmap showed that the 
ftp server can't be access anonymously. Then I visited the the webpage. Webpage tells us to set user-agent to our codename.
As you can see that the message was written by Agent R, then the codenames for detectives must be some alphabet. 
Open burpsuite and intercept the traffic. Send the request to intruder and set the user-agent field to §§. 
Now get a list A-Z and start the attack. If you check the length, you will see that payload C has the length of **422**.
Now see the response. In location field: **agent_C_attention.php**.
This is the **secret page**.

### What is the agent name?
The name of the agent can be found on the website: **chris**.

## Hash cracking and brute-force 
### FTP password
To find the FTP password, you can either use hydra or [ncrack](https://nmap.org/ncrack/).
```ncrack --user chris -P wordlists/rockyou.txt ftp://<target-ip> ```
The password is: **crystal**

### Zip file password
Log into ftp server and download all files. I tried to read meta-data of the images but it was no luck. 
The .txt file does say that password is stored inside the images. I had to look up online and found strings command can be helpful.
``` string cutie.png ``` does display something useful: 
```
To_agentR.txt
W\_z#
2a>=
To_agentR.txt
EwwT
```
To extract hidden files, we can use [binwalk](https://www.kali.org/tools/binwalk/). 
```binwalk -e cutie.png ```
We now get a directory containing the zip file. 
The access the zip file we need a password. It's time to call the john.
Depending on your version of john, you may execute this command differntly. 
I am using john-the-ripper from snap.
```john-the-ripper.zip2john 8702.zip > hash.txt ```
Now we have the hash, let's crack it.
``` john hash.txt rockyou.txt ```
The password is: **alien**.

### steg password
Now using the password, we can unzip the file. We have some encrypted text: **QXJlYTUx**. 
Using the [hash identifier](https://hashes.com/en/tools/hash_identifier), we can see that the string is in base64.
Decrypt the string, the answer is: **Area51**

### Who is the other agent (in full name)?
To find the name of the other agent was kinda hard. After searching online, I stumbled on [stegcrack](https://www.kali.org/tools/stegcracker/)
