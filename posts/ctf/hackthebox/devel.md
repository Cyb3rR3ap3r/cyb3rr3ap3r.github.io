---
layout: page
title: Devel - HackTheBox
permalink: /posts/ctf/hackthebox/devel.html
---

# Devel - HackTheBox
----

First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 10.10.10.5`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dd57331d-0e6f-4660-b560-945125880ce5/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dd57331d-0e6f-4660-b560-945125880ce5/Untitled.png)

Looks like we have 2 ports open.  First thing that I notice is the FTP contents from the nmap results.  It appears to contain a starter page for an IIS server.  Since we have port 80 running IIS, maybe the FTP server is hosting the root directory of the HTTP server.  Lets see if port 80 is hosting a default IIS page.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/65a47617-05ba-4f95-8563-38afc4b41e86/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/65a47617-05ba-4f95-8563-38afc4b41e86/Untitled.png)

Great!  Now for a PoC, lets see if we can copy a file over via FTP and execute it via HTTP.  This will help prove if we are dealing with the actually root directory or just a backup.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/164e6247-c125-4391-9628-9a80b16c290a/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/164e6247-c125-4391-9628-9a80b16c290a/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b450d3dc-88ee-47e1-81f7-375083aea54d/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b450d3dc-88ee-47e1-81f7-375083aea54d/Untitled.png)

Awesome.  We should be able to upload a malicious .aspx file and execute it to get a shell back.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/266cde63-a1a9-4299-803b-49dd22fcc735/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/266cde63-a1a9-4299-803b-49dd22fcc735/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bd6568e7-96ca-49b4-b6f9-52cbc7ff3e24/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bd6568e7-96ca-49b4-b6f9-52cbc7ff3e24/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e5535e0f-3739-447d-94f9-ea99dff0be0f/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e5535e0f-3739-447d-94f9-ea99dff0be0f/Untitled.png)

I copied over WinPEAS.exe but was unable to get the 32bit version to run correctly..  I decided to run the .bat version but didn't find much so decided to run WinPrivCheck.bat as I like the way it checks for uninstalled patches on the system that may have kernel exploits we could use.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a67069d3-d4b0-4538-90d7-9e2c0c7eda32/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a67069d3-d4b0-4538-90d7-9e2c0c7eda32/Untitled.png)

I looked through a few of these and found that MS11-046 worked and already has a compiled executable here.

[abatchy17/WindowsExploits](https://github.com/abatchy17/WindowsExploits)

Just simply copy the executable over and run.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dd66c4e0-c2b8-45d9-9089-012613041b66/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dd66c4e0-c2b8-45d9-9089-012613041b66/Untitled.png)
