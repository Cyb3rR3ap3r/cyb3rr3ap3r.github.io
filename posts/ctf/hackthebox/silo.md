---
layout: page
title: Silo - HackTheBox
permalink: /posts/ctf/hackthebox/silo.html
---

# Silo - HackTheBox
----

First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 10.10.10.82`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/92abbaa3-55fb-4de5-97d9-e8b01102b865/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/92abbaa3-55fb-4de5-97d9-e8b01102b865/Untitled.png)

Looks like we have quite a few ports open, lets check out the web server on port 80.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/61ea47e4-fc4b-488a-8bd0-3eb46ebe3da4/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/61ea47e4-fc4b-488a-8bd0-3eb46ebe3da4/Untitled.png)

Looks like we have a default IIS webpage.  I fuzzed for directories and see what we can find but came up short.  I enumerated a few other services and came up short as well.  I figure we are going to need to exploit the Oracle Database which I have limited experience on so this should be a fun learning experience.

After some research, it seems that the open source tool ODAT is gonna be the best tool for enumerating the Oracle database.

[quentinhardy/odat](https://github.com/quentinhardy/odat)

This tool can be tricky to get working but for the most part the README installation will get you going.  I did have trouble installing the Oracle Instant Client via the RPM files but following this guide helped.

[rapid7/metasploit-framework](https://github.com/rapid7/metasploit-framework/wiki/How-to-get-Oracle-Support-working-with-Kali-Linux)

Once it is all installed, we can start enumerating for SIDs

`odat.py sidguesser -s 10.10.10.82`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c24a1216-c0f5-4b7a-80a0-65e819762239/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c24a1216-c0f5-4b7a-80a0-65e819762239/Untitled.png)

Looks like an SID of XE was found.  Now we need some credentials, lets use ODAT with a wordlist of various Oracle accounts

[`odat.py](http://odat.py/) passwordguesser -s 10.10.10.82 -d XE --accounts-file accounts/accounts_multiple.txt`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/014da9aa-b928-4d18-8774-43bd4490af01/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/014da9aa-b928-4d18-8774-43bd4490af01/Untitled.png)

Awesome!  Now that we have a valid SID and credentials, we can use ODAT to upload a file to the server.  Lets try to upload an ASPX reverse shell so we can execute on the webserver.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/13fe8ed9-66a5-46b3-a2df-62cf35f58a3c/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/13fe8ed9-66a5-46b3-a2df-62cf35f58a3c/Untitled.png)

Now just go to the webpage to execute the payload

[http://10.10.10.82/shell.aspx](http://10.10.10.82/shell.aspx)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dbe5d17f-5617-4ada-ba6e-9cd8b93a7219/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dbe5d17f-5617-4ada-ba6e-9cd8b93a7219/Untitled.png)

Now that we have a shell, we can begin our enumeration.  Looking at out privileges, we can see that **SeImpersonatePrivilege** is enabled, meaning that we may be able to Impersonate tokens with a potato attack.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eefa089b-a8f7-4221-bdf1-d1bb3b293e81/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eefa089b-a8f7-4221-bdf1-d1bb3b293e81/Untitled.png)

We can copy over juicypotato.exe along with an msfvenom reverse shell and run to impersonate the system user and gain a privileged shell.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/60969613-9ab6-4c61-a59c-4ac4431d6e1a/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/60969613-9ab6-4c61-a59c-4ac4431d6e1a/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/66608e3c-bb04-448f-ba26-78e9bdf79ad1/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/66608e3c-bb04-448f-ba26-78e9bdf79ad1/Untitled.png)
