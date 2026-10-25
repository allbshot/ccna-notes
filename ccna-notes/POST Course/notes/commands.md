# CCNA Command Cheat Sheet

Config on the left, verification on the right. Add commands as you meet them in labs and Boson explanations.

---

## Basic device setup

```
hostname R1
enable secret <password>
service password-encryption
no ip domain-lookup
banner motd # Authorized access only #
line console 0
 logging synchronous
 exec-timeout 0 0
```
Verify: `show running-config` · `show version` · `show startup-config`
Save: `copy running-config startup-config` (or `wr`)

## SSH

```
ip domain-name lab.local
username admin privilege 15 secret <password>
crypto key generate rsa modulus 2048
ip ssh version 2
line vty 0 15
 login local
 transport input ssh
```
Verify: `show ip ssh` · `show ssh` · `show users`

---

## Network Access

### VLANs and access ports
```
vlan 10
 name USERS
interface fa0/1
 switchport mode access
 switchport access vlan 10
```
Verify: `show vlan brief` · `show interfaces fa0/1 switchport`

### Trunks
```
interface fa0/24
 switchport trunk encapsulation dot1q   ! only on switches that support ISL (e.g., 3550)
 switchport mode trunk
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,99
 switchport nonegotiate
```
Verify: `show interfaces trunk` · `show interfaces fa0/24 switchport`

### STP
```
spanning-tree mode rapid-pvst
spanning-tree vlan 10 root primary
spanning-tree vlan 10 priority 4096
interface fa0/1
 spanning-tree portfast
 spanning-tree bpduguard enable
```
Verify: `show spanning-tree` · `show spanning-tree vlan 10` · `show spanning-tree summary`

### EtherChannel (LACP)
```
interface range fa0/23 - 24
 channel-group 1 mode active
interface port-channel 1
 switchport mode trunk
```
Verify: `show etherchannel summary` · `show etherchannel port-channel`

### CDP / LLDP
```
cdp run
lldp run
```
Verify: `show cdp neighbors detail` · `show lldp neighbors detail`

### Inter-VLAN routing — router-on-a-stick
```
interface g0/0
 no shutdown
interface g0/0.10
 encapsulation dot1Q 10
 ip address 192.168.10.1 255.255.255.0
```
Verify: `show ip interface brief` · `show ip route connected`

### Inter-VLAN routing — SVIs (L3 switch)
```
ip routing
interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
```
Verify: `show ip interface brief` · `show ip route`

---

## IP Connectivity

### Static routes
```
ip route 10.0.0.0 255.0.0.0 192.168.1.2          ! next hop
ip route 0.0.0.0 0.0.0.0 192.168.1.2             ! default
ip route 10.0.0.0 255.0.0.0 192.168.2.2 150      ! floating static (AD 150)
ipv6 unicast-routing
ipv6 route 2001:db8:2::/64 2001:db8:1::2
```
Verify: `show ip route static` · `show ipv6 route` · `ping` · `traceroute`

### OSPFv2 (single area)
```
router ospf 1
 router-id 1.1.1.1
 network 192.168.1.0 0.0.0.255 area 0
 passive-interface g0/1
 default-information originate
interface g0/0
 ip ospf cost 10
 ip ospf priority 100
```
Verify: `show ip ospf neighbor` · `show ip ospf interface brief` · `show ip protocols` · `show ip route ospf`

### HSRP
```
interface g0/0
 standby 1 ip 192.168.10.254
 standby 1 priority 110
 standby 1 preempt
```
Verify: `show standby brief`

### Administrative distance (memorize)
| Source | AD |
|--------|----|
| Connected | 0 |
| Static | 1 |
| eBGP | 20 |
| EIGRP | 90 |
| OSPF | 110 |
| RIP | 120 |
| iBGP | 200 |

---

## IP Services

### NAT / PAT
```
interface g0/0
 ip nat inside
interface g0/1
 ip nat outside
access-list 1 permit 192.168.10.0 0.0.0.255
ip nat inside source list 1 interface g0/1 overload          ! PAT
ip nat inside source static 192.168.10.5 203.0.113.5         ! static NAT
```
Verify: `show ip nat translations` · `show ip nat statistics`

### DHCP server and relay
```
ip dhcp excluded-address 192.168.10.1 192.168.10.10
ip dhcp pool USERS
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 8.8.8.8
interface g0/0.20
 ip helper-address 10.1.1.1      ! relay
```
Verify: `show ip dhcp binding` · `show ip dhcp pool`

### NTP, syslog, SNMP
```
ntp server 10.1.1.1
logging host 10.1.1.50
logging trap warnings
snmp-server community <string> RO
```
Verify: `show ntp status` · `show ntp associations` · `show logging` · `show clock detail`

---

## Security Fundamentals

### ACLs
```
access-list 10 permit 192.168.10.0 0.0.0.255                          ! standard: place near destination
ip access-list extended BLOCK-WEB
 deny tcp 192.168.10.0 0.0.0.255 any eq 80
 permit ip any any
interface g0/0
 ip access-group BLOCK-WEB in                                          ! extended: place near source
```
Verify: `show access-lists` · `show ip interface g0/0`

### Port security
```
interface fa0/1
 switchport mode access
 switchport port-security
 switchport port-security maximum 2
 switchport port-security mac-address sticky
 switchport port-security violation restrict
```
Verify: `show port-security` · `show port-security interface fa0/1`
Recover err-disabled: `shutdown` then `no shutdown`

### DHCP snooping + DAI
```
ip dhcp snooping
ip dhcp snooping vlan 10
interface fa0/24
 ip dhcp snooping trust
ip arp inspection vlan 10
interface fa0/24
 ip arp inspection trust
```
Verify: `show ip dhcp snooping binding` · `show ip arp inspection`

---

## Troubleshooting go-to commands

| Problem | First commands |
|---------|----------------|
| Interface down | `show ip interface brief` · `show interfaces` |
| Can't reach another VLAN | `show vlan brief` · `show interfaces trunk` |
| Route missing | `show ip route` · `show ip protocols` |
| OSPF neighbor not forming | `show ip ospf neighbor` · `show ip ospf interface` (check area, timers, subnet, passive) |
| ACL blocking traffic | `show access-lists` (check hit counters) |
| NAT not translating | `show ip nat translations` · check inside/outside on interfaces |
