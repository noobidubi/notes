#IT #🖥 #🏫 
# What is a Network?
A **Network** is where all of your devices communicate with each other, but also how all of **your** devices can communicate with the whole world (_world wide web_)

Here is an Image from [HTB academy](https://academy.hackthebox.com/module/34/section/297 )that might provide some clarity:
![[Pasted image 20241001155504.png]]
# Network Types
there are a ton network types but it would be useless to talk about all. Here are a few basic ones:
## Wide Area Network(WAN)
usually when dealing with a network you get a **WAN IP**(public IP) and a **LAN IP**(private IP)

the **WAN** is public meaning that a lot of networks are joined together making world wide communication possible(*that means that when your visiting a website the connection generally goes over the WAN*)

Also important to mention would be that some companies and government agencies have an **Internal WAN** as a security measure that way they are not easily reachable through the **Public WAN**

## Local Area Network(LAN)
A **LAN** is what's you local network, your LAN IP is **not public** and somewhere in this range(RFC 1918, 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16). Each Network has its own list of **LAN IP's** that are note accessible for another network.

A basic example of the difference between a **LAN** and a **WAN** would be:

If you start an HTTP server like this:  
`sudo python3 -m http.server 80`,  
it will only be accessible within your **LAN** (*with the LAN IP e.g. 192.168.178.20/24*). This means that if you try to access it from another network, you won’t be able to connect as you will have the wrong IP because your trying to access a **LAN IP** and not a **WAN IP**.

To make the HTTP server accessible publicly (over the **WAN** or Wide Area Network), you would need to enable **port forwarding** on your router. Once port forwarding is set up, you can use your router's **WAN IP** to connect to the server from outside your local network.

#### Wireless Local Area Network (WLAN)
a **LAN** without cables

