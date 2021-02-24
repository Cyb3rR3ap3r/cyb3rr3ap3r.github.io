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

![image](https://user-images.githubusercontent.com/50459517/109064875-8b666f80-76b0-11eb-94a8-827894fe89d8.png)

# Wireshark Breakdown

First, lets breakdown what we have here.

We have the client (10.10.14.31) connecting to an HTTP web server (10.10.10.5).  Lets take a look at what happens when the client navigates to the website hosted on the server.

![image](https://user-images.githubusercontent.com/50459517/109064909-94efd780-76b0-11eb-9398-29025183b401.png)

As you can see, we have 3 packets with the TCP protocol before we see the HTTP traffic come through.  These 3 packets are the 3-way handshake.

Lets take a deeper look into these packets.

![image](https://user-images.githubusercontent.com/50459517/109064933-9caf7c00-76b0-11eb-8129-ec0a49a55a6e.png)

Here is some of the details in packet 1.  As we can see, the Source IP is 10.10.14.31 and the Destination IP is 10.10.10.5.  We can also see the Source port that gets opened up for this connection on the client is 35316 and the Destination port is of course 80.  We also see that the SYN flag is set.

![image](https://user-images.githubusercontent.com/50459517/109064966-a6d17a80-76b0-11eb-8373-af66b9113ec6.png)

Here is some of the details in packet 2.  As we can see, the Source IP is now 10.10.10.5 and the Destination IP is 10.10.14.31.  We can also see the Source port is now 80 and the Destination port is 35316.  Finally we see that both the SYN and ACK flags are set.

![image](https://user-images.githubusercontent.com/50459517/109064986-b05ae280-76b0-11eb-91f3-298dbca23ac3.png)

Here we see some of the details in packet 3.  As we can see, the Source IP is again 0.10.14.31 and the Destination IP is 10.10.10.5.  We can also see the Source port is again 35316 and the Destination port is 80.  Finally we see that ACK flag is set.

# Conclusion

That's it.  An overview of the TCP 3 way handshake.  As we continue to understand other network activities, you will see this handshake quite often as all TCP communications start with it.  Next we will take a look at a popular tool called Nmap and how some of its various features work.
