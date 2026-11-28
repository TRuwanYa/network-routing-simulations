# OSPF Routing — Configuration and Experimental Analysis

## Overview

This project extends the earlier static routing and RIPv2 simulations by implementing Open Shortest Path First (OSPFv2) in Cisco Packet Tracer. The existing five-router campus network is reused to investigate link-state routing, cost-based path selection, network resilience, hierarchical routing and routing protocol authentication.

The objective is not simply to establish connectivity again, but to examine how OSPF manages routing information and responds to changes in network conditions.

## 1. Baseline: Single-Area OSPF

[Open baseline Packet Tracer configuration](./ospf-baseline.pkt) · [Baseline observations](./observations/)

All five routers initially participate in Area 0 using OSPF process 1. The redundant core connects four departmental LANs and an administration network, with R5 providing the connection toward an external network.

![Baseline routing infrastructure](00-baseline-observations/01%20routing-infrastructure.png)

| Network | Subnet | Router |
|---|---|---|
| CO Department | 10.0.1.0/24 | R1 |
| EE Department | 10.0.2.0/24 | R2 |
| Mechanical Department | 10.0.3.0/24 | R3 |
| Civil Department | 10.0.4.0/24 | R4 |
| Administration | 10.0.5.0/24 | R2 |
| Inter-router links | 10.0.6.0/30, .4/30, .8/30, .12/30, .16/30 | R1–R5 |

### Baseline observations

#### OSPF process and network advertisements

`show ip protocols` confirms process 1, router ID 1.1.1.1, Area 0 network statements and administrative distance 110 on R1.

![OSPF protocol configuration](00-baseline-observations/02%20ospf-protocols.png)

#### Neighbor discovery and adjacency formation

R1 establishes FULL adjacencies with R2, R3 and R5. The serial links use point-to-point OSPF, while the Ethernet link to R5 uses DR/BDR election.

![R1 neighbor table](00-baseline-observations/03%20neighbors-R1.png)

#### Link-state database (LSDB)

R1 and R4 contain the same five Router LSAs and one Network LSA, with matching sequence numbers and checksums in the captured outputs. Their LSA ages differ because the screenshots were taken at different times. This demonstrates a shared view of Area 0 topology.

![R1 LSDB](00-baseline-observations/04%20ospf-database-R1.png)

![R4 LSDB](00-baseline-observations/05%20ospf-database-R4.png)

#### Route calculation and equal-cost multipath

R1 learns remote departmental networks as intra-area routes (`O`). The Civil Department network has two equal-cost next hops (10.0.6.2 and 10.0.6.13), both with cost 129. The Mechanical Department route has cost 65.

![R1 OSPF routing table](00-baseline-observations/06%20routing-table-R1.png)

#### Connectivity verification

Ping tests to 10.0.5.1 and 10.0.4.3 each returned four replies with zero packet loss. Traceroute tests to the Mechanical and EE departments completed successfully.

![Baseline connectivity tests](00-baseline-observations/07%20connectivity-test.png)

These observations establish the reference configuration for the following experiments. External-network reachability is not established by these internal tests.

## 2. Experiments

### Experiment 01 — Cost-Based Routing

[Experiment documentation](./01-cost-based-routing/README.md) · [Packet Tracer configuration](./01-cost-based-routing/ospf-cost-routing.pkt)

The R1-to-R3 interface cost was increased from 64 to 250. OSPF selected the longer R1 → R2 → R4 → R3 path with a cumulative destination-network cost of 193 instead of 251 through the direct link. Traceroute confirmed the path change. This demonstrates why OSPF can prefer a route with more hops, unlike RIPv2's hop-count metric.

### Experiment 02 — Network Resilience

[Experiment documentation](./02-network-resilience/README.md) · [Packet Tracer configuration](./02-network-resilience/ospf-redundant-network.pkt)

Using the modified-cost network, the preferred R1–R2 link was disabled to observe adjacency loss, route recalculation and failover to R1–R3. The route cost changed from 193 to 251. Restoring the link allows the preferred route to be selected again. A continuous ping test recorded 220 successful replies out of 221 packets (99.55% delivery). Exact convergence time was not measured.

### Experiment 03 — Multi-Area OSPF

[Experiment documentation](./03-multi-area-ospf/README.md) · [Packet Tracer configuration](./03-multi-area-ospf/ospf-multi-area.pkt)

The manually modified cost was reset, and departmental LANs were assigned to Areas 10, 20, 30 and 40 while inter-router links remained in Area 0. R1–R4 became Area Border Routers. Inter-area routes (`O IA`), Type 3 Summary LSAs and separate area databases demonstrated OSPF's hierarchical architecture. The small topology illustrates the mechanism rather than measuring scalability gains.

### Experiment 04 — Secure OSPF

[Experiment documentation](./04-secure-ospf/README.md) · [Packet Tracer configuration](./04-secure-ospf/ospf-authentication.pkt)

MD5 authentication was enabled on the R1–R3 link. Matching keys permitted FULL adjacency; mismatched keys caused adjacency loss and traffic rerouting through R2 and R4. Correcting the key restored the direct route with cost 65, and final ping tests achieved zero packet loss. Authentication protects routing message integrity and helps restrict unauthorized participation; it does not encrypt traffic. Only the selected link was authenticated, and MD5 was used for simulator compatibility.

## 3. Progression from RIPv2 to OSPF

| Feature | RIPv2 | OSPFv2 |
|---|---|---|
| Routing approach | Distance-vector | Link-state, SPF algorithm |
| Metric | Hop count | Cumulative interface cost |
| Network hierarchy | Flat | Backbone and multiple areas |
| Routing updates | Periodic and triggered | Event-driven LSA flooding and periodic refresh |
| Maximum usable hop count | 15 | No equivalent 15-hop limit |
| Failure recovery | Dynamic route updates | Adjacency monitoring and SPF recalculation |
| Authentication | Supports authentication, including MD5 | Supports authentication integrated with adjacencies and area/interface configuration |

Both protocols support dynamic recovery, equal-cost routing and authentication. The experiments demonstrate OSPF's more flexible metric and hierarchical design, not that those shared capabilities are unique to OSPF. No controlled RIPv2-versus-OSPF convergence benchmark was performed.



## 4. Reproducing the experiments

Open the corresponding `.pkt` file in Cisco Packet Tracer and inspect the router CLI using `show ip protocols`, `show ip ospf neighbor`, `show ip ospf database`, `show ip ospf interface brief` and `show ip route ospf`. Use `ping` and `tracert` from end devices to verify forwarding. Follow the individual experiment documentation to reproduce configuration changes.

## Conclusion

The simulations demonstrate how OSPF builds a link-state view of a network, selects routes using configurable costs, adapts to failures, supports hierarchical network organization and authenticates routing exchanges. Together, the baseline and four experiments provide a practical progression from basic dynamic routing toward more flexible, resilient and structured network infrastructure.
