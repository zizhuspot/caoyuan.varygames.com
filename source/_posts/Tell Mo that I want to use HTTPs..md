---
title: Tell Mo that I want to use HTTPs.
date: 2023-11-07 05:04:00
categories: 
  - Backend
keywords: 
tags: 
  - Interviews
  - Backend
  - Security
  - recognize
  - development
  - framework
  - technical
  - network firewall
description: Years later, in the face of the technical interviewer of Green Alliance Technology, the program ape Xiao ❤ will recall the distant afternoon when the hacker Tom showed him the instant disintegration of the network firewall.
cover: https://s2.loli.net/2023/11/07/msIoX1JhrclUuGg.webp
---
Table of Contents

1. Introduction
2. What are HTTPs?
3. Customizing Certificates
4. Enabling HTTPs on the server side
5. Postscript

## 1. Introduction

"The global construction team, anywhere, at any time, will put 'security' in the first place! And network project security, in turn, is the top priority of Internet products, so can you say a little bit about how data transmission in project development is secured?"

Less than recalling, I only heard the interviewer ask slowly and methodically.

"Heh, construction team, as a new age migrant worker I certainly know that safety comes first. **Wannabe can't eat fish while constructing! **" At this point, I would like to talk nonsense, but the opposite is a smiling interviewer, not good enough to refuse, so I think for a moment slowly answered: "We are in the project development on-line, in order to interface data transmission security, access may need to use HTTPs".

At this point there are small partners have questions, data security access, not with POST request can be?

The fact is certainly not the case, "we are in the process of interface design, GET request access security is the worst, because the transfer of data will be spliced in the URL to send directly, while the POST request will be the transfer of data into the Body body, not visible in the address bar, but there is no difference between the two in the transmission process. **To achieve secure data transfer, HTTPs must also be used". **

## 2. What are HTTPs?

### 2.1 Introduction to HTTPs

"So tell me about your understanding of HTTPs", the interviewer didn't seem to be very surprised by this answer and asked again.

"In the network, the HTTP protocol communication is ubiquitous, familiar with the RESTful API development program apes know, even if the POST request, in the network transmission due to the reasons such as plaintext transmission, is still not very secure. At this point, we need to use HTTPs, in general terms **HTTPs = HTTP + data encryption + authentication + integrity protection**."

> The essence of HTTPs is to add a new layer between the HTTP application layer and the TCP transport layer. Assuming that we use the TLS protocol for HTTPs encryption, HTTPs is the communication between HTTP and TLS, and then the communication between TLS and TCP.

### 2.2 Encryption Process

![](https://s2.loli.net/2023/11/07/msIoX1JhrclUuGg.webp)

I went on to say, "HTTPs encryption is roughly divided into two phases, the **Certificate Validation** and **Data Transfer** phases, which interact as follows:"

#### 2.2.1 Certificate verification process

1) The browser initiates HTTPs request;

(2) The server generates a pair of public and private keys, **the private key is stored by the server itself, and the public key is put in the HTTPs certificate and returned to the client**, and the content of the certificate also contains the website address, certificate authority, expiration date, etc. The client verifies the legitimacy of the certificate (**certificate validation** and **data transmission**);

(3) The client verifies the legitimacy of the certificate (e.g. whether the URL in the certificate is the same as the current URL and whether the certificate has expired), and if it is not legitimate, it prompts an alarm;

#### 2.2.2 Data Transmission Phase

1) When the certificate is verified to be legal, the client generates a random number as the key of the symmetric algorithm;

2） **The client encrypts the random number through the public key and transmits it to the server**;

3) After receiving the encrypted random number, the server side decrypts the random number through its own private key;

4) After the server gets the symmetric key (random number), it symmetrically encrypts the returned result data.

## 3. Homemade certificates

"Okay, so if you were asked to design a server for HTTPs, how would you do it?" The interviewer seemed to want to hear more real-world experience, so he went on to ask.

I was not willing to show weakness, so I added: "After understanding the principle of HTTPs and the authentication process, let's sort out the three things necessary to open HTTPs secure communication: **CA certificate, server certificate and server private key **."

1. CA (Certification Authority) certificate is issued by a CA organization to prove the legitimacy of an entity's identity. 2;
2. the CA certificate needs to be installed on the client machine, i.e. the browser, to verify the authenticity of the server certificate;
3. the server's certificate is used to send to the client, the server private key to do data encryption and decryption.

"So, when we want to open an HTTPs service, we need to apply for a certificate first. there are two common software for making homemade certificates: OpenSSL and XCA. the former is a command line interface and the latter is a graphical interface. to make sure that the process is visualized, we can use XCA to make the certificate."

### 3.1 Creating a certificate with XCA

At this point, the clever and attentive reader may realize: if the interviewer wants to see me applying for a certificate, then there is clearly not enough time! However, in order to let the reader better familiarize with the HTTPs encryption process, the thoughtful program ape ❤ ❤ After the interview, we sorted out the following process:

First, download and install the XCA tool, download address: [xca.hohnstaedt.de/](https://link.juejin.cn?target=http%3A%2F%2Fxca.hohnstaedt.de%2F "http://xca.hohnstaedt.de/") , I downloaded the latest [XCA version 2.4.0](https://link.juejin.cn?target=https%3A%2F%2Fwww.hohnstaedt.de%2Fxca%2Findex.php%2Fdownload "https://www. hohnstaedt.de/xca/index.php/download").

1) After the installation is completed, we open the xca software, click File -> New Database on the main page (select the storage directory of certificate application, and then enter the password)

![](https://s2.loli.net/2023/11/07/kOEGSY2o6Ch3FfQ.webp)

2) Switch to the certificate page and click on create certificate (I have already created a certificate, so it already exists in the certificate field, so please ignore it)

![](https://s2.loli.net/2023/11/07/gBQ9VnLrMNuvHmj.webp)

3) On the source page, select Signature, Algorithm, and Template, respectively, and finally click "Apply all information from the template"

![](https://s2.loli.net/2023/11/07/LAsSoXvxZ7dKN1n.webp)

4) switch to the main page, fill in the internal name of the certificate, the rest of the options can be filled in casually, and finally generate a new key

![](https://s2.loli.net/2023/11/07/tGopyknX2QTBswL.webp)

(5) Click Create, OK to return, in the newly generated certificate to create a certificate [Note: When choosing the signature, select "Use this CA certificate for signing", because we are now applying for server-side certificates, so select the template to choose the default TLS_server].

![](https://s2.loli.net/2023/11/07/VmsoOwWb2gPtqMG.webp)

6）Switch to the main body again, fill in the name and other information, click to generate a new key, OK after the completion of the creation to return to the

![](https://s2.loli.net/2023/11/07/cjNsRyP6QFbupre.webp)

At this point, the CA certificate and server certificate are created, and then exported.

### 3.2 Exporting Certificate

1) On the certificate page, export the CA certificate.

![](https://s2.loli.net/2023/11/07/hmVOKDbN9tCRW2e.webp)

2) Export server-side certificate

![](https://s2.loli.net/2023/11/07/5EM71DNgmTeBAna.webp)

3) On the private key page, export the server-side private key

![](https://s2.loli.net/2023/11/07/tXo4gQmjqVvxsup.webp)

At this point, we are done generating certificates using XCA. Next, turn on HTTPs_ on the _server_ using the certificate and private key.

## 4. Enabling HTTPs on the server side

The example here is in Golang. Other languages are similar in that you only need to switch to HTTPs when HTTPs are listening for services.

To control whether HTTPs are turned on in Go, you can add three new parameters to the project startup entry:

```go
safeMode := flag.Bool("safe_mode", true, "https mode")
certPath := flag.String("cert_path", "D://runSpace//wecom//ca//server.crt", "server ca")
keyPath := flag.String("key_path", "D://runSpace//wecom//ca//server.pem", "private key")
```

1）如下图所示

![](https://s2.loli.net/2023/11/07/UtulK8kRHoMnFJm.webp)

2) safeMode controls whether HTTPs are turned on or not.

![](https://s2.loli.net/2023/11/07/BOAHrMYIaSu8GlC.webp)

The logs are as follows

```csharp
[] Listening and serving HTTPS on :80
```

3) After the startup is complete, you can type [https://127.0.0.1:80](https://127.0.0.1:80) into your browser to access the

![](https://s2.loli.net/2023/11/07/Hdor3V6nSmPxCEl.webp)

Since the CA certificate is not added to the browser, so the access will be prompted, we choose [Advanced -> continue to go] can be.

(4) If you do not want to report this kind of prompt every time you visit, we can add the CA certificate just applied for in the browser, Google Chrome import process is as follows

![](https://raw.githubusercontent.com/zqwuming/blogimage/img/img/b750cb19cc3b48728229e1bb15584668%7Etplv-k3u1fbpfcp-zoom-in-crop-mark%3A1512%3A0%3A0%3A0.awebp)

5）【Manage Certificates -> Import】Import the certificate you just applied in XCA, close the browser and restart.

![](https://s2.loli.net/2023/11/07/53KOXYbrNAVqiW2.webp)

At this point, the HTTPs service can be accessed normally. If you want to switch to HTTP access, just set the command line parameter **safe_mode to false**.

> The full code is at [GitHub](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fyangfx15%2Fwecom%2Fblob%2Fmain%2Fmain.go "https://github. com/yangfx15/wecom/blob/main/main.go").

## 5. Postscript

The interviewer nodded slightly and asked again, "During the HTTPs encryption process, why are private key authentication and data transmission, two ways of asymmetric encryption and symmetric encryption, respectively, used?"

I smiled wryly, to the interviewer dug a million times the pit, they really all voluntarily stepped up: "This is of course determined by the characteristics of both of them, which:"

* Symmetric encryption: that is, ** both parties to a message use the same key to encrypt and decrypt the message ** using symmetric cryptographic coding techniques. Since the algorithm is public, the key cannot be disclosed to the public. It has a small amount of computation and fast encryption speed. The disadvantages are insecurity and difficulty in key management, such as AES, IDEA.
* Asymmetric encryption: ** can only be encrypted and decrypted by pairs of public and private keys, generally public key encryption, private key decryption **. Process: Party A generates a pair of keys and discloses one of them as a public key; Party B gets the public key, encrypts the data and sends it to Party A, which decrypts it with a dedicated private key. Secure, but slower encryption, such as the RSA algorithm.

"HTTPs encryption uses **hybrid encryption**, which combines the advantages of symmetric encryption and asymmetric encryption, and transmits the symmetric encrypted public key through asymmetric encryption. In this way, both the security of the public key transmission process [asymmetric encryption] and the efficiency of the data exchange [symmetric encryption] are ensured."

The interviewer nodded slightly, "Uh-huh, that's enough of that question for now, next ......" . With a good start, the interview process my legs are not shaking, the mouth is not shivering, **HTTPs also do not want to learn, the evening call on the old silent together to eat fish! ** emmm really good day ah ~~
