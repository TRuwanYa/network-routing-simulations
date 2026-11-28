<div align="center">

# 🌐 Network Routing Simulations

**Network Architecture · Dynamic Routing · Resilience · Routing Security**

![Cisco](https://img.shields.io/badge/Cisco-Packet%20Tracer-049FD9?style=for-the-badge&logo=cisco&logoColor=white)
![IPv4](https://img.shields.io/badge/IPv4-VLSM-334155?style=for-the-badge)
![OSPF](https://img.shields.io/badge/OSPF-Implemented-047857?style=for-the-badge)
![RIPv2](https://img.shields.io/badge/RIPv2-Implemented-047857?style=for-the-badge)

A progressive, experiment-driven investigation of computer network routing, redundancy, resilience and interoperability using Cisco Packet Tracer.

[Objectives](#-project-objectives) · [Roadmap](#-project-roadmap) · [Campus Network](#02--multi-router-network) · [OSPF Experiments](#-ospf-experiments) · [Getting Started](#-getting-started)

</div>

---

## 📖 Overview

This repository progresses from fundamental static and dynamic routing to a redundant multi-router campus network. Independent simulations investigate routing metrics, link-state operation, fault tolerance, hierarchical routing, authentication and edge network services. Future stages will consolidate comparative measurements and explore hybrid routing architectures.

### Project highlights

| Campus infrastructure | Routing experiments | Network security |
| :---: | :---: | :---: |
| **5 routers · 4 departments** | **4 focused OSPF experiments** | **Authenticated OSPF adjacency** |
| Redundant backbone and VLSM | Cost, failover and hierarchical routing | Key mismatch and recovery testing |

## 🎯 Project objectives

- Implement and verify static and dynamic IPv4 routing.
- Design redundant networks using subnetting and variable-length subnet masks (VLSM).
- Investigate route discovery, routing metrics, path selection and equal-cost multipath routing (ECMP).
- Evaluate network behaviour during backbone link failures and restoration.
- Demonstrate default routing, NAT/PAT and simulated external access.
- Investigate hierarchical OSPF routing and routing message authentication.
- Compare routing protocols under consistent experimental conditions and explore route redistribution.

## 🧭 Project roadmap

| Project | Scope | Status |
|---|---|---|
| [01 — Basic Routing](./01-basic-routing/) | Static routing, RIPv2, inter-LAN connectivity | ![Completed](https://img.shields.io/badge/Completed-047857) |
| [02 — Multi-Router Network](./02-multi-router-network/) | Campus architecture, RIPv2, OSPF, NAT/PAT and planned EIGRP | ![In progress](https://img.shields.io/badge/In%20progress-D97706) |
| 03 — Comparative Routing Analysis | Convergence, packet loss and recovery measurements | ![Planned](https://img.shields.io/badge/Planned-64748B) |
| 04 — Hybrid Routing | Redistribution, route preference and routing policies | ![Planned](https://img.shields.io/badge/Planned-64748B) |

## 01 — Basic Routing

Introduces routing between two departmental LANs using independent static routing and RIPv2 simulations. Routing tables and connectivity tests demonstrate the difference between manually configured and dynamically learned routes.

**[Explore basic routing →](./01-basic-routing/)**

## 02 — Multi-Router Network

Extends the network into a redundant campus architecture with four departmental LANs, a central administration server, four internal routers and an edge router connected to a simulated external server.

### 🏛️ Campus architecture

![Campus network topology](./02-multi-router-network/network-topology.png)

### Router interface addressing

![Router interface addressing](./02-multi-router-network/router-interface-addressing.png)

Each dynamic routing protocol uses an independent copy of the same base topology and addressing scheme.

### Routing protocol implementations

| Protocol | Routing approach | Metric | Status |
|---|---|---|---|
| [RIPv2](./02-multi-router-network/ripv2/) | Distance vector | Hop count | Completed |
| [OSPF](./02-multi-router-network/03-ospf-routing/) | Link state | Cumulative interface cost | Completed |
| EIGRP | Advanced distance vector | Bandwidth and delay by default | Planned |

### RIPv2 implementation

Investigates dynamic route learning, interdepartmental connectivity, equal-cost paths, backbone failure and restoration, default route propagation, NAT/PAT and simulated HTTP access.

**[Explore RIPv2 →](./02-multi-router-network/ripv2/)**

### 🔬 OSPF experiments

The OSPF implementation begins with a single-area Area 0 configuration, verifying neighbor adjacencies, link-state databases, routing tables and end-to-end connectivity. Four subsequent experiments investigate capabilities beyond basic route learning.

| Experiment | Investigation | Key result |
|---|---|---|
| [Baseline OSPF](./02-multi-router-network/03-ospf-routing/) | Neighbor discovery, LSDB and SPF routing | FULL adjacencies and equal-cost routes observed |
| [01 — Cost-based routing](./02-multi-router-network/03-ospf-routing/01-cost-based-routing/) | Interface cost manipulation | Longer path selected when its cost was lower (193 versus 251) |
| [02 — Network resilience](./02-multi-router-network/03-ospf-routing/02-network-resilience/) | Backbone link failure and restoration | Automatic failover; 220 of 221 ping packets received during the continuous test |
| [03 — Multi-area OSPF](./02-multi-router-network/03-ospf-routing/03-multi-area-ospf/) | Hierarchical routing, ABRs and Type 3 LSAs | Inter-area routes (`O IA`) and area-specific LSDBs verified |
| [04 — Secure OSPF](./02-multi-router-network/03-ospf-routing/04-secure-ospf/) | MD5 authentication and key mismatch | Adjacency rejection, alternate path selection and recovery verified |

**[Explore the complete OSPF implementation →](./02-multi-router-network/03-ospf-routing/)**

### Routing protocol comparison

| Characteristic | RIPv2 | OSPF |
|---|---|---|
| Algorithm | Distance vector | Dijkstra shortest path first |
| Path selection | Hop count | Cumulative cost |
| Network organization | Flat | Hierarchical areas |
| Routing information | Distance-vector updates | Link-state advertisements |
| Hop-count limitation | 15 usable hops | No equivalent 15-hop limit |
| Authentication | Supported | Supported, including area-based configuration |
| ECMP | Supported | Supported |

OSPF offers more flexible path selection and hierarchical network organization. Both protocols support authentication and redundant routing. Comparative convergence performance remains to be measured under controlled conditions.

**[Explore the multi-router network →](./02-multi-router-network/)**

## 03 — Comparative Routing Analysis

> Planned — controlled comparative experiments.

This stage will consolidate results from independent protocol simulations rather than duplicate the network topology. Wherever possible, the same endpoints, failed backbone link, interface settings and test procedure will be used.

Measurements will include convergence and recovery time, packet loss during disruptions, route restoration and selected routing paths. Repeated trials and documented conditions will support meaningful comparisons. Numerical comparisons will be added only after measurements are collected.

## 04 — Hybrid Routing

> Planned — routing interoperability.

This stage will investigate static and dynamic routes within a single network, redistribution between routing domains, administrative distance, route filtering and loop prevention. The exact topology and policies will be defined during implementation.

## 🔧 Tools and technologies

| Category | Technologies |
|---|---|
| Simulation | Cisco Packet Tracer, Cisco IOS CLI |
| Network design | IPv4, VLSM, Ethernet, serial links |
| Routing | Static routing, RIPv2, OSPF; EIGRP planned |
| Verification | ICMP, ping, traceroute, routing tables, LSDB inspection |
| Edge services | Default routing, NAT/PAT, TCP/HTTP |
| Routing security | OSPF message-digest authentication |
| Documentation | Git, GitHub, Markdown |

## 🚀 Getting started

1. Clone or download this repository.
2. Open a completed `.pkt` file in Cisco Packet Tracer.
3. Consult the corresponding project README for addressing, configuration and verification procedures.
4. Inspect routing tables using `show ip route` and test connectivity using `ping` and `tracert`.
5. Use a separate copy of the simulation when modifying configurations or reproducing failure experiments.

## 📌 Scope and limitations

This repository is an educational simulation environment, not a production network. Packet Tracer timing and protocol behaviour may differ from physical Cisco equipment. External connectivity uses the documentation address range `203.0.113.0/24` and does not represent real Internet access. Internal backbone redundancy does not eliminate the single edge uplink as a point of failure.

MD5 authentication is demonstrated for educational and simulator compatibility purposes on one OSPF link; it does not encrypt traffic or secure the entire routing domain. Modern deployments should prefer stronger supported authentication mechanisms. Planned experiments and theoretical protocol characteristics are distinguished from measured results.

---

<div align="center">

**Ruwanya Herath**


</div>
