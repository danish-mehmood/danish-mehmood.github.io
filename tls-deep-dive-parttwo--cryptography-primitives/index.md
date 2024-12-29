# SSL/TLS deep dive part 2: cryptography primitives


<!--more-->

{{< admonition note "Table Of Contents" >}}

:(fa-solid fa-arrow-right-long): Part 1 - [Introduction to TLS](https://danish-mehmood.github.io/tls-deep-dive/)

:(fa-solid fa-arrow-right-long): Part 2 - **Crytography Primitives** :(fa-solid fa-arrow-left-long): you are here :(fa-solid fa-location-crosshairs):

{{< /admonition >}}

In my last blog post, I tried to put TLS in a context in which it is easier to see why TLS is needed in the first place, what problems it solves, and what services it provides to make our communication over a public network secure.

In this blog I want to tell you about some cryptography principles behind TLS, it is a prerequisite to grok TLS, in this blog post I will write about hashing algorithms,**_HMAC, Encryption, public-private_**keys, etc.

## Hashing Algorithms 🧮

Hashing algorithms are a category of algorithms that take input of arbitrary length and output a string of fixed length.

Hashing is ubiquitous in computer science, it has so many applications and is a useful category of algorithms.

let me describe a very basic hashing algorithm, suppose I have a hashing algorithm that takes in a string “hello” and outputs the sum of the numbers at which every letter of the string occurs in the alphabetical order.

so if the input is “hello” the output would be 8 + 5 + 12 + 12 + 15 = 52.

likewise, you can see the output for any given input, it’s a very simple implementation.

although this example algorithm is good for example purposes but is a horrible hashing algorithm, you would be thinking how could someone categorize hash functions as good or bad,

the answer is good hashing algorithms should follow the following 4 rules

1. Infeasible to produce a given digest.

2. Impossible to extract the original message.

3. Slight changes produce a big difference in the output.

4. The size of the output should be fixed.

if you try to filter the hashing algorithm we discussed through these 4 rules, it won’t check a single rule, which is why it is a terrible hashing algorithm.

## Hands-On

Next, I would demonstrate a good hashing algorithm called sha-256.

If you want to follow along and have Linux then open up the terminal and follow along otherwise you can use an online sha-256 generator.
![practical](images/posts/TLS/part2/handson1.webp)

as you can see when I gave “hello“ as an input to the sha256sum Linux utility it outputs a random string. if you give the same input you will get a similar output as me.

as sha-256 is a good hashing algorithm and is being used in the real world, it follows all 4 rules I stated.

it is close to impossible to generate all the inputs given the output or the other way around, telling the input which generated the specific output,

it also satisfies the rule of output being of specific size, sha-256 given input of any size would always output a string of size 256 bits.

it also satisfies the rule which states that a small change in input should result in a big change of output, see the following example.
![practical](images/posts/TLS/part2/handson2.webp)

Only introducing a period at the end of the last input has changed the output drastically (this is called an avalanche effect) from,

**5891b5b522d5df086d0ff0b110fbd9d21bb4fc7163af34d08286a2e846f6be03**

to

**5d382abfb5da6294d05e4afade40adcdfc85ac56a6a2e6d94b9734c41c6ddbf3**

## Collisions 💥 🏎

collision occurs when two **different** inputs map to the **same** output,

to understand collisions consider the following example, suppose we have a hash function that takes arbitrary input and outputs fixed size three bit string.

the total unique strings that our function would be able to output would be **_23= 2 × 2 × 2 = 8_**.

which is very small if we give our function 9 **different** inputs the two inputs are bound to map to the same output string.

this is called collision, and it is inevitable when you have output of a fixed size but in the case of good hashing algorithms like sha-256 which has 256-bit size output, it is very unlikely to happen because 2^256 is a humongous number.

## Integrity, Authentication And HMAC:

Integrity as I explained in the last post is a property of data reaching the recipient as it leaves the sender, unchanged.

TLS uses hashing to achieve integrity.
![hmac](images/posts/TLS/part2/hmac.webp)
consider the following scenario. blue user wants to send a message to the green user over the internet and also wants to ensure the integrity of the message. for that blue user does not only send the message but also the hash of the message to the green user, the green user gets the message and hash, hashes the message, and compares the produced hash with the hash sent with the message by the blue user and if the hashes match the green user would know that the message hasn’t been changed.

**_Do you notice any problem in this flow? It's quite easy to spot._**

the man in the middle could easily get the message as well as the hash, change the message append the hash of a modified message, and send it to the green user and the green user would never know that the message was changed in transition because the message and the hash would match at his end.

## HMAC And Authentication

To overcome this problem, before sending any message green and blue user agree on a key (we will go deep into this, how this is done for now just know that they exchange a key), then when they are sending a message they don’t only hash the message they want to send, instead they hash both message and the key together and then send the message towards the other side, the remaining flow remains the same.

the result of hashing the message and the key is called message authentication code,

notice that the parties don’t only need to agree on the key but also how they are going to hash the key with the message because remember, appending the key to the message then hashing and putting the key before the message then hashing would result in different hashes.

to make sure these things don’t become a reason for conflict there is a whole RFC standard that guides and standardizes the process called HMAC.

there is one more thing you need to notice, HMAC does not only provide integrity it also provides authentication because when you are putting the key with the message and then hashing, the recipient wouldn’t only be sure of the integrity of the message but also they would know that this message could only be sent by the person who has the key (which was exchanged in the beginning of the process).

> So HMAC is how TLS achieves integrity and authentication.

## Encryption 🔢:

As we discussed in the last post TLS also provides confidentiality, it uses encryption to achieve the goal of confidentiality.

Encryption is a very simple affair, you take an input, feed it to some encryption algorithm, and get the output, in the case of encryption the input is called plain text and the output is called cipher text.

the simple encryption in which you don’t use any key and just shuffle the plain text to get the cipher text is not scalable, because to send the message to every different person you would need to come up with a new algorithm to encrypt.

this is the reason we use key-based encryption in which the algorithms are known to everyone but the keys used are private. using this approach we can use different keys with the same algorithm and the cipher text will not break.

there are two ways to do the key-based encryption, one is asymmetric cryptography and the other is symmetric cryptography.

## Asymmetric cryptography:

in this type of cryptography, you have two pieces of keys one is public and the other is private, one you could use for encryption and the other for decryption. these two keys are mathematically related, if you encrypt with one key only the other key in the pair could decrypt it.

asymmetric cryptography has some weaknesses, it is compute intensive it takes a lot of compute power to encrypt and decrypt, and secondly the cipher text is large compared to plain text.

it also has a very useful pro, which is the private key is never shared with anyone, it always remains with the user and public key could be known by anyone, it does not matter.

## Symmetric cryptography:

in a symmetric crypto setup we only have one key which is used for both encryption and decryption, and this has an inherent weakness as we would need to share the key and anyone can capture it in transit,

symmetric cryptography is very fast at working on large amounts of data and the cipher text it produces is equal in size to the plain text.

> For bulk data, symmetric crypto is used and for small amounts of data asymmetric crypto is used

Public-Private Key Encryption 🔑:
![practical](images/posts/TLS/part2/pubprivkey.webp)
let’s try to understand the use of private and public keys in TLS in a more coherent way.

### Confidentiality:

suppose Jim wants to send Pam a message and wants confidentiality, what he would do is encrypt the message using “Pam’s public key” and send the message over the wire, as the message was encrypted using Pam’s public key, only Pam’s private key could decrypt and only pam can read it hence confidentiality is achieved.

### Authentication (digital signatures) and Integrity 🖊:

now consider another scenario where Jim wants to send Pam a message and does not care about confidentiality but wants Pam to know that the message was indeed sent by him, to achieve this he would now use his private key to encrypt the message and when the message would reach pam she could only decrypt using Jim’s public key that she has and now she would know that the message is from him.

this technique is called digital signatures. because the user is using encryption to prove his or her identity.

but notice that digital signatures also have a side effect which is “integrity”. because when Pam was able to decrypt the message successfully she also got the confirmation the message was not modified in transition because if it was changed then she won’t be able to decrypt it.

### Hybrid Encryption:

In the real world, the two kinds of encryption are not used in isolation, but they are used together because they complement each other so well and cancel each others weakness’s

remember that asymmetric cryptography is not feasible for bulk data and symmetric crypto is.

and that asymmetric crypto is safe in a way that private keys are never shared and symmetric cryptography has a drawback that we need to share the same key among users and there is a danger of a secret key being captured in transit.

> well in the real world, a hybrid model is used in which asymmetric cryptography is used to exchange the keys and then further communication among the involved parties is encrypted and decrypted using symmetric keys. This way we get best of both worlds.

**What about the signatures then?**
as we have discussed before asymmetric encryption is used to digitally sign messages but with a single key in the symmetric crypto signatures are not possible. and asymmetric crypto is not good for large amounts of data so how do we sign data to prove the identity?

we can use the smaller version of the message for signature purposes, this is where hashing comes in, we can hash the message first and then sign it using a private key, and add it to the message, then the recipient can decrypt using public key getting the hash, then hashing the original message and comparing the two hashes would know the sender is the expected one.

> this achieves once again authentication and integrity.

This is it for this blog entry in future posts I will go deeper into the implementation and consequences of TLS.

I hope now you have understood what role encryption and hashing plays in TLS and how everything fits together.

