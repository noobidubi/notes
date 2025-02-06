#IT #🖥 #🏫 
# What is a Network?
A **Network** is where all of your devices communicate with each other, but also how all of **your** devices can communicate with the whole world (_world wide web_)

Here is an Image from [HTB academy](https://academy.hackthebox.com/module/34/section/297 )that might provide some clarity:
![[Pasted image 20241001155504.png]]
# Network Types
there are more network types but I wont list all as the rest isn't as important
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


# Networking Topologies
A network Topologie is split into the two network types (**LAN** / **WAN**)

the **LAN** Topologie could be one or more of the #LAN-Topologie when there is more than one in place we call it a **Hybrid** but more on that later

the **WAN** Topologie is managed by your ISP and your **LAN** accesses it through a device like your router
	_in the images provided by HTB there are just Host A, Host B, ... you can imagine that one of the hosts is a router then the network can also access the **WAN**_
## Bus
#LAN-Topologie
![[Pasted image 20241011155009.png]]
a **BUS** is an older type of network and luckily isn't being used today.
That's because it is based on _coaxial cable_ which has the limitation of only one host being able to send at a time.

when a **BUS** tries to connect to a **WAN** one of the hosts needs to be a router.
### Example
**A:** _is a router_ 
**B:** _a computer that tries to go on a website_ 
	**B** will send a request to the router(**A**) that sends a request to the **WAN** to www.example.com then the router(**A**) will get the response and send it over the **LAN** back to **B**
the downside is that when **C**(_a telephone_) tries to make a call while someone is trying to access the web, then it won't work for anybody. _because only one host can send at a time._
