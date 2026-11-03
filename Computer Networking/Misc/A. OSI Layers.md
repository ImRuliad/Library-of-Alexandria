## Table Of Contents

---

- [[#OSI Layers|OSI Layers]]
  - [[#Resources|Resources]]
  - [[#Related Notes|Related Notes]]

# OSI Layers

| No. | Name               | Protocol(s)                                                                                                          | Remarks                                                                  |
| --- | ------------------ | -------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ |
| 7   | Application Layer  | [D. DNS](D.%20DNS.md), [C. HTTP](C.%20HTTP.md), [HTTPS](C.%20HTTP.md), IMAP, POP3, SMTP, SSH                                              | Concerned with headers, cookies, content, etc.                           |
| 6   | Presentation Layer |                                                                                                                      | Concerned with encryption.                                               |
| 5   | Session Layer      |                                                                                                                      | Attach session tag.                                                      |
| 4   | Transport Layer    | [ALPN](tls.md/#:~:text=alpn), OCSP, [SNI](tls.md/#:~:text=sni), SSL, TCP, TLS, UDP                                   | Attach ports and sequence nos. Each individual part is called a segment. |
| 3   | Network Layer      | [AH](B.%20IPSec.md#ah), BGP, EIGRP, ESP, ICMP, IGRP, [IKE](B.%20IPSec.md#ike), IP, [IPSEC](B.%20IPSec.md), ISAKMP, IS-IS, OSPF, RIP | Attach IP add. Each individual part is called a packet.                  |
| 2   | Data Link Layer    | ARP, DHCP                                                                                                            | Attach MAC add. Each individual part is called a frame.                  |
| 1   | Physical Layer     |                                                                                                                      | Data transfer in 1s and 0s.                                              |

<p align="center">
  <br />
  <br />
  <img src="https://user-images.githubusercontent.com/50140864/101920490-e3352480-3bf1-11eb-8a76-18fc51052498.png" />
</p>

> NOTE: Corresponding to the OSI model, there exists a [TCP/IP model](https://networkinterview.com/tcp-ip-model-vs-osi-model) as well.

## ***Resources***
---
-   [The OSI Model](https://www.youtube.com/watch?v=7IS7gigunyI)
-	[What the Session, Presentation and Application Layers actually do](https://www.youtube.com/watch?v=2iFFRqzX3yE)
-   [The Internet that Wasn't](files/research-papers/the-internet-that-wasnt.pdf) (OSI)
-   [Encapsulation explained and why Ping (part of ICMP) doesn't work at layer three](https://www.youtube.com/watch?v=2shvrp0-yHw) and even then [ICMP is considered as a layer three protocol](https://serverfault.com/questions/511965/why-is-icmp-categorized-as-a-layer-3-protocol).
-   [Protocols](./protocols.md)

## Related Notes

- [[Books/Computer Networking (A Top Down Approach)/Chapter 1/1.5 Protocol Layers and Their Service Models.md|1.5 Protocol Layers and Their Service Models]] — This note compares the OSI model with the Internet protocol-layer service model.
- [[Computer Networking/Protocols/A. Table of Protocols.md|A. Table of Protocols]] — The protocol table provides a broader reference for the protocols mapped here.
- [[Computer Networking/Protocols/HTTP.md|HTTP]] — HTTP is one of the application-layer protocols listed here.
- [[Computer Networking/Protocols/DNS.md|DNS]] — DNS is one of the application-layer protocols listed here.
- [[Computer Networking/Protocols/TCP.md|TCP]] — TCP is a transport-layer protocol listed here.
- [[Computer Networking/Security/IPSec.md|IPSec]] — IPsec provides network-layer security protocols.
