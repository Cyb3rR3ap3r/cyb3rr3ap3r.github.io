---
layout: page
title: Secret - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/secret.html
---

# Secret - CyberSecLabs
----


First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.4`

![image](https://user-images.githubusercontent.com/50459517/103782775-81ec4f80-4ffd-11eb-95c4-423357fbe607.png)

We can quickly see we have a lot of ports open.  Looking at ports and services running on this machine like 53 (dns), 88 (kerberos), and 3268 (ldap), we can pretty much guess this is a Windows Domain Controller.

First thing I want to do here is see if any Null Sessions are available on the SMB service.  I like to use **smbclient** mostly for this.

`smbclient -L //172.31.1.4`

![image](https://user-images.githubusercontent.com/50459517/103782824-92042f00-4ffd-11eb-8df1-69208aa00fe2.png)

We have some default shares we see often in Windows machines such as C$, ADMIN$, and IPC$.  However these most of the time require authentication to access..

![image](https://user-images.githubusercontent.com/50459517/103782867-a0eae180-4ffd-11eb-8b17-6c8f3c0c6c31.png)

However the **Office_Share** looks interesting.. Lets check this out and see what we find.

![image](https://user-images.githubusercontent.com/50459517/103782905-ac3e0d00-4ffd-11eb-9afb-202bf350264d.png)

Looks like **Office_Share** is a null session, allowing us unauthenticated access to the share.  It looks like we have a few user folders in here.  Lets dig around in these and see what we can find.

Looking into each user, I did not find any files, however I did find a file under the **Template** folder called **Default_Password.txt** with its content being "**SecretOrg!"** 

So we can take a the users we see in this share, create a wordlist and try to log in using this password.  

![image](https://user-images.githubusercontent.com/50459517/103783081-e14a5f80-4ffd-11eb-9201-ee87fd635232.png)

We could try these one by one with the password we found, but that is not practical.  Before we try to authenticate anywhere though I would like to enumerate ldap some first to see if we can confirm any of the users we found.

To do this, we can use a tool called **ldapsearch** which is great for querying data from an LDAP server.  To do this, we need to get the Base DN value for the server.  The Base DN is the starting point an LDAP server uses when searching for users authentication within your Directory.

To get this value, we can use **ldapsearch** to enumerate the base namingcontexts which will show us the Base DN value we need to start querying information from the LDAP server.

`ldapsearch -x -h 172.31.1.4 -s base namingcontexts`

![image](https://user-images.githubusercontent.com/50459517/103783113-ec04f480-4ffd-11eb-9e39-400135cd5324.png)

Now that we have the Base DN value, we can set it with the **-b** flag and see if we can dump information off the LDAP server anonymously.

![image](https://user-images.githubusercontent.com/50459517/103783148-f6bf8980-4ffd-11eb-9af9-5f97b3c99e4c.png)

Unfortunately we cannot, I messed around with trying to authenticate with the password provided in the file but decided to move on for the time being.  I next tried to brute force SMB with the user list we created along with the default password using **hydra**

`hydra -L user.txt -P pass.txt 172.31.1.4 smb -V -f`

![image](https://user-images.githubusercontent.com/50459517/103783190-02ab4b80-4ffe-11eb-974f-6cd72a6c138f.png)

**Hydra** failed for some reason but **ncrack** shows all accounts using the "FirstnameLastname" naming scheme have the default password.

`ncrack -U user.txt -P pass.txt 172.31.1.4 -p 445`

![image](https://user-images.githubusercontent.com/50459517/103783214-0c34b380-4ffe-11eb-8e1b-736a436c2e15.png)

I tried to access the Office_Share again with login using the SECRET domain but I didn't find anything different than the anonymous login provided.  So I decided to try to get a shell using **psexec**

![image](https://user-images.githubusercontent.com/50459517/103783259-1787df00-4ffe-11eb-99ae-fe831875da5a.png)

Seems like the server may be blocking this connection.  Maybe due to permissions of that user.  Lets try the other users.

![image](https://user-images.githubusercontent.com/50459517/103783291-253d6480-4ffe-11eb-8966-297a867022f0.png)

I completely missed something in the **ncrack** results, both "joecakes" and "jcakes" username returned as positive usernames, and it looks like **jcakes** successfully authenticates with the AD server, meaning the other usernames are most likely false positives.  However it looks like none of the shares are writeable. 

Since we know these seem to be valid credentials, lets try to use **evil-winrm** to get a shell.

`evil-winrm -i 172.31.1.4 -u jcakes -p SecretOrg!`

![image](https://user-images.githubusercontent.com/50459517/103783323-2f5f6300-4ffe-11eb-950f-8dc1210e9150.png)

Now that we have a shell, we can begin our post-enumeration to try and escalate our privilege's further on the machine.  Lets copy **SharpHound.exe** onto the machine and then download the .zip file it creates to upload into **BloodHound.**

![image](https://user-images.githubusercontent.com/50459517/103783356-3a19f800-4ffe-11eb-8ff0-3c1d564f30f4.png)

![image](https://user-images.githubusercontent.com/50459517/103783394-4736e700-4ffe-11eb-9d94-2ef03376d776.png)

After looking through the data found, there unfortunately isn't much to go on at the moment.  While looking into this, I also decided to upload **winpeas** to the machine and let it enumeration for possible privilege escalation vectors.  I was able to find some AutoLogin credentials stored on the machine.

![image](https://user-images.githubusercontent.com/50459517/103783416-50c04f00-4ffe-11eb-8a30-079ed86b87ba.png)

Now lets use ncrack again to see if we can brute force any logins with this new password.

`ncrack -U user.txt -P pass.txt 172.31.1.4 -p 445`

![image](https://user-images.githubusercontent.com/50459517/103783450-5c137a80-4ffe-11eb-957b-d55d7d0304a8.png)

Looks like the user **bdover** has AutoLogin enabled with these credentials, lets use **evil-winrm** again to login to this new user.

![image](https://user-images.githubusercontent.com/50459517/103783484-6766a600-4ffe-11eb-80a0-585629131cb3.png)

Now lets go back to our BloodHound data and see if we can find a path from Ben Dover to Domain Admin.

![image](https://user-images.githubusercontent.com/50459517/103783516-7188a480-4ffe-11eb-8ba9-1a75e24feed4.png)

Looks like we have a couple of different options.  It looks like **bdover** is a member of the **Service Accounts** group.  The **Service Accounts** group is part of the **Administrators** group.  Then from the **Administrators** group, we have a couple ways to **Domain Admin**.

We can get more information by right clicking on the line and clicking on Help.  This will give us some information on this line including possible ways to abuse it.

![image](https://user-images.githubusercontent.com/50459517/103783554-7d746680-4ffe-11eb-82ce-f5646ef6668a.png)

After reading, we can see all we have to do is run **net group "Domain Admins" bdover /add /domain** to add **bdover** to the **Domain Admins** group

![image](https://user-images.githubusercontent.com/50459517/103783588-882efb80-4ffe-11eb-9c77-fd4a74a56162.png)

Now that our user is part of the Domain Admins group, we should be able to access the system.txt flag from the Administrators Desktop, unfortunately we get an error when trying to read it..

![image](https://user-images.githubusercontent.com/50459517/103783612-9250fa00-4ffe-11eb-8802-26b02fa57bae.png)

So looks like we need to log in as the local Administrator to access this file.  Luckily we should be able to dump the hashes with **mimikatz** since we are in the Domain Admins group.

Lets copy over mimikatz and run on the machine and run it.

![image](https://user-images.githubusercontent.com/50459517/103783653-9e3cbc00-4ffe-11eb-9983-755ebf587ec3.png)

Unfortuneatly just running it in interactive mode throws it in a loop for some reason..  To elimnate this, we can put the commands we want to run in quotes then finish with "exit" to exit mimikatz so it doesn't loop.  The below command will ensure mimikatz is running as admin (privilege::debug) and then try to dump all of the NTLM hashes on the machine.

`mimikatz64.exe "privilege::debug" "lsadump::lsa /patch" "exit"`

![image](https://user-images.githubusercontent.com/50459517/103783697-aac11480-4ffe-11eb-8e70-34338a478d29.png)

Now that we have the hash for **Administrator,** we can use this hash to authenticate with **evil-winrm**

`evil-winrm -i 172.31.1.4 -u Administrator -H '4d801e8c043133366056b5cd6fdcc2c7'`

![image](https://user-images.githubusercontent.com/50459517/103783744-b8769a00-4ffe-11eb-94da-8c7423f0c605.png)

Now we can get the system.txt hash to complete the box.

![image](https://user-images.githubusercontent.com/50459517/103783783-c3c9c580-4ffe-11eb-9a49-e96f5524bfec.png)
