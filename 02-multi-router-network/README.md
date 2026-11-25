# Multi-Router Campus Network

A Cisco Packet Tracer simulation of a departmental campus network with a redundant router backbone, central administration services and a simulated external network. The project provides a common topology for studying dynamic routing, path selection, resilience and edge network services.

## Objectives

- Design a segmented IPv4 campus network using variable-length subnet masks (VLSM).
- Configure Ethernet and serial router interfaces and verify directly connected networks.
- Provide redundant paths between departmental routers.
- Evaluate dynamic routing protocols on an identical physical topology.
- Investigate route convergence, equal-cost paths and simulated external access using NAT/PAT.

## Physical network topology

![Complete campus network topology](network-topology.png)

The network contains four engineering departments, each with four PCs and two interconnected access switches. A central administration server connects through a dedicated switch to R2. Four Cisco 2911 routers form a serial ring, and an ISR4331 edge router (R5) connects R1 to an external server. The external network represents an isolated simulated Internet service, not the real Internet.

| Router | Role | Model |
|---|---|---|
| R1 | Computer Engineering and edge uplink | Cisco 2911 |
| R2 | Electrical Engineering and Administration | Cisco 2911 |
| R3 | Mechanical Engineering | Cisco 2911 |
| R4 | Civil Engineering | Cisco 2911 |
| R5 | Edge router | Cisco ISR4331 |

## Logical topology and interface addressing

![Router interface addressing](router-interface-addressing.png)

The ring follows R1–R3–R4–R2–R1, with an additional Ethernet connection from R1 to R5. The ring provides alternate internal paths; the single edge uplink remains a single point of failure for external access.

### LAN addressing

| Network | Subnet | Host addresses | Default gateway |
|---|---|---|---|
| Computer Engineering (CO) | 10.0.1.0/24 | 10.0.1.1–10.0.1.4 | 10.0.1.254 |
| Electrical Engineering (EE) | 10.0.2.0/24 | 10.0.2.1–10.0.2.4 | 10.0.2.254 |
| Mechanical Engineering (ME) | 10.0.3.0/24 | 10.0.3.1–10.0.3.4 | 10.0.3.254 |
| Civil Engineering (CE) | 10.0.4.0/24 | 10.0.4.1–10.0.4.4 | 10.0.4.254 |
| Administration | 10.0.5.0/24 | 10.0.5.1 | 10.0.5.254 |
| External server | 203.0.113.0/24 | 203.0.113.10 | 203.0.113.1 |

All LANs use mask `255.255.255.0`. The external range is TEST-NET-3, reserved for documentation.

### Backbone subnet allocation

| Link | Subnet | Endpoint A | Endpoint B | Broadcast |
|---|---|---|---|---|
| R1–R3 | 10.0.6.0/30 | 10.0.6.1 | 10.0.6.2 | 10.0.6.3 |
| R3–R4 | 10.0.6.4/30 | 10.0.6.5 | 10.0.6.6 | 10.0.6.7 |
| R4–R2 | 10.0.6.8/30 | 10.0.6.9 | 10.0.6.10 | 10.0.6.11 |
| R2–R1 | 10.0.6.12/30 | 10.0.6.13 | 10.0.6.14 | 10.0.6.15 |
| R1–R5 | 10.0.6.16/30 | 10.0.6.17 | 10.0.6.18 | 10.0.6.19 |

Each point-to-point link uses `255.255.255.252`, providing exactly two usable IPv4 addresses. Using /24 LANs and /30 backbone links demonstrates VLSM and avoids allocating an entire /24 to each router link.

### Router interface assignments

| Router | Interface | IPv4 address | Connection |
|---|---|---|---|
| R1 | Gi0/0 | 10.0.1.254/24 | CO LAN |
| R1 | Gi0/1 | 10.0.6.17/30 | R5 |
| R1 | Se0/3/0 | 10.0.6.1/30 | R3 (DCE) |
| R1 | Se0/3/1 | 10.0.6.14/30 | R2 (DCE) |
| R2 | Gi0/0 | 10.0.2.254/24 | EE LAN |
| R2 | Gi0/1 | 10.0.5.254/24 | Admin LAN |
| R2 | Se0/3/0 | 10.0.6.13/30 | R1 |
| R2 | Se0/3/1 | 10.0.6.10/30 | R4 (DCE) |
| R3 | Gi0/0 | 10.0.3.254/24 | ME LAN |
| R3 | Se0/3/0 | 10.0.6.2/30 | R1 |
| R3 | Se0/3/1 | 10.0.6.5/30 | R4 |
| R4 | Gi0/0 | 10.0.4.254/24 | CE LAN |
| R4 | Se0/3/0 | 10.0.6.9/30 | R2 |
| R4 | Se0/3/1 | 10.0.6.6/30 | R3 (DCE) |
| R5 | Gi0/0/0 | 10.0.6.18/30 | R1 |
| R5 | Gi0/0/1 | 203.0.113.1/24 | External server |

## Base configuration

The 2911 routers use HWIC-2T serial modules. Serial DCE interfaces supply clocking at 64000 bps. Access switches operate at Layer 2; departmental segmentation is achieved through separate routed LANs rather than a shared VLAN trunk architecture.

Example configuration on R1:

```cisco
enable
configure terminal
interface GigabitEthernet0/0
 description CO LAN
 ip address 10.0.1.254 255.255.255.0
 no shutdown
exit
interface Serial0/3/0
 description Link to R3
 ip address 10.0.6.1 255.255.255.252
 clock rate 64000
 no shutdown
end
write memory
```

Verify interface status and directly connected routes:

```cisco
show ip interface brief
show ip route
show controllers serial 0/3/0
```

Test each host's default gateway and adjacent router addresses using `ping`. The base simulation intentionally does not provide complete interdepartmental routing or NAT/PAT; those features belong to the protocol-specific implementations.

## Routing implementations

| Implementation | Status | Documentation |
|---|---|---|
| RIPv2 | Implemented | [RIPv2 experiment](ripv2/README.md) |
| OSPF | Planned | Single-area routing and convergence comparison |
| EIGRP | Planned | DUAL, composite metrics and convergence comparison |

Full static routing is not repeated here because it was demonstrated in the preceding basic network project. The extended topology focuses on dynamic routing and resilience.

## Running the simulation

1. Open [extended-network-base.pkt](extended-network-base.pkt) in Cisco Packet Tracer to inspect the unmodified base network.
2. Open the protocol-specific simulation to reproduce the corresponding experiment.
3. Use the interface addressing table and protocol README to inspect and verify the configuration.

## Tools and scope

Cisco Packet Tracer, Cisco IOS CLI, IPv4, VLSM, Ethernet, serial WAN links and dynamic routing. This is an educational simulation; performance and convergence observations are not production hardware benchmarks. Redundancy is limited to the internal ring, and no production firewall, ISP or high-availability edge design is implied.
