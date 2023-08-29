---
layout: post
title: Biohazard
date: 2023-08-24 08:24:16 +0700
tags: [cipher, base, root, stego]
toc: true
---

<p class="message">A CTF room based on the old-time survival horror game, Resident Evil. Can you survive until the end?</p>

> **Instructions**
> Welcome to Biohazard room, a puzzle-style CTF. Collecting the item, solving the puzzle and escaping the nightmare is your top priority. Can you survive until the end?

Because this room is a type of **puzzle-style CTF** → I will follow up the questions instead of going through normal phases of penetration testing. Let’s start!

### Deploy the machine and start the nightmare

No answer needed

### How many open ports?

Run `nmap` and you will find the answer ⇒ **3**

```tsx
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-22 09:53 EDT
Warning: 10.10.85.235 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.85.235
Host is up (0.19s latency).
Not shown: 65532 closed tcp ports (reset)
PORT   STATE SERVICE
21/tcp open  ftp
22/tcp open  ssh
80/tcp open  http
```

### What is the team name in operation

Visit the main page of the **http** service (port `80`):

![Untitled](/assets/Biohazard%20images/Untitled.png)

⇒ The answer is: **The STARS alpha team**

### What is the emblem flag

Click on the `mansion` and follow up its link:

![Untitled](/assets/Biohazard%20images/Untitled%201.png)

Press `Ctrl + U` or right-click and view the Page’s Source:

```tsx
<!doctype html>
        <head>
                <title>Main hall</title>
                <h1 align="center">Main hall</h1>
        </head>

        <body>
        <img alt="mainhall" src="../images/Mainhall12.jpg" style="display: block;margin-left: auto;margin-right: auto; width: 50%;"/>

        <p>The team reach the mansion safe and sound. However, it appear that Chris is missing</p>
	<p>Jill try to open the door but stopped by Weasker</p>
        <p>Suddenly, a gunshot can be heard in the nearby room. Weaker order Jill to make an investigate on the gunshot. Where is the room?</p>
	<!-- It is in the /diningRoom/ -->
        </body>

</html>
```

Notice at the comment line → Go to the `/diningRoom`

![Untitled](/assets/Biohazard%20images/Untitled%202.png)

Click on **YES**

![Untitled](/assets/Biohazard%20images/Untitled%203.png)

Answer: **emblem{fec832623ea498e20bf4fe1821d58727}**

### What is the lock pick flag?

Get back and refresh the page:

![Untitled](/assets/Biohazard%20images/Untitled%204.png)

Enter the previous `emblem` flag but it’s not correct. View the page source again and notice at the comment line:

```tsx
<!-- SG93IGFib3V0IHRoZSAvdGVhUm9vbS8= -->
```

Decode the string with **base64**:

```tsx
How about the /teaRoom/
```

Route to the `/teaRoom`

![Untitled](/assets/Biohazard%20images/Untitled%205.png)

Click on the **Lockpick** and save the flag:

![Untitled](/assets/Biohazard%20images/Untitled%206.png)

Asnwer: **lock_pick{037b35e2ff90916a9abf99129c8e1837}**

### What is the music sheet flag?

Visit the `/artRoom`

![Untitled](/assets/Biohazard%20images/Untitled%207.png)

![Untitled](/assets/Biohazard%20images/Untitled%208.png)

Save the list for easier following up all the directions. Then move on with the `/barRoom/`

![Untitled](/assets/Biohazard%20images/Untitled%209.png)

Enter the previous **lockpick** flag:

![Untitled](/assets/Biohazard%20images/Untitled%2010.png)

Click on **READ**:

![Untitled](/assets/Biohazard%20images/Untitled%2011.png)

Decode the string with **base32**:

```tsx
┌──(kali㉿kali)-[~/TryHackMe/Biohazard]
└─$ echo "NV2XG2LDL5ZWQZLFOR5TGNRSMQ3TEZDFMFTDMNLGGVRGIYZWGNSGCZLDMU3GCMLGGY3TMZL5" | base32 -d
music_sheet{362d72deaf65f5bdc63daece6a1f676e}
```

Answer: **music_sheet{362d72deaf65f5bdc63daece6a1f676e}**

### What is the gold emblem flag?

Get back to the browser and enter the **music sheet** flag:

![Untitled](/assets/Biohazard%20images/Untitled%2012.png)

![Untitled](/assets/Biohazard%20images/Untitled%2013.png)

Click YES:

![Untitled](/assets/Biohazard%20images/Untitled%2014.png)

Answer: **gold_emblem{58a8c41a9d08b8a4e38d02a4d7ff4843}**

### What is the shield key flag

Go back and refresh the **Secret bar room** page then enter the flag:

![Untitled](/assets/Biohazard%20images/Untitled%2015.png)

![Untitled](/assets/Biohazard%20images/Untitled%2016.png)

The **gold_emblem** key is not the right one, remember the first **emblem** key? Use it!

![Untitled](/assets/Biohazard%20images/Untitled%2017.png)

![Untitled](/assets/Biohazard%20images/Untitled%2018.png)

The `rebecca` might be an username or a key for decrypt/decode something. Save it for later use.

Go back to the first found room `/diningRoom/` and enter the **gold_emblem** key:

![Untitled](/assets/Biohazard%20images/Untitled%2019.png)

![Untitled](/assets/Biohazard%20images/Untitled%2020.png)

Decode the string with **Vigenere Cipher** and the key is **rebecca**:

![Untitled](/assets/Biohazard%20images/Untitled%2021.png)

Go to `/the_great_shield_key.html`:

![Untitled](/assets/Biohazard%20images/Untitled%2022.png)

Answer: **shield_key{48a7a9227cd7eb89f0a062590798cbac}**

### What is the blue gem flag?

Open the list `MansionMap` and move on to the next one `/diningRoom2F/`

![Untitled](/assets/Biohazard%20images/Untitled%2023.png)

View it’s source:

```tsx
<html>
        <head>
                <title>Dining room 2F</title>
                <h1 align="center">Dining room 2F</h1>
        </head>

        <body>
        <img alt="dining room 2F" src="../images/Vlcsnap-2015-01-26-08h54m37s183.png" style="display: block;margin-left: auto;margin-right: auto; width: 50%;"/>

	<p>Once Jill reach the room, she saw a tall status with a shiining blue gem on top of it. However, she can't reach it</p>
	<!-- Lbh trg gur oyhr trz ol chfuvat gur fgnghf gb gur ybjre sybbe. Gur trz vf ba gur qvavatEbbz svefg sybbe. Ivfvg fnccuver.ugzy -->
        </body>

</html>
```

Decode the comment text with **ROT13:**

```tsx
┌──(kali㉿kali)-[~/TryHackMe/Biohazard]
└─$ hURL --rot13 -f diningRoom2F_comment.txt

Original file   :: diningRoom2F_comment.txt
ROT13 decoded   :: You get the blue gem by pushing the status to the lower floor. The gem is on the diningRoom first floor. Visit sapphire.html
```

Visit the `sapphire.html` from the `/diningRoom/`:

![Untitled](/assets/Biohazard%20images/Untitled%2024.png)

Answer: **blue_jewel{e1d457e96cac640f863ec7bc475d48aa}**

### What is the FTP username? What is the FTP password?

Take a look at the list of rooms again (`MansionMap.html`) → Visit the room `/tigerStatusRoom/`

![Untitled](/assets/Biohazard%20images/Untitled%2025.png)

**_gem_** is a kind of jewel and notice at the tiger’s eye in the picture with blue color → Enter the **blue_jewel** flag:

![Untitled](/assets/Biohazard%20images/Untitled%2026.png)

There are totally 4 **crests**! Let’s decode the **crest1** with **base64** and **base32**:

```tsx
┌──(kali㉿kali)-[~/TryHackMe/Biohazard]
└─$ echo "S0pXRkVVS0pKQkxIVVdTWUpFM0VTUlk9" | base64 -d | base32 -d
RlRQIHVzZXI6IG
```

The **crest2** is in the `/galleryRoom/`

![Untitled](/assets/Biohazard%20images/Untitled%2027.png)

![Untitled](/assets/Biohazard%20images/Untitled%2028.png)

Decode it with **base32** and **base52**:

```tsx
┌──(kali㉿kali)-[~/TryHackMe/Biohazard]
└─$ echo "GVFWK5KHK5WTGTCILE4DKY3DNN4GQQRTM5AVCTKE" | base32 -d | base58 -d
h1bnRlciwgRlRQIHBh
```

The **crest3** is in the `/armorRoom/` by using the **shield** **key** flag:

![Untitled](/assets/Biohazard%20images/Untitled%2029.png)

![Untitled](/assets/Biohazard%20images/Untitled%2030.png)

![Untitled](/assets/Biohazard%20images/Untitled%2031.png)

Decode the **crest3** with **base64** + **binary** + **hex**:

![Untitled](/assets/Biohazard%20images/Untitled%2032.png)

→ **Crest3**: c3M6IHlvdV9jYW50X2h

The final **crest4** could be found from the `/attic/` using the **shield key** flag too:

![Untitled](/assets/Biohazard%20images/Untitled%2033.png)

![Untitled](/assets/Biohazard%20images/Untitled%2034.png)

![Untitled](/assets/Biohazard%20images/Untitled%2035.png)

```tsx
┌──(kali㉿kali)-[~/TryHackMe/Biohazard]
└─$ echo "gSUERauVpvKzRpyPpuYz66JDmRTbJubaoArM6CAQsnVwte6zF9J4GGYyun3k5qM9ma4s" | base58 -d > crest4_hex

┌──(kali㉿kali)-[~/TryHackMe/Biohazard]
└─$ hURL --hex -f crest4_hex

Original file     :: crest4_hex
ASCII/RAW DEcoded :: pZGVfZm9yZXZlcg==
```

Combine all the **crests** and we have:

```tsx
RlRQIHVzZXI6IGh1bnRlciwgRlRQIHBhc3M6IHlvdV9jYW50X2hpZGVfZm9yZXZlcg==
```

Decode it with **base64**:

```tsx
┌──(kali㉿kali)-[~/TryHackMe/Biohazard]
└─$ cat crest_combination.txt | base64 -d
FTP user: hunter, FTP pass: you_cant_hide_forever
```

### Where is the hidden directory mentioned by Barry?

Use the **FTP creds** to login FTP:

```tsx
┌──(kali㉿kali)-[~/TryHackMe/Biohazard]
└─$ ftp 10.10.224.135
Connected to 10.10.224.135.
220 (vsFTPd 3.0.3)
Name (10.10.224.135:kali): hunter
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls -la
229 Entering Extended Passive Mode (|||38340|)
150 Here comes the directory listing.
drwxrwxrwx    2 1002     1002         4096 Sep 20  2019 .
drwxrwxrwx    2 1002     1002         4096 Sep 20  2019 ..
-rw-r--r--    1 0        0            7994 Sep 19  2019 001-key.jpg
-rw-r--r--    1 0        0            2210 Sep 19  2019 002-key.jpg
-rw-r--r--    1 0        0            2146 Sep 19  2019 003-key.jpg
-rw-r--r--    1 0        0             121 Sep 19  2019 helmet_key.txt.gpg
-rw-r--r--    1 0        0             170 Sep 20  2019 important.txt
226 Directory send OK.
```

Transfer all the files to local machine. Then read the `important.txt`:

```tsx
┌──(kali㉿kali)-[~/TryHackMe/Biohazard/ftp]
└─$ cat important.txt
Jill,

I think the helmet key is inside the text file, but I have no clue on decrypting stuff. Also, I come across a /hidden_closet/ door but it was locked.

From,
Barry
```

Answer: **/hidden_closet/**

### Password for the encrypted file?

Use `steghide` to extract the `001-key.jpg` to get the **key1**:

```tsx
┌──(kali㉿kali)-[~/TryHackMe/Biohazard/ftp]
└─$ steghide --extract -sf 001-key.jpg
Enter passphrase:
wrote extracted data to "key-001.txt".
```

Use `exiftool` to get the **key 2** from the **Comment** field:

```tsx
┌──(kali㉿kali)-[~/TryHackMe/Biohazard/ftp]
└─$ exiftool 002-key.jpg
ExifTool Version Number         : 12.57
File Name                       : 002-key.jpg
Directory                       : .
File Size                       : 2.2 kB
File Modification Date/Time     : 2019:09:19 02:08:31-04:00
File Access Date/Time           : 2023:08:23 07:07:35-04:00
File Inode Change Date/Time     : 2023:08:23 07:07:35-04:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Comment                         : 5fYmVfZGVzdHJveV9
Image Width                     : 100
Image Height                    : 80
Encoding Process                : Progressive DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 100x80
Megapixels                      : 0.008

┌──(kali㉿kali)-[~/TryHackMe/Biohazard/ftp]
└─$ cat key-002.txt
5fYmVfZGVzdHJveV9
```

Use `binwalk` to extract the `003-key.jpg`:

```tsx
┌──(kali㉿kali)-[~/TryHackMe/Biohazard/ftp]
└─$ binwalk -e 003-key.jpg

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             JPEG image data, JFIF standard 1.01
1930          0x78A           Zip archive data, at least v2.0 to extract, uncompressed size: 14, name: key-003.txt
2124          0x84C           End of Zip archive, footer length: 22
```

Get the **key 3** inside the extracted folder:

```tsx
┌──(kali㉿kali)-[~/TryHackMe/Biohazard/ftp]
└─$ cd _003-key.jpg.extracted

┌──(kali㉿kali)-[~/TryHackMe/Biohazard/ftp/_003-key.jpg.extracted]
└─$ cat key-003.txt
3aXRoX3Zqb2x0
```

Combine **3 keys** and decode it with **base64**:

```tsx
┌──(kali㉿kali)-[~/TryHackMe/Biohazard/ftp]
└─$ cat key* | tr -d "\n" > keys

┌──(kali㉿kali)-[~/TryHackMe/Biohazard/ftp]
└─$ cat keys
cGxhbnQ0Ml9jYW5fYmVfZGVzdHJveV93aXRoX3Zqb2x0
┌──(kali㉿kali)-[~/TryHackMe/Biohazard/ftp]
└─$ cat keys | base64 -d
plant42_can_be_destroy_with_vjolt
```

Answer: **plant42_can_be_destroy_with_vjolt**

### What is the helmet key flag?

Use the result as the key to extract the `helmet_key.txt.gpg` and get the key:

```tsx
┌──(kali㉿kali)-[~/TryHackMe/Biohazard/ftp]
└─$ gpg helmet_key.txt.gpg
gpg: WARNING: no command supplied.  Trying to guess what you mean ...
gpg: AES256.CFB encrypted data
gpg: encrypted with 1 passphrase
```

```tsx
┌──(kali㉿kali)-[~/TryHackMe/Biohazard/ftp]
└─$ cat helmet_key.txt
helmet_key{458493193501d2b94bbab2e727f8db4b}
```

### What is the SSH login username?

Re-visit the left room in the `MansionMap.html` which is `/studyRoom/` and enter the **helmet key** flag:

![Untitled](/assets/Biohazard%20images/Untitled%2036.png)

![Untitled](/assets/Biohazard%20images/Untitled%2037.png)

Click **EXAMINE** or copy its link and use `wget` to download a file:

```tsx
┌──(kali㉿kali)-[~/TryHackMe/Biohazard]
└─$ wget http://10.10.224.135/studyRoom28341c5e98c93b89258a6389fd608a3c/doom.tar.gz
--2023-08-23 07:20:28--  http://10.10.224.135/studyRoom28341c5e98c93b89258a6389fd608a3c/doom.tar.gz
Connecting to 10.10.224.135:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 149 [application/x-gzip]
Saving to: ‘doom.tar.gz’

doom.tar.gz                  100%[==============================================>]     149  --.-KB/s    in 0s

2023-08-23 07:20:29 (24.6 MB/s) - ‘doom.tar.gz’ saved [149/149]
```

Use `gunzip` to unzip the file and then use `tar -xvf` to extract the data inside:

```tsx
┌──(kali㉿kali)-[~/TryHackMe/Biohazard]
└─$ gunzip doom.tar.gz

┌──(kali㉿kali)-[~/TryHackMe/Biohazard]
└─$ tar -xvf doom.tar
eagle_medal.txt

┌──(kali㉿kali)-[~/TryHackMe/Biohazard]
└─$ cat eagle_medal.txt
SSH user: umbrella_guest
```

Answer: **umbrealla_guest**

### What is the SSH login password?

Visit the `/hidden_closet/` and enter the **helmet key** flag:

![Untitled](/assets/Biohazard%20images/Untitled%2038.png)

![Untitled](/assets/Biohazard%20images/Untitled%2039.png)

Click \***\*EXAMINE:**

![Untitled](/assets/Biohazard%20images/Untitled%2040.png)

Answer: **T_virus_rules**

### Who the STARS bravo team leader

![Untitled](/assets/Biohazard%20images/Untitled%2041.png)

Answer: **Enrico**

### Where you found Chris?

SSH to the target system using the found creds:

```tsx
umbrella_guest@umbrella_corp:~$ id
uid=1001(umbrella_guest) gid=1001(umbrella) groups=1001(umbrella)
umbrella_guest@umbrella_corp:~$ ls -l
total 0
umbrella_guest@umbrella_corp:~$ ls -la
total 64
drwxr-xr-x  8 umbrella_guest umbrella 4096 Sep 20  2019 .
drwxr-xr-x  5 root           root     4096 Sep 20  2019 ..
-rw-r--r--  1 umbrella_guest umbrella  220 Sep 19  2019 .bash_logout
-rw-r--r--  1 umbrella_guest umbrella 3771 Sep 19  2019 .bashrc
drwxrwxr-x  6 umbrella_guest umbrella 4096 Sep 20  2019 .cache
drwxr-xr-x 11 umbrella_guest umbrella 4096 Sep 19  2019 .config
-rw-r--r--  1 umbrella_guest umbrella   26 Sep 19  2019 .dmrc
drwx------  3 umbrella_guest umbrella 4096 Sep 19  2019 .gnupg
-rw-------  1 umbrella_guest umbrella  346 Sep 19  2019 .ICEauthority
drwxr-xr-x  2 umbrella_guest umbrella 4096 Sep 20  2019 .jailcell
drwxr-xr-x  3 umbrella_guest umbrella 4096 Sep 19  2019 .local
-rw-r--r--  1 umbrella_guest umbrella  807 Sep 19  2019 .profile
drwx------  2 umbrella_guest umbrella 4096 Sep 20  2019 .ssh
-rw-------  1 umbrella_guest umbrella  109 Sep 19  2019 .Xauthority
-rw-------  1 umbrella_guest umbrella 7546 Sep 19  2019 .xsession-errors
```

Go to `.jailcell`:

```tsx
umbrella_guest@umbrella_corp:~$ cd .jailcell/
umbrella_guest@umbrella_corp:~/.jailcell$ ls -la
total 12
drwxr-xr-x 2 umbrella_guest umbrella 4096 Sep 20  2019 .
drwxr-xr-x 8 umbrella_guest umbrella 4096 Sep 20  2019 ..
-rw-r--r-- 1 umbrella_guest umbrella  501 Sep 20  2019 chris.txt
```

Answer: **jailcell**

### Who is the traitor?

Read the file `christ.txt`

```tsx
umbrella_guest@umbrella_corp:~/.jailcell$ cat chris.txt
Jill: Chris, is that you?
Chris: Jill, you finally come. I was locked in the Jail cell for a while. It seem that weasker is behind all this.
Jil, What? Weasker? He is the traitor?
Chris: Yes, Jill. Unfortunately, he play us like a damn fiddle.
Jill: Let's get out of here first, I have contact brad for helicopter support.
Chris: Thanks Jill, here, take this MO Disk 2 with you. It look like the key to decipher something.
Jill: Alright, I will deal with him later.
Chris: see ya.

MO disk 2: albert
```

Answer: **weasker**

### The login password for the traitor?

Re-visit the `/hidden_closet/` and enter the **helmet key** flag. Then click **READ**:

![Untitled](/assets/Biohazard%20images/Untitled%2042.png)

Use the key `ablert` from the `chrits.txt` and decode the string with **Vigenere Cipher**:

![Untitled](/assets/Biohazard%20images/Untitled%2043.png)

Answer: **stars_members_are_my_guinea_pig**

### The name of the ultimate form

Switch to user `weasker` using the login password found:

```tsx
umbrella_guest@umbrella_corp:~/.jailcell$ su weasker
Password:
weasker@umbrella_corp:/home/umbrella_guest/.jailcell$ cd
weasker@umbrella_corp:
```

Read the note of `weasker`:

```tsx
weasker@umbrella_corp:~$ ls -la
total 80
drwxr-xr-x  9 weasker weasker 4096 Sep 20  2019 .
drwxr-xr-x  5 root    root    4096 Sep 20  2019 ..
-rw-------  1 weasker weasker   18 Sep 20  2019 .bash_history
-rw-r--r--  1 weasker weasker  220 Sep 18  2019 .bash_logout
-rw-r--r--  1 weasker weasker 3771 Sep 18  2019 .bashrc
drwxrwxr-x 10 weasker weasker 4096 Sep 20  2019 .cache
drwxr-xr-x 11 weasker weasker 4096 Sep 20  2019 .config
drwxr-xr-x  2 weasker weasker 4096 Sep 19  2019 Desktop
drwx------  3 weasker weasker 4096 Sep 19  2019 .gnupg
-rw-------  1 weasker weasker  346 Sep 20  2019 .ICEauthority
drwxr-xr-x  3 weasker weasker 4096 Sep 19  2019 .local
drwx------  5 weasker weasker 4096 Sep 19  2019 .mozilla
-rw-r--r--  1 weasker weasker  807 Sep 18  2019 .profile
drwx------  2 weasker weasker 4096 Sep 19  2019 .ssh
-rw-r--r--  1 weasker weasker    0 Sep 20  2019 .sudo_as_admin_successful
-rw-r--r--  1 root    root     534 Sep 20  2019 weasker_note.txt
-rw-------  1 weasker weasker  109 Sep 20  2019 .Xauthority
-rw-------  1 weasker weasker 5548 Sep 20  2019 .xsession-errors
-rw-------  1 weasker weasker 6749 Sep 20  2019 .xsession-errors.old
weasker@umbrella_corp:~$ cat weasker_note.txt
Weaker: Finally, you are here, Jill.
Jill: Weasker! stop it, You are destroying the  mankind.
Weasker: Destroying the mankind? How about creating a 'new' mankind. A world, only the strong can survive.
Jill: This is insane.
Weasker: Let me show you the ultimate lifeform, the Tyrant.

(Tyrant jump out and kill Weasker instantly)
(Jill able to stun the tyrant will a few powerful magnum round)

Alarm: Warning! warning! Self-detruct sequence has been activated. All personal, please evacuate immediately. (Repeat)
Jill: Poor bastard
```

Answer: **Tyrant** _(Weasker: Let me show you the ultimate lifeform, the Tyrant.)_

### The root flag

```tsx
weasker@umbrella_corp:~$ sudo -l
[sudo] password for weasker:
Matching Defaults entries for weasker on umbrella_corp:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User weasker may run the following commands on umbrella_corp:
    (ALL : ALL) ALL
```

The `weasker` user can run all `sudo` commands → Simply type `sudo su` to switch to the **root** user:

```tsx
weasker@umbrella_corp:~$ sudo su
root@umbrella_corp:/home/weasker# id
uid=0(root) gid=0(root) groups=0(root)
root@umbrella_corp:/home/weasker# cd /root
root@umbrella_corp:~# cat root.txt
In the state of emergency, Jill, Barry and Chris are reaching the helipad and awaiting for the helicopter support.

Suddenly, the Tyrant jump out from nowhere. After a tough fight, brad, throw a rocket launcher on the helipad. Without thinking twice, Jill pick up the launcher and fire at the Tyrant.

The Tyrant shredded into pieces and the Mansion was blowed. The survivor able to escape with the helicopter and prepare for their next fight.

The End

flag: 3c5794a00dc56c35f2bf096571edf3bf
```
