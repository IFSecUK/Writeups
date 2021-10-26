Hi, welcome back again. Today we will be doing shocker. Its a Linux box. Lets get going.

<H2>Enumeration</H2>
The name of the box suggests something related to shell shock. So lets see what surprises are ahead.
Nmap scan:

```Bash
PORT     STATE SERVICE REASON         VERSION
80/tcp   open  http    syn-ack ttl 63 Apache httpd 2.4.18 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     syn-ack ttl 63 OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
```

Connecting to ssh on port 2222 doesn't reveal any banner of any sorts. Also this version of ssh is vulnerable to username enumeration which might come handy later. 

```Bash
┌──(root💀kali)-[/home/rishabh/HTB/shocker]
└─# searchsploit openssh 7.2                                                                                   130 ⨯
----------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                     |  Path
----------------------------------------------------------------------------------- ---------------------------------
OpenSSH 2.3 < 7.7 - Username Enumeration                                           | linux/remote/45233.py
OpenSSH 2.3 < 7.7 - Username Enumeration (PoC)                                     | linux/remote/45210.py
OpenSSH 7.2 - Denial of Service                                                    | linux/dos/40888.py
OpenSSH 7.2p1 - (Authenticated) xauth Command Injection                            | multiple/remote/39569.py
OpenSSH 7.2p2 - Username Enumeration                                               | linux/remote/40136.py
OpenSSH < 7.4 - 'UsePrivilegeSeparation Disabled' Forwarded Unix Domain Sockets Pr | linux/local/40962.txt
OpenSSH < 7.4 - agent Protocol Arbitrary Library Loading                           | linux/remote/40963.txt
OpenSSH < 7.7 - User Enumeration (2)                                               | linux/remote/45939.py
OpenSSHd 7.2p2 - Username Enumeration                                              | linux/remote/40113.txt
----------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results

```

So now our only focus should be port 80. Similarly you can also check for version exploits for this apache but there aren't any except for DoS and Local Priv Esc which are of no use to us. You can start nikto scans in the background if it finds anything useful.
Nikto scans didn't find anything interesting. The homepage contains an image saying "Don't Bug me"

![](Pasted%20image%2020211025215937.png)

The source code doesn't contain any hints or comments which can aid us in our next step. Next step would be to run directory bruteforcing tool like dirb with common.txt wordlist. 

![](Pasted%20image%2020211025220113.png)

It did find one directory cgi-bin but its access code is 403 which is forbidden. Lets try with a bigger wordlist, if it finds any other directories. No luck!! I even included extensions like php,txt,cgi but no luck. I got stuck at this point banging my head against the wall. I have to admit, I just peaked at one of the walkthroughs just to see a hint and to my surprise it was one of the siliest things I missed. I never searched for sh extension files. I ran my gobuster and included sh as extensions and got user.sh as one of the files. 

![](Pasted%20image%2020211025224246.png)

I will give you guys a tip. These hacking platforms always hide a hint behind those machine names. If the machine's name is shocker, it must be related to shell shock vulnerability. This vulnerability lets us execute system commands from environment variables unintentionally. This is a great resource if you want to understand about this vulnerability and how to exploit it: https://www.breachlock.com/shellshock-bash-remote-code-execution-vulnerability-explained/

<H2>Initial Foothold</H2>

Here is how I managed to read /etc/passwd file: 

![](Pasted%20image%2020211025225210.png)

Getting shell from here was a quite a challenge for me. I tried using netcat but it was throwing code 500. I tried reading user's id_rsa file, but there also I failed. Using this article https://pentesterlab.com/exercises/cve-2014-6271/course and pentester monkey bash reverse shell, I got a shell back.

![](Pasted%20image%2020211025230721.png)

![](Pasted%20image%2020211025230809.png)

<H2>Privilege Escalation</H2>

User shelly has lxd privilieges which is a straight priv esc path. But there's no harm to enumerate more. Possibily we can find more vectors for escalation. Transferred my linpeas script and let it do the work. User shelly can run perl as root without password. 

![](Pasted%20image%2020211025231643.png)

Lets go to our favorite website gtfobins and look at sudo entry for perl. Here it is: 

![](Pasted%20image%2020211025231800.png)

Simple and neat. No hassle.

![](Pasted%20image%2020211025231949.png)

Voila!! We have pwned this machine. This was an easy rated box by HTB but nevertheless it was fun to exploit the shellshock vulnerability. Meet you tomorrow with another box.















