# Kubernetes Networking

A Kubernetes cluster presents a flat Pod network, then layers stable virtual Services, DNS discovery, L7 Ingress routing, TLS termination, and distributed NetworkPolicy controls on top. The useful diagnostic habit is to trace a request across those layers rather than treating “the network” as one opaque thing.

## Flat Pod network: the three contracts

Kubernetes asks its CNI implementation to uphold three rules:

1. Every Pod gets a unique IP. Containers in the same Pod share that Pod IP and can all use the same port without collisions with other Pods.
2. Any Pod can communicate directly with any other Pod without NAT — source and destination Pod IPs stay intact.
3. Nodes and Pods can communicate directly, also without NAT.

This differs from traditional Docker port mapping (`NodeIP:port → containerIP:port`). At microservice scale, port mappings cause collisions, dynamic allocation complexity, and self-registration errors: the address a container knows is not the address others can reach. In the flat model, Pods are first-class network participants; apps see their real routable address, discovery records `ServiceName → PodIP`, and packet captures need no mental NAT translation.

Physically, the cluster can span racks and availability zones. The CNI makes it behave like one connected L3 space, often via routing or an overlay such as IP-in-IP. The address-planning implications sit in [[computer-networking-tcp-ip]].

## Service discovery: CoreDNS

Pod IPs are short-lived, so hardcoding even a stable-looking `ClusterIP` is brittle and unreadable. CoreDNS gives Services stable names, typically `service.namespace.svc.cluster.local`:

- It runs as a replicated Deployment in `kube-system`, behind the `kube-dns` Service.
- It watches the Kubernetes API, automatically creating/updating DNS records when Services change.
- Pods use `/etc/resolv.conf` search paths, so a same-namespace Service can normally be reached as just `backend-service`.

DNS is a pre-flight lookup, not part of the later data path: it maps a meaningful Service name to the Service virtual IP; kube-proxy/eBPF then handles Service-to-endpoint forwarding.

## Ingress: one L7 entrance, two separate concepts

Giving every microservice a cloud `LoadBalancer` costs public IPs and load balancers, and standard L4 load balancing sees only IP and port. **Ingress** adds content-aware L7 routing, reading HTTP host and path so a single public entry can direct `api.example.com` and `/static` to different Services.

Do not confuse:

- **Ingress Resource** — a declarative rulebook stored in the API: host/path → Service. By itself it does nothing.
- **Ingress Controller** — the working reverse-proxy implementation (ingress-nginx, Traefik, AWS ALB controller). It watches Ingress resources and reloads its runtime configuration.

Typical external path: public DNS → cloud LoadBalancer (L4) → NodePort / Service handling → Ingress Controller (L7) → selected backend Service → Pod. Internal traffic instead resolves a Service through CoreDNS, then reaches its endpoints without visiting the public edge.

On EKS the same path is usually automated. AWS Load Balancer Controller watches Ingress objects and creates or reuses an ALB plus Target Group. External DNS writes the hostname into Route53. TLS commonly terminates at the ALB. Ready Pod IPs are registered as targets; a failing Readiness probe drops the Pod from the Target Group before kube-proxy is the interesting part. See [[eks-platform-engineering]].

## TLS termination at the edge

At the Ingress, TLS can terminate: the controller decrypts HTTPS, chooses a backend based on host/path, then commonly forwards HTTP inside the cluster. Centralising this gives one place for certificates and cipher policy, removes expensive TLS handshake work from each app Pod, and lets application teams focus on the service.

The certificate lives in a `kubernetes.io/tls` Secret with `tls.crt` and private `tls.key`. Base64 encoding is **not encryption**; RBAC still has to protect the Secret. TLS termination simplifies the inside path but changes the security boundary: the plaintext portion now begins at the edge. For zero-trust or sensitive traffic, use re-encryption / TLS passthrough / a mesh as appropriate, not a casual assumption that “internal” means safe. See [[https-tls-web-security]].

## NetworkPolicy: default-open needs deliberate isolation

Kubernetes' flat network is intentionally permissive: Pods are non-isolated by default. That makes a compromised internet-facing Pod a potential lateral-movement pivot into databases or internal APIs.

A **NetworkPolicy** is a distributed L3/L4 firewall intent:

- `podSelector` identifies protected Pods; selected Pods become isolated.
- `ingress` says *who may connect to me*; `egress` says *where may I connect*.
- Rules can select peers by Pod labels, Namespace labels, CIDR `ipBlock`, and TCP/UDP ports.
- It is allow-list logic: after isolation, only explicitly permitted traffic passes.

Critical caveat: the API object is only intent. The selected CNI must enforce NetworkPolicy (e.g., Cilium or Calico); otherwise Kubernetes accepts the YAML but no packets are filtered. Production policy design should start with explicit defaults (including DNS egress), then narrowly allow the actual paths.

## A request-tracing checklist

For an external HTTPS request:

1. Public DNS resolves the hostname to the cloud load balancer.
2. The L4 load balancer forwards into the cluster, usually via a NodePort/Service path.
3. The Ingress Controller terminates TLS (if configured), reads host/path, and selects a Service.
4. Service forwarding selects a backend Pod endpoint.
5. NetworkPolicy may allow or block the final Pod traffic.

For a Pod-to-Service request:

1. The Pod resolves the Service name through CoreDNS.
2. It receives the stable ClusterIP and sends traffic through the Service implementation to an endpoint Pod.
3. NetworkPolicy controls whether this east-west traffic is allowed.

That ordering makes failures tractable: resolve DNS first, distinguish L4 reachability from L7 routing, inspect certificate/Ingress config, then validate endpoints and policies.

## Related pages

- [[container-and-kubernetes-foundations]] — the course underneath this page: why Pod IPs move, and why Service/Ingress exist.
- [[computer-networking-tcp-ip]] — TCP/IP layers, CIDR, and Pod address planning beneath this model.
- [[https-tls-web-security]] — certificate verification and TLS mechanics that Ingress termination applies.
- [[http2-multiplexing]] — L7 protocol behavior; gRPC over HTTP/2 is common for east-west services.
- [[helm-kubernetes-package-manager]] — package and version this infrastructure without duplicating manifests.
- [[eks-platform-engineering]] — ALB controller, External DNS, IRSA, and the request-path used in the current EKS course.
- [[aws-vpc-subnet-design]] — private subnets and Security Groups under those controllers.

## References
