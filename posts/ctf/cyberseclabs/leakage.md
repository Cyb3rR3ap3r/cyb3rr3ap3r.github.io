---
layout: page
title: Leakage - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/leakage.html
---

# Leakage - CyberSecLabs
----



First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.6`

![image](https://user-images.githubusercontent.com/50459517/109024901-d61dc280-7683-11eb-805a-b6a5a1c7c299.png)

We see that we have a few interesting ports open.  First lets check out port 80 and see what is running on the HTTP server.

![image](https://user-images.githubusercontent.com/50459517/109024946-e0d85780-7683-11eb-8bda-2c957f25d81a.png)

We get redirected to **/users/sign_in** page that looks like a GitLab instance.  I saw in the nmap scan that **robots.txt** might have some good information.  While I check out that I also decided to run a directory buster to see if it can catch anything as well.

![image](https://user-images.githubusercontent.com/50459517/109024983-eb92ec80-7683-11eb-8606-7a98866d1089.png)

**robots.txt** has a bunch of information, after checking out these I found we can access some public repositories as the user **jonathan**

![image](https://user-images.githubusercontent.com/50459517/109025025-f51c5480-7683-11eb-8366-9b2f26907b6d.png)

After some enumeration of these respostories, I found some activity where jonathan changed some information in the **config.php** file under the **CMS** repository.

![image](https://user-images.githubusercontent.com/50459517/109025065-006f8000-7684-11eb-873e-9f0c7c549ffe.png)

Looks like jonathan hard coded his password into the **config.php** file then decided to remove.  Lucky for us he didn't remove any of the activity history.  Lets try to login as jonathan and see what we can find.

![image](https://user-images.githubusercontent.com/50459517/109025116-0f563280-7684-11eb-8e48-648405012a3c.png)

Looks like we now have access to a new project called **security**.  Sounds interesting, lets check it out.

![image](https://user-images.githubusercontent.com/50459517/109025145-167d4080-7684-11eb-9cd4-c04dc96f63fe.png)

Looks like we have an id_rsa private key.  Lets see if we can use this to login as jonathan via SSH.

![image](https://user-images.githubusercontent.com/50459517/109025201-2137d580-7684-11eb-8b42-179c0225094c.png)

Looks like we need a passphase.. Lets load this key into **ssh2john** to get a hash we can crack with John the Ripper.

![image](https://user-images.githubusercontent.com/50459517/109025239-29901080-7684-11eb-84af-d52748364d5a.png)

Now we can take the contents and load into john to crack.

![image](https://user-images.githubusercontent.com/50459517/109025278-33b20f00-7684-11eb-94f8-13446e2fb240.png)

Looks like the passphase is **scooby**.  Lets try this and see.

![image](https://user-images.githubusercontent.com/50459517/109025319-3dd40d80-7684-11eb-897a-89e06184ccae.png)

We are in!  Now lets copy over **LinPEAS** and see if we can escalate our privileges.

![image](https://user-images.githubusercontent.com/50459517/109025377-4af0fc80-7684-11eb-8d19-8d91fdf2ce4e.png)

Looks like nano has the SUID bit permission enabled.  Lets see if we can view the contents of /etc/shadow

`/bin/nano /etc/shadow`

![image](https://user-images.githubusercontent.com/50459517/109025418-55ab9180-7684-11eb-87e0-d7ff583bef6c.png)

We can!  Lets take the has of root and load it into **John the Ripper**

![image](https://user-images.githubusercontent.com/50459517/109025466-5e9c6300-7684-11eb-89a9-2345360c54f0.png)

Looks like the root password is chocolate.  Lets try to log in.

![image](https://user-images.githubusercontent.com/50459517/109025505-678d3480-7684-11eb-8732-820f87df70a9.png)

Now we are root!
