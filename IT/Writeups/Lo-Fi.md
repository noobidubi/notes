a basic 5 min CTF featuring web exploitation.

# Enumeration
a basic nmap scan to see what were dealing with.
```bash
$ nmap 10.10.233.155
..Snip..
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http
```

## http server
Just a basic http server that has some embedded YouTube links to Lofi live streams

Found two parameters while scanning through the site
```http
GET /?search=chill HTTP/1.1
Host: 10.10.233.155
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.6723.70 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.10.233.155/
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

The other one seems a bit more interesting
```http
GET /?page=chill.php HTTP/1.1
Host: 10.10.233.155
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.6723.70 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://10.10.233.155/
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
```

# Exploitation
I realized that the **page** parameter isn't secure and allows for path traversal. So I can add something like `/?page=../../../etc/passwd` and get the contents of that file.

I got the hint that the flag is in the root directory so let's see if that's true...
![[Pasted image 20250520201122.png]]

# Conclusion
Just a fun little 5 min CTF didn't have any issues with this one. Maybe I should try some harder CTF's.

**What did I learn?**
- Do writeups after completing the CTF *not while solving it*