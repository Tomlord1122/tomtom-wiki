# EKS Platform Engineering

The current Heptabase course treats an EKS platform as a stack of contracts, not a cluster plus some YAML. Terraform publishes identities. Helmfile consumes them. Each later layer assumes the previous one can fail.

> **Account and IAM bound the blast radius. Private subnets and Security Groups bound the path. Namespace and IRSA bound the workload. KMS and CSI bound the secret. Review and parameter checks bound the deploy.**

## Identity: two locks, temporary credentials

AWS IAM is modeled as two separate documents:

- **Trust Policy**: who may assume the role.
- **Permission Policy**: what the role may do after it is assumed.

Both are required. `sts:AssumeRole` on the caller is not enough if the target role does not list that caller.

Inside the cluster, **IRSA** applies the same idea to Pods. EKS exposes an OIDC issuer. A ServiceAccount is annotated with a role ARN. The Pod Identity webhook injects `AWS_ROLE_ARN` and `AWS_WEB_IDENTITY_TOKEN_FILE`. The SDK exchanges that projected token with STS for short-lived credentials. A Node Instance Role is no longer the shared superpower of every Pod on the node.

The Trust Policy condition is the part that actually closes the hole:

- GitHub OIDC must lock `token.actions.githubusercontent.com:sub` to a specific repo and ref. A federated principal with no `:sub` condition trusts any GitHub workflow that knows the role ARN.
- IRSA must lock `system:serviceaccount:<namespace>:<name>`. A role that only names the OIDC provider can be assumed by any ServiceAccount in the cluster.

`kubeconfig` and **EKS Access Entries** are the human and CI side of the same story: cluster auth is an identity mapping, not a file you copy forever.

## Secrets: three layers

Git remembers every character. Secrets need the opposite. The notes split the problem:

1. **KMS** is the root of trust. Keys stay in HSM. Callers have permission to use a key, not a copy of the key.
2. **Storage splits by lifetime.** SOPS encrypts config-layer values that should live in Git (Helmfile environment parameters). Secrets Manager holds runtime secrets (database passwords, third-party tokens) that should not enter Git history.
3. **Secrets Store CSI Driver** is the last mile. A `SecretProviderClass` tells the AWS provider what to fetch. The CSI mount happens before the main container starts, using the Pod's IRSA identity. Failure here leaves the Pod in `ContainerCreating`.

Reloader (or an equivalent restart signal) is required after a runtime secret rotates. Mounting a new file does not, by itself, make a process reread it. Environment-specific KMS keys keep an leaked `int` ciphertext from unlocking `prod`.

## Ingress, DNS, and TLS

External traffic is a controller loop, not a static load balancer object:

1. An Ingress (plus controller annotations) describes host, path, TLS, and grouping.
2. **AWS Load Balancer Controller** creates or reuses an ALB and Target Group, then registers Ready Pod IPs.
3. **External DNS** writes the hostname to Route53.
4. TLS terminates at the ALB in the current design. Certificates stay at the edge; in-cluster traffic is a separate trust decision.

Share one ALB across Ingresses when host or group annotations allow it. Each extra ALB is a fixed cost and another DNS target. Keep production and non-production traffic on separate load balancers and hosted zones so a staging Ingress cannot steal a production hostname.

## Scale and observe

Horizontal scaling is two loops:

- **HPA** changes replica count from CPU, memory, or custom metrics.
- **Cluster Autoscaler** (or an equivalent node provisioner) adds or removes nodes when Pods are unschedulable or nodes are empty.

They are not substitutes. HPA without node capacity just creates Pending Pods. Node autoscaling without HPA just gives the same replica count a bigger machine pool. Volume expansion and resource requests belong in the same conversation: a PVC that cannot grow, or a request that does not fit the node group, looks like a scale failure.

Observability is treated as three pillars with Terraform-owned foundations:

- **Prometheus** scrapes metrics.
- **Loki + Alloy** collect logs.
- **Tempo** collects traces.
- Grafana reads those backends. Alert routes belong in the infra repo when they need SNS or similar AWS objects.

Retention is a cost control, not an afterthought. Logs and traces are elastic spend; unbounded retention is a design choice.

## Six-layer defense and operations

The course's defense-in-depth model, outer to inner:

1. One AWS account per environment. Integration chaos cannot delete production, and service quotas do not leak across accounts.
2. IAM and OIDC. No long-lived access keys. Temporary credentials bound to repo, branch, or ServiceAccount.
3. Network. Nodes and data stores in private subnets. Inbound only through the ALB. Security Groups reference other Security Groups, not `0.0.0.0/0`.
4. Kubernetes RBAC, Namespaces, and IRSA. A compromised app Pod gets that app's role, not the node role.
5. Encryption chain. KMS, SOPS, Secrets Manager, CSI. Stolen etcd objects should not be the production password.
6. Deploy guards. Branch and environment parameter checks, required review, version pins.

Cost is designed, not leftover. Fixed cost includes the EKS control plane, per-AZ NAT Gateways, and ALB hours. Elastic cost includes EC2, logs, traces, and storage. The notes prefer Namespaces over extra clusters, shared ALBs, scheduled non-prod shutdown, and retention limits.

Troubleshooting follows the same layers as the request path: DNS and TLS, ALB target health, Ingress and Service endpoints, Pod probes and events, then IRSA and CSI, then node pressure and autoscaler decisions. Guessing at the application first skips the contracts that usually broke.

## Related pages

- [[terraform-infrastructure-as-code]]: infra-repo, stack outputs, account isolation, and GitHub OIDC.
- [[helm-kubernetes-package-manager]]: app-repo Helmfile layering and the apply side of the dual pipeline.
- [[aws-vpc-subnet-design]]: private subnets, NAT, Security Groups, and discovery tags under the cluster.
- [[kubernetes-networking]]: Services, Ingress, TLS, and NetworkPolicy that the controllers program.
- [[https-tls-web-security]]: certificate and handshake mechanics used at the ALB edge.
- [[opentelemetry-lgtm]]: Collector and LGTM as the observability pillars this stack deploys.

## References

