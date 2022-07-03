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
