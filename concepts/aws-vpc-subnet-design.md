# AWS VPC and Subnet Design

A VPC is the logical network boundary for an AWS workload. Subnets divide that address space into Availability Zone scoped segments, while route tables decide how those segments can communicate. The useful mental model is simple:

> **VPC defines the address boundary. Subnet places a workload in one AZ. Route tables determine whether it has a path out.**

A subnet is not inherently public or private. It is public when its associated route table sends `0.0.0.0/0` to an Internet Gateway and workloads have a reachable public address. A private subnet normally sends outbound internet traffic through a NAT Gateway, or has no internet route at all. Keep databases, worker nodes, and other sensitive workloads private by default.

## Address planning

Start with a VPC CIDR sized for actual growth, then allocate non-overlapping subnet ranges per AZ and role. A `/16` such as `10.0.0.0/16` has 65,536 addresses; splitting it into `/24` subnets gives 256 addresses per subnet. AWS reserves five addresses in each subnet, so a `/24` supplies 251 usable addresses.

Use Terraform's `cidrsubnet(prefix, newbits, netnum)` rather than hand-calculating ranges. For example, `cidrsubnet("10.0.0.0/16", 8, 1)` produces `10.0.1.0/24`. Combining it with a keyed `for_each` map makes the layout data-driven and lets a new AZ receive a predictable range.

## A resilient baseline

For a multi-AZ application:

1. Create one VPC with DNS support enabled.
2. Create public and private subnets in at least two AZs.
3. Attach an Internet Gateway to the VPC.
4. Route public subnets to the Internet Gateway.
5. Route private subnet egress to NAT Gateways, with the availability and cost model chosen explicitly.
6. Place public entry points such as load balancers in public subnets; place compute and data services in private subnets.
7. Apply Security Groups and Network ACLs as separate controls. Routing makes a path possible, it does not make it safe.

This foundation sits below an EKS or Kubernetes layer. It must accommodate Pod and Service address planning, NAT capacity, and failure isolation without accidentally turning internal workloads into public ones.

## EKS-aware network contracts

The VPC is also a contract with the cluster layer, not merely a collection of routes.

- **Plan Pod address space early.** With the AWS VPC CNI, Pods normally consume VPC IP addresses. When the primary CIDR becomes scarce, attach a deliberately planned secondary CIDR. The current study material uses RFC 6598 space such as `100.64.0.0/16` for EKS Pods, after checking that it will not overlap with connected networks.
- **Choose the NAT failure domain explicitly.** One NAT Gateway is cheaper but makes private-subnet egress in other AZs depend on that AZ. A NAT Gateway and Elastic IP per AZ cost more, but preserves image pulls, external API access, and telemetry when another AZ fails. The EIP also provides a stable outbound allowlist address.
- **Express trust by identity.** Security Groups are stateful: response traffic for an allowed connection is automatically permitted. Prefer SG-to-SG references for EKS control-plane, node, and workload paths instead of allowing broad CIDR ranges. Keep the rule direction and owning component clear.
- **Publish discovery metadata.** AWS Load Balancer Controller and similar integrations discover eligible subnets through Kubernetes-specific tags. Tags are therefore part of the infrastructure API, alongside VPC ID, private/public subnet IDs, route-table IDs, and security-group IDs exposed as narrow Terraform outputs.

A practical boundary is a foundation stack that owns VPC, DNS, subnets, gateways, tags, and security primitives, while EKS and workload stacks consume its reviewed outputs. This avoids hard-coded IDs and makes the cross-stack dependency visible.

That boundary is now also a security and cost boundary. Worker nodes and data stores stay in private subnets with no public IP. Inbound traffic should hit an ALB, not a node. Security Groups stay identity-based (SG-to-SG), which is Layer 3 of the six-layer model in [[eks-platform-engineering]]. NAT Gateways are both a failure domain and a fixed monthly cost: one per AZ is the high-availability choice, and it is also why extra clusters are expensive compared with extra Namespaces.

## Related pages

- [[terraform-infrastructure-as-code]]: the desired-state workflow and module boundaries for this design.
- [[computer-networking-tcp-ip]]: CIDR calculation and IP-capacity fundamentals.
- [[kubernetes-networking]]: cluster networking and the traffic layers that run on this foundation.
- [[eks-platform-engineering]]: private-subnet plus ALB ingress, account isolation, and NAT as a designed cost.

## References

