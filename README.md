# NMAP Scan Types (Work on Progress)
---

This document will try to compile all the scan types that nmap can do with the traffic captures for better understanding of how they work at a network level.

### TCP SYN Scan (-sS)
----
This type of scan does not complete a full TCP connection. It sends a SYN TCP packet, and depending on the target response, nmap decides status of the port based on the response to a SYN Probe:

| Probe Response | Status |
|-|-|
|TCP SYN/ACK response|Open|
|TCP RST response|Closed|
|No response received|Filtered|
|ICMP unreachable error (type 3, code 1, 2, 3, 9, 10, or 13)| Filtered|


#### Open Port
Nmap receives a SYN,ACK from the target, then does not establish the connection and nmap sends a RST.
```markup
    ATACKER ------> SYN -------> TARGET
    ATACKER <------ SYN,ACK <------- TARGET
    ATACKER ------> RST -------> TARGET
```
```bash
20:56:33.633394 IP 192.168.1.61.35266 > 192.168.1.1.ssh: Flags [S], seq 483337882, win 1024, options [mss 1460], length 0
20:56:33.637824 IP 192.168.1.1.ssh > 192.168.1.61.35266: Flags [S.], seq 626137850, ack 483337883, win 14600, options [mss 1460], length 0
20:56:33.637856 IP 192.168.1.61.35266 > 192.168.1.1.ssh: Flags [R], seq 483337883, win 0, length 0
```

#### Closed Port 
Nmap receives a RST from the target.
```markup
    ATACKER ------> SYN -------> TARGET
    ATACKER <------ RST <------- TARGET
```
```bash
20:57:05.215340 IP 192.168.1.61.52423 > 192.168.1.1.smtp: Flags [S], seq 764755950, win 1024, options [mss 1460], length 0
20:57:05.218495 IP 192.168.1.1.smtp > 192.168.1.61.52423: Flags [R.], seq 0, ack 764755951, win 0, length 0
```

#### Filtered Port
nmap does not receive anything from the target (packet dropped) or receives an ICMP Unreachable error.
```markup
    ATACKER ------> SYN -------> TARGET
    ATACKER ------> SYN -------> TARGET
    No response
```
```bash
21:04:59.672061 IP 192.168.1.61.55381 > scanme.nmap.org.http: Flags [S], seq 692149444, win 1024, options [mss 1460], length 0
21:05:00.564158 IP 192.168.1.61.55383 > scanme.nmap.org.http: Flags [S], seq 692280518, win 1024, options [mss 1460], length 0
```

### TCP Connect Scan (-sT)
---
This type of scan establishes a full TCP connection. 

#### Open Port

```markup
    ATACKER ------> SYN -------> TARGET
    ATACKER <------ SYN,ACK <------- TARGET
    ATACKER ------> ACK -------> TARGET
    ATACKER ------> RST -------> TARGET
```
```bash
21:13:04.200743 IP 192.168.1.61.45178 > 192.168.1.1.ssh: Flags [S], seq 2141274705, win 64240, options [mss 1460,sackOK,TS val 4229566320 ecr 0,nop,wscale 7], length 0
21:13:04.204891 IP 192.168.1.1.ssh > 192.168.1.61.45178: Flags [S.], seq 4214332755, ack 2141274706, win 14480, options [mss 1460,sackOK,TS val 3077863243 ecr 4229566320,nop,wscale 4], length 0
21:13:04.204954 IP 192.168.1.61.45178 > 192.168.1.1.ssh: Flags [.], ack 1, win 502, options [nop,nop,TS val 4229566324 ecr 3077863243], length 0
21:13:04.205058 IP 192.168.1.61.45178 > 192.168.1.1.ssh: Flags [R.], seq 1, ack 1, win 502, options [nop,nop,TS val 4229566324 ecr 3077863243], length 0
```

#### Closed Port
It's the same behaviour as the SYN Scan

#### Filtered Port
Same behaviour as the SYN Scan

### TCP FIN, NULL, and Xmas (-sF, -sN, -sX)
---
This three types of scans work in the same behaviour, just change the flags of the header:
- NULL: Does not set any bits in the header (0)
- FIN: Sets only the FIN bit
- Xmas: Sets the FIN, PSH and URG flags.

It exploits a definition in the TCP RFC 793, that if the target port receives any packet that does not contain SYN/RST/ACK, then:
- If target port is closed, the target respond with a RST
- If target port it's open, there is no response at all
> This type of scan gives false results on modern systems, specially for closed ports.
| Probe Response | Status |
|-|-|
|No response received |Open or Filtered|
|TCP RST packet|Closed|
|ICMP unreachable error (type 3, code 1, 2, 3, 9, 10, or 13)|Filtered|

#### FIN
When the port it's open:
```bash
    # Target port 22 it's open for this example
    nmap -sF 192.168.1.1 -p22
    PORT   STATE         SERVICE
    22/tcp open|filtered ssh
```
```bash
    # No response, port is NOT closed
    23:39:06.007022 IP 192.168.1.61.48111 > 192.168.1.1.ssh: Flags [F], seq 3108583640, win 1024, length 0
    23:39:06.107481 IP 192.168.1.61.48113 > 192.168.1.1.ssh: Flags [F], seq 3108714714, win 1024, length 0
```
And when port it's closed
```bash
# Target port 25 it's closed
nmap -sF 192.168.1.1 -p25
PORT   STATE         SERVICE
22/tcp open|filtered ssh
```
```bash
# No response, port is closed
23:39:06.007022 IP 192.168.1.61.48111 > 192.168.1.1.ssh: Flags [F], seq 3108583640, win 1024, length 0
23:39:06.107481 IP 192.168.1.61.48113 > 192.168.1.1.ssh: Flags [F], seq 3108714714, win 1024, length 0
```
#### NULL
```bash
# Target port 22 it's open for this example
nmap -sN 192.168.1.1 -p22
PORT   STATE         SERVICE
22/tcp open|filtered ssh
```
```bash
# No response, port is NOT closed
23:39:06.007022 IP 192.168.1.61.48111 > 192.168.1.1.ssh: Flags [F], seq 3108583640, win 1024, length 0
23:39:06.107481 IP 192.168.1.61.48113 > 192.168.1.1.ssh: Flags [F], seq 3108714714, win 1024, length 0
```











