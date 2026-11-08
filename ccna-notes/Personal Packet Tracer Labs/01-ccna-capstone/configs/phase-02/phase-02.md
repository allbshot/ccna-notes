# DSW1

![alt text](image.png)

- What it does:

vtp mode transparent makes the switch keep its own VLAN database. It ignores VTP updates from other switches but still forwards them. In server or client mode, one wrong update with a higher revision number can wipe every switch’s VLANs. Transparent mode prevents that.
The vlan commands create the HQ VLANs. Each VLAN is a separate broadcast domain with its own subnet. 666 (BLACKHOLE) is a dead end for unused ports, and 999 (NATIVE) is a dedicated VLAN for untagged trunk traffic.
spanning-tree mode rapid-pvst changes STP from the default PVST+ to Rapid PVST+, Cisco’s per-VLAN version of 802.1w. It converges in seconds instead of 30–50, and each VLAN can have a different root bridge.

![alt text](image-1.png)

- What it does:

switchport mode trunk makes the port an 802.1Q trunk, carrying many VLANs on one link by adding a 4-byte tag to each frame. The 3650 only supports dot1q, so you don’t choose an encapsulation.
native vlan 999 moves untagged traffic onto an unused VLAN. That defends against VLAN-hopping (double-tagging) attacks. Both ends must match, or CDP reports a mismatch.
allowed vlan 10,20,30,40,99 is manual pruning: only the production VLANs can cross the trunk.
nonegotiate turns off DTP. Otherwise an attacker could negotiate a trunk with your switch.
channel-group 1 mode active bundles G1/0/23 and G1/0/24 into Port-channel 1 using LACP. STP sees one logical link, so neither physical link gets blocked and you get double the bandwidth. LACP modes are active and passive. Active/active and active/passive work; passive/passive never forms. PAgP, Cisco’s protocol, uses desirable and auto.
The trunk settings are on the member ports first because every member of an EtherChannel must match exactly. A member that doesn’t match gets suspended.
Port-channel 1 is the logical interface. It gets the same trunk settings, because in IOS the port-channel config governs the members.

![alt text](image-2.png)

- What it does: 

G1/0/2 goes to ASW1 and G1/0/3 goes to ASW2. These are hard-coded trunks with the same settings as Po1. VLANs 666 and 999 are left off the allowed list on purpose, because nothing should travel in them.

Block D: root bridge placement (enter on DSW1)

![alt text](image-3.png)

- What it does:

root primary sets DSW1’s priority to 24576 for VLANs 10, 30 and 99, so it becomes their root bridge. The lowest bridge ID wins (priority first, then MAC address). The default priority is 32768 plus the VLAN number.
root secondary sets 28672 for VLANs 20 and 40, so DSW1 is the backup root if DSW2 fails.
Why these VLANs: DSW1 will also be the HSRP active gateway for 10, 30 and 99 in Phase 3. Keeping the STP root and the gateway on the same switch means traffic goes straight up to its gateway, not across Po1 first.

![alt text](image-4.png)

- What it does: 

shuts down every unused port and parks it in BLACKHOLE. A port that gets re-enabled by accident still lands somewhere harmless. G1/0/1 (the uplink to EDGE) is skipped on purpose. It becomes a routed port in Phase 3.


# DSW2

![alt text](image-5.png)

What it does: 

the same as DSW1. In transparent mode nothing syncs, so every switch needs its own copy of the VLANs.

![alt text](image-6.png)

- What it does: 

the other half of Po1. Once both sides are active, LACP negotiates and the bundle comes up. Allow about 30 seconds, or use Fast Forward Time.

![alt text](image-7.png)

- forgot to add link between DSW1 and ASW2

![alt text](image-8.png)

- What it does: 

DSW2’s links to ASW1 (G1/0/2) and ASW2 (G1/0/3). These second uplinks create the redundant loops that STP has to block.

![alt text](image-9.png)

- What it does: 

the mirror image of DSW1. DSW2 is root for VLANs 20 and 40, matching its future HSRP active role, and the backup root for 10, 30 and 99. If either distribution switch dies, the other becomes root for every VLAN.

![alt text](image-10.png)

- What it does: 

the same port hygiene as DSW1.

# ASW1

![alt text](image-11.png)

- What it does: 

the same VLAN database as the DSWs. VLAN 99 already exists from Phase 1, and re-entering it is harmless. An access switch needs every VLAN its trunks carry, not only the VLANs on its own ports. If a VLAN doesn’t exist locally, the switch drops that VLAN’s frames.

![alt text](image-12.png)

- What it does: 

G0/1 goes to DSW1 and G0/2 goes to DSW2. These settings match the DSW side exactly, so the native VLAN mismatch messages stop once this is in.

![alt text](image-13.png)

What it does:

Fa0/1 goes to PC-STAFF (VLAN 10), and Fa0/2 goes to PC-SALES (VLAN 20).
switchport mode access hard-codes the port as an access port, so it never negotiates a trunk.
spanning-tree portfast skips the listening and learning states and goes straight to forwarding, so PCs don’t wait for STP. Use it only on end-host ports.
spanning-tree bpduguard enable err-disables the port if it receives a BPDU, which stops a rogue switch on a user port. You may see a PortFast warning; it’s informational.

![alt text](image-14.png)

- What it does: 

parks and shuts down Fa0/3–24. Both gigabit ports are uplinks, so this covers every remaining port.


# ASW2

![alt text](image-15.png)

- What it does: 

the same as ASW1.

![alt text](image-16.png)

- What it does: 

G0/1 goes to DSW1’s G1/0/3 and G0/2 goes to DSW2’s G1/0/3, with settings that match the DSW side.

![alt text](image-17.png)

- What it does: 

Fa0/1 goes to PC-GUEST (VLAN 30), Fa0/2 goes to SRV1 (VLAN 40), and Fa0/24 goes to ADMIN-PC (VLAN 99). ADMIN-PC sits in the management VLAN so it can be the only host allowed to SSH to devices in Phase 8.

![alt text](image-18.png)

- What it does: 

shuts down Fa0/3–23. The range stops at 23 because Fa0/24 is ADMIN-PC.

# BR-SW

![alt text](image-19.png)

- What it does: 

creates the branch’s own VLAN set: users (50), management (98), BLACKHOLE and NATIVE. It gets the same VTP protection and STP mode as HQ.

![alt text](image-20.png)

- What it does: 

this is the trunk BR1 will route over with router-on-a-stick in Phase 3. Native VLAN 999 matches the encapsulation dot1Q 999 native subinterface BR1 gets then. It will show down for now, because BR1’s G0/0 is still shut down.

![alt text](image-21.png)

- What it does: 

Fa0/1 goes to PC-BR in VLAN 50, with PortFast and BPDU guard. Everything else, including the unused G0/2, is shut down in VLAN 666.

# ADMIN-PC 

![alt text](image-22.png)

- What it does: 

gives ADMIN-PC a static management address. The gateway won’t exist until HSRP in Phase 3, but ASW1 and ASW2 are in the same VLAN, so you can already test Layer 2 reachability to them.


# VERIFY

![alt text](image-23.png)

![alt text](image-24.png)

![alt text](image-25.png)

![alt text](image-26.png)

![alt text](image-29.png)

![alt text](image-30.png)

![alt text](image-31.png)

![alt text](image-32.png)

![alt text](image-33.png)

# Fix: rebuild Po1 on both DSW1 and DSW2

![alt text](image-27.png)

![alt text](image-28.png)

- What it does:

no interface port-channel 1 deletes the broken logical interface and its mismatched settings.
default interface range g1/0/23 - 24 wipes both member ports back to factory defaults, so no partial config is left over from the failed paste.
shutdown keeps the ports down while you rebuild them, so LACP doesn’t try to bundle half-configured ports and suspend them again.
Member and Po1 trunk settings must be identical on both members, on Po1, and on the far switch. If they aren’t, IOS suspends the ports. The (s) flag you saw is that check in action.
no shutdown at the end brings both ports up together, already matching, so LACP negotiates cleanly.