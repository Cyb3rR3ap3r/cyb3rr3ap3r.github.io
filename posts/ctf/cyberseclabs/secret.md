---
layout: page
title: Secret - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/secret.html
---

# Secret - CyberSecLabs
----


First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.4`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a20e524f-93ba-499b-9cd4-385154e82a04/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a20e524f-93ba-499b-9cd4-385154e82a04/Untitled.png)

We can quickly see we have a lot of ports open.  Looking at ports and services running on this machine like 53 (dns), 88 (kerberos), and 3268 (ldap), we can pretty much guess this is a Windows Domain Controller.

First thing I want to do here is see if any Null Sessions are available on the SMB service.  I like to use **smbclient** mostly for this.

`smbclient -L //172.31.1.4`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/02bc97d1-942c-4784-978e-9e50f1283496/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/02bc97d1-942c-4784-978e-9e50f1283496/Untitled.png)

We have some default shares we see often in Windows machines such as C$, ADMIN$, and IPC$.  However these most of the time require authentication to access..

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cd5c0eed-cdb6-4875-835c-3d7b5307c7df/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/cd5c0eed-cdb6-4875-835c-3d7b5307c7df/Untitled.png)

However the **Office_Share** looks interesting.. Lets check this out and see what we find.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f103a6fb-6b8e-4646-ad5c-f3f5dfc68976/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f103a6fb-6b8e-4646-ad5c-f3f5dfc68976/Untitled.png)

Looks like **Office_Share** is a null session, allowing us unauthenticated access to the share.  It looks like we have a few user folders in here.  Lets dig around in these and see what we can find.

Looking into each user, I did not find any files, however I did find a file under the **Template** folder called **Default_Password.txt** with its content being "**SecretOrg!"** 

So we can take a the users we see in this share, create a wordlist and try to log in using this password.  

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ef5c1ce4-e106-4a7d-a40d-7ca49ff83421/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ef5c1ce4-e106-4a7d-a40d-7ca49ff83421/Untitled.png)

We could try these one by one with the password we found, but that is not practical.  Before we try to authenticate anywhere though I would like to enumerate ldap some first to see if we can confirm any of the users we found.

To do this, we can use a tool called **ldapsearch** which is great for querying data from an LDAP server.  To do this, we need to get the Base DN value for the server.  The Base DN is the starting point an LDAP server uses when searching for users authentication within your Directory.

To get this value, we can use **ldapsearch** to enumerate the base namingcontexts which will show us the Base DN value we need to start querying information from the LDAP server.

`ldapsearch -x -h 172.31.1.4 -s base namingcontexts`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7e4d59fe-3a9c-437a-a2fe-826a9e86d1e8/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7e4d59fe-3a9c-437a-a2fe-826a9e86d1e8/Untitled.png)

Now that we have the Base DN value, we can set it with the **-b** flag and see if we can dump information off the LDAP server anonymously.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/807d1a48-fbf3-4722-a1aa-a0989b09cccf/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/807d1a48-fbf3-4722-a1aa-a0989b09cccf/Untitled.png)

Unfortunately we cannot, I messed around with trying to authenticate with the password provided in the file but decided to move on for the time being.  I next tried to brute force SMB with the user list we created along with the default password using **hydra**

`hydra -L user.txt -P pass.txt 172.31.1.4 smb -V -f`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b00f2124-142a-4df9-ac65-236dc7e2f88c/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b00f2124-142a-4df9-ac65-236dc7e2f88c/Untitled.png)

**Hydra** failed for some reason but **ncrack** shows all accounts using the "FirstnameLastname" naming scheme have the default password.

`ncrack -U user.txt -P pass.txt 172.31.1.4 -p 445`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/db79a6b5-2ffa-417f-ba19-29be93df9a17/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/db79a6b5-2ffa-417f-ba19-29be93df9a17/Untitled.png)

I tried to access the Office_Share again with login using the SECRET domain but I didn't find anything different than the anonymous login provided.  So I decided to try to get a shell using **psexec**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e658c8a2-4d1a-4008-a9a2-eb999736715c/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e658c8a2-4d1a-4008-a9a2-eb999736715c/Untitled.png)

Seems like the server may be blocking this connection.  Maybe due to permissions of that user.  Lets try the other users.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0d7ce24d-6752-4a53-ba13-8f584b48688f/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0d7ce24d-6752-4a53-ba13-8f584b48688f/Untitled.png)

I completed missed something in the **ncrack** results, both "joecakes" and "jcakes" username returned as positive usernames, and it looks like **jcakes** successfully authenticates with the AD server, meaning the other usernames are most likely false positives.  However it looks like none of the shares are writeable. 

Since we know these seem to be valid credentials, lets try to use **evil-winrm** to get a shell.

`evil-winrm -i 172.31.1.4 -u jcakes -p SecretOrg!`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/27ad90bb-4e66-4172-9c87-ebb79b3babf7/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/27ad90bb-4e66-4172-9c87-ebb79b3babf7/Untitled.png)

Now that we have a shell, we can begin our post-enumeration to try and escalate our privilege's further on the machine.  Lets copy **SharpHound.exe** onto the machine and then download the .zip file it creates to upload into **BloodHound.**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7f2c2f85-8724-4787-bc55-465d45b93761/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7f2c2f85-8724-4787-bc55-465d45b93761/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8b0ed593-622f-46f5-a833-83d40b17ae41/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/8b0ed593-622f-46f5-a833-83d40b17ae41/Untitled.png)

After looking through the data found, there unfortunately isn't much to go on at the moment.  While looking into this, I also decided to upload **winpeas** to the machine and let it enumeration for possible privilege escalation vectors.  I was able to find some AutoLogin credentials stored on the machine.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a7968866-ca4e-4d17-a08c-c3e310ec893a/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a7968866-ca4e-4d17-a08c-c3e310ec893a/Untitled.png)

Now lets use ncrack again to see if we can brute force any logins with this new password.

`ncrack -U user.txt -P pass.txt 172.31.1.4 -p 445`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5930ae6c-22e7-4d8a-a1e1-4d3875226225/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5930ae6c-22e7-4d8a-a1e1-4d3875226225/Untitled.png)

Looks like the user **bdover** has AutoLogin enabled with these credentials, lets use **evil-winrm** again to login to this new user.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/199d706e-54d3-47d7-984b-564e0280b15d/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/199d706e-54d3-47d7-984b-564e0280b15d/Untitled.png)

Now lets go back to our BloodHound data and see if we can find a path from Ben Dover to Domain Admin.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/12d62d63-74b5-4464-b1d3-8487eeb55891/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/12d62d63-74b5-4464-b1d3-8487eeb55891/Untitled.png)

Looks like we have a couple of different options.  It looks like **bdover** is a member of the **Service Accounts** group.  The **Service Accounts** group is part of the **Administrators** group.  Then from the **Administrators** group, we have a couple ways to **Domain Admin**.

We can get more information by right clicking on the line and clicking on Help.  This will give us some information on this line including possible ways to abuse it.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/58f0ed6d-0de7-437b-9dfd-3f1500316bd7/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/58f0ed6d-0de7-437b-9dfd-3f1500316bd7/Untitled.png)

After reading, we can see all we have to do is run **net group "Domain Admins" bdover /add /domain** to **add bdover** to the **Domain Admins** group

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/edc8306d-df8b-49fc-9dcb-269b86f53285/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/edc8306d-df8b-49fc-9dcb-269b86f53285/Untitled.png)

Now that our user is part of the Domain Admins group, we should be able to access the system.txt flag from the Administrators Desktop, unfortunately we get an error when trying to read it..

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fefdfe40-fd95-4b47-adf7-698bf98dc000/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/fefdfe40-fd95-4b47-adf7-698bf98dc000/Untitled.png)

So looks like we need to log in as the local Administrator to access this file.  Luckily we should be able to dump the hashes with **mimikatz** since we are in the Domain Admins group.

Lets copy over mimikatz and run on the machine and run it.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e15cde64-3384-4f7e-b0b7-760db62f9697/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e15cde64-3384-4f7e-b0b7-760db62f9697/Untitled.png)

Unfortuneatly just running it in interactive mode throws it in a loop for some reason..  To elimnate this, we can put the commands we want to run in quotes then finish with "exit" to exit mimikatz so it doesn't loop.  The below command will ensure mimikatz is running as admin (privilege::debug) and then try to dump all of the NTLM hashes on the machine.

`mimikatz64.exe "privilege::debug" "lsadump::lsa /patch" "exit"`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1781f096-975c-4f2f-ba2e-f10d6d3c7589/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1781f096-975c-4f2f-ba2e-f10d6d3c7589/Untitled.png)

Now that we have the hash for **Administrator,** we can use this hash to authenticate with **evil-winrm**

`evil-winrm -i 172.31.1.4 -u Administrator -H '4d801e8c043133366056b5cd6fdcc2c7'`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3de84630-6c28-45bc-9f8a-7e0edf000412/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3de84630-6c28-45bc-9f8a-7e0edf000412/Untitled.png)

Now we can get the system.txt hash to complete the box.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/84ea01e3-830c-4850-965c-c8edacc9ee10/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/84ea01e3-830c-4850-965c-c8edacc9ee10/Untitled.png)
