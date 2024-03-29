## Host Discovery (Ping Scan)



### List Scan (-sL)
---
Lists the hosts in the specified network or IP address by doing a reverse DNS resolution. This means that does not send any packet or probe to targets. 
```bash
    nmap -sL 192.168.1.32/32
    Starting Nmap 7.92 ( https://nmap.org ) at 2022-07-03 23:59 CEST
    Nmap scan report for 192.168.1.32
    Nmap done: 1 IP address (0 hosts up) scanned in 0.01 seconds
```
```bash
23:59:44.536594 IP 192.168.1.61.59629 > DNS_SERVER.domain: 35086+ PTR? 32.1.168.192.in-addr.arpa. (43)
23:59:44.545681 IP DNS_SERVER.domain > 192.168.1.61.59629: 35086 NXDomain 0/1/0 (120)   <-- No reverse resolution DNS
```

### Disable Port Scan (-sn)
---
Disables port scan during host discovery. This type of scan sends:
- ICMP echo request
- TCP SYN to port 443
- TCP ACK to port 80
- ICMP Timestamp request
```bash
    nmap -sn 8.8.8.8
```
```bash
   # ICMP echo request
   20:34:19.494174 IP 192.168.1.34 > dns.google: ICMP echo request, id 2434, seq 0, length 8
   # TCP SYN to 443
   20:34:19.494209 IP 192.168.1.34.49974 > dns.google.https: Flags [S], seq 2031498717, win 1024, options [mss 1460], length 0
   # TCP ACK to port 80 
   20:34:19.494214 IP 192.168.1.34.49974 > dns.google.http: Flags [.], ack 2031498717, win 1024, length 0
   # ICMP Timestamp query
   20:34:19.494255 IP 192.168.1.34 > dns.google: ICMP time stamp query id 40525 seq 0, length 20
   # TCP SYN ACK from target to attacker
   20:34:19.509434 IP dns.google.https > 192.168.1.34.49974: Flags [S.], seq 3606970364, ack 2031498718, win 65535, options [mss 1430], length 0
   # TCP RST from attacker to close connection on 443
   20:34:19.509504 IP 192.168.1.34.49974 > dns.google.https: Flags [R], seq 2031498718, win 0, length 0
   # ICMP repply from target
   20:34:19.510480 IP dns.google > 192.168.1.34: ICMP echo reply, id 2434, seq 0, length 8
   # No responses to TCP ACK on port 80 and ICMP timestamp
```

### Disable Ping (-Pn)
---
Disables the ping on nmap during port scans or host discovery. If not disabled, if a host is not responding to ping, it's not scanned.
Capture of a normal port scan (notice the ICMP resuqest and response):
```bash
   ## Command is nmap 8.8.8.8 -p443
   # Sends ICMP echo request
   20:43:28.454266 IP 192.168.1.34 > dns.google: ICMP echo request, id 7637, seq 0, length 8
   # Sends TCP scan to port 80 and 443 and the timestamp query (same as with -sn options
   20:43:28.454299 IP 192.168.1.34.41391 > dns.google.https: Flags [S], seq 2725091738, win 1024, options [mss 1460], length 0
   20:43:28.454305 IP 192.168.1.34.41391 > dns.google.http: Flags [.], ack 2725091738, win 1024, length 0
   20:43:28.454310 IP 192.168.1.34 > dns.google: ICMP time stamp query id 49365 seq 0, length 20
   # ICMP reply to the ICMP request sent
   20:43:28.488234 IP dns.google > 192.168.1.34: ICMP echo reply, id 7637, seq 0, length 8
   20:43:28.488349 IP dns.google.https > 192.168.1.34.41391: Flags [S.], seq 3695684294, ack 2725091739, win 65535, options [mss 1430], length 0
   20:43:28.488375 IP 192.168.1.34.41391 > dns.google.https: Flags [R], seq 2725091739, win 0, length 0
   20:43:28.558928 IP 192.168.1.34.41647 > dns.google.https: Flags [S], seq 1939023093, win 1024, options [mss 1460], length 0
   20:43:28.573765 IP dns.google.https > 192.168.1.34.41647: Flags [S.], seq 1912564117, ack 1939023094, win 65535, options [mss 1430], length 0
   20:43:28.573827 IP 192.168.1.34.41647 > dns.google.https: Flags [R], seq 1939023094, win 0, length 0
```
With the -Pn, see the difference:
```bash
   ## command nmap -Pn 8.8.8.8 -p443
   # Directly sends the TCP SYN without discovery
   20:47:32.006543 IP 192.168.1.34.42984 > dns.google.https: Flags [S], seq 2171424091, win 1024, options [mss 1460], length 0
   20:47:32.021837 IP dns.google.https > 192.168.1.34.42984: Flags [S.], seq 63304799, ack 2171424092, win 65535, options [mss 1430], length 0
   20:47:32.021862 IP 192.168.1.34.42984 > dns.google.https: Flags [R], seq 2171424092, win 0, length 0
```
### TCP SYN Ping (-PS[port-list])
---
This discovery uses an empty TCP packet with the SYN flag set to a defined port. The default port it's 80, but others can be specified with the PORT-LIST parameter.
```bash
   ## command nmap -sn -PS443,53,80 8.8.8.8
   # Directly sends the TCP SYN without discovery and just scanning specified ports
   # Sends SYN to port 53
   11:01:39.136486 IP 192.168.1.34.37313 > dns.google.domain: Flags [S], seq 2733092374, win 1024, options [mss 1460], length 0                            
   # Sends SYN to port 80
   11:01:39.136529 IP 192.168.1.34.37313 > dns.google.http: Flags [S], seq 2733092374, win 1024, options [mss 1460], length 0                                           
   # Sends SYN to port 443
   11:01:39.136539 IP 192.168.1.34.37313 > dns.google.https: Flags [S], seq 2733092374, win 1024, options [mss 1460], length 0                                         
   # Receive SYN,ACK from target port 53. Port open, host up
   11:01:39.153387 IP dns.google.domain > 192.168.1.34.37313: Flags [S.], seq 2742099961, ack 2733092375, win 65535, options [mss 1430], length 0                       
   # Receiving a SYN,ACK from target port 443. Port open, host up
   11:01:39.153391 IP dns.google.https > 192.168.1.34.37313: Flags [S.], seq 2367239538, ack 2733092375, win 65535, options [mss 1430], length 0                       
   # Send RST to target to close connection 
   11:01:39.153434 IP 192.168.1.34.37313 > dns.google.domain: Flags [R], seq 2733092375, win 0, length 0                                                               
   11:01:39.153457 IP 192.168.1.34.37313 > dns.google.https: Flags [R], seq 2733092375, win 0, length 0 
```

### TCP ACK Ping (-PA[port-list])
---
Similar like previous one but this time just sends the ACK flag. This is usefull if there is any filtering for SYN packets (the usual). When a host receives an ACK packet, target should repply with a RST (per the TCP RFC)
```bash
   ## command nmap -sn -PA443,53,80 8.8.8.8
   # Send ACK to port 53
   11:07:30.552753 IP 192.168.1.34.50625 > dns.google.domain: Flags [.], ack 2571984450, win 1024, length 0                                                             
   # Send ACK to port 80
   11:07:30.552791 IP 192.168.1.34.50625 > dns.google.http: Flags [.], ack 2571984450, win 1024, length 0                                                               
   # Send ACK to port 443
   11:07:30.552801 IP 192.168.1.34.50625 > dns.google.https: Flags [.], ack 2571984450, win 1024, length 0                                                             
   # Received a RST packet from target ports 53 and 443 
   11:07:30.570329 IP dns.google.domain > 192.168.1.34.50625: Flags [R], seq 2571984450, win 0, length 0                                                               
   11:07:30.570332 IP dns.google.https > 192.168.1.34.50625: Flags [R], seq 2571984450, win 0, length 0 
```

### UDP Ping (-PU[port-list])
---
Sends a empty (for uncommon ports, common ports sends a payload) UDP packet to UDP port 40125 if no port is specified. When a closed port is scanned, the target should return a ICMP port unreachable packet what means that the host is up. 
```bash
   ## command nmap -sn -PU scanme.nmap.org
   # Send the UDP packet to 40125 UDP port
   11:20:38.041028 IP 192.168.1.34.33546 > scanme.nmap.org.40125: UDP, length 40
   # Receive a ICMP port unreachable
   11:23:55.494310 IP scanme.nmap.org > 192.168.1.34: ICMP scanme.nmap.org udp port 40125 unreachable, length 76
```

### ICMP Ping types (-PE, -PP, and -PM)
---

### IP Protocol Ping (-PO[protocol list])
---

### ARP Scan (-PR)
---


