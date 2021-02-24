---
layout: page
title: TCP Handshake - Wireshark
permalink: /posts/other/how-wireshark/tcp-3.html
---

# What is a TCP Handshake

The TCP Handshake is the process which is used in a TCP/IP network to make a connection between a client and a server.  It is a three-step process that requires both the client and server to exchange synchronization and acknowledgment packets before the real data communication process starts.

TCP traffic always begins with this handshake to initialize the communication.  

First the client sends a synchronization packet (SYN) to the server, asking to start a connection.

Second the server sends an acknowledgment and synchronization packet (SYN ACK) to the clients, which signifies a response to the SYN for the client as well as sending its own SYN to the client.

Third the client responses with an acknowledgment packet (ACK) which signifies a response to the servers SYN packet.  The connection is now established and the data transfer can begin.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/598031fd-07d9-4539-a6e3-e7735b23eb70/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/598031fd-07d9-4539-a6e3-e7735b23eb70/Untitled.png)

# Wireshark Breakdown

First, lets breakdown what we have here.

We have the client (10.10.14.31) connecting to an HTTP web server (10.10.10.5).  Lets take a look at what happens when the client navigates to the website hosted on the server.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3e3e4b83-101e-4feb-b4b2-d7beb22371ea/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3e3e4b83-101e-4feb-b4b2-d7beb22371ea/Untitled.png)

As you can see, we have 3 packets with the TCP protocol before we see the HTTP traffic come through.  These 3 packets are the 3-way handshake.

Lets take a deeper look into these packets.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/574f0517-42ce-4a90-a25b-d41651c8e040/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/574f0517-42ce-4a90-a25b-d41651c8e040/Untitled.png)

Here is some of the details in packet 1.  As we can see, the Source IP is 10.10.14.31 and the Destination IP is 10.10.10.5.  We can also see the Source port that gets opened up for this connection on the client is 35316 and the Destination port is of course 80.  We also see that the SYN flag is set.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4850d27c-5cb7-436d-896c-ce5fa974de34/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/4850d27c-5cb7-436d-896c-ce5fa974de34/Untitled.png)

Here is some of the details in packet 2.  As we can see, the Source IP is now 10.10.10.5 and the Destination IP is 10.10.14.31.  We can also see the Source port is now 80 and the Destination port is 35316.  Finally we see that both the SYN and ACK flags are set.

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/206d751f-4825-47b7-b784-d0d7cb3fd630/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/206d751f-4825-47b7-b784-d0d7cb3fd630/Untitled.png)

Here we see some of the details in packet 3.  As we can see, the Source IP is again 0.10.14.31 and the Destination IP is 10.10.10.5.  We can also see the Source port is again 35316 and the Destination port is 80.  Finally we see that ACK flag is set.

# Conclusion

That's it.  An overview of the TCP 3 way handshake.  As we continue to understand other network activities, you will see this handshake quite often as all TCP communications start with it.  Next we will take a look at a popular tool called Nmap and how some of its various features work.
