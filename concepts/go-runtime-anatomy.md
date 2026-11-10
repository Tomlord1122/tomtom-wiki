# Anatomy of Go Runtime: Functions, Memory, Concurrency

The Heptabase **go-backend** board (2026-09-15) now holds a chapter map of *The Anatomy of Go* pp. 414–873. It is the runtime oral underneath [[go-memory-model]]. The book is the source; this page is the spine, not a substitute for the chapter cards.

## Three machines, one runtime

- **Chapter 6 — Functionality.** Function call as stack-frame contract (SP, PC, `argp`). Closures as `funcval` with value versus reference capture. Defer in three eras: heap, stack, open-coded bitmaps. Panic/recover as a `_panic` chain that `mcall`s onto g0. Observation: pprof's four sample groups and delta diffs; runtime tracer with pseudo-P; PGO feeding the compiler.
- **Chapter 7 — Memory.** Binary image (TEXT, RODATA, globals, pointer-free BSS) into a TCMalloc-shaped heap: arena, palloc chunk, 67 size classes, span, then `mcache` / `mcentral` / `mheap`. Goroutine stacks grew from segmented to contiguous `copystack`. Escape analysis, not `new`, decides stack versus heap (10MB variable cap, 64KB implicit heap move). GC is four phases (sweep termination, root sharding, concurrent mark, sweep) with a hybrid write barrier and PACER assist.
- **Chapter 8 — Concurrency.** CSP as data structures: `hchan` plus `sudog` and wait queues, `select` as `scase` with `pollorder` randomization and `lockorder` deadlock avoidance. Timers as a 4-ary min-heap on P. MPG boot: `m0`/`g0` system stacks, TLS, main goroutine. Local run queue of 256 plus `runnext`. Work stealing. Cooperative plus SIGURG async preemption via `stackPreempt` and sysmon. Network poller on epoll/kqueue.

## What this page is not

It does not retell arrays/slices/strings/maps — that stays on [[go-memory-model]]. It is the "why does a leaked goroutine survive a deploy" layer: scheduler, stacks, and GC.

## Related pages

- [[go-memory-model]]: Chapter 3 collections as machines.
- [[go-layered-backend]]: service shape that sits on this runtime.
- [[distributed-systems-course]]: the lab view of goroutines, races, mutex versus channel, and `sync.Cond`. This page stays the runtime anatomy. The course page stays the lab rules, including "do not RPC while holding the lock."

## References

