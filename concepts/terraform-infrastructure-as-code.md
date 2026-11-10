# Terraform: Infrastructure as Code

Terraform treats infrastructure as a desired-state program rather than a series of console actions. The working loop is:

> **configuration + provider credentials + current state → plan → reviewed apply → updated state**

This makes cloud resources reproducible, reviewable, and comparable across environments. It does not remove the need to understand the underlying platform. It makes that understanding executable, for better or for spectacularly repeatable mistakes.

## The core building blocks

- **Provider**: plugin that translates Terraform configuration into a platform API, such as AWS.
- **Resource**: an object Terraform creates or manages, such as `aws_vpc`, an IAM role, or a Kubernetes cluster.
- **Data source**: read-only lookup of existing infrastructure or external data.
- **Input variable**: configuration supplied by the caller or environment.
- **Local**: named expression for internal reuse.
- **Output**: selected value exposed after apply, often for humans or a downstream stack.
- **State**: Terraform's record of the real objects it manages and their identities. It is operational data, not a cache to casually delete.

`terraform plan` compares desired configuration against state and provider observations without changing infrastructure. `terraform apply` executes the approved graph of changes. Make plans visible in code review, because an apparently harmless input can mean replace rather than update.

## Model the dependency graph, not command order

Terraform infers dependencies from references such as `aws_subnet.private.vpc_id = aws_vpc.main.id`. This produces a graph that can be safely parallelized where possible and ordered where necessary.

Use `depends_on` only when a real dependency exists but is not visible through a value reference. Overusing it makes the graph more opaque and can serialize work needlessly. Lifecycle rules are equally sharp tools: `create_before_destroy`, `prevent_destroy`, and `ignore_changes` encode important deployment behavior, but can also hide drift or make a replacement impossible if used without intent.

## Repeatable environments

A reusable stack normally separates stable module logic from environment-specific inputs:

- Root configuration composes modules and chooses a backend, region, and environment values.
- Modules expose narrow variables and outputs, rather than reaching across unrelated module internals.
- `tfvars` files supply non-secret environment values.
- A Makefile or equivalent wraps consistent commands such as `init`, `fmt`, `validate`, `plan`, and `apply`.
- Remote state uses a backend with locking and version history. State files can contain sensitive values, so access needs the same care as production credentials.

For cross-stack values, prefer an explicit module output or a deliberately governed `terraform_remote_state` read. The latter couples stacks through a state contract, so version and protect that contract deliberately.

## Foundation stacks as APIs

A VPC foundation should expose a small, stable output surface rather than require EKS or application stacks to look inside its implementation. Typical outputs include the VPC ID, public and private subnet IDs, route-table IDs, security-group IDs, and DNS or shared-endpoint identifiers.

This makes the dependency honest: a service stack consumes an interface, not a hidden resource name. Keep outputs intentional because state can contain sensitive values. For cloud integrations, resource tags can be part of that interface too: for example, Kubernetes controllers discover eligible subnets by their tags rather than a hard-coded subnet ID.

## Two repos, two change frequencies

The current platform notes split work by blast radius, not by team fashion:

- **infra-repo (Terraform)** owns slow-changing AWS objects: VPC, EKS cluster, IAM roles, KMS, Secrets Manager containers, Route53 zones, and similar foundations. Its useful output is identity: ARN, VPC ID, cluster name, OIDC issuer.
- **k8s-app-repo (Helmfile)** owns fast-changing cluster objects: Deployments, Services, Ingress, HPA, CSI SecretProviderClass, and Prometheus rules. It consumes those identities as environment values instead of recreating the cloud objects.

The handoff is "foundation first, application after." Terraform writes outputs; Helmfile environment files such as `env/prod/aws.yaml.gotmpl` read them. A new IRSA role is created in the infra repo, published as an ARN, then referenced by a ServiceAccount in the app repo. The two pipelines stay separate so a bad Helm value cannot delete a VPC, and a Terraform reviewer does not need application-deploy permission.

Design a new component the same way: ask which stacks must already exist (VPC, hosted zone, SNS topic, OIDC provider) before choosing a resource type. Place shared primitives under `aws/`; place workload-owned objects under `service/`.

## Environments, accounts, and CI identity

Repeatable environments now include an account boundary, not only a `tfvars` file:

- One AWS account per environment (`int`, `stg`, `prod`) plus a shared account for CI. Cross-account work uses explicit `sts:AssumeRole`.
- Pin Terraform and provider versions. Protect production branches and required reviews.
- Prefer GitHub Actions OIDC over long-lived access keys. The workflow needs `id-token: write`, then `aws-actions/configure-aws-credentials` assumes an environment-specific role.
- The IAM Role Trust Policy is the real lock. A federated principal without a `:sub` condition trusts every GitHub repo that knows the role ARN. Lock `token.actions.githubusercontent.com:sub` to `repo:org/name:ref:refs/heads/main` (or an equivalent branch/environment pattern).
- Choose the role from validated environment input (`INT_ROLE_ARN` / `STG_ROLE_ARN` / `PROD_ROLE_ARN`). Do not let a workflow in the integration account assume the production role.

This is the same identity idea as IRSA inside the cluster: temporary credentials, bound to a specific caller, with Trust Policy and Permission Policy as two separate locks. See [[eks-platform-engineering]].

## Practical safety rules

1. Run `fmt` and `validate` before planning.
2. Review a saved plan before applying it to shared infrastructure.
3. Never put secrets in committed `tfvars`, plain outputs, or broadly readable state.
4. Keep network address capacity and availability-zone layout intentional from the beginning.
5. Treat state migration, resource import, and `-target` as controlled operations, not normal workflow.

## Related pages

- [[aws-vpc-subnet-design]]: Terraform implementation of a multi-AZ AWS network foundation.
- [[computer-networking-tcp-ip]]: CIDR, subnet capacity, and the network concepts behind infrastructure choices.
- [[helm-kubernetes-package-manager]]: application deployment layer that consumes Terraform outputs.
- [[eks-platform-engineering]]: IRSA, OIDC, secrets, and the six-layer defense model that sits on this foundation.

## References

