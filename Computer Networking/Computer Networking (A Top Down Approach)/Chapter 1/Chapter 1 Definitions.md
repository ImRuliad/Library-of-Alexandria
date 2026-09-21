
| Term                           | Definition |
| ------------------------------ | ---------- |
| Packet                         |            |
| Router                         |            |
| Link-Layer Switch              |            |
| Store-And-Forward-Transmission |            |
| Guided Medium                  |            |
| Shared Medium                  |            |
| Digital Subscriber Line        |            |
| Cable                          |            |
| Unshielded Twisted Pair        |            |
| Fiber to the Home              |            |
| Hybrid Fiber Coax              |            |
| Client                         |            |
| Server                         |            |
| Socket Interface               |            |
| Distributed Applications       |            |
| Transmission Control Protocal  |            |
| Internet Protocal              |            |
| Route                          |            |
| Path                           |            |
| Packet Switches                |            |
| Communication Links            |            |
| Hosts / End systems            |            |
|                                |            |

---
#### ***Equations***

$$
L = Bits\;in\;a\;packet
$$
$$
R = bits/sec\;of\;transmission\;link\;(transmission\;rate)
$$
$$
z=time\;to\;transmit\;packet
$$
$$
z = L/R\;seconds
$$
$$
N = Number\;of\;links
$$
---
***End-to-End Delay (Nodal)***
$$
\begin{align}
d_{end-to-end} = N \frac{L}{R}
\end{align}
$$
---
***Nodal Processing Delay***
*Time required to examine the packet's header and determine where to direct packet*

$$
d_{proc}
$$
---
***Queueing Delay***
*Amount of time a packet waits to be transmitted onto the queue of the link*

$$
d_{queue}
$$
---
***Transmission Delay***
*Amount of time a packet waits to be transmitted out of the queue*

$$
d_{trans} = L/R
$$
---
***Propagation Delay***
*Time taken to propagate a bit from the beginning of the current link to the next node*

---
***Average Rate Packets Arrive at Queue***

$$
a = packets/sec
$$
---
***Average Rate Bits Arrive at Queue***

$$
L_{a} = Avg\;rate\;bits\;arrive\;in\;queue
$$
---
***Traffic Intensity***
*The ratio between the average rate bits arrive at queue and average rate bits leave the queue*

$$
L_{a}/R
$$
---
***End-To-End Delay (Total)***
*The total delay from source to destination*

$$
d_{end-to-end}=N(d_{proc}+d_{trans}+d_{prop})
$$
