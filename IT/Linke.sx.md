# Enumeration
so im not sure what im allowed to do or not so im just gonna start with some **Passive Enumeration**...
```sh
╭─janosch@archlinux /home/janosch  ‹system›
╰─$ dnsenum --dnsserver 192.174.68.104 --enum -p 0 -s 0 -o subdomains.txt -f ./git/SecLists/Discovery/DNS/subdomains-top1million-110000.txt linke.sx
..Snip..
Brute forcing with ./git/SecLists/Discovery/DNS/subdomains-top1million-110000.txt:
_____________________________________________________________________________

owncloud.linke.sx.                       3600     IN    A        152.53.46.97
dali.linke.sx.                           3600     IN    A        152.53.46.97
synapse.linke.sx.                        3600     IN    A        152.53.46.97
element.linke.sx.                        3600     IN    A        152.53.46.97
postgresql.linke.sx.                     3600     IN    A        152.53.46.97
dokuwiki.linke.sx.                       3600     IN    A        152.53.46.97
```

#### Manual subdomain Enumeration _(Burp passive scan)_
```
linke.sx
authelia.linke.sx
dali.linke.sx
dokuwiki.linke.sx
element.linke.sx
espe.linke.sx
forgejo.linke.sx
hedgedoc.linke.sx
owncloud.linke.sx
postgresql.linke.sx
static.requarks.io
synapse.linke.sx
vikunja.linke.sx
wiki-js.linke.sx
zackeneule.linke.sx
zeitbild.linke.sx
```

## Zeitbild
### Web-Frontend [git](https://forgejo.linke.sx/zeitbild/frontend-dali) [link](dali.linke.sx)
