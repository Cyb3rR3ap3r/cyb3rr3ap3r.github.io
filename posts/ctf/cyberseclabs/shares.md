---
layout: page
title: Shares - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/shares.html
---

# Shares - CyberSecLabs
----


First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.7`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b17dbf8d-b79c-4a89-88ca-f8fcbb3b552a/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b17dbf8d-b79c-4a89-88ca-f8fcbb3b552a/Untitled.png)

Looks like we have a few interesting things open.  After looking, I believe we will need to check out nfs and see what shares we have access.  However I first started with FTP and HTTP just to see what was there.

FTP didn't allow for anonymous login so nothing there.  HTTP was just a static site for a Pet Shop.  I enumerated for directories while checking out NFS and that returned nothing as well.  So lets focus on NFS for now.

First lets run **showmount** to see if we can enumerate what shares we have access too.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3bcdfa5a-6578-45ef-a7a2-5e1c34356fee/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3bcdfa5a-6578-45ef-a7a2-5e1c34356fee/Untitled.png)

Looks like we can access **/home/amir** so lets try to mount this directory.

`mount -t nfs 172.31.1.7:/home/amir /root/misc/mount`

The above command will mount the NFS share to the directory **/root/misc/mount** on my local machine.  Once it is mounted, lets check out its contents.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/aee57838-61d3-4ac2-9aaf-0ad3f5128925/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/aee57838-61d3-4ac2-9aaf-0ad3f5128925/Untitled.png)

As expected we see the home directory for the user **amir.**  we also see in this directory a **.ssh** folder that contains the users **id_rsa** private key.

Lets use this key to login as **amir** via SSH.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/38f9fba0-bbd6-4731-ac42-23ecce61a92a/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/38f9fba0-bbd6-4731-ac42-23ecce61a92a/Untitled.png)

Unfortunately that didn't work due to us not having permissions to this file.  Lucky for us it looks like the user has a backup of the file that we do have access too.  So lets copy the contents of that file to a new file and try to login again.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/21576452-826e-4a85-a278-a18b2b6e5407/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/21576452-826e-4a85-a278-a18b2b6e5407/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5a823414-2b42-4316-bd2b-e13529bf7e18/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5a823414-2b42-4316-bd2b-e13529bf7e18/Untitled.png)

Looks like the private key is protected with a password.  Lets create a hash with **ssh2john** and then try to try to check with John The Ripper.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/67bc947e-1eb0-4d3e-ade0-f29f24e6460e/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/67bc947e-1eb0-4d3e-ade0-f29f24e6460e/Untitled.png)

Looks like the password is **hello6**.  Lets try it.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/63d3f447-b669-437f-a435-c8c92e8b0c8c/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/63d3f447-b669-437f-a435-c8c92e8b0c8c/Untitled.png)

We are in.  Now lets copy over **LinPEAS** and see if we can escalate out privileges.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0790b2a6-96ea-4fcf-898e-e9d0b0f5f443/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0790b2a6-96ea-4fcf-898e-e9d0b0f5f443/Untitled.png)

Looks like we have a couple different binaries we can run as the user **amy**

I spent some time with the binary **pkexec** first but didn't get anywhere.. So I moved on to **python3** and got right in!

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/85dd7337-a823-4aab-a0f4-d1f692c114ed/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/85dd7337-a823-4aab-a0f4-d1f692c114ed/Untitled.png)

Now that we are **amy** we can run LinPEAS again and see if we can find a path to root

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6fcab457-cc98-4540-a281-355d61283bca/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/6fcab457-cc98-4540-a281-355d61283bca/Untitled.png)

Looks like we can run ssh with sudo and no password.  Looking at GTFOBins, we can abuse this with the following command that will give us an elevated shell as root

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f0e89e77-e66f-47a2-a3ea-cd64b89b1e48/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f0e89e77-e66f-47a2-a3ea-cd64b89b1e48/Untitled.png)
