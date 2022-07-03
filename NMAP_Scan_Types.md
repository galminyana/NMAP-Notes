# NMAP Scan Types (Work on Progress)
---

This document will try to compile all the scan types that nmap can do with the traffic captures for better understanding of how they work at a network level.

### TCP SYN Scan (-sS)
----
This type of scan does not complete a full TCP connection. It sends a SYN TCP packet, and depending on the target response, nmap decides status of the port based on the response to a SYN Probe:

```markup
   # Probe Response         # Status   #
   #------------------------#----------#
   # TCP SYN/ACK response   # Open     #
   # TCP RST response       # Closed   #
   # No response received   # Filtered #
   # ICMP unreachable error # Filtered #   <--  ICMP type 3, code 1,2,3,9,10,13
```

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

### TCP FIN, NULL, and Xmas Scan Types (-sF, -sN, -sX)
---

This three types of scans work in the same behaviour, just change the flags of the header:
- NULL: Does not set any bits in the header (0)
- FIN: Sets only the FIN bit
- Xmas: Sets the FIN, PSH and URG flags.

It exploits a definition in the TCP RFC 793, that if the target port receives any packet that does not contain SYN/RST/ACK, then:
- If target port is closed, the target respond with a RST
- If target port it's open, there is no response at all

```markup
   # Probe Response         # Status           #
   #------------------------#------------------#
   # No response received   # Open or Filtered #
   # TCP RST packet         # Closed           #
   # ICMP unreachable error # Filtered         #   <--  ICMP type 3, code 1,2,3,9,10,13
```
This type of scans don't work on systems not supporting the RFC. Won't work on Windows and will work on Unix like systems. 
See different packet capture for each scan type for an open port
```bash
    # Target port 22 it's open for this example
    nmap -sN 192.168.1.1 -p22
    PORT   STATE         SERVICE
    22/tcp open|filtered ssh
```
#### FIN Scan 
```bash
    # No response, port is NOT closed
    23:39:06.007022 IP 192.168.1.61.48111 > 192.168.1.1.ssh: Flags [F], seq 3108583640, win 1024, length 0
    23:39:06.107481 IP 192.168.1.61.48113 > 192.168.1.1.ssh: Flags [F], seq 3108714714, win 1024, length 0
```
#### NULL Scan
```bash
    # No response, port is NOT closed
    00:00:55.948950 IP 192.168.1.61.53762 > 192.168.1.1.ssh: Flags [none], win 1024, length 0
    00:00:56.053450 IP 192.168.1.61.53764 > 192.168.1.1.ssh: Flags [none], win 1024, length 0
```
#### Xmas Scan
```bash
    # No response, port is NOT closed
    00:02:13.803776 IP 192.168.1.61.63799 > 192.168.1.1.ssh: Flags [FPU], seq 1151864062, win 1024, urg 0, length 0
    00:02:13.906083 IP 192.168.1.61.63801 > 192.168.1.1.ssh: Flags [FPU], seq 1151995132, win 1024, urg 0, length 0
```
### ACK Scan (-sA)
---
It only sets the ACK flag in the probe packet. Usefull to check for firewall rules. Mainly it detects if ports are filtered or unfiltered. If receives a RST, means that either the port is open or closer, hence, the port is unfiltered. When no response or ICMP error received, means the port is filtered
```markup
   # Probe Response         # Status           #
   #------------------------#------------------#
   # TCP RST response       # Unfiltered       #
   # No response received   # Filtered         #
   # ICMP unreachable error # Filtered         #   <--  ICMP type 3, code 1,2,3,9,10,13
```
- Unfiltered Port
```bash
    # Scanning a open port, not filtered
    nmap -p22 -sA 192.168.1.1 -Pn
    PORT   STATE      SERVICE
    22/tcp unfiltered ssh
   
    # RST is received as port is not filtered but open
    0:20:36.807220 IP 192.168.1.61.59915 > 192.168.1.1.ssh: Flags [.], ack 4272271275, win 1024, length 0
    00:20:36.810550 IP 192.168.1.1.ssh > 192.168.1.61.59915: Flags [R], seq 4272271275, win 0, length 0
```
- Filtered Port
```bash
    # Scanning a open port, not filtered
    nmap -p25 -sA XXXXXXXXXX -Pn
    PORT   STATE      SERVICE
    22/tcp unfiltered ssh
   
    # RST is received as port is not filtered but open
    00:22:39.010278 IP 192.168.1.61.49530 > XXXXXXXXXX.smtp: Flags [.], ack 3820581451, win 1024, length 0
    00:22:40.021481 IP 192.168.1.61.49532 > XXXXXXXXXX.smtp: Flags [.], ack 3820712521, win 1024, length 0
```
### TCP Window Scan (-sW)
---
Not always can rely on this type of scan, as only works on certain systems. Works like ACK Scan, just examining the response for the RST response for the TCP Window Size:
- Open ports, return a positive window size
- Closed ones return a zero window
```markup
   # Probe Response                              # Status           #
   #---------------------------------------------#------------------#
   # TCP RST response with non-zero window field # Unfiltered       #
   # TCP RST response with zero window field     # Filtered         #
   # No response received                        # Filtered         #
   # ICMP unreachable error                      # Filtered         #   <--  ICMP type 3, code 1,2,3,9,10,13
```

### TCP Mainmon Scan (-sM)
---
Not so usefull in actual systems. It sends a probe with FIN and ACK flags, and then should receive a RST response as expected. However, some systems simply drop the packet for open ports. 
```markup
   # Probe Response         # Status           #
   #------------------------#------------------#
   # No response received   # Unfiltered       #
   # TCP RST packet         # Filtered         #
   # ICMP unreachable error # Filtered         #   <--  ICMP type 3, code 1,2,3,9,10,13
```
- Open Port
```bash
    # Scanning a open port, not filtered. Pot 22. Device is a Router
    nmap -p22 -sA 192.168.1.1 -Pn
    PORT   STATE         SERVICE
    22/tcp open|filtered ssh
   
    # RST is received as port is not filtered but open
    00:41:54.075699 IP 192.168.1.61.61169 > 192.168.1.1.ssh: Flags [F.], seq 0, ack 1929847251, win 1024, length 0
    00:41:54.177003 IP 192.168.1.61.61171 > 192.168.1.1.ssh: Flags [F.], seq 0, ack 1929716177, win 1024, length 0
```

- Closed Port
```bash
    # Scanning a closed port, on Windows10 machine
    nmap -p25 -sM 192.168.1.41 -Pn
    PORT   STATE  SERVICE
    25/tcp closed smtp
   
    # RST is received as port is not filtered but open
    00:43:51.093734 IP 192.168.1.61.56250 > 192.168.1.41.smtp: Flags [F.], seq 0, ack 1859801841, win 1024, length 0
    00:43:51.096878 IP 192.168.1.41.smtp > 192.168.1.61.56250: Flags [R], seq 1859801841, win 0, length 0
```
### TCP Iddle Scan (-sI)
---
Scan a target without sending any packet from the attacker machine to the target host. This scan relies in that every IP packet on internet has a fragment identification number (IP ID) and many operating systems increment this number for each packet sent. The attack can detect a open, closer or filtered port, and the process works as follows for each case:

- Open Port
```markup
    # Step 1: Probe the Zombie's IP ID
    ATTACKER ------> SYN/ACK ------------> ZOMBIE
    ZOMBIE --------> RST + IPID = X -----> ATTACKER
    #Step 2: Attacker sends a forged a SYN spoofing with zombie's IP as the source address to target
             The Zombie, not expecting this SYN/ACK, responds with a RST, increasing it's IP ID
    ATTACKER ------> Spoofed SYN --------> TARGET
    TARGET --------> SYN/ACK ------------> ZOMBIE
    ZOMBIE --------> RST + IPID = X+1 ---> TARGET
    #Step 3: Attacker probes zombine again for the IP ID
             IP ID increased in 2 units, then port it's open
    ATTACKER ------> SYN/ACK ------------> ZOMBIE
    ZOMBIE --------> RST + IPID= X+2 ----> ATTACKER
```
```bash
   TBD
```

- Closed Port
```markup
    # Step 1: Probe the Zombie's IP ID
    ATTACKER ------> SYN/ACK ------------> ZOMBIE
    ZOMBIE --------> RST + IPID = X -----> ATTACKER
    #Step 2: Attacker sends a forged a SYN spoofing with zombie's IP as the source address to target
             As the Zombie receives a RST, does not change it's IPID as there is no forged packet
    ATTACKER ------> Spoofed SYN --------> TARGET
    TARGET --------> RST ------------> ZOMBIE
    #Step 3: Attacker probes zombine again for the IP ID
             Only increased in 1, then port is closed
    ATTACKER ------> SYN/ACK ------------> ZOMBIE
    ZOMBIE --------> RST + IPID= X+1 ----> ATTACKER
```
```bash
   TBD
```

- Filtered Port
```markup
    # Step 1: Probe the Zombie's IP ID
    ATTACKER ------> SYN/ACK ------------> ZOMBIE
    ZOMBIE --------> RST + IPID = X -----> ATTACKER
    #Step 2: Attacker sends a forged a SYN spoofing with zombie's IP as the source address to target
             As the Zombie receives a RST, does not change it's IPID as there is no forged packet
    ATTACKER ------> Spoofed SYN --------> TARGET
    #Step 3: Attacker probes zombine again for the IP ID
             Only increased in 1, then port is closed
    ATTACKER ------> SYN/ACK ------------> ZOMBIE
    ZOMBIE --------> RST + IPID= X+1 ----> ATTACKER
```
```bash
   TBD
```
The difficulty of this attack, t's to find the right Zombie's host. Usually, hosts that implement a basic network stacks, are vulnerable to IP ID traffic detection. Also, there is a NSE Script called `ipidseq.nse` to help identify zombie candidates. This scripts probes a host to classify it's IP ID generation method:
```bash
   # Find random vulnerable hosts
   nmap -p80 --script ipidseq -iR 1000
```

When the Zombie is not valid, usually will get the following error:
```bash
   …cannot be used because it has not returned any of our probes — perhaps it is down or firewalled.
   QUITTING!”
```

### FTP Bounce Scan (-b)
---
In the FTP RFT 959, sates about FTP servers to proxy FTP connections, for an user to connect to a FTP server and then ask files to be sent to a 3th party server. This can be abused to cause thr FTP server to scan other hosts ports, simply asking the FTP proxy server to send files to each port of the target host. Actually, FTP servers removed this feature because the abuse it can imply.

```bash
    nmap -Pn -b ftp.vulnerable.com target_host  <-- Default credentials anonymous with password wwwuser@
    nmap -Pn -b user:pass@ftp.vulnerable.com target_host
```

### TCP Protocol Scan (-sO)
---
To determine which IP protocols are supported by target hosts. It cycles, by default, throught the 256 protocol numbers to check. The `-p` option can be used to select the protocol number to be scaned instead the port (this scan does not scan ports ;-) ).

```markup
   # Probe Response                                 # Status                #
   #------------------------------------------------#-----------------------#
   # Any response in any protocol from target host  # Open                  #
   # ICMP protocol unreachable erro                 # Closed                #  <--  ICMP type 3, code 2
   # Other ICMP unreachable errors                  # Filtered              #  <--  ICMP type 3, code 1,3,9,10,13
   # No response received                           # Open|Filtered         #   
```

Check IP Protocol numbers at [https://en.wikipedia.org/wiki/List_of_IP_protocol_numbers](https://en.wikipedia.org/wiki/List_of_IP_protocol_numbers)
```bash
    # Windows 10 Host
    nmap -sO 192.168.1.47 -p1
    PROTOCOL STATE SERVICE
    1        open  icmp
```
```bash
    18:54:36.710519 IP 192.168.1.61 > 192.168.1.47: ICMP echo request, id 643, seq 0, length 8
    18:54:36.711205 IP 192.168.1.47 > 192.168.1.61: ICMP echo reply, id 643, seq 0, length 8
```
```bash
    # Router
    nmap -sO 192.168.1.1 -p1,6,17
    PROTOCOL STATE         SERVICE
    1        open          icmp
    6        open          tcp
    17       open|filtered udp
```
```bash
    18:57:48.906290 IP 192.168.1.61.46371 > 192.168.1.1.40125: UDP, length 0
    18:57:48.906330 IP 192.168.1.61 > 192.168.1.1: ICMP echo request, id 13828, seq 0, length 8
    18:57:48.906343 IP 192.168.1.61.46371 > 192.168.1.1.http: Flags [.], ack 2171228364, win 1024, length 0
    18:57:48.919569 IP 192.168.1.1.http > 192.168.1.61.46371: Flags [R], seq 2171228364, win 0, length 0
    18:57:48.919896 IP 192.168.1.1 > 192.168.1.61: ICMP echo reply, id 13828, seq 0, length 8
    18:57:50.011378 IP 192.168.1.61.46373 > 192.168.1.1.40125: UDP, length 0
    18:57:50.479656 IP 192.168.1.1 > 192.168.1.61: ICMP echo request, id 17888, seq 0, length 64
    18:57:50.479695 IP 192.168.1.61 > 192.168.1.1: ICMP echo reply, id 17888, seq 0, length 64
    18:57:58.658836 IP 192.168.1.47.54043 > 192.168.1.61.2054: UDP, length 28
    18:57:58.658938 IP 192.168.1.61 > 192.168.1.47: ICMP 192.168.1.61 udp port 2054 unreachable, length 64
```

### Custom SYN/FIN Scan (--scanflags SYNFIN)
---
Sometimes firewalls only block connections for incoming packets with the SYN flag set only, not the SYN/ACK for in case it's comming from a related connection originated from the target host. Some hosts 
```bash
    nmap -sS --scanflags SYNFIN TARGET 
```
### Custom PSH Scan (--scanflags PSH)
---
```bash
    nmap -sF --scanflags PSH TARGET
```
