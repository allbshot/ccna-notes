# Phase 1: Base Config and Device Hardening

**Goal:** Give every network device a baseline config: hostname, secured privileged access, local AAA, SSH-only remote access and a login banner. Then address the router interfaces, loopbacks and switch management SVIs.

**Status:** ⬜ Not started / 🟨 In progress / ✅ Complete

**Saved file:** `pkt/capstone-phase1.pkt`

---

## 1. Baseline hardening (all network devices)

I applied this to ISP, EDGE, BR1, DSW1, DSW2, ASW1, ASW2 and BR-SW, changing the hostname each time.

```
enable
configure terminal
hostname EDGE
no ip domain-lookup
ip domain-name lab.local
enable secret Cisco123!
service password-encryption
username admin privilege 15 secret Admin123!
banner motd # Authorized access only #
line console 0
 login local
 logging synchronous
 exec-timeout 10 0
crypto key generate rsa general-keys modulus 2048
ip ssh version 2
line vty 0 4
 login local
 transport input ssh
 exec-timeout 10 0
end
copy running-config startup-config
```

> Switches use `line vty 0 15` instead of `0 4`.

### What each command does

| Command | What it does |
| --- | --- |
| `hostname` | |
| `no ip domain-lookup` | |
| `ip domain-name lab.local` | |
| `enable secret` | |
| `service password-encryption` | |
| `username admin privilege 15 secret` | |
| `banner motd` | |
| `line console 0` → `login local` | |
| `logging synchronous` | |
| `exec-timeout 10 0` | |
| `crypto key generate rsa general-keys modulus 2048` | |
| `ip ssh version 2` | |
| `line vty 0 4` / `0 15` | |
| `transport input ssh` | |
| `copy running-config startup-config` | |

---

## 2. Router interfaces and loopbacks

### ISP

![ISP config](images/isp-config.png)

| Interface | Address | Connects to |
| --- | --- | --- |
| G0/0 | 203.0.113.1/30 | EDGE G0/0 |
| G0/1 | 8.8.8.1/24 | WEB-SRV |

**Notes:**

---

### EDGE

![EDGE config](images/edge-config.png)

| Interface | Address | Connects to |
| --- | --- | --- |
| Lo0 | 1.1.1.1/32 | OSPF router ID (Phase 4) |
| G0/0 | 203.0.113.2/30 | ISP |
| G0/1 | 10.0.0.1/30 | DSW1 |
| G0/2 | 10.0.0.5/30 | DSW2 |
| S0/0/0 | 10.0.0.9/30 | BR1 (EDGE is DCE) |

| Command | What it does |
| --- | --- |
| `interface loopback0` | |
| `no shutdown` | |
| `clock rate 64000` | |

**Notes:**

---

### BR1

![BR1 config](images/br1-config.png)

| Interface | Address | Connects to |
| --- | --- | --- |
| Lo0 | 4.4.4.4/32 | OSPF router ID (Phase 4) |
| S0/0/0 | 10.0.0.10/30 | EDGE (DTE end) |

**Notes:**

---

## 3. Switch management SVIs

```
vlan 99
 name MGMT
interface vlan 99
 ip address 10.10.99.11 255.255.255.0
 no shutdown
ip default-gateway 10.10.99.1
```

| Command | What it does |
| --- | --- |
| `vlan 99` / `name MGMT` | |
| `interface vlan 99` | |
| `ip default-gateway` | |

### ASW1 (10.10.99.11)

![ASW1 config](images/asw1-config.png)

**Notes:**

### ASW2 (10.10.99.12)

![ASW2 config](images/asw2-config.png)

**Notes:**

### BR-SW (10.20.98.11, VLAN 98, gw 10.20.98.1)

![BR-SW config](images/br-sw-config.png)

**Notes:**

### DSW1 / DSW2

![DSW1 config](images/dsw1-config.png)

![DSW2 config](images/dsw2-config.png)

**Notes:**

---

## 4. Servers

| Device | IP / Mask | Gateway | DNS |
| --- | --- | --- | --- |
| WEB-SRV | 8.8.8.8/24 | 8.8.8.1 | none |
| SRV1 | 10.10.40.10/24 | 10.10.40.1 | 10.10.40.10 |

![WEB-SRV IP config](images/web-srv-ip.png)

![SRV1 IP config](images/srv1-ip.png)

---

## 5. Verification

| Command | Device | What I expected | Result |
| --- | --- | --- | --- |
| `show ip interface brief` | All routers | Interfaces up/up with the correct IPs | |
| `show controllers s0/0/0` | EDGE | DCE cable, clock rate 64000 | |
| `ping 10.0.0.10` | EDGE | Serial link to BR1 works | |
| `ping 203.0.113.1` | EDGE | ISP link works | |
| `show ip ssh` | All | SSH enabled, version 2.0 | |
| `ssh -l admin <ip>` | Any | Login works with local credentials | |

![show ip interface brief on EDGE](images/edge-show-ip-int-brief.png)

![show controllers s0/0/0](images/edge-show-controllers.png)

![Ping across serial link](images/edge-ping-br1.png)

![show ip ssh](images/show-ip-ssh.png)

---

## 6. Problems I hit and how I fixed them

| Problem | Cause | Fix |
| --- | --- | --- |
| | | |

---

## 7. What I learned

-
-
-

---

## Config files in this folder

- [ ] `ISP.txt`
- [ ] `EDGE.txt`
- [ ] `BR1.txt`
- [ ] `DSW1.txt`
- [ ] `DSW2.txt`
- [ ] `ASW1.txt`
- [ ] `ASW2.txt`
- [ ] `BR-SW.txt`
