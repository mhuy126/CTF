---
layout: post
title: TryHackMe - Classic Passwd
date: 2023-10-08 23:50:00 +0700
tags: [security, assembly reversing, bypass]
toc: true
---

<p class="message">Practice your skills in reversing and get the flag bypassing the login</p>

I forgot my password, can you give me access to the program?

| Title      | Classic Passwd                       |
| ---------- | ------------------------------------ |
| Difficulty | Medium                               |
| Author     | N0obit4                              |
| Tags       | security, assembly reversing, bypass |

<aside>
💡 My note: This room is very simple <strong>IF</strong> you know how to use the debugger tools (gdb, ltrace, radare2,…)

</aside>

## Overview

This is the target file that need to analyze to get the flag:

```
┌──(kali㉿kali)-[~/TryHackMe/classicpasswd]
└─$ ls -l && file *
total 20
-rwxr--r-- 1 kali kali 16928 Oct  8 12:09 Challenge.Challenge
Challenge.Challenge: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=b80ce38cb25d043128bc2c4e1e122c3d4fbba7f7, for GNU/Linux 3.2.0, not stripped
```

Run as the first try and it requires a **username** for authentication:

```
┌──(kali㉿kali)-[~/TryHackMe/classicpasswd]
└─$ ./Challenge.Challenge
Insert your username: idontknow

Authentication Error
```

## Analyze

### Method 1: Radare2 (r2)

List the functions of file:

```
┌──(kali㉿kali)-[~/TryHackMe/classicpasswd]
└─$ r2 Challenge.Challenge
Warning: run r2 with -e bin.cache=true to fix relocations in disassembly
[0x000010a0]> aaaa
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Finding and parsing C++ vtables (avrr)
[x] Type matching analysis for all functions (aaft)
[x] Propagate noreturn information (aanr)
[x] Finding function preludes
[x] Enable constraint types analysis for variables
[0x000010a0]> afl
0x000010a0    1 43           entry0
0x000010d0    4 41   -> 34   sym.deregister_tm_clones
0x00001100    4 57   -> 51   sym.register_tm_clones
0x00001140    5 57   -> 50   sym.__do_global_dtors_aux
0x00001090    1 6            sym.imp.__cxa_finalize
0x00001180    1 5            entry.init0
0x00001000    3 23           sym._init
0x00001380    1 1            sym.__libc_csu_fini
0x00001185    4 260          sym.vuln
0x00001384    1 9            sym._fini
0x00001320    4 93           sym.__libc_csu_init
0x000012f6    1 31           main
0x00001289   10 109          sym.gfl
0x00001030    1 6            sym.imp.strcpy
0x00001040    1 6            sym.imp.puts
0x00001050    1 6            sym.imp.printf
0x00001060    1 6            sym.imp.strcmp
0x00001070    1 6            sym.imp.__isoc99_scanf
0x00001080    1 6            sym.imp.exit
```

Navigate the `main()` function to see what it does:

```
[0x000010a0]> s main
[0x000012f6]> pdf
            ; DATA XREF from entry0 @ 0x10bd
┌ 31: int main (int argc, char **argv, char **envp);
│           0x000012f6      55             push rbp
│           0x000012f7      4889e5         mov rbp, rsp
│           0x000012fa      b800000000     mov eax, 0
│           0x000012ff      e881feffff     call sym.vuln
│           0x00001304      b800000000     mov eax, 0
│           0x00001309      e87bffffff     call sym.gfl
│           0x0000130e      b800000000     mov eax, 0
│           0x00001313      5d             pop rbp
└           0x00001314      c3             ret
```

The `main()` function calls to 2 more functions `sym.vuln()` and `sym.gfl()`.

While the `sym.glf()` function returns the flag of the room:

```
[0x000012f6]> s sym.gfl
[0x00001289]> pdf
            ; CALL XREF from main @ 0x1309
┌ 109: sym.gfl ();
│           ; var signed int var_8h @ rbp-0x8
│           ; var signed int var_4h @ rbp-0x4
│           0x00001289      55             push rbp
│           0x0000128a      4889e5         mov rbp, rsp
│           0x0000128d      4883ec10       sub rsp, 0x10
│           0x00001291      c745fcd5c852.  mov dword [var_4h], 0x52c8d5
│       ┌─< 0x00001298      eb4f           jmp 0x12e9
│       │   ; CODE XREF from sym.gfl @ 0x12f0
│      ┌──> 0x0000129a      817dfc788a63.  cmp dword [var_4h], 0x638a78
│     ┌───< 0x000012a1      7542           jne 0x12e5
│     │╎│   0x000012a3      c745f8741400.  mov dword [var_8h], 0x1474  ; 't\x14'
│    ┌────< 0x000012aa      eb30           jmp 0x12dc
│    ││╎│   ; CODE XREF from sym.gfl @ 0x12e3
│   ┌─────> 0x000012ac      817df8302100.  cmp dword [var_8h], 0x2130
│  ┌──────< 0x000012b3      7523           jne 0x12d8
│  │╎││╎│   0x000012b5      8b55f8         mov edx, dword [var_8h]
│  │╎││╎│   0x000012b8      8b45fc         mov eax, dword [var_4h]
│  │╎││╎│   0x000012bb      89c6           mov esi, eax
│  │╎││╎│   0x000012bd      488d3d790d00.  lea rdi, str.THM_d_d        ; 0x203d ; "THM{%d%d}" ; const char *format
│  │╎││╎│   0x000012c4      b800000000     mov eax, 0
│  │╎││╎│   0x000012c9      e882fdffff     call sym.imp.printf         ; int printf(const char *format)
│  │╎││╎│   0x000012ce      bf00000000     mov edi, 0
│  │╎││╎│   0x000012d3      e8a8fdffff     call sym.imp.exit
│  │╎││╎│   ; CODE XREF from sym.gfl @ 0x12b3
│  └──────> 0x000012d8      8345f801       add dword [var_8h], 1
│   ╎││╎│   ; CODE XREF from sym.gfl @ 0x12aa
│   ╎└────> 0x000012dc      817df80e2700.  cmp dword [var_8h], 0x270e
│   └─────< 0x000012e3      7ec7           jle 0x12ac
│     │╎│   ; CODE XREF from sym.gfl @ 0x12a1
│     └───> 0x000012e5      8345fc01       add dword [var_4h], 1
│      ╎│   ; CODE XREF from sym.gfl @ 0x1298
│      ╎└─> 0x000012e9      817dfc88d077.  cmp dword [var_4h], 0x77d088
│      └──< 0x000012f0      7ea8           jle 0x129a
│           0x000012f2      90             nop
│           0x000012f3      90             nop
│           0x000012f4      c9             leave
└           0x000012f5      c3             ret
```

```
│  │╎││╎│   0x000012bb      89c6           mov esi, eax
│  │╎││╎│   0x000012bd      488d3d790d00.  lea rdi, str.THM_d_d        ; 0x203d ; "THM{%d%d}" ; const char *format
│  │╎││╎│   0x000012c4      b800000000     mov eax, 0
│  │╎││╎│   0x000012c9      e882fdffff     call sym.imp.printf         ; int printf(const char *format)
```

The another `sym.vuln()` carries on the authentication process that compare the **input value** with the **authenticated string**:

```
[0x00001289]> s sym.vuln
[0x00001185]> pdf
            ; CALL XREF from main @ 0x12ff
┌ 260: sym.vuln ();
│           ; var char *dest @ rbp-0x2c0
│           ; var char *s2 @ rbp-0x23e
│           ; var int64_t var_236h @ rbp-0x236
│           ; var int64_t var_232h @ rbp-0x232
│           ; var char *src @ rbp-0x230
│           ; var int64_t var_30h @ rbp-0x30
│           ; var int64_t var_28h @ rbp-0x28
│           ; var int64_t var_20h @ rbp-0x20
│           ; var int64_t var_18h @ rbp-0x18
│           ; var int64_t var_16h @ rbp-0x16
│           ; var int64_t var_dh @ rbp-0xd
│           ; var int64_t var_5h @ rbp-0x5
│           ; var int64_t var_1h @ rbp-0x1
│           0x00001185      55             push rbp
│           0x00001186      4889e5         mov rbp, rsp
│           0x00001189      4881ecc00200.  sub rsp, 0x2c0
│           0x00001190      48b84d616465.  movabs rax, 0x207962206564614d ; 'Made by '
│           0x0000119a      488945f3       mov qword [var_dh], rax
│           0x0000119e      c745fb346e6f.  mov dword [var_5h], 0x6e6f6e34 ; '4non'
│           0x000011a5      c645ff00       mov byte [var_1h], 0
│           0x000011a9      48b868747470.  movabs rax, 0x2f2f3a7370747468 ; 'https://'
│           0x000011b3      48ba67697468.  movabs rdx, 0x632e627568746967 ; 'github.c'
│           0x000011bd      488945d0       mov qword [var_30h], rax
│           0x000011c1      488955d8       mov qword [var_28h], rdx
│           0x000011c5      48b86f6d2f6e.  movabs rax, 0x69626f306e2f6d6f ; 'om/n0obi'
│           0x000011cf      488945e0       mov qword [var_20h], rax
│           0x000011d3      66c745e87434   mov word [var_18h], 0x3474  ; 't4'
│           0x000011d9      c645ea00       mov byte [var_16h], 0
│           0x000011dd      48b841474236.  movabs rax, 0x6435736a36424741 ; 'AGB6js5d'
│           0x000011e7      488985c2fdff.  mov qword [s2], rax
│           0x000011ee      c785cafdffff.  mov dword [var_236h], 0x476b6439 ; '9dkG'
│           0x000011f8      66c785cefdff.  mov word [var_232h], 0x37   ; '7'
│           0x00001201      488d3dfc0d00.  lea rdi, str.Insert_your_username:_ ; 0x2004 ; "Insert your username: " ; const char *format
│           0x00001208      b800000000     mov eax, 0
│           0x0000120d      e83efeffff     call sym.imp.printf         ; int printf(const char *format)
│           0x00001212      488d85d0fdff.  lea rax, [src]
│           0x00001219      4889c6         mov rsi, rax
│           0x0000121c      488d3df80d00.  lea rdi, [0x0000201b]       ; "%s" ; const char *format
│           0x00001223      b800000000     mov eax, 0
│           0x00001228      e843feffff     call sym.imp.__isoc99_scanf ; int scanf(const char *format)
│           0x0000122d      488d95d0fdff.  lea rdx, [src]
│           0x00001234      488d8540fdff.  lea rax, [dest]
│           0x0000123b      4889d6         mov rsi, rdx                ; const char *src
│           0x0000123e      4889c7         mov rdi, rax                ; char *dest
│           0x00001241      e8eafdffff     call sym.imp.strcpy         ; char *strcpy(char *dest, const char *src)
│           0x00001246      488d95c2fdff.  lea rdx, [s2]
│           0x0000124d      488d8540fdff.  lea rax, [dest]
│           0x00001254      4889d6         mov rsi, rdx                ; const char *s2
│           0x00001257      4889c7         mov rdi, rax                ; const char *s1
│           0x0000125a      e801feffff     call sym.imp.strcmp         ; int strcmp(const char *s1, const char *s2)
│           0x0000125f      85c0           test eax, eax
│       ┌─< 0x00001261      750e           jne 0x1271
│       │   0x00001263      488d3db40d00.  lea rdi, str._nWelcome      ; 0x201e ; "\nWelcome" ; const char *s
│       │   0x0000126a      e8d1fdffff     call sym.imp.puts           ; int puts(const char *s)
│      ┌──< 0x0000126f      eb16           jmp 0x1287
│      ││   ; CODE XREF from sym.vuln @ 0x1261
│      │└─> 0x00001271      488d3daf0d00.  lea rdi, str._nAuthentication_Error ; 0x2027 ; "\nAuthentication Error" ; const char *s
│      │    0x00001278      e8c3fdffff     call sym.imp.puts           ; int puts(const char *s)
│      │    0x0000127d      bf00000000     mov edi, 0
│      │    0x00001282      e8f9fdffff     call sym.imp.exit
│      │    ; CODE XREF from sym.vuln @ 0x126f
│      └──> 0x00001287      c9             leave
└           0x00001288      c3             ret
```

Let’s break down the process a little bit to understand what is going on!

These line is the **input** process:

```
|						 0x00001201      488d3dfc0d00.  lea rdi, str.Insert_your_username:_ ; 0x2004 ; "Insert your username: " ; const char *format
│           0x00001208      b800000000     mov eax, 0
│           0x0000120d      e83efeffff     call sym.imp.printf         ; int printf(const char *format)
│           0x00001212      488d85d0fdff.  lea rax, [src]
│           0x00001219      4889c6         mov rsi, rax
│           0x0000121c      488d3df80d00.  lea rdi, [0x0000201b]       ; "%s" ; const char *format
│           0x00001223      b800000000     mov eax, 0
│           0x00001228      e843feffff     call sym.imp.__isoc99_scanf ; int scanf(const char *format)
```

From here, it compares the **input value** as the `*s1` variable with the **expected value** as the `*2` variable:

```
|						0x00001254      4889d6         mov rsi, rdx                ; const char *s2
│           0x00001257      4889c7         mov rdi, rax                ; const char *s1
│           0x0000125a      e801feffff     call sym.imp.strcmp         ; int strcmp(const char *s1, const char *s2)
│           0x0000125f      85c0           test eax, eax
```

To discover the real value of the `*s2` value which must be input to bypass the authentication process, set the breakpoint before the `strcmp()` function is called which is at the location **0x00001257**:

```
[0x00001185]> db 0x00001257
```

Then type `ood ''` to open the **read-write mode** that allow us to input the value and follow the process:

```
[0x00001185]> ood ''
File dbg:///home/kali/TryHackMe/classicpasswd/Challenge.Challenge '' reopened in read-write mode
```

Type `dc` (_Continue execution_) to execute the process and input any value:

```
[0x7f852a363140]> dc
Insert your username: idontknow
hit breakpoint at: 0x5618ba20f257
```

Now the execution hits the breakpoint that set earlier. Choose of the registers `rdi` or `rax` from the **assembly script** `mov rdi,rax` to see what are they containning:

```
[0x556f18049257]> px @rdi
- offset -       0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x7fff41e9dee0  6964 6f6e 746b 6e6f 7700 0418 6f55 0000  idontknow...oU..
0x7fff41e9def0  f0da 4302 507f 0000 e0ca 4402 507f 0000  ..C.P.....D.P...
0x7fff41e9df00  78da 53a4 5c42 0100 4164 4502 507f 0000  x.S.\B..AdE.P...
0x7fff41e9df10  0000 0000 0000 0000 801f 0000 ffff 0000  ................
0x7fff41e9df20  0000 0000 0000 0000 0002 ea41 ff7f 0000  ...........A....
0x7fff41e9df30  01c1 4602 507f 0000 40b7 2302 507f 0000  ..F.P...@.#.P...
0x7fff41e9df40  a8c8 4602 507f 0000 b0ba 4602 507f 0000  ..F.P.....F.P...
0x7fff41e9df50  6385 4ba4 5c42 0100 c0c2 4602 507f 0000  c.K.\B....F.P...
0x7fff41e9df60  bbf2 4147 4236 6a73 3564 3964 6b47 3700  ..AGB6js5d9dkG7.
0x7fff41e9df70  6964 6f6e 746b 6e6f 7700 0000 0000 0000  idontknow.......
0x7fff41e9df80  0000 0000 0000 0000 0000 0000 0000 0000  ................
0x7fff41e9df90  2802 fb41 ff7f 0000 0000 0000 0101 0000  (..A............
0x7fff41e9dfa0  2f2f 2f2f 2f2f 2f2f 2f2f 2f2f 2f2f 2f2f  ////////////////
0x7fff41e9dfb0  2f68 6f6d 652f 6b61 6c69 2f54 7279 4861  /home/kali/TryHa
0x7fff41e9dfc0  0000 ff00 0000 0000 0000 0000 0000 0000  ................
0x7fff41e9dfd0  ff00 0000 0000 0000 0000 0000 0000 0000  ................
```

```
[0x556f18049257]> px @rax
- offset -       0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x7fff41e9dee0  6964 6f6e 746b 6e6f 7700 0418 6f55 0000  idontknow...oU..
0x7fff41e9def0  f0da 4302 507f 0000 e0ca 4402 507f 0000  ..C.P.....D.P...
0x7fff41e9df00  78da 53a4 5c42 0100 4164 4502 507f 0000  x.S.\B..AdE.P...
0x7fff41e9df10  0000 0000 0000 0000 801f 0000 ffff 0000  ................
0x7fff41e9df20  0000 0000 0000 0000 0002 ea41 ff7f 0000  ...........A....
0x7fff41e9df30  01c1 4602 507f 0000 40b7 2302 507f 0000  ..F.P...@.#.P...
0x7fff41e9df40  a8c8 4602 507f 0000 b0ba 4602 507f 0000  ..F.P.....F.P...
0x7fff41e9df50  6385 4ba4 5c42 0100 c0c2 4602 507f 0000  c.K.\B....F.P...
0x7fff41e9df60  bbf2 4147 4236 6a73 3564 3964 6b47 3700  ..AGB6js5d9dkG7.
0x7fff41e9df70  6964 6f6e 746b 6e6f 7700 0000 0000 0000  idontknow.......
0x7fff41e9df80  0000 0000 0000 0000 0000 0000 0000 0000  ................
0x7fff41e9df90  2802 fb41 ff7f 0000 0000 0000 0101 0000  (..A............
0x7fff41e9dfa0  2f2f 2f2f 2f2f 2f2f 2f2f 2f2f 2f2f 2f2f  ////////////////
0x7fff41e9dfb0  2f68 6f6d 652f 6b61 6c69 2f54 7279 4861  /home/kali/TryHa
0x7fff41e9dfc0  0000 ff00 0000 0000 0000 0000 0000 0000  ................
0x7fff41e9dfd0  ff00 0000 0000 0000 0000 0000 0000 0000  ................
```

Detect your input value and the **authenticated value** is standing before. Use that value as input value and re-execute the file then get the flag:

```
┌──(kali㉿kali)-[~/TryHackMe/classicpasswd]
└─$ ./Challenge.Challenge
Insert your username: AGB6js5d9dkG7

Welcome
THM{65235128496}
```

### Method 2: ltrace

This is method is much more easier to know the exactly expected input value. Using `ltrace` and the filename as the argument to start the debug process:

```
┌──(kali㉿kali)-[~/TryHackMe/classicpasswd]
└─$ ltrace ./Challenge.Challenge
printf("Insert your username: ")                                       = 22
__isoc99_scanf(0x55d1e275301b, 0x7ffea1f749c0, 0, 0Insert your username:
```

Input any value you want and it will display exactly the **authentication process** with the function `strcmp()` and the **authenticated value** inside as plaintext:

```
printf("Insert your username: ")                                       = 22
__isoc99_scanf(0x55d1e275301b, 0x7ffea1f749c0, 0, 0Insert your username: idontknow
)                   = 1
strcpy(0x7ffea1f74930, "idontknow")                                    = 0x7ffea1f74930
strcmp("idontknow", "AGB6js5d9dkG7")                                   = 40
puts("\nAuthentication Error"
Authentication Error
)                                         = 22
exit(0 <no return ...>
+++ exited (status 0) +++
```

Simply use that value and re-execute the file to get the flag:

```
┌──(kali㉿kali)-[~/TryHackMe/classicpasswd]
└─$ ./Challenge.Challenge
Insert your username: AGB6js5d9dkG7

Welcome
THM{65235128496}
```
