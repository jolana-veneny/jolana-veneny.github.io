---
layout: post
tag: WriteUps
author: JLN
---
## WriteUp: IP Man (NahamCon 22)
In this post I want to show you the IP Man networking challenge and how I and [Cameron Blankenbuehler](https://www.cblanken.dev/) (who deserves most of the credit for getting this challenge) solved it at the NahamCon Europe 22 CTF. This was hands down the best CTF challenge I have done so far and it's a shame that there are not more networking challenges in other CTFs!

The IP Man challenge was worded as follows: 
>Funky HTTP servers are in the house! We've leaked the source code for an unusual web server, and we're sure that if you can make it accept your request, you'll be golden!

You are provided with a python file, a server address link, and you are asked to launch the challenge in the browser.

First, let's think about what this challenge wants us to do. From the challenge description we get the sense that you need to connect to the server to get the flag. Specifically, the challenge says that you need to send a request to the server. How does one connect to an HTTP server and send a request?

The short answer is packets. First, you have to establish a TCP connection via a process called the three-way handshake. It is a specific exchange of packets between a client and a server. You can read about the details [here](https://coderscat.com/tcp-3-way-handshake-in-detail/). Once you establish the connection, you are ready to send the HTTP get request packet, like the challenge instructs you.

This should be the general idea for solving the challenge, but we were also told the server is "unusual." Let's take a look at the python code which appears to be a script that runs on the server.
 
```
"""
IP Man
"""

from http.server import SimpleHTTPRequestHandler
import socketserver
import os
import threading

from scapy.all import *
from netfilterqueue import NetfilterQueue

# Create queue
os.system("iptables -A INPUT -p tcp --dport 8080 -j NFQUEUE --queue-num 1")
# Eliminate outbound unreachable messages
os.system("iptables -I OUTPUT -p icmp -m icmp --icmp-type port-unreachable -j DROP")
print("Created queue")


def run_server():
    try:
        # Define the server's port number
        PORT = 8080
        SimpleHTTPRequestHandler.index_file = "/opt/index.html"
        httpd = socketserver.ThreadingTCPServer(("", PORT), SimpleHTTPRequestHandler)
        httpd.serve_forever()
    except Exception as exc:
        print(exc)


def callback(packet):
    """
    Handles incoming packets and does some ~magic~.
    """

    # Extract the payload of the packet from NFQueue
    data = packet.get_payload()

    # Parse it with scapy
    scapy_pkt = IP(data)

    print("Got packet from %s..." % scapy_pkt[IP].src)

    if scapy_pkt[IP].id != 0x1337:
        packet.drop()
        return

    # If we're good, accept the packet.
    packet.accept()


threading.Thread(target=run_server, daemon=True).start()
nfqueue = NetfilterQueue()  # create a new NFQueue
nfqueue.bind(1, callback)  # bind callback to queue-num 1
try:
    nfqueue.run()  # run nfqueue until user shutdown
except KeyboardInterrupt:
    os.system("iptables -D INPUT -p tcp --dport 8080 -j NFQUEUE --queue-num 1")
    os.system("iptables -D OUTPUT -p icmp -m icmp --icmp-type port-unreachable -j DROP")
```

The important part in this script is the `if` condition in the callback function. It instructs the server to drop any packet whose ID does not equal to 0x1337. This means that all the packets we craft for the challenge have to carry the ID value of 0X1337.

The script also reveals that the server uses a tool called [scapy](https://scapy.readthedocs.io/en/latest/) to inspect the packets. It is a packet manipulation program that can not only inspect packets but also create them. Interestingly, this tool is capable of launching a DoS attack, but we will use it to create our TCP handshake and the GET request.

You can either run scapy from the command line or as a Python script. We will do the latter as we have to create several specific packets and writing them out in the command line could get a bit clumsy. 

So let's put our python script together.
First, we import scapy:

```
from scapy.all import *
```

Now, we need to find out the server IP address and the port so that we know where to send the packets. Using the `nslookup` command we find that the challenge.nahamcon.com server is located at `35.188.181.240` and we use the port generated in the challenge, in this case `30157`.

<img src="https://user-images.githubusercontent.com/101567957/210428762-653718fb-513a-498c-98e1-541691c14998.png" width="700">

We also need to pick a random port on our side, say `7081`, and an ACK number of `1`. We initialize these as variables to get:

```
dst_ip = '35.188.181.240'
dst_port = 30157
source_port = 7081
ack_num = 1
```
With this out of the way, we can start designing the initial SYN packet: 

```
# SYN packet
syn = IP(dst=dst_ip, id=0x1337)/TCP(sport=source_port, dport=dst_port, flags='S', seq=ack_num)
Ether()/syn

syn_ack = sr1(syn)
print('Received SYN/ACK', syn_ack.seq)
```
The syn packet needs to have the S flags attribute to designate it as a SYN packet and let's not forget the id attribute of 0x1337, otherwise the server will drop it. The sr1 command sends the packet to the server one time and returns the response.

However, if you run the script now and check what is happening at Wireshark, you'll see something peculiar:

<img src="https://user-images.githubusercontent.com/101567957/210428867-c4064698-4d3e-416d-b27f-c2e7f10d6b94.png" width="max">

*(The port numbers in this screenshot are different from the ones I use in the script)*

The SYN packet was sent to the server and the server responded with a SYN, ACK packet, which is what we hoped would happen. But the client resets the connection with an RST packet. Peculiar indeed. [This blog post](http://blog.facilelogin.com/2010/12/hand-crafting-tcp-handshake-with-scapy.html) gives a good explanation on why Linux kernel automatically blocks the incoming SYN, ACK packet and how to fix this behavior with IP tables.

Now, we can proceed to the ACK packet. It follows the same pattern as the SYN packet, but we need to change the flags attribute to `A` and increase the ack_num by 1.

```
# ACK packet
ack = IP(dst=dst_ip, id=0x1337)/TCP(sport=source_port, dport=dst_port, flags='A', seq=ack_num+1, ack=syn_ack.seq+1)
print(ack.seq)
print("SENDING ACK PACKET")
sr(ack)
```
At this point, we finished the three-way handshake and we can confirm in Wireshark that it has been succesful.

<img src="https://user-images.githubusercontent.com/101567957/210428974-be67070f-7ee8-4615-bca7-82efa1a6f083.png" width="max">

*(Again, the port numbers in the screenshot are different from the ones used in the script.)*

Lastly, we craft and send the GET request packet:

```
# HTTP packet
getStr = 'GET / HTTP/1.1\r\nHost: challenge.nahamcon.com\r\n\r\n'
request = IP(dst=dst_ip, id=0x1337) / TCP(dport=dst_port, sport=source_port,
             seq=ack_num+1, ack=syn_ack.seq+1, flags='A') / getStr
#request.show()
print("SENDING HTTP PACKET")
resp = sr1(request)
```

We can see in Wireshark that we successfully created the three-way handshake and sent the GET request. And we received an HTTP packet back from the server!
<img src="https://user-images.githubusercontent.com/101567957/210429036-5339be55-34f6-4182-8302-397be832a3ff.png" width="max">

And the incoming 200 packet text contains our flag! 
<img src="https://user-images.githubusercontent.com/101567957/210429102-421d16a8-b0a6-4a4f-ad19-34d7aaa8b481.png" width="max">
