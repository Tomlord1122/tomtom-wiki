# Computer Networking: TCP/IP Mental Model

A practical networking model for reasoning about web requests and Kubernetes traffic. TCP/IP is the implementation that runs the Internet; OSI is still valuable as a shared troubleshooting vocabulary. The two models evolved independently rather than TCP/IP being a literal implementation of OSI.

## Four layers

| TCP/IP layer | Main responsibility | Typical examples |
|---|---|---|
| Link | Local-hop delivery using frames and hardware or virtual interfaces | Ethernet, Wi‑Fi, MAC addresses, Kubernetes bridges and veth pairs |
| Internet | Logical addressing and best-effort routing between networks | IP |
| Transport | Application-to-application delivery and connection identity | TCP, UDP, ports |
| Application | User-facing protocols and the meaning of the data | HTTP, DNS, TLS, gRPC |

The OSI mapping is approximate: OSI Physical + Data Link become Link; OSI Network becomes Internet; OSI Transport remains Transport; and OSI Session + Presentation + Application become Application. The End-to-End Principle explains why many responsibilities remain at the communicating endpoints rather than in the network core.

## Tracing a request

DNS is a pre-flight step: a hostname must resolve to an IP before the sender can construct the Internet-layer destination. The request is then encapsulated as it moves down the stack — application data inside a TCP or UDP segment, inside an IP packet, inside a Link-layer frame — and decapsulated at the destination.

Servers commonly listen on well-known ports such as 80 (HTTP), 443 (HTTPS), and 22 (SSH). Clients use ephemeral source ports so one device can maintain many simultaneous connections. The OS uses the five-tuple (source IP, source port, destination IP, destination port, protocol) to demultiplex incoming traffic to the correct process.

## Kubernetes and debugging

Kubernetes makes the same layering visible. A Service’s `Port` is the cluster-facing entry point and `TargetPort` identifies the application port inside the selected Pod. Virtual bridges and veth pairs participate in local-hop delivery, while IP routing and transport ports carry traffic across the cluster.

The model gives a useful first diagnostic split:

- Ping failure usually points toward the Internet layer or below.
- A port-connectivity failure points toward the Transport layer.
- HTTP status or TLS failures are Application-layer problems, even when lower-layer connectivity works.

This is a troubleshooting heuristic, not a replacement for observing the actual path and protocol state.

## CIDR, subnet capacity, and Kubernetes address planning

CIDR notation writes the number of contiguous network-prefix bits after a slash. For IPv4, a `/24` is equivalent to `255.255.255.0`: 24 network bits and 8 host bits. The total number of addresses is `2^(32-prefix)`. In a traditional subnet, the network ID and broadcast address are reserved, so the usual usable-host formula is `2^(32-prefix)-2` (with small-subnet and platform-specific exceptions).

This makes CIDR a practical capacity and routing tool, not just shorthand. A `/16` contains 65,536 IPv4 addresses; a `/24` contains 256 total and conventionally 254 usable addresses. CIDR replaced rigid class A/B/C allocation with arbitrary prefixes from `/0` through `/32`, reducing waste and allowing precise network slices.

Kubernetes applies the same hierarchy to Pod addressing:

- **Cluster CIDR / pod-network-cidr** is the total pool reserved for Pods, such as `10.244.0.0/16`.
- **Node CIDR** is a delegated slice of that pool assigned to one Node, often `/24` in common CNI configurations.
- Pods receive unique, routable addresses from their Node’s slice. Kubernetes’ intended flat Pod network allows Pod-to-Pod traffic across Nodes without relying on NAT for internal communication.

The sizing implication is operational: a `/16` gives a cluster room to grow, while a `/24` Node slice gives roughly 250 Pod addresses after reservations. Exhausting the address pool can prevent scheduling even when compute and memory remain available. Cluster, Node, Service, and external address ranges must also be non-overlapping and routable through the chosen CNI; otherwise routing becomes ambiguous.

### Special IPv4 addresses

Not every address in a block is an ordinary host address. The network address identifies the subnet, the broadcast address reaches all hosts on a local subnet, and loopback (`127.0.0.0/8`, commonly `127.0.0.1`) refers back to the local machine. Private ranges are intended for internal networks (`10.0.0.0/8`, `172.16.0.0/12`, and `192.168.0.0/16`) and are not globally routed. Link-local addresses (`169.254.0.0/16`) are used when a host cannot obtain a normal address, while unspecified (`0.0.0.0`) and limited broadcast (`255.255.255.255`) have control-plane meanings rather than identifying ordinary destinations.

These reservations explain why an IP that looks numerically available may still be unusable. Always interpret an address together with its prefix, network role, and platform rules; Kubernetes and cloud providers may reserve additional addresses inside each subnet.

## Related pages

- [[cloud-native-ai-infrastructure]] — networking is part of the platform substrate supporting AI workloads.
- [[cometline-architecture]] — a systems view benefits from separating layers and responsibilities.
- [[cometline-agent-runtime]] — the same “trace the request through layers” habit applies to tool and provider flows.
- [[https-tls-web-security]] — TLS is an Application-layer protocol over TCP; certificate and handshake failures belong above basic port connectivity.
- [[http2-multiplexing]] — HTTP/2 uses TCP's connection behavior while fixing application-layer queueing.
- [[kubernetes-networking]] — applies Layer 3 routing, DNS, Service virtual IPs, L7 ingress, and policy inside a cluster.

## References
