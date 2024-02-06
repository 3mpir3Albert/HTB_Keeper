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

Due to the fairly updated version of SSH, a technological scan was performed on port 80.

[![whatweb.png](https://i.postimg.cc/T3yRxZyZ/whatweb.png)](https://postimg.cc/gxW9qtHD)

The tools do not yield much information, so the website was accessed to show what it contains:

[![Web.png](https://i.postimg.cc/N0mD3jQL/Web.png)](https://postimg.cc/Lq4jfRYM)

As you can see in the image, the subdomain to audit is tickets.keeper.htb, and so in /etc/hosts there are two domains linked to one IP.

By repeating the technological scan on port 80, but changing the subdomain, the following information can be seen:

[![whatweb2.png](https://i.postimg.cc/CMHVkbnr/whatweb2.png)](https://postimg.cc/3WR6H4B2)

The presence of request tracker software is notable. Therefore, the Internet was searched for vulnerabilities, but nothing was found. Due to the lack of information on vulnerabilities, another path was taken. Default root credentials were found:
- USER: root
- PASSWORD: password

Entering the credentials in the login panel found at http://tickets.keeper.htb would not give root access to the request tracking software. But, if you do a resource scan you may see new paths where it is possible to find new login panels, as it is shown below:

```bash
wfuzz -c -t 200 -hc 302 -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt http://tickets.keeper.htb/FUZZ
```

[![wfuzz.png](https://i.postimg.cc/BnqTT4rD/wfuzz.png)](https://postimg.cc/crbtZy44)
## Exploitation Phase

At http://tickets.keeper.htb/rt there is another authentication panel that can be used to enter the root credentials and enter "request tracker" as this user. Once there, you can access as much information as possible, as the access was made with the user with maximum privileges, so all the ticket history was accessed and this information can be seen there:

[![Info.png](https://i.postimg.cc/XqQC45zN/Info.png)](https://postimg.cc/q66RGNrW)

There seems to be a problem with a memory dump of the keepass program in a file located in Lise's home directory, so probably the way to hack this machine is to get access to lnorgaard home and try to get the master password to decrypt the file and see the content. But how to access Lise's account?
