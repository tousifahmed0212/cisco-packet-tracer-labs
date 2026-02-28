# Inter‑VLAN Routing (Router‑on‑a‑Stick) Lab

This lab extends the VLAN trunking scenario by adding a router to provide inter‑VLAN connectivity.  When multiple VLANs exist on a Layer‑2 switch, devices in different VLANs cannot communicate without a Layer‑3 device to route traffic between them.  A common solution in small networks is a **router‑on‑a‑stick**, where a single physical interface on a router is configured with multiple sub‑interfaces, each carrying traffic for one VLAN.

## Topology

| Device | Interface | IP address | Description |
| --- | --- | --- | --- |
| **PC1** | NIC | 192.168.10.10/24 | Host in VLAN 10 |
| **PC2** | NIC | 192.168.20.10/24 | Host in VLAN 20 |
| **S1** | Fa0/1 | Access port for PC1 | VLAN 10 |
| **S1** | Fa0/2 | Access port for PC2 | VLAN 20 |
| **S1** | Gig0/1 | Trunk to router R1 | VLANs 10 & 20 |
| **R1** | Gig0/0.10 | 192.168.10.1/24 | Sub‑interface for VLAN 10 |
| **R1** | Gig0/0.20 | 192.168.20.1/24 | Sub‑interface for VLAN 20 |

### Mermaid diagram

```mermaid
graph LR
    PC1((PC1<br/>192.168.10.10/24)) -- Fa0/1 → S1[[Switch S1]]
    PC2((PC2<br/>192.168.20.10/24)) -- Fa0/2 → S1
    S1 -- Trunk (VLAN 10,20) → R1_G0((Router R1 G0/0
sub‑interfaces .10/.20))
    R1_G0 -- .10 → VLAN10_GW((Gateway 192.168.10.1))
    R1_G0 -- .20 → VLAN20_GW((Gateway 192.168.20.1))
```

## Tasks

1. **Build the topology.**  Place one switch (S1), one router (R1) and two PCs.  Connect each PC to S1 (Fa0/1 and Fa0/2).  Connect S1 Gig0/1 to R1 Gig0/0 using a straight‑through cable.
2. **Create VLANs and assign ports.**  On S1, create VLAN 10 and VLAN 20 and assign Fa0/1 and Fa0/2 to these VLANs respectively.  See the VLAN trunk lab for the exact commands【45094192213535†L235-L273】.
3. **Configure the trunk link.**  Set S1’s Gig0/1 interface to trunk mode and allow VLANs 10 and 20.  On the router, no special trunking commands are required because the router will recognise 802.1Q tags on sub‑interfaces.  Example for S1:

   ```plaintext
   S1(config)# interface GigabitEthernet0/1
   S1(config-if)# switchport trunk encapsulation dot1q
   S1(config-if)# switchport mode trunk
   S1(config-if)# switchport trunk allowed vlan 10,20
   ```

4. **Configure router sub‑interfaces.**  On R1, create a sub‑interface for each VLAN under the physical Gig0/0 interface.  Use 802.1Q encapsulation and assign an IP address (which will act as the default gateway for that VLAN):

   ```plaintext
   R1(config)# interface GigabitEthernet0/0
   R1(config-if)# no shutdown
   R1(config-if)# exit
   
   ! Sub-interface for VLAN 10
   R1(config)# interface GigabitEthernet0/0.10
   R1(config-subif)# encapsulation dot1Q 10
   R1(config-subif)# ip address 192.168.10.1 255.255.255.0
   R1(config-subif)# exit

   ! Sub-interface for VLAN 20
   R1(config)# interface GigabitEthernet0/0.20
   R1(config-subif)# encapsulation dot1Q 20
   R1(config-subif)# ip address 192.168.20.1 255.255.255.0
   R1(config-subif)# exit
   ```

5. **Set default gateways.**  Configure PC1 to use `192.168.10.1` as its default gateway and PC2 to use `192.168.20.1`.
6. **Test connectivity.**  Ping from PC1 to PC2.  The router should route traffic between VLANs.  Use `show ip interface brief` on the router to verify sub‑interface status.

## Configuration files

The [configs](configs/) folder contains example configurations for S1 and R1.  Apply them to save time when building the lab.
