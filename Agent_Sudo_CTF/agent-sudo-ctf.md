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
```stegcracker cute-alien.jpg pass.txt ```. (Save the password in a txt file)
You will get .out file. Open it to see the name of the agent **james** and password **hackerrules**.

## Capture the user flag 
With the information, use ssh login.
### What is the user flag?
The flag is in the home directory: **b03d975e8c92a7c04146cfa7a5a313c7**
### What is the incident of the photo called? 
Use scp to copy the image to local: ``` scp james@10.10.229.189:~/Alien_autospy.jpg .```. Here I was again dumbfounded and have to look at the hint. 
Its say reverse and Fox News. Use Google reverse image search and find the article by Fox News.
The answer is: **Roswell alien autopsy**

## Privilege escalation 
Coming to the fun part.

### CVE number for the escalation 
Use ```sudo -l``` and we see that ```(ALL, !root) /bin/bash```. 
If you google this, you will find there is a [Security Bypass exploit](https://www.exploit-db.com/exploits/47502). 
Download the exploit from exploit-db and using ``` scp 47502.py james@10.10.229.189:~ ``` copy it to the remote.
Now ```python3 47502.py ```.
You should be logged as root now. 
Btw CVE is: **CVE-2019-14287**.

### What is the root flag?
The root flag is in the root home directory: **b53a02f55b57d4439e3341834d70c062**

### (Bonus) Who is Agent R?
The name of Agent R is also found in that file: **DesKel**

