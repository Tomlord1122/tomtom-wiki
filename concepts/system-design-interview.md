# System Design Interview: Template, NFRs, and Estimation

Senior system-design interviews score the reasoning process, not a memorized box diagram. The Heptabase Ace System Design Interviews track (Course Material board, late August 2026) treats design as a funnel: lock the problem, name the constraints, prove the scale, then draw. Jumping to high-level design first is the junior failure mode because it leaves out the "why."

## The 6-step funnel

1. **Clarify functional requirements** — lock scope. "Design TikTok" is not a spec; 45 minutes cannot cover the whole product.
2. **Define NFRs** — the north star. Every later box should exist because a constraint demanded it.
3. **Resource estimation (BOE)** — a feasibility study, not decoration. Numbers decide sharding, caches, and queues.
4. **Core entities and API design** — the contract. Prevents magic boxes that cannot carry the required dataflow.
5. **High-level design** — Services, Dataflow, Storage (SDS) mapped onto the numbers.
6. **Detailed design** — the deep dive where seniority shows: trade-offs, failure modes, mitigations.

Suggested 45-minute rhythm: 5m clarify, 5m NFR/BOE, 5m API, 10m HLD, 15m deep dive, 5m wrap-up. Spend time on high-ROI trade-offs (storage engines, consistency, tail latency), not boilerplate load balancers. Name flaws in your own design before the interviewer does.

## SDS as a completeness check

- **Services**: how logic is bounded and scaled (monolith vs microservices).
- **Dataflow**: how data moves (sync vs async, push vs pull, protocol on every arrow).
- **Storage**: where data lives and which consistency model it uses.

A finished sketch should have no blank spots: service boundary, arrow protocol, and storage model all justified.

## Four NFR pillars

| Pillar | What it forces |
|---|---|
| Scalability | Horizontal scale, load balancers, later sharding |
| Availability | Redundancy, no SPOF, failover; measured in nines |
| Performance | Latency (CDN, cache, locality) vs throughput (partition writes) |
| Consistency | Strong/ACID vs eventual; CAP shows up as a product choice |

Amazon's shopping cart is the stock example: prefer availability over a perfectly consistent cart so users can always add items. A bank cannot make that trade.

## First principles: hardware to architecture

Architectural choices should follow a reasoning chain:

`hardware latency → IOPS ceiling (1 / latency) → business QPS the box can actually serve`

Treat CPU and RAM as effectively free at interview granularity (L1/L2 nanoseconds, RAM ~100ns). Disk and network are expensive: SSD random ~150µs, same-DC RTT ~0.5ms, HDD seek ~10ms, transcontinental ~150ms. A 1,000 read QPS service on HDD cannot physically keep up (~100 IOPS per spindle), so a cache is a necessity derived from physics, not a best-practice sticker.

## QPS tiers and conversion

**1,000 QPS** is the watershed between "buy a bigger box" and "you now have a distributed system" (connection limits, lock contention, vertical-scale ROI collapse).

| Tier | QPS | Typical shape |
|---|---|---|
| Monolith | 1–100 | One app, one database |
| Scalable web app | 100–1,000 | Load balancer, replicas, cache |
| Distributed | 1k–100k | Sharding, queues, independent services |
| Global giant | 100k+ | Multi-region, CDN/edge, speed of light |

Convert DAU to RPS with the **100k shortcut**: 86,400 seconds/day ≈ 10^5, so 100k users × 1 request/day ≈ 1 RPS, with a built-in safety margin. Distinguish **RPS** (inbound API) from **QPS** (downstream queries); one user request can fan out into many storage queries. That fan-out is often the real bottleneck.

## Tail latency and peak buffers

Systems do not degrade linearly. Queuing delay dominates near ~80% utilization; a small extra burst fills queues, holds connections, and snaps. Design for **P99**, not the average: one slow dependency among many amplifies tail latency for the whole request.

Never provision for daily average. Consumer peaks are typically 2–3× average; flash sales can be 5–10×. Stay under the 80% knee, so a 10k average QPS design may need ~3× headroom.

## Worked example: TikTok-scale estimation

For 1B DAU and 100 views/user/day, the 100k shortcut yields ~1M average view RPS and ~2.3M peak. 1% daily uploaders → ~10M uploads/day (~116/s). Transcoded storage ~200MB/upload → ~2 PB/day. Those numbers justify CDN (egress is terabytes per second), object storage, and an async transcode queue: the upload API cannot transcode in the request path.

## Request lifecycle (interview narration)

A request is not "client → server → DB." Narrate the white space: DNS (cache → recursive resolver → hierarchy; DNS is a SPOF, as in Dyn 2016), TCP 3-way handshake (1 RTT) plus TLS (1 RTT in TLS 1.3, more in 1.2), L7 load balancer (TLS termination, path routing, health checks), then app / cache / storage hops. At 100ms RTT, handshakes alone can spend 200–300ms before application bytes move. See [[computer-networking-tcp-ip]] and [[https-tls-web-security]].

## Trade-offs as negotiation, not puzzles

Late-August through 1 September notes on Ace System Design Interviews treat design as a multi-stakeholder negotiation, not a riddle with one right box diagram. The engineer wants changeability; product wants a date; SRE wants pages they can survive; finance wants a bill that does not surprise anyone. Name the constraint you are buying and the one you are selling.

Four recurring axes:

- **Availability vs consistency** — see CAP below. A like-button can be wrong for a second; a ledger cannot.
- **Latency vs cost** — extra replicas, bigger caches, and multi-region writes all show up as money.
- **Complexity vs reliability** — every new moving part is another failure mode you now own.
- **Now vs later** — design the 10× you can defend this year, not the fantasy 1000× topology.

Devil's-advocate yourself before the interviewer does: state the failure mode of your own choice ("if this cache is wrong, users see yesterday's like count") and the mitigation.

## CAP in practice, then the consistency spectrum

CAP consistency means **linearizability**, not ACID's C. Availability means every non-failing node answers. Partition tolerance is not optional on a real network: the Atlantic cable, or a bad route, will split the cluster. "Pick two" is the junior slogan; the senior question is what the node does when the link is gone.

- **CP**: refuse or wait rather than be wrong. Banks, ledgers, unique inventory. Tools that block or elect a single writer.
- **AP**: answer with whatever this replica has. Feeds, presence, shopping carts that must accept the add. Staleness is a time window, not a corrupted `1+1=3`.

Inconsistency is replication delay: the New York write has not yet reached London. Strong consistency pays a latency tax waiting for everyone. Eventual consistency is the scalability default. Causal consistency is the middle: preserve happens-before for conversations ("your reply cannot appear before my message") without global linearizability. Hybrid systems are normal: payments CP, social graph AP.

The 2026-10-25 course notes say the same partition fact more sharply: on a real network P is required, so the live choice is C versus A, not "pick any two." They also name the loop that creates the tax: shard for size, then faults become normal, then replication for survival, then replicas disagree. That course track lives on [[distributed-systems-course]]. It does not replace this interview framing.

## Fan-out, celebrities, and payments

**Fan-out on write (push)** precomputes each follower's inbox. Reads are cheap; a celebrity write explodes. **Fan-in on read (pull)** stores one canonical post and assembles timelines at read time. Writes are cheap; reads pay. The hybrid is the interview answer for mixed graphs: push for ordinary users, pull for celebrities, and a For-You / recommendation path that is neither inbox.

Stripe-style payments are a different deep dive: a **Payment Intent** state machine, auth then capture, and the double-write problem between "charge succeeded" and "our DB recorded it." Transactional outbox plus a relay, plus **idempotency keys** with TTL, are how at-least-once workers do not double-charge. That is the same idempotency claim [[durable-execution-temporal]] makes for activities, in a different domain.

## Related pages

- [[computer-networking-tcp-ip]]: layer model and request tracing that this interview narration sits on.
- [[durable-execution-temporal]]: exactly-once as history plus idempotent activities, not as a broker myth.
- [[go-memory-model]]: parallel language-internals track, not interview process.
- [[distributed-systems-course]]: the course version of CAP, replication cost, and RPC ambiguity.
- [[grpc-service-contracts]]: HTTP/2 contracts and why an L4 balancer sticks a gRPC client to one pod.

## References

