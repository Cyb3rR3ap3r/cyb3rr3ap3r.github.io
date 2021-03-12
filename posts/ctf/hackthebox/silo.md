---
layout: page
title: Silo - HackTheBox
permalink: /posts/ctf/hackthebox/silo.html
---

# Silo - HackTheBox
----

First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 10.10.10.82`

![image](https://user-images.githubusercontent.com/50459517/110979859-c7ddd080-832a-11eb-9bc3-49978ab1e23d.png)

Looks like we have quite a few ports open, lets check out the web server on port 80.

![image](https://user-images.githubusercontent.com/50459517/110979889-d3c99280-832a-11eb-9ca5-94304c490de0.png)

Looks like we have a default IIS webpage.  I fuzzed for directories and see what we can find but came up short.  I enumerated a few other services and came up short as well.  I figure we are going to need to exploit the Oracle Database which I have limited experience on so this should be a fun learning experience.

After some research, it seems that the open source tool ODAT is gonna be the best tool for enumerating the Oracle database.

[https://github.com/quentinhardy/odat](https://github.com/quentinhardy/odat)

This tool can be tricky to get working but for the most part the README installation will get you going.  I did have trouble installing the Oracle Instant Client via the RPM files but following this guide helped.

[https://github.com/rapid7/metasploit-framework/wiki/How-to-get-Oracle-Support-working-with-Kali-Linux](https://github.com/rapid7/metasploit-framework/wiki/How-to-get-Oracle-Support-working-with-Kali-Linux)

Once it is all installed, we can start enumerating for SIDs

`odat.py sidguesser -s 10.10.10.82`

![image](https://user-images.githubusercontent.com/50459517/110979966-f0fe6100-832a-11eb-9b4c-8b7d4981def9.png)

Looks like an SID of XE was found.  Now we need some credentials, lets use ODAT with a wordlist of various Oracle accounts

[`odat.py](http://odat.py/) passwordguesser -s 10.10.10.82 -d XE --accounts-file accounts/accounts_multiple.txt`

![image](https://user-images.githubusercontent.com/50459517/110980014-fd82b980-832a-11eb-8ea0-604d3d6c613d.png)

Awesome!  Now that we have a valid SID and credentials, we can use ODAT to upload a file to the server.  Lets try to upload an ASPX reverse shell so we can execute on the webserver.

![image](https://user-images.githubusercontent.com/50459517/110980121-2014d280-832b-11eb-9f63-bb44eb2cbc02.png)


Now just go to the webpage to execute the payload

[http://10.10.10.82/shell.aspx](http://10.10.10.82/shell.aspx)

![image](https://user-images.githubusercontent.com/50459517/110980165-2c992b00-832b-11eb-8236-e46a5fc3a579.png)


Now that we have a shell, we can begin our enumeration.  Looking at out privileges, we can see that **SeImpersonatePrivilege** is enabled, meaning that we may be able to Impersonate tokens with a potato attack.

![image](https://user-images.githubusercontent.com/50459517/110980185-31f67580-832b-11eb-9ff1-c0ed341d2374.png)


We can copy over juicypotato.exe along with an msfvenom reverse shell and run to impersonate the system user and gain a privileged shell.

![image](https://user-images.githubusercontent.com/50459517/110980203-3884ed00-832b-11eb-8ed9-6491247bdf9f.png)

![image](https://user-images.githubusercontent.com/50459517/110980234-40dd2800-832b-11eb-96be-fbaa55aaadb3.png)

