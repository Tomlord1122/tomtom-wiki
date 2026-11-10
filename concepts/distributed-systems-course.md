# Distributed systems course

The Database and Distributed System board is an MIT 6.824-style course, not an interview drill. The 2026-10-25 and 2026-11-05 notes start from "why leave one machine" and stop, so far, at RPC ambiguity and the Go concurrency the labs need. Later lectures (primary-backup, CRAQ, distributed transactions) are videos on the Course Material child board, not notes yet.

## Why a second machine creates a new problem

A single server fails all-or-nothing. Several servers fail in part, and partial failure is the normal case. The notes chain the pressure:

1. One machine cannot hold the traffic or the data, so you shard.
2. Many machines mean something is always broken.
3. Fault tolerance means a copy of each shard. That copy is replication.
4. Replication is also how replicas disagree. The client that crashes mid-write is the example: two copies can both believe they are right.

Performance, fault tolerance, physical placement, and security are the four reasons to distribute. Consistency is the tax, not the goal. This matches the interview page [[system-design-interview]], which already treats inconsistency as replication delay. The course adds the loop that forces you there, and it is stricter about CAP wording. See the tension note below.

## A timeout is not a result

Go's RPC package is the lab tool because it is small. The semantic point is older than Go. After `Transfer` over the network times out, the client cannot tell these apart:

- the request never arrived;
- the server died before running it;
- the server ran it and died before replying;
- the reply was dropped.

That is a failure to know, not a failure to execute. At-least-once means the client retries, so the server may run the call more than once. The handler must be safe to repeat, or the retry moves money twice. The interview page's payment idempotency keys are the same rule in another costume. [[durable-execution-temporal]] says the same thing about activities.

## Go concurrency for the labs

These notes are the course's view of goroutines, not the runtime anatomy on [[go-runtime-anatomy]].

- A goroutine is a lightweight thread in one address space: its own PC, registers, and stack, and a shared heap. The stack starts small (about 2KB) and grows. Go picks threads over an event loop so the lab code can stay sequential while I/O waits overlap.
- `n++` is load, increment, store. A race is shared access, at least one write, and no synchronization. The race detector only sees paths that actually ran.
- A mutex protects an invariant, not a variable. The point of the lock is that no observer sees the middle of a transfer. Channels move ownership. A master that owns the `fetched` map and receives URLs is cleaner than workers sharing that map.
- Do not start an RPC while holding the lock. Two servers can each wait for the other and never release. That is the Raft taboo in the notes.
- `sync.Cond`: the waiter does lock, `for !condition { Wait() }`, work, unlock. `for`, not `if`, because the condition can change between wakeup and reacquiring the lock. `Wait` unlocks and sleeps as one step, then must win the mutex again before it returns.

## Course map, as of this snapshot

Edited notes: introduction and why distribute, replication's hidden cost, CAP, RPC delivery, Lesson 1, Lesson 2 (goroutines through condition variables). Two-line stubs only: Go and goroutines, mutex and channel, deadlock, the next-token machine, how pretraining works. Those stubs are not yet a claim this wiki should repeat.

## Tension with the interview page

The course card title says that in modern systems P is required, so the real choice is between C and A. [[system-design-interview]] says the same thing in different words: partition tolerance is not optional, and "pick two" is the junior slogan. No contradiction in the claim. The course card is the sharper one-line version. Do not soften it back into "pick any two."

## Related pages

- [[system-design-interview]]: CAP, consistency spectrum, and payment idempotency.
- [[grpc-service-contracts]]: the transport and contract layer around RPC.
- [[go-runtime-anatomy]]: scheduler, channels, and select as runtime structures.
- [[durable-execution-temporal]]: exactly-once as history plus idempotent activities.

## References

- [[2026-11-08-heptabase-knowledge-graph]] — Lesson 1, Lesson 2, "Why Distribute at All", "Replication's Hidden Cost", "RPC Delivery Semantics", and the CAP card, edited 2026-10-25 to 2026-11-06.
