# Secure OSPF

## Objective

Demonstrate OSPFv2 message-digest authentication, rejection of incorrectly authenticated routing messages, alternative path selection and adjacency recovery in Cisco Packet Tracer.

## Why routing authentication is needed

Without authentication, a device on an OSPF-enabled link that meets the protocol requirements may establish an adjacency and introduce misleading routing information. This can cause incorrect forwarding, traffic redirection or loss of reachability. Authentication restricts routing-message acceptance to peers possessing the shared secret and helps detect unauthorized modification of messages. It does not establish that every authenticated router is trustworthy.

## Experimental setup

The multi-area topology from Experiment 03 is reused without changing addressing or area assignments. Authentication is applied only to the R1–R3 backbone link (10.0.6.0/30), using Serial0/3/0 on both routers. The destination is ME-PC2 (10.0.3.2).

## Authentication mechanism

Both routers use a shared secret to generate and verify an MD5 authentication digest. The key ID identifies the configured key. A mismatched digest causes OSPF packets to be rejected; an existing adjacency can expire when valid Hello packets are no longer received. Authentication does not encrypt traffic.

## Implementation and observations

### 1. Baseline

R1 initially has three FULL neighbors, including R3. The tested interface is up/up with cost 64 and no message-digest authentication.

![Before authentication](observations/01%20before-authentication.png)

### 2. Enable authentication

Configure both R1 and R3 on the tested interface:

```cisco
configure terminal
interface Serial0/3/0
 ip ospf authentication message-digest
 ip ospf message-digest-key 1 md5 OSPFLab123
end
```

R1 reports message-digest authentication enabled, key ID 1, and a FULL adjacency with R3.

![Matching authentication keys and preferred path](matching%20keys.png)

*Figure: Matching keys allow the R1–R3 adjacency to reach FULL. The preferred route to the Mechanical Department has cost 64 + 1 = 65.*

![Authentication enabled](observations/02%20authentication-enabled.png)

### 3. Introduce a mismatched key

On R3:

```cisco
configure terminal
interface Serial0/3/0
 no ip ospf message-digest-key 1
 ip ospf message-digest-key 1 md5 WrongKey123
end
```

After the adjacency expires, R3 lists only R4 as a FULL neighbor. Its route to the CO network (10.0.1.0/24) has cost 193 through R4.

![Mismatched authentication keys and backup path](mismatched%20keys.png)

*Figure: A key mismatch removes the logical adjacency without requiring a physical link shutdown. The calculated backup cost to the destination LAN is 64 + 64 + 64 + 1 = 193. The diagram illustrates the expected physical link state, which was not separately captured during the mismatch.*

![Authentication failure](observations/03%20authentication-failure.png)

Traceroute from CO-PC1 to ME-PC2 still completes through R1, R2, R4 and R3, demonstrating alternative path selection rather than complete connectivity failure.

![Alternative path verification](observations/04%20connection-failure-verification.png)

### 4. Restore authentication

On R3:

```cisco
configure terminal
interface Serial0/3/0
 no ip ospf message-digest-key 1
 ip ospf message-digest-key 1 md5 OSPFLab123
end
```

R1 again lists three FULL neighbors. The inter-area route to 10.0.3.0/24 returns to cost 65 via 10.0.6.2 (Serial0/3/0).

![Authentication restored](observations/05%20authentication-restored.png)

### 5. Verify connectivity

```text
ping 10.0.3.2
tracert 10.0.3.2
```

All four ping packets were received, with 0% loss and an average round-trip time of 3 ms. Traceroute confirms the direct R1–R3 path.

![Connectivity verification](observations/06%20connectivity-verification.png)

## Results

| Stage | R1–R3 adjacency | Forwarding path from CO to ME |
|---|---|---|
| Before authentication | FULL | Direct path expected from baseline topology |
| Matching keys | FULL | Direct path expected |
| Mismatched keys | Lost | R1 → R2 → R4 → R3 (observed) |
| Corrected key | FULL | R1 → R3 (observed) |

The alternative forward path has an expected cost of 193 (64 + 64 + 64 + 1); the restored forward cost of 65 is directly verified. The failure screenshot records R3's reverse route cost, not R1's forward metric. Exact convergence time and packet loss during the mismatch were not measured. An up/up interface during the mismatch was not separately captured.

## Comparison with RIPv2

Both RIPv2 and OSPFv2 support authentication, including MD5. The improvement demonstrated here is the integration of authentication with OSPF’s link-state and hierarchical routing architecture, not the introduction of authentication itself.

| Feature | RIPv2 | OSPFv2 |
|---|---|---|
| Authentication | Supports MD5 and simple password authentication | Supports MD5 and simple password authentication; stronger HMAC-SHA mechanisms are also standardized |
| Routing information | Distance-vector updates | Link-state protocol messages |
| Neighbor management | No OSPF-style adjacency state machine | Explicit adjacency states and database synchronization |
| Configuration scope | Interface-based | Interface-based or area-wide authentication policy |
| Alternative path selection | Hop count | Accumulated interface cost |

In OSPF, rejected Hello packets can cause an adjacency to expire, triggering link-state updates and shortest-path recalculation. The authenticated adjacency therefore has a direct relationship with the topology used for route computation. Area-wide authentication policies also provide a convenient way to enforce authentication across a hierarchical network. The experiment verifies these adjacency and routing effects on one link; it does not benchmark security or convergence against an authenticated RIPv2 network.

## Limitations and security considerations

- Only one link is authenticated; the entire routing domain is not secured.
- MD5 is a legacy mechanism chosen for simulator compatibility. Prefer stronger authentication such as HMAC-SHA-256 where supported.
- Authentication protects routing-message authenticity and integrity, not confidentiality or all forms of routing attack.
- The displayed keys are demonstration-only credentials and must not be reused in production.

## Conclusion

The experiment demonstrates that matching authentication keys permit OSPF adjacency, while mismatched keys prevent continued authenticated neighbor communication. The redundant topology preserves reachability through an alternative path, and correcting the key restores the original adjacency and preferred route. Final connectivity was verified with zero packet loss in the four-packet test.
