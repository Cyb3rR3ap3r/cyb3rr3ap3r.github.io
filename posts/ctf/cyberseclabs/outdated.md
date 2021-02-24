---
layout: page
title: Outdated - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/outdated.html
---

# Outdated - CyberSecLabs
----


First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.22`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b1f8753d-cbd0-4027-bbfb-af19a4f094bd/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b1f8753d-cbd0-4027-bbfb-af19a4f094bd/Untitled.png)

First thing I want to do is check for nfs shares on this machine with **showmount**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c52c9737-4346-477d-9a7d-00ee61231f54/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c52c9737-4346-477d-9a7d-00ee61231f54/Untitled.png)

I looked through these directories and didn't find anything, but we can keep these usernames in mind incase we need them later.  Now lets check FTP.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2b0db3b9-825b-401a-8005-d55145d00f12/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2b0db3b9-825b-401a-8005-d55145d00f12/Untitled.png)

Anonymous login didn't work, lets check the version and see if we can find an exploit for this.

[Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/36742)

Found this exploit for a remote file copy via this version of FTP that is unauthenticated.  Lets try this out.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3e65b2b7-73d8-4059-8fa0-5488f56da94a/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3e65b2b7-73d8-4059-8fa0-5488f56da94a/Untitled.png)

Looks like we can copy the files with **CPFR** and **CPTO** so lets see if this works.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eda4b37d-8937-4994-a3b1-d13f7b7263e8/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eda4b37d-8937-4994-a3b1-d13f7b7263e8/Untitled.png)

Looks like it recognizes this is a file.  Lets try to copy the file to **/var/nfsbackups** and access via the NFS share we mounted earlier

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a75edd1b-f91e-4ac5-9947-f440c4e5bfb1/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a75edd1b-f91e-4ac5-9947-f440c4e5bfb1/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d8b4e749-4078-4ab4-a36f-aeb13de1998c/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d8b4e749-4078-4ab4-a36f-aeb13de1998c/Untitled.png)

Looks like it copied successfully.  I also tried to copy the **/etc/shadow** file but it failed with permission denied.  After researching a little I found a RCE PoC that uses this same vulnerability to inject malicious php code that we may be able to use to execute commands on the machine. 

[Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/36803)

Unfortunately I'm not sure if these is a way to interoperate the php code without an open HTTP port we can access..  So my next thought was to look at the **/etc/passwd** file we copied over and see who is a valid user and see if we can get access to the private ssh key.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4d29ce63-f340-4130-966a-e7bdf44dae5b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4d29ce63-f340-4130-966a-e7bdf44dae5b/Untitled.png)

Looks like **daniel** is a valid user, lets see if we can copy the private key over.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8aea19a2-4942-4b55-b55d-9ea103f720ee/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8aea19a2-4942-4b55-b55d-9ea103f720ee/Untitled.png)

Awesome!  Now lets see if we can authenticate using this certificate with SSH

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bfc86f70-9105-40b4-a5cf-f3769b00e924/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/bfc86f70-9105-40b4-a5cf-f3769b00e924/Untitled.png)

It worked.  Now lets see if we can escalate our privileges.  We will copy over LinPEAS and see what it finds.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b2a2d2bf-c559-4b36-a45a-4207ee1109a8/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b2a2d2bf-c559-4b36-a45a-4207ee1109a8/Untitled.png)

Looks like the **/var/nfsbackups** NFS share has no_root_squash enabled meaning if we copy a file over from a mounted point that file will be owned by root, therefore we should be able to create a shell as root with this.

I will copy over a one-liner that creates a C file that will execute /bin/bash that I got from TCM Priv-Esc course.

`echo 'int main() { setgid(0); setuid(0); system("/bin/bash"); return 0; }' > /tmp/1/x.c`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/41d96508-ae77-41e3-a52d-20bfc41c1704/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/41d96508-ae77-41e3-a52d-20bfc41c1704/Untitled.png)

Now all we need to do is execute via the ssh session we have and it should spawn a shell as root.

NOTE:  I forgot, you need to set the SUID bit in order to execute as root

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b8255a23-989a-41f7-95ae-e3662cf6d868/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b8255a23-989a-41f7-95ae-e3662cf6d868/Untitled.png)

Now you can execute and get shell as root.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fc17b0de-8254-49b7-bf1b-21a0c9801143/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fc17b0de-8254-49b7-bf1b-21a0c9801143/Untitled.png)
