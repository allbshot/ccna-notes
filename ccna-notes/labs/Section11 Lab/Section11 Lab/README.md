# Lab 1: Basic VLAN Configuration

## Goal
Set up two VLANs on a switch and verify connectivity.

## Topology
(Add a screenshot or exported image from Packet Tracer here, e.g. `topology.png`)

## Steps
1. Create VLAN 10 (Sales) and VLAN 20 (Engineering)
2. Assign switch ports to each VLAN
3. Verify with `show vlan brief`

## Switch Config

```
enable
configure terminal
vlan 10
 name Sales
vlan 20
 name Engineering
interface range fa0/1-2
 switchport mode access
 switchport access vlan 10
interface range fa0/3-4
 switchport mode access
 switchport access vlan 20
end
write memory
```

## What I Learned
- (Write a sentence or two here — this is more useful for review later than the config itself)
