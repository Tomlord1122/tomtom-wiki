---
title: Container and Kubernetes Foundations
updated: 2026-10-10
source: Heptabase whiteboard "Container & Network Foundation"
---

# Container and Kubernetes Foundations

The DevOp board [Container & Network Foundation](https://app.heptabase.com/5c2ef201-88c7-4a10-b65e-73612dc04438/whiteboard/e09d255c-5c1d-49ff-80a5-8eff5ccf6d4f) is one course, not a pile of Docker tips. It walks from "why a process needs a private world" to "why you never talk to that process by its IP."

> A container is a process with a private view and a budget. An image is the frozen recipe. Kubernetes does not babysit the process. It keeps comparing the cluster to a YAML contract and fixing the gap.

Later pages assume this contract already exists. [[kubernetes-networking]] starts at the flat Pod network. [[eks-platform-engineering]] starts at AWS identity and the ALB. [[helm-kubernetes-package-manager]] starts once the manifests are worth packaging. This page is the path underneath them.

## Why the process gets a private world

Three failures show up before any YAML:

- **Dependency hell.** Two services want different versions of the same library. A shared host has one user-space copy.
- **Environment drift.** Laptop and server disagree on timezone, case sensitivity, compiler, or firewall. "Works on my machine" is that gap.
- **Snowflake servers.** Months of unscripted tweaks make a machine impossible to rebuild. Fear of failure follows.

A container is not a small VM. It is an ordinary process on the host kernel. **Namespaces** give it a private view: PID so it can be PID 1, NET so it has its own IP and port space, MNT so its root filesystem is not the host's. **cgroups** give it a budget so a noisy neighbor cannot eat the node.

The VM comparison is the efficiency claim, not a security claim. A VM boots a guest OS on a hypervisor. A container shares the host kernel, starts in milliseconds, and stays small because libraries live in the container's user space and only the syscall crosses into the kernel. The notes treat EC2 as the VM layer and containers-on-EKS as both: VM isolation outside, process isolation inside.

Four properties follow:

| Property | What it buys |
|---|---|
| Isolation | Two library versions can share a machine |
| Portability | The image carries the environment, so laptop and server run the same artifact |
| Lightweight | No guest OS, so density goes up |
| Immutability | You replace a bad container; you do not SSH in and patch it |

Image and container match struct and instance. The image is the static blueprint. The container is one running instance.

## The image is a stack, then a thin writable layer

An image is a stack of immutable read-only layers: base OS, runtime, dependencies, app. Registries store a shared layer once. Build cache reuses unchanged layers, which is why `go mod download` belongs before `COPY` of source.

At `docker run`, the runtime adds an empty writable layer. Copy-on-write copies a file up into that layer only when the container changes it. One hundred containers from a 500 MB image do not copy 500 MB each. Deleting the container deletes the writable layer and returns you to the image.

A production Dockerfile is two stages. `golang` builds the binary. A small runner image copies only that binary. The compiler, module cache, and source do not ship.

Distribution is tag, push, pull. A name without a registry host means Docker Hub. A company registry (ECR, Artifact Registry, Harbor) keeps the proprietary image private. Later pushes upload only changed layers.

Single-host networking is already a preview of the cluster problem:

- `docker0` plus a veth pair gives each container `eth0` and a private IP, often `172.17.0.x`.
- `localhost` inside container A is not container B. User-defined networks add DNS, so `db:5432` survives an IP change. Compose service names are that DNS.
- `-p 8000:8080` is host port to container port. The app still listens on 8080. The kernel NATs the host port through the bridge.
- A named volume (`postgres-data:/var/lib/postgresql/data`) is Docker-managed disk, not a folder on the laptop.

`docker run` by hand cannot keep a service up, scale it, balance it, or upgrade it without a gap. That shortage is the door into Kubernetes.
## Pets become cattle

The old model names the server, remembers its IP, and nurses it when it breaks. The board's cloud-native model does not depend on any one container. A failed one is replaced. What matters is whether the system matches a sentence like: "Keep 3 healthy replicas of this API, on whichever nodes, behind one name."

| Hand-run failure | What replaces the babysitting |
|---|---|
| Container dies at 3 AM | A controller notices actual count ≠ desired count and creates a Pod |
| Traffic spike | Replica count or HPA changes the desired count |
| Several replicas, no front door | A Service with one IP and DNS |
| Upgrade takes the site down | Rolling update replaces v1 with v2 while healthy Pods stay in rotation |

## The brain records intent; the muscle runs it

A cluster is a pool of CPU and memory, not a list of machines you SSH into. The control plane decides. Worker nodes execute. The notes split the brain into four, and every one of them talks through the API server. None writes etcd except the API server.

| Piece | Job |
|---|---|
| API server | Only front door. Authn, authz, validation. kubectl is an HTTP client of this API |
| etcd | Consistent key-value ledger of desired and recorded state. Raft over availability. Watch, don't poll. Lose etcd and the brain forgets the cluster even if containers are still running |
| Controller manager | Thermostats. Node, Deployment, Namespace, and others observe, compare, act |
| Scheduler | Filter nodes that cannot fit the Pod, then score the rest. It assigns a node. It does not start the container |

A `kubectl apply` is not "start my binary." The API server stores desired state. A controller notices 0 of 3 Pods and creates unscheduled Pod objects. The scheduler writes node names back. Only then does the kubelet on that node see a PodSpec.

Each worker runs the same three:

- **kubelet** — the foreman. It does not choose Pods. It starts what the scheduler assigned, restarts a dead container, and reports "still alive / out of memory."
- **container runtime** — pull the image, set up namespaces. The kubelet does not do that itself. The notes name the CRI split (containerd / CRI-O under the kubelet, not "Docker as the cluster API").
- **kube-proxy** — programs node rules (iptables or IPVS) so a Service IP reaches a current Pod IP. It is not an Nginx that receives and re-sends every packet.

If the scheduler is down for a minute, already-running Pods keep running. That is the point of the split. On EKS the provider runs those four brain processes; you still own the worker contract.

## YAML is the contract

Every object is `apiVersion`, `kind`, `metadata`, `spec`. Metadata is the sticky note (name, labels). Spec is the desired state. Labels are how other objects find this one. They are not a unique ID.

A Pod is the atom: one or more containers that share a network namespace and can share volumes. Two patterns sit inside that atom:

- **Init container** runs to completion before the app starts.
- **Sidecar** runs beside the app and adds a function without changing app code.

A raw Pod is a lesson, not a production object. If its node dies, nothing recreates it. It cannot scale, and a version change is delete-then-create. The ownership chain is Deployment → ReplicaSet → Pod.

- The ReplicaSet only counts. It finds its Pods by **label selector**, not by Pod name.
- The Deployment owns the version change. A new image creates a new ReplicaSet and shrinks the old one.
- The selector and the Pod template labels must match. That is the glue. A mismatch means the ReplicaSet does not see the Pods it created.

`spec.strategy.rollingUpdate` has two budgets:

- **maxSurge** — how many extra Pods may exist above desired. Default 25%.
- **maxUnavailable** — how many may be missing. Default 25%.

`Recreate` kills all old Pods first. The board treats that as the downtime you were trying to avoid.

## Requests schedule; limits punish

A request is a reservation the scheduler checks before placement. A limit is the ceiling enforced on the node.

CPU is compressible: hit the limit and the process is throttled, not killed. Memory is not: the kernel OOMKills the container, and a tight limit plus a restart policy looks like `CrashLoopBackOff`. The notes say to check OOMKill before blaming the code.

`requests == limits` for both CPU and memory puts the Pod in **Guaranteed** QoS. `limit > request` is Burstable. Neither set is BestEffort, and BestEffort is the first to be evicted.

Probes ask different questions, and mixing the answers causes useless restarts:

| Probe | Question | Failure |
|---|---|---|
| Startup | Have you finished booting? | Restart, but only after the startup budget |
| Liveness | Are you stuck or dead? | Kill and restart the container |
| Readiness | Can you take a request right now? | Remove the Pod from Service endpoints. Do not restart it |

HTTP GET (`/healthz`, 200–399), TCP socket, and exec are the three pokes. A process that has not crashed can still be a zombie. Liveness is for that. Readiness is for "alive, but the database connection is not up yet."

HPA changes replica count from a metric, usually CPU utilization against the **request**. No request means no utilization ratio, so HPA has nothing to scale on. Min, max, and a stabilization window exist to stop flapping.

## Pod IPs are not addresses you call

The CNI on each node wires a new Pod IP at birth. Restart, rollout, or node loss means a new IP. Hard-coding it breaks during the next rolling update.

A Service is the stable front desk:

- It gets a ClusterIP and a DNS name for its lifetime.
- A label selector defines the group. The Endpoints object lists Pod IPs that pass readiness.
- kube-proxy on every node learns that list, so a Pod on node A can call a Service whose Pods are on node B.
- Same-namespace DNS is the short name. The full name is `service.namespace.svc.cluster.local`.

Service type is who may walk up to that desk:

- **ClusterIP** — cluster-only. Databases and private services stay here.
- **NodePort** — a port on every node. The board calls this barely used in production.
- **LoadBalancer** — the cloud provisions a real load balancer and points it at those node ports.

That last type is still L4. It does not read an HTTP path. One LoadBalancer per microservice also means one cloud load balancer per microservice.

Ingress is the L7 rulebook, and it is two objects:

- The **Ingress resource** says host/path → Service. Alone, it is a row in etcd.
- The **Ingress controller** (NGINX, Envoy, or the AWS Load Balancer Controller) watches those rows and programs a real proxy or cloud load balancer.

Path routing splits one domain. Host routing splits sites. TLS termination at that edge means the app Pod often sees HTTP. One Secret renewal can cover every route. The plaintext boundary moves to the edge; "internal" is not automatically safe. [[kubernetes-networking]] keeps the longer version of that warning.

The board's AWS fork:

- **In-cluster NGINX behind an NLB** is one cheap L4 pipe, cloud-agnostic, and yours to scale and certificate.
- **AWS Load Balancer Controller** makes a real ALB do L7 and ACM TLS. With the VPC CNI it can target Pod IPs and skip NodePort. That is the pattern the notes say most companies use.

Route 53 aliases the ugly ALB DNS name to the brand domain. Multiple ALBs show up when teams or trust boundaries should not share one listener. [[eks-platform-engineering]] is the operational version of that path.

## Config does not live in the image

A hostname change should not require a rebuild. The 12-factor line on this board is: the image is the code, the environment is the config.

- A **ConfigMap** holds non-secret key-value data. Env injection is a snapshot at process start. A volume mount is a file the kubelet can refresh without you baking a new image. The app still has to re-read the file.
- A **Secret** is the same shape, Base64-encoded. Encoding is not encryption. A Secret YAML in git is a leaked password. Real protection is RBAC plus encryption at rest in etcd, and even that is not a substitute for a proper secret store. [[eks-platform-engineering]] continues this into KMS, SOPS, and Secrets Manager.
- A **Namespace** is the folder that stops an experiment from sharing a name and a blast radius with production. It is not a network firewall.
- **RBAC** is subject + Role + RoleBinding. Least privilege: the CI job that patches a Deployment does not get `delete` on the cluster. A ServiceAccount is the identity for a non-human.

## How to read this against the rest of the wiki

This board does not cover NetworkPolicy, CoreDNS internals, IRSA, or Helm. Those are already compiled:

- [[kubernetes-networking]] — flat Pod network, CoreDNS, Ingress/TLS, NetworkPolicy.
- [[eks-platform-engineering]] — account, IAM, ALB, secrets chain.
- [[helm-kubernetes-package-manager]] — stop copying these manifests by hand.
- [[computer-networking-tcp-ip]] — the packet and the address plan under the CNI.

## References
