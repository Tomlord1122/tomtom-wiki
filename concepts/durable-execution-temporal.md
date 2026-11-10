# Durable Execution and Temporal

Temporal is a workflows-as-code durable execution platform. It persists Event History and replays deterministic workflow code so a long-running distributed process can survive worker crashes, network failure, and restarts. It is not a microservice orchestrator slogan, and the Temporal Service does not run business code. Workers do.

The Heptabase Temporal board (early September 2026) is the source for the product-shaped notes below. The general claim belongs on this page. The mail-security product mapping was removed on 2026-10-10.

## The pain it actually solves

A payment / inventory / notify flow can last seconds or months. If a process dies, in-memory progress, timers, and retry state die with it. Teams then invent a state machine, a queue, cron, a dedup table, and a recovery job. Temporal writes execution progress to durable history so the same workflow function can rebuild state and continue.

```
Client ---- start/signal/query/update ----> Temporal Service
                                             |
                                             | durable Event History
                                             v
                                          Database
                                             ^
                                             |
Worker ---- long-poll Task Queue ------------+
  - runs Workflow code
  - runs Activity code
```

- **Temporal Service**: coordinates execution, persists history, owns timers, dispatches tasks.
- **Worker**: deployed by the app team, polls a task queue, runs workflow and activity code.

## Workflow, activity, command, event

- **Workflow**: the durable business process. State is rebuilt by replaying history, so the code must be deterministic: no hidden I/O, no `time.Now` as a business decision, no random without a workflow API.
- **Activity**: the side effect that may talk to the world (HTTP, DB, email). Activities fail, time out, and retry. They usually need idempotency.
- **Workflow Task**: "replay this workflow and emit the next commands." Not a business activity.
- **Activity Task**: "run this activity function." Result or failure becomes history events.

A typical `ExecuteActivity(ChargeCard)` cycle: worker receives history → SDK replays to the current point → workflow emits a Schedule Activity command → service records it and dispatches an activity task → activity result becomes an event → the next workflow task continues from there. Replay is how memory is reconstructed, not a debug feature.

## What Temporal is not

It is not a replacement for an ingress queue when upstream already delivers work as messages. It is not a substitute for a business database: case status still lives in something like db-manager. It is not observability: OTel spans tell you the request tree; Temporal UI tells you which signal a workflow is waiting on. See [[opentelemetry-lgtm]].

Server-side, Temporal does not execute the workflow function inside its own process. If the worker is down, history still grows timers, but no new activity runs. Capacity planning is therefore about worker concurrency versus waiting executions: thousands of parked workflows can be cheap; unbounded concurrent activities are not.

## Close-loop instantiation (2026-09-20)

A new root **Temporal** board (21 cards) plus close-loop notes restate the same split in this repo's language.

- **Workflow decides; Activity touches the world.** FN may wait 30 minutes for remediation and 48 hours for SOLVE. `time.Sleep` in a goroutine dies on deploy. Replay uses `workflow.Now(ctx)` and `workflow.NewTimer`; a successful activity is not re-executed because its result is already in history.
- **Caller-owned activity interfaces.** `workflow.go` must not import `internal/client`. Activities take stdlib `context.Context` so OTel spans continue. See [[go-layered-backend]].
- **Skip is a product rule, not a Temporal feature.** `checkShouldSkipSolutionFlow` skips when TMASE ≥ 5.0 and mail type matches.
- **Do not increment metrics in workflow code.** Replay will count again. Counters live in an activity or interceptor.
- **Signals are named history channels, not sockets.** `GetSignalChannel` plus `NewTimer` plus `Selector.Select` wait for callback or timeout. Worker restart does not sleep 48 hours again. Workflow ID is `caseId` so a callback that only has the case can `SignalWorkflow(ctx, caseID, "", const, payload)` with empty runID (current execution). `ALLOW_DUPLICATE_FAILED_ONLY`: a live ID is not replaced; a failed one may restart. A typo in the signal name does not fail compile; the workflow waits until timeout.
- **SQS only opens the door.** `ExecuteWorkflow` success means input is in Temporal's DB; then delete the SQS message immediately. Do not hold visibility for 48 hours. AGENTS.md: signal is the only resume — do not write the DB as a backup path when callback fails.
- **Test without Docker:** `WorkflowTestSuite` + `env.SignalWorkflow`.

OTel spans and Temporal history answer different questions: `sqs.process` ends when the message is deleted; the 48-hour wait lives in history, not in that span. See [[opentelemetry-lgtm]].

## Related pages

- [[opentelemetry-lgtm]]: traces versus workflow history.
- [[system-design-interview]]: async pipelines and exactly-once as interview trade-offs, not as a product claim.

## References

