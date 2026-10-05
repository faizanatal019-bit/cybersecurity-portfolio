# CCNA Lab Series – Switch, Router, and Network Fundamentals (Labs 1–6)

Hands-on labs completed as part of COMP1016 (Foundations of Networking & Cyber Security), using physical Cisco switches/routers, console cables, and Tera Term. Progression: basic device config → traffic analysis → subnet design → secure remote access.

---

## Lab 1 – Basic Switch and End Device Configuration
Configured a switch and two PCs with static IP addressing on the same subnet, set up console/enable passwords, and verified end-to-end connectivity.

```
Switch(config)#hostname S1
S1(config)#enable secret class
S1(config)#line console 0
S1(config-line)#password cisco
S1(config-line)#login
S1(config)#interface vlan 1
S1(config-if)#ip address 192.168.1.1 255.255.255.0
S1(config-if)#no shutdown
```

**Result:** Both PCs could ping each other and the switch once addressing and interfaces were correctly brought up.

---

## Lab 2 – Viewing the Switch MAC Address Table
Explored how a switch learns and stores MAC addresses dynamically.

```
S1#show mac address-table
S1#clear mac address-table dynamic
```

Compared table entries against each device's real NIC address (`ipconfig /all`) and the ARP cache (`arp -a`) before and after pings. Hit a duplicate-IP conflict mid-lab — the switch logged `%IP-4-DUPADDR`, which confirmed both the error-detection behaviour of IOS and the importance of coordinating addressing in a shared lab environment.

---

## Lab 3 – Examining Ethernet Frames with Wireshark
Captured live ICMP traffic to compare Layer 2 (MAC) vs Layer 3 (IP) addressing behaviour.

```
C:\Users\student>ping 192.168.254.1       (local gateway)
C:\Users\student>ping www.cisco.com       (remote host)
```

**Finding:** pinging the gateway showed matching destination MAC and IP. Pinging a remote host (resolved to 23.221.132.98) kept the same destination MAC (still the gateway) while the destination IP changed to the remote server — concrete proof that a PC only ever talks directly to its default gateway at Layer 2; everything beyond that is routed.

---

## Lab 4 – VLSM Subnetting and Router Configuration
Designed a VLSM scheme splitting a 192.168.33.128/25 block into six right-sized subnets, then implemented it on physical routers (BR1, BR2).

| Subnet | Hosts Needed | CIDR | Network |
|---|---|---|---|
| BR1 LAN | 40 | /26 | .128 |
| BR2 LAN | 25 | /27 | .192 |
| BR2 IoT | 5 | /29 | .224 |
| BR2 CCTV | 4 | /29 | .232 |
| BR2 HVAC/C2 | 4 | /29 | .240 |
| BR1–BR2 Link | 2 | /30 | .248 |

```
BR1(config)#hostname BR1
BR1(config)#no ip domain-lookup
BR1(config)#enable secret class
BR1(config)#interface g0/0/0
BR1(config-if)#description Link to BR2
BR1(config-if)#ip address 192.168.33.249 255.255.255.252
BR1(config-if)#no shutdown
BR1(config)#interface g0/0/1
BR1(config-if)#description BR1 LAN
BR1(config-if)#ip address 192.168.33.129 255.255.255.192
BR1(config-if)#no shutdown
```

```
BR1#ping 192.168.33.250
Success rate is 80 percent (4/5), round-trip min/avg/max = 1/40/158 ms
```

Link confirmed live between BR1 and BR2 — single dropped packet consistent with normal first-ping ARP delay, not a fault.

---

## Lab 5 – Building a Switch and Router Network
Built a two-subnet topology with a router separating two LANs (one via direct connection, one through a switch), configuring IPv4 and IPv6 on router interfaces.

```
R1(config)#interface g0/0/0
R1(config-if)#ip address 192.168.0.1 255.255.255.0
R1(config-if)#no shutdown
R1(config)#interface g0/0/1
R1(config-if)#ip address 192.168.1.1 255.255.255.0
R1(config-if)#no shutdown
R1(config)#ipv6 unicast-routing
```

Verified routing behaviour directly with `show ip route` and `show ip interface brief`, and worked through real connectivity failures (interface status, PC IP config, Windows Firewall) to isolate why initial pings between subnets failed before the router was configured.

---

## Lab 6 – Configuring Network Devices with SSH
Replaced insecure Telnet access with SSH on both router and switch.

```
R1(config)#ip domain-name cisco.com
R1(config)#crypto key generate rsa
% Choose a key modulus: 2048
R1(config)#username admin password Adm1nP@55
R1(config)#line vty 0 4
R1(config-line)#transport input ssh telnet
R1(config-line)#login local
```

Verified secure access both from a PC (Tera Term SSH client) and device-to-device:

```
S1#ssh -l admin 192.168.1.1
Password:
R1>
```

Troubleshot an SSH timeout by checking `show ip ssh`, confirming VTY `transport input` settings, and testing with Telnet as a comparison — isolating the issue to incomplete SSH configuration on one device rather than a network-level block.

---

## Skills Demonstrated
- Cisco IOS CLI configuration (console and remote)
- Static IP addressing and VLSM subnet design
- Layer 2 vs Layer 3 traffic analysis (Wireshark)
- Secure remote access (SSH, RSA keys, local authentication)
- Systematic troubleshooting: interface status checks, staged pings, duplicate IP resolution, SSH/VTY misconfiguration isolation

## Screenshots
*(Insert: MAC address table output, Wireshark gateway-vs-remote comparison, VLSM ping verification on BR1, SSH session established from S1 to R1)*
