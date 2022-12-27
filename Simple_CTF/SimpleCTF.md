# Simple CTF 

## How many services are running under port 1000?
Use nmap to find the ports. We can see that ssh isn't running on it's default port. Therefore, the answer to the question is: **2**
Let's further investigate. Connecting to the ftp server (username anonymous), we find a file called **ForMitch.txt** in pub directory. 
Now we visit the website. It's a standard Apache Default Page. Nothing much.
Using Gobuster, we enumerate the subpages and find a page: **/simple**

## What is running on the higher port?
SSH

## What's the CVE you're using against the application? 
Visiting the **ip-address/simple**, we see that it uses CMS Made Simple version 2.2.8. Googling 'CMS Made Simple version 2.2.8 exploit' gives a link to
[exploit-db](https://www.exploit-db.com/exploits/46635). There we find the CVE: **CVE-2019-9053**.

## To what kind of vulnerability is the application vulnerable?
From the exploit-db, we can find the vulnerablity: **SQLi**

## What's the password?
Download the exploit from the exploit-db and run it. You will get the hash of the password.
Use hashcat or john-the-ripper to crack it: **secret**.

## Where can you login with the details obtained?
Remeber the open ports: one of them is for **ssh**. 

## What's the user flag?
Time to ssh to the server. Remeber that the ssh is running on non-default port. 
Therefore ```ssh -p 2222 mitch@10.10.103.23```. 
Use the obtained password to login.

The flag: **G00d j0b, keep up!**

(note: type bash to spawn bash).

## Is there any other user in the home directory? What's its name?
```ls /home ``` to see the other user: **sunbath**. 

## What can you leverage to spawn a privileged shell?
Use ```sudo -l``` to see which commands this user can run as sudo. In our case, the user mitch can run **vim**.

## What's the root flag?
On [GTFOBin](), we find the following command: ```sudo vim -c ':!/bin/sh' ```.
Now we are root, we can change to the root directory to find the flag: **W3ll d0n3. You made it!**
