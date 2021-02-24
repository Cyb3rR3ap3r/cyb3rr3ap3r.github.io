---
layout: page
title: Outdated - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/outdated.html
---

# Outdated - CyberSecLabs
----


First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.22`

![image](https://user-images.githubusercontent.com/50459517/109031232-25ff8800-768a-11eb-9f6d-ef6d8fc231ac.png)

First thing I want to do is check for nfs shares on this machine with **showmount**

![image](https://user-images.githubusercontent.com/50459517/109031272-2f88f000-768a-11eb-9281-2ea9613a7ac1.png)

I looked through these directories and didn't find anything, but we can keep these usernames in mind incase we need them later.  Now lets check FTP.

![image](https://user-images.githubusercontent.com/50459517/109031306-3879c180-768a-11eb-8b23-4d280714c8fc.png)

Anonymous login didn't work, lets check the version and see if we can find an exploit for this.

[Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/36742)

Found this exploit for a remote file copy via this version of FTP that is unauthenticated.  Lets try this out.

![image](https://user-images.githubusercontent.com/50459517/109031397-4e878200-768a-11eb-9f68-d6bf333321d1.png)

Looks like we can copy the files with **CPFR** and **CPTO** so lets see if this works.

![image](https://user-images.githubusercontent.com/50459517/109031431-57785380-768a-11eb-9740-e8b31e4ed67b.png)

Looks like it recognizes this is a file.  Lets try to copy the file to **/var/nfsbackups** and access via the NFS share we mounted earlier

![image](https://user-images.githubusercontent.com/50459517/109031465-62cb7f00-768a-11eb-904a-7783e8fbe3d2.png)

![image](https://user-images.githubusercontent.com/50459517/109031524-6ced7d80-768a-11eb-876b-d71390049bcb.png)

Looks like it copied successfully.  I also tried to copy the **/etc/shadow** file but it failed with permission denied.  After researching a little I found a RCE PoC that uses this same vulnerability to inject malicious php code that we may be able to use to execute commands on the machine. 

[Offensive Security's Exploit Database Archive](https://www.exploit-db.com/exploits/36803)

Unfortunately I'm not sure if these is a way to interoperate the php code without an open HTTP port we can access..  So my next thought was to look at the **/etc/passwd** file we copied over and see who is a valid user and see if we can get access to the private ssh key.

![image](https://user-images.githubusercontent.com/50459517/109031575-78d93f80-768a-11eb-8192-f32acbf7e1b5.png)

Looks like **daniel** is a valid user, lets see if we can copy the private key over.

![image](https://user-images.githubusercontent.com/50459517/109031629-82fb3e00-768a-11eb-9845-fbc2fba714d8.png)

Awesome!  Now lets see if we can authenticate using this certificate with SSH

![image](https://user-images.githubusercontent.com/50459517/109031667-8bec0f80-768a-11eb-9ab9-d88356c84385.png)

It worked.  Now lets see if we can escalate our privileges.  We will copy over LinPEAS and see what it finds.

![image](https://user-images.githubusercontent.com/50459517/109031717-99a19500-768a-11eb-9db0-1ba7efbcfece.png)

Looks like the **/var/nfsbackups** NFS share has no_root_squash enabled meaning if we copy a file over from a mounted point that file will be owned by root, therefore we should be able to create a shell as root with this.

I will copy over a one-liner that creates a C file that will execute /bin/bash that I got from TCM Priv-Esc course.

`echo 'int main() { setgid(0); setuid(0); system("/bin/bash"); return 0; }' > /tmp/1/x.c`

![image](https://user-images.githubusercontent.com/50459517/109031751-a0c8a300-768a-11eb-94aa-ba7c4b2d50cf.png)

Now all we need to do is execute via the ssh session we have and it should spawn a shell as root.

NOTE:  I forgot, you need to set the SUID bit in order to execute as root

![image](https://user-images.githubusercontent.com/50459517/109031797-aaeaa180-768a-11eb-84c0-2ef38a723062.png)

Now you can execute and get shell as root.

![image](https://user-images.githubusercontent.com/50459517/109031830-b3db7300-768a-11eb-9271-d5b7971326ea.png)
