# OSPF Cost-Based Routing

## 1. Objective

This experiment investigates OSPF's cost-based path selection using the existing university network topology. It demonstrates how OSPF can select a longer route with a lower cumulative interface cost rather than relying on hop count, as RIPv2 does.

## 2. Network Environment

- Simulator: Cisco Packet Tracer
- Routing protocol: OSPFv2
- OSPF process ID: 1
- Routing area: Area 0
- Source: CO-PC1 (10.0.1.1)
- Destination: ME-PC2 (10.0.3.2)
- Destination network: 10.0.3.0/24

The existing five-router topology was reused without modifying its physical connections or IP addressing.

## 3. Baseline Routing Behaviour

The original serial interface costs were 64, and the destination LAN interface cost was 1.

![Original routing topology](before%20modification.png)

| Path | Cost calculation | Total cost |
| --- | --- | ---: |
| R1 → R3 | 64 + 1 | 65 |
| R1 → R2 → R4 → R3 | 64 + 64 + 64 + 1 | 193 |

OSPF initially selected the direct path through R3.

![Original traceroute](observations/01%20original-path-before%20modification.png)

## 4. Cost Modification

The outgoing OSPF cost of R1's Serial0/3/0 interface, connected to R3, was increased to 250.

```cisco
configure terminal
interface Serial0/3/0
 ip ospf cost 250
end
```

This changes the routing metric without changing the physical bandwidth or disabling the link. The modification is directional because only R1's outgoing interface cost was changed.

The configured cost was verified using:

```cisco
show ip ospf interface Serial0/3/0
```

![Modified interface cost](observations/04%20modified-interface-cost.png)

## 5. Routing Behaviour After Modification

![Modified routing topology](after%20modification.png)

| Path | Cost calculation | Total cost |
| --- | --- | ---: |
| R1 → R3 | 250 + 1 | 251 |
| R1 → R2 → R4 → R3 | 64 + 64 + 64 + 1 | 193 |

OSPF recalculated its shortest path and selected the alternative route through R2 and R4.

### Routing Table Verification

```cisco
show ip route ospf
```

![Updated routing table](observations/02%20new-routing-table.png)

The route to 10.0.3.0/24 was installed with administrative distance 110, metric 193 and next hop 10.0.6.13 through Serial0/3/1.

### Connectivity and Path Verification

```text
ping 10.0.3.2
tracert 10.0.3.2
```

![Post-modification connectivity and traceroute](observations/03%20after-path-modification.png)

The traceroute confirmed the longer forwarding path, while all four ICMP echo requests received replies.

## 6. Experimental Results

| Parameter | Before modification | After modification |
| --- | --- | --- |
| Selected path | R1 → R3 | R1 → R2 → R4 → R3 |
| OSPF metric | 65 | 193 |
| Next hop | 10.0.6.2 | 10.0.6.13 |
| Outgoing interface | Serial0/3/0 | Serial0/3/1 |
| Routers traversed | 2 | 4 |
| Traceroute | Successful | Successful |
| Packet loss | Not measured in the original traceroute | 0% (4/4 replies) |

## 7. Comparison with RIPv2

RIPv2 uses hop count and would prefer the direct path in this topology if all links remained available. OSPF instead uses cumulative interface costs and selected the longer path after the direct link's cost was increased. This comparison is theoretical; a new RIP test was not conducted in this experiment.

The experiment demonstrates configurable routing preferences, not an increase in physical bandwidth or measured throughput.

## 8. Conclusion

The experiment successfully demonstrated OSPF's cost-based routing mechanism. Increasing the cost of the direct R1–R3 link caused OSPF to select a longer but lower-cost alternative path through R2 and R4. Post-modification connectivity was verified with zero packet loss across four ping requests. The results demonstrate that OSPF selects routes according to cumulative cost rather than hop count.
