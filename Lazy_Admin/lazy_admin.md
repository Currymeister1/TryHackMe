# Lazy Admin
An interesting room to practice reverse shell. Unlike other rooms, here you only need to answer two questions. 
But don't let that fool you. Finding these two flags isn't that easy.

## What is the user flag?
After deploying the machine, use ``` nmap -sV -sC <target-ip> ```.
We see that two ports are open: **ssh** and **http**.
Going to the website, we are greeted with the default apache config page.
Time to use gobuster. ```gobuster -u <target-ip> -w known_directory_names ```
We quickly find the **<target-ip>/content** page. Navigating there we can see that the web app is 
powered by SweetRice. 
Go to [exploit-db.com](https://www.exploit-db.com) to find a useful exploit.
There are few exploits. By reading them, the [Back-Up Disclosure](https://www.exploit-db.com/exploits/40718) one looks promising.
According to proof of concept: 'You can **access** to all **mysql backup** and download them from this directory.
http://localhost/**inc/mysql_backup**'

Remember that the Sweet Rice is hosted on <target-ip>/content. Append to this url ***inc/mysql_backup**.
There we find a single .sql file. Downlaod it and search for useful information.
Inside the file we can find the username: **manager** and hash: **42f749ade7f9e195bf475f37a44cafcb**. 
Use an online hash cracking tool, we get the password: **Password123**.

Going back to exploit-db, you can find another useful exploit: [Arbitary File Upload](https://www.exploit-db.com/exploits/40716). 
With this exploit, we will upload our reverse shell. 
Get the reverse shell from [Pentestmonkey](https://github.com/pentestmonkey/php-reverse-shell). 
And edit the ip and port. To find your ip go to https://tryhackme.com/access. 
(Note you need to change the extension from php to **php5**.
```python3 40716.py
   <target-ip>/content
   manager
   Password123
   php-reverse-shell.php5 ```
 After running the command you will see the link to your upload. Before clicking on it run ```nc -lvnp <port-for-reverse-shell> ```
 
 Click on the link, you will get the reverse shell. Navigate to ```/home/itguy``` to find the first flag: **THM{63e5bce9271952aad1113b6f1ac28a07}**
 
 ## What is the root flag?
 To get the root flag, run ``` sudo -l ``` to check which command this user can run as root. 
 We see that perl and a script called backup.pl in the **itguy home directory** can be run by this user.
 The backup.pl runs the script **/etc/copy.sh**. Which in itself is a reverse shell. 
 We have the permission to **write** to the **copy.sh**, but nano and vim aren't available. 
 Instead use echo to replace the target ip to your openvpn internal ip and the port to your desired port.
 Run a seperate instance of netcat: ```nc -lvnp <your-port>
 Run the script: ```sudo perl /home/itguy/backup.pl```
 Now you are logged as root. The flag is: **THM{6637f41d0177b6f37cb20d775124699f}**
