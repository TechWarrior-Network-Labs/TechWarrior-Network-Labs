# Tech Warrior Lab – [VLAN + TRUNK + SVI]
> Built and documented by Charly Tech Warrior

---

## Objective
> Demonstrate VLAN segmentation and trunking across two switches


## Topology
![Topology Diagram](VLAN-TRUNK-SVI.drawio.png)

---

Configuration Summary Best Practices Examples
 
  conf t
interface vlan 1
 shutdown
 conf t
vlan 99
 name MANAGEMENT/NATIVE
exit

interface g0/0
 switchport trunk allowed vlan remove 1
 switchport trunk allowed vlan add 99
interface vlan 99
 ip address 192.168.99.1 255.255.255.0
 no shutdown
 
 conf t
interface g0/0
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,99
 
If the trunk connects to:

another switch

a router-on-a-stick

a firewall

a L3 switch

…you must configure the same exact native VLAN on the other side.
interface g0/1
 switchport trunk native vlan 99
 switchport trunk allowed vlan 10,20,30,99
 
 3. Important Notes

Native VLAN is untagged traffic on a trunk.

If you make VLAN 99 both management and native, it’s common but not the most secure — modern best practice is to make the native VLAN an unused VLAN (like 999).

But for labs, using VLAN 99 as native + management is totally fine.

---
