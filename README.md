# Write up
## Introduction
Before starting to present the way it has been taken to solve Keeper machine, I will present an outline with the concepts that will be used to solve challenge:
- Use of default credentials in the Request Tracker software.
- Connection to Keeper via SSH, due to a information leakage in Request Tracker.
- Keepass 2.x vulnerability exploit (CVE-2023-32784).
- Obtaining the root's Openssh private key from the PuTTy key stored in Keepass.
## Reconnaissance phase
As usual, reconnaissance phase was the first phase executed due to the fact that a hacker needs to know as much information about his target as possible before starting to exploit posible vulnerabilities.

To do this, you need to add a new line in the ''/etc/hosts'' file, where you define which IP is linked to which domain name. As shown in the following image, if you want to access keeper.htb, you are going to send a request to 10.10.11.227 really:

[![hosts.png](https://i.postimg.cc/8zBhdMYt/hosts.png)](https://postimg.cc/SnR2k2kM)

As you can see, there is one more subdomain linked to the same IP. Why this is the case is explained below.

Next, Nmap is used to scan all ports on the server. The tool output is as follows:

[![nmap.png](https://i.postimg.cc/MGWqQhc3/nmap.png)](https://postimg.cc/nXwy8SGq)

As shown in the last image, two open ports were discovered exposing two different services:
- Port 22: SSH service.
- Port 80: HTTP service.

Due to the fairly updated version of SSH, a technological scan was performed on port 443.

[![whatweb.png](https://i.postimg.cc/T3yRxZyZ/whatweb.png)](https://postimg.cc/gxW9qtHD)

The tools do not yield much information, so the website was accessed to show what it contains:

[![Web.png](https://i.postimg.cc/N0mD3jQL/Web.png)](https://postimg.cc/Lq4jfRYM)

As you can see in the image, the subdomain to audit is tickets.keeper.htb, and so in /etc/hosts there are two domains linked to one IP.
