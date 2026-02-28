# Dynamic Routing Lab (OSPF Example)

While static routes are easy to configure, they do not scale to larger networks.  Dynamic routing protocols such as **OSPF (Open Shortest Path First)** automatically learn routes and adapt to topology changes.  This lab introduces OSPF in a simple three‑router environment.

## Topology

Three routers (R1, R2 and R3) form a triangle.  Each router is also connected to a directly attached LAN with a PC.  OSPF will be used on the router links to exchange route information.

| Device | Interface | IP address | Network |
| --- | --- | --- | --- |
| **R1** | Gig0/0 | 10.0.12.1/30 | Link to R2 |
| **R1** | Gig0/1 | 10.0.13.1/30 | Link to R3 |
| **R1** | Gig0/2 | 192.168.1.1/24 | LAN1 |
| **R2** | Gig0/0 | 10.0.12.2/30 | Link to R1 |
| **R2** | Gig0/1 | 10.0.23.2/30 | Link to R3 |
| **R2** | Gig0/2 | 192.168.2.1/24 | LAN2 |
| **R3** | Gig0/0 | 10.0.13.2/30 | Link to R1 |
| **R3** | Gig0/1 | 10.0.23.3/30 | Link to R2 |
| **R3** | Gig0/2 | 192.168.3.1/24 | LAN3 |
| **PC1** | NIC | 192.168.1.10/24 | LAN1 |
| **PC2** | NIC | 192.168.2.10/24 | LAN2 |
| **PC3** | NIC | 192.168.3.10/24 | LAN3 |

### Mermaid diagram

```mermaid
graph TB
    subgraph Router_R1
        R1_LAN((LAN1 192.168.1.0/24))
        R1--10.0.12.1/30--R1_R2
        R1--10.0.13.1/30--R1_R3
    end
    subgraph Router_R2
        R2_LAN((LAN2 192.168.2.0/24))
        R2--10.0.12.2/30--R1_R2
        R2--10.0.23.2/30--R2_R3
    end
    subgraph Router_R3
        R3_LAN((LAN3 192.168.3.0/24))
        R3--10.0.13.2/30--R1_R3
        R3--10.0.23.3/30--R2_R3
    end
    PC1((PC1<br/>192.168.1.10)) -- LAN1 --> R1_LAN
    PC2((PC2<br/>192.168.2.10)) -- LAN2 --> R2_LAN
    PC3((PC3<br/>192.168.3.10)) -- LAN3 --> R3_LAN
    R1_R2(( )) -- link --> R2
    R1_R3(( )) -- link --> R3
    R2_R3(( )) -- link --> R3
```

## Tasks

1. **Build the topology.**  Place three routers and three PCs.  Connect the routers in a triangle using serial or Gigabit interfaces.  Connect each router to its own PC on a separate LAN.
2. **Assign IP addresses.**  Configure the interfaces with the IP addresses from the table.  Assign default gateways to the PCs (e.g. PC1 uses `192.168.1.1`).  Verify local connectivity with `ping`.
3. **Enable OSPF.**  On each router, enable OSPF process 1 and advertise the connected networks.  OSPF network statements use **wildcard masks** (the inverse of the subnet mask) to identify which interfaces belong to the routing process; for example, a `/30` subnet (`255.255.255.252`) has a wildcard mask of `0.0.0.3`【801102245745170†L112-L123】.  Assign a router ID (RID) on each router.  For example, on R1:

   ```plaintext
   R1(config)# router ospf 1
   R1(config-router)# router-id 1.1.1.1
   R1(config-router)# network 10.0.12.0 0.0.0.3 area 0
   R1(config-router)# network 10.0.13.0 0.0.0.3 area 0
   R1(config-router)# network 192.168.1.0 0.0.0.255 area 0
   ```

   Repeat similar network statements on R2 and R3, adjusting the networks and router IDs.
4. **Verify OSPF adjacency.**  Use `show ip ospf neighbor` to ensure the routers form adjacencies.  Use `show ip route` to verify that routes to the remote LANs appear as OSPF routes (`O` code).
5. **Test end‑to‑end connectivity.**  From PC1, ping PC2 and PC3.  The pings should succeed once OSPF is running.  Disconnect a link and observe how OSPF converges to the new path.

## Configuration files

Sample router configurations for R1, R2 and R3 are provided in the [configs](configs/) directory.  They include IP addressing and OSPF network statements.  Use them as a starting point.
