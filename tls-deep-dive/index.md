# SSL/TLS Deep Dive


<!--more-->

> Disclaimer: I will use the term TLS instead of SSL in this blog to get rid of me writing and you reading “SSL/TLS“ again and again, TLS and SSL are different versions of the same protocol and TLS is the one in use.

## What This Blog Post Is About

This deep dive would be a one-stop guide for anyone trying to grok Transport layer security.

This guide will be broken down into multiple blog entries “this“ being the first of many, so that no one reading this deep dive gets overwhelmed or bogged down by details. this strategy provides necessary pauses and enough time for anyone to assimilate the provided information.

TLS/SSL is not hard to understand but there is so much to cover, to understand TLS we first need to provide a wider context for it, persevere through this deep dive and I guarantee you will understand TLS.

**_This blog series will cover more or less following topics_**

**_what problem does TLS solve, what are the primitives, what claims does it make, what is a public-private key, what is a digital signature, what are certificates, certificate authorities, certificate chains, replay attacks, non-repudiation, public key infra, integrity, TLS versions and more…_**

let’s dive 🤿…..

## The Problem And The Solution

### Internet, HTTP, and HTTPS

![tunnel](images/posts/TLS/tunnel.webp)

keep in mind that the above image is good for understanding the technology at hand but it is a model and not the reality so view it abstractly while reading further.

The Internet is a big network of interconnected routers (not only routers but let’s simplify) as you can see in the above image. the job of this internetwork is to take the data packets from node at point 1 to node at point 2.

consider a client requests the server for an HTML page, the server will prepare the HTML and send it back to the client and the client will display the HTML.

**_there are 2 very simple yet nontrivial points you have to keep in mind regarding this communication_** 📑

- There is no encryption involved in this very simple communication I described

- And once the data leaves the client or the server, the client and server lose control of the data, It is the job of the internet to route data however it wants

note that the majority of the traffic on the internet is from websites and websites don’t always present information, sometimes they want to retrieve information from the users as well.

for example passwords or bank information.

communicating such information between the nodes using the aforementioned model of communication would be stupid, as anyone between the client and server could eavesdrop and read what is being sent.

**Do you see the problem?** 🎯

This is the problem TLS is trying to solve, TLS before communicating anything on the internet creates a secure tunnel through which the information flows.

HTTP which uses TLS is called HTTPS, there is no other difference, but HTTP is not the only protocol for which TLS is tailored, TLS is generic and it does not care about the protocol underlying it, some other applications of TLS include FTPS and TLS VPN using which you can use your company’s network sitting at home.

## How Does TLS Accomplish Data Security?

![tls](images/posts/TLS/tls.webp)
**_Note: It is very important to understand two things_**

1. TLS does not **_literally_** create a tunnel because as you may or may not know internet uses a model called packet switching to transfer data ( basically in a packet-switched network every data packet takes a different route instead of taking one dedicated route ). **_So how could TLS create a tunnel?_**

2. second TLS does not encrypt the underlying protocol headers as TLS operates on the Transport layer it won’t encrypt the TCP or IP headers but rather it would encrypt the payload like passwords or bank information

## What Should TLS Do To Data To Make Communication Safe?📡

> Recap, when I said the data leaves the node, it is not in our control anymore, anyone can read our sent data, someone reading our data on the wire is called a man-in-a-middle attack, remember TLS can not just prevent the man-in-the-middle from getting the packets but it can encrypt the data captured by middle man to render it useless to the middle man.

TLS provides 3 main services

**1. Confidentiality:** only the entities for whom data is meant could read the data, this service is provided using encryption.

**2. Integrity:** data is not susceptible to change on the wire it should reach the recipient as it left the sender (no changes not even an extra period or space ), this service is provided by hashing.

**3. Authentication:** the third and last service of TLS is to make sure that people or parties are who they claim to be, this service is provided using **_public key infrastructure_**.

> To understand these services we need to first discuss some cryptography primitives which I will write about in the future part of this blog series.

## Some Web-related Attacks For Perspective

### TLS Is Anti-Replay:

Replay attacks are very common on the web in which a malicious user could intercept the user’s requests and later on replay those requests to achieve some bad motives.

For example, if the user is using a bank website to transfer funds of $100 and the man in the middle captures the request and replays to request multiple times to empty the user account

TLS prevents these kinds of attacks using sequence numbers for every request and if the request with the same sequence number is repeated it is rejected.

### TLS Promotes Non-Repudiation:

Google defines repudiation in the following way

> to refuse to have anything to do with

If someone sends you something on the internet and later refuses to accept that they sent you anything , this won’t be possible because of 2 services that TLS provides namely Integration and Authentication.

## The End

This is it for this blog entry in future posts I will go deeper into the implementation and consequences of TLS. giving all this context before going deep was very important to cement the understanding

I hope now you have understood the motivations for having a protocol like TLS and looking forward to understanding more deeply.

