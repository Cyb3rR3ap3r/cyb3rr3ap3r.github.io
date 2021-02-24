---
layout: page
title: Leakage - CyberSecLabs
permalink: /posts/ctf/cyberseclabs/leakage.html
---

# Leakage - CyberSecLabs
----



First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.6`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/edb424ae-b1b7-4343-acf7-81625c21a51b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/edb424ae-b1b7-4343-acf7-81625c21a51b/Untitled.png)

We see that we have a few interesting ports open.  First lets check out port 80 and see what is running on the HTTP server.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e47723f2-2e2a-41e2-a6a9-7442ebef73fe/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e47723f2-2e2a-41e2-a6a9-7442ebef73fe/Untitled.png)

We get redirected to **/users/sign_in** page that looks like a GitLab instance.  I saw in the nmap scan that **robots.txt** might have some good information.  While I check out that I also decided to run a directory buster to see if it can catch anything as well.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/df7593f9-f5f2-415a-bb44-6ccf06bc4118/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/df7593f9-f5f2-415a-bb44-6ccf06bc4118/Untitled.png)

**robots.txt** has a bunch of information, after checking out these I found we can access some public repositories as the user **jonathan**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d4c883c9-3a01-454e-b00c-7cb11693bc46/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d4c883c9-3a01-454e-b00c-7cb11693bc46/Untitled.png)

After some enumeration of these respostories, I found some activity where jonathan changed some information in the **config.php** file under the **CMS** repository.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eb3ee448-8b39-4648-b25a-e60884b9748c/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eb3ee448-8b39-4648-b25a-e60884b9748c/Untitled.png)

Looked like jonathan hard coded his password into the **config.php** file then decided to remove.  Lucky for us he didn't remove any of the activity history.  Lets try to login as jonathan and see what we can find.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/be62b1c5-c1ce-4753-8b0d-4c2ea8b60015/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/be62b1c5-c1ce-4753-8b0d-4c2ea8b60015/Untitled.png)

Looks like we now have access to a new project called **security**.  Sounds interesting, lets check it out.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2c4cf1ff-d4cf-414c-9548-a5042794ef6b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/2c4cf1ff-d4cf-414c-9548-a5042794ef6b/Untitled.png)

Looks like we have an id_rsa private key.  Lets see if we can use this to login as jonathan via SSH.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d6c1eaf1-d3f6-4066-b909-d35311911dc1/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/d6c1eaf1-d3f6-4066-b909-d35311911dc1/Untitled.png)

Looks like we need a passphase.. Lets load this key into **ssh2john** to get a hash we can crack with John the Ripper.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/385374f1-7bf7-4996-be7a-8323aa327b63/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/385374f1-7bf7-4996-be7a-8323aa327b63/Untitled.png)

Now we can take the contents and load into john to crack.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0b396068-8608-4028-b850-05a8395c854c/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0b396068-8608-4028-b850-05a8395c854c/Untitled.png)

Looks like the passphase is **scooby**.  Lets try this and see.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c40ae409-320e-41e4-bfc4-81696cce4427/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/c40ae409-320e-41e4-bfc4-81696cce4427/Untitled.png)

We are in!  Now lets copy over **LinPEAS** and see if we can escalate our privileges.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a8f1e19d-cae4-4969-b55d-f912b151f755/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a8f1e19d-cae4-4969-b55d-f912b151f755/Untitled.png)

Looks like nano has the SUID bit permission enabled.  Lets see if we can view the contents of /etc/shadow

`/bin/nano /etc/shadow`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/957378ff-d0c8-471d-95ad-bc3d18a3774e/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/957378ff-d0c8-471d-95ad-bc3d18a3774e/Untitled.png)

We can!  Lets take the has of root and load it into **John the Ripper**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/821bfae2-e7cd-4315-8b34-5e9ffca9e219/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/821bfae2-e7cd-4315-8b34-5e9ffca9e219/Untitled.png)

Looks like the root password is chocolate.  Lets try to log in.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1f2713b9-2788-4909-bd5b-554802bb07ce/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1f2713b9-2788-4909-bd5b-554802bb07ce/Untitled.png)

Now we are root!
