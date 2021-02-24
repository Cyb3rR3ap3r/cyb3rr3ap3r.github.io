---
layout: page
title: Shares - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/shares.html
---

# Shares - CyberSecLabs
----


First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.7`

![image](https://user-images.githubusercontent.com/50459517/109027534-842a6c00-7686-11eb-9051-4268f8dcc2da.png)

Looks like we have a few interesting things open.  After looking, I believe we will need to check out NFS and see what shares we have access.  However I first started with FTP and HTTP just to see what was there.

FTP didn't allow for anonymous login so nothing there.  HTTP was just a static site for a Pet Shop.  I enumerated for directories while checking out NFS and that returned nothing as well.  So lets focus on NFS for now.

First lets run **showmount** to see if we can enumerate what shares we have access too.

![image](https://user-images.githubusercontent.com/50459517/109027577-90162e00-7686-11eb-82f1-aa8a0c1327d3.png)

Looks like we can access **/home/amir** so lets try to mount this directory.

`mount -t nfs 172.31.1.7:/home/amir /root/misc/mount`

The above command will mount the NFS share to the directory **/root/misc/mount** on my local machine.  Once it is mounted, lets check out its contents.

![image](https://user-images.githubusercontent.com/50459517/109027617-9a382c80-7686-11eb-99b3-8d3a3ab58ff8.png)

As expected we see the home directory for the user **amir.**  we also see in this directory a **.ssh** folder that contains the users **id_rsa** private key.

Lets use this key to login as **amir** via SSH.

![image](https://user-images.githubusercontent.com/50459517/109027668-a623ee80-7686-11eb-8aad-5a4d87b3980d.png)

Unfortunately that didn't work due to us not having permissions to this file.  Lucky for us it looks like the user has a backup of the file that we do have access too.  So lets copy the contents of that file to a new file and try to login again.

![image](https://user-images.githubusercontent.com/50459517/109027708-b1771a00-7686-11eb-8b1d-463546d34570.png)

![image](https://user-images.githubusercontent.com/50459517/109027748-bc31af00-7686-11eb-9258-7bbe55f867cf.png)

Looks like the private key is protected with a password.  Lets create a hash with **ssh2john** and then try to try to check with John The Ripper.

![image](https://user-images.githubusercontent.com/50459517/109027791-c653ad80-7686-11eb-8bd2-5040c71296dd.png)

Looks like the password is **hello6**.  Lets try it.

![image](https://user-images.githubusercontent.com/50459517/109027842-d3709c80-7686-11eb-8e21-28f89fc75214.png)

We are in.  Now lets copy over **LinPEAS** and see if we can escalate out privileges.

![image](https://user-images.githubusercontent.com/50459517/109027874-dcfa0480-7686-11eb-8963-932ed7557359.png)

Looks like we have a couple different binaries we can run as the user **amy**

I spent some time with the binary **pkexec** first but didn't get anywhere.. So I moved on to **python3** and got right in!

![image](https://user-images.githubusercontent.com/50459517/109027920-e6836c80-7686-11eb-818e-1ca65e9bdb4f.png)

Now that we are **amy** we can run **LinPEAS** again and see if we can find a path to root

![image](https://user-images.githubusercontent.com/50459517/109027957-ef743e00-7686-11eb-8ec1-a0461b18a30c.png)

Looks like we can run ssh with sudo and no password.  Looking at GTFOBins, we can abuse this with the following command that will give us an elevated shell as root

![image](https://user-images.githubusercontent.com/50459517/109027991-f8fda600-7686-11eb-8425-0db1d3219388.png)
