# OSPF Network Resilience

## Objective

Evaluate automatic failover and route restoration in a redundant single-area OSPF network by simulating an inter-router link failure and observing adjacency changes, routing metrics and packet delivery.

## Network and methodology

The five-router university topology from Experiment 01 is reused without changing IP addressing. R1's Serial0/3/0 interface retains its configured cost of 250; the remaining ring links have a cost of 64. Traffic from CO-PC1 to the Mechanical Department host `10.0.3.2` initially follows R1 → R2 → R4 → R3 (cost 193, including the destination LAN cost of 1).

### 1. Normal operation

The initial traceroute confirms the preferred path, and the ping test reports no packet loss.

![Initial path](observations/01%20paths-before-failure.png)

### 2. Simulated link failure

The R1–R2 link is disabled on R1:

```cisco
configure terminal
interface Serial0/3/1
 shutdown
end
```

![Link failure](observations/02%20link-failure.png)

### 3. Adjacency and routing changes

```cisco
show ip ospf neighbor
show ip route ospf
```

R2 disappears from R1's neighbor table while R3 and R5 remain fully adjacent. OSPF recalculates the route to `10.0.3.0/24`, selecting the direct R1–R3 link with a cumulative cost of 251.

![Neighbors after failure](observations/03%20neighbor-table-after-failure.png)

![Routes after failure](observations/04%20routing-table-after-failure.png)

### 4. Connectivity and packet loss

```text
tracert 10.0.3.2
ping 10.0.3.2
```

The traceroute confirms the backup path and all four post-failover ping packets are delivered.

![Connectivity after failure](observations/05%20connectivity-after-failure.png)

The continuous ping observation records 221 packets sent, 220 received and one lost. The calculated loss is approximately 0.45%, while Packet Tracer displays 1%. The recorded round-trip times are 1 ms minimum, 27 ms maximum and 6 ms average.

![Packet loss observations](results/06%20packet-loss-observations.png)

### 5. Link restoration

```cisco
configure terminal
interface Serial0/3/1
 no shutdown
end
show ip ospf neighbor
show ip route ospf
```

R2 returns to FULL adjacency and R1 automatically reinstalls the preferred route with cost 193.

![After restoration](observations/07%20after-restoration.png)

## Results

| Metric | Normal | Failed link | Restored link |
| --- | --- | --- | --- |
| Selected router path | R1–R2–R4–R3 | R1–R3 | R1–R2–R4–R3 (inferred from restored routing state) |
| OSPF cost | 193 | 251 | 193 |
| Next hop | 10.0.6.13 | 10.0.6.2 | 10.0.6.13 |
| Outgoing interface | Serial0/3/1 | Serial0/3/0 | Serial0/3/1 |
| R1 neighbor count | 3 (baseline) | 2 | 3 |
| Ping loss in individual tests | 0% | 0% | Not separately measured |

## Discussion

OSPF adapts to topology changes through link-state advertisements and shortest-path recalculation. The higher-cost backup route preserves reachability after the preferred link fails. Restoration of the link allows OSPF to select the lower-cost route again without manual route changes.

The experiment demonstrates redundancy against the tested single-link failure, not protection against every router or link failure. The continuous ping summary does not establish the exact convergence time or isolate packet loss by event. No controlled RIPv2 convergence comparison was performed.

## Conclusion

The redundant OSPF network successfully demonstrated automatic failover and route restoration. Connectivity was available after convergence, with only one packet lost in the continuous ping observation. These results illustrate the benefits of redundant paths and dynamic link-state routing.

## Files

- `ospf-redundant-network.pkt` — Packet Tracer simulation
- `results/` — experimental screenshots
