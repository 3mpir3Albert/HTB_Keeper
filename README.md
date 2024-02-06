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

Due to the fairly up-to-date version of SSH, it seems that the way to go is to perform a technology scan on port 80.

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

There seems to be a problem with a memory dump of the keepass program in a file located in Lise's home directory, so probably the way to hack this machine is to access lnorgaard's home and try to get the master password to decrypt the file and see the content. But how to access Lise's account? After logging into the website it is possible to go to the top bar and in the middle of it hover the mouse over the administration dropdown and then a subsection called Users appears. By clicking on it a new page appears with all the users registered in "Request Tracker". As you can see, the user lnorgaard appears, so click on him and inspect the user information. In a text box below you will see some credentials:

[![Leak.png](https://i.postimg.cc/PJYV3812/Leak.png)](https://postimg.cc/XBNgXqHC)

If you try to see if the credentials are reused in other services such as SSH, you will be able to log in with the user lnorgaard.

[![Access.png](https://i.postimg.cc/9Ms2vTK9/Access.png)](https://postimg.cc/2bdgWqzj)
## Post-Exploitation Phase

After gaining access to the user lnorgaard, it would be time to do a system recognition, but remember that there is a file in the Lise home path that contains the Keepass memory dump. So it's time to introduce CVE-2023-32784.
### CVE-2023-32784

The vulnerability affects all Keepass software versions below 2.54. The only requirement is to have a memory dump. It does not matter where the dump comes from, whether it is a process dump, a swap file (a dedicated space on a computer's storage device that is used as virtual memory. Virtual memory is a memory management technique that allows the operating system to use part of the storage device as if it were additional RAM), a hibernation file (this is a system file used to support the hibernation function. Hibernation is a power-saving mode in which the contents of a computer's RAM are saved to the hard disk or other non-volatile storage device before the system is shut down) or a RAM dump.

Keepass works with custom text boxes for many purposes, for example to enter the master password or to edit passwords, among others.
When a person writes in these boxes, a leftover string is created in memory, which is practically impossible to delete. If you type "Password" seven leftover strings are created in memory:
- ·a
- ··s
- ···s
- ····w
- ·····o
- ······r
- ·······d

Guessing the first character will retrieve the master password in plain text.

To recover the leftover strings there is a script programmed in C# that will help you to automate this process (https://github.com/vdohney/keepass-password-dumper).
