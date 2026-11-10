# OpenTelemetry and LGTM

OpenTelemetry is the shared language for producing, describing, processing, and exporting telemetry. It is not Grafana, Prometheus, or a database. LGTM is the usual open-source backend split that consumes those signals: Loki (logs), Grafana (query and correlation UI), Tempo (traces), and Mimir (Prometheus-compatible metrics). The Heptabase Interview track (Tech Stack × 工作專案, early September 2026) treats this pair as interview vocabulary, then maps it back onto close-loop work. The 2026-09-12 DevOp notes add the course spine: monitoring vs observability, the three signals, API/SDK/Collector, auto vs manual, Collector pipelines, and a Prometheus track that now has its own page ([[prometheus-metrics]]).

## OTel is the pipeline, not the store

Without OTel, each APM vendor ships its own SDK, agent, format, and metadata. Switching backends means re-instrumenting. OTel standardizes generation and transport so an app can speak vendor-neutral API, SDK, semantic conventions, and OTLP:

```
Application / Infrastructure
        |
        | API + SDK / auto-instrumentation
        v
OTel signals with Resource + Context
        |
        | OTLP
        v
OpenTelemetry Collector
        |
        +--> metrics backend (Mimir / Prometheus / vendor)
        +--> logs backend    (Loki / vendor)
        +--> traces backend  (Tempo / vendor)
```

Monitoring checks failure modes you already named (CPU over 90%, CrashLoopBackOff you already dashboarded). Observability is the ability to explore unknown internal state from external evidence — the unknown-unknowns a microservice graph creates when Pods and IPs vanish under Kubernetes. Installing OTel does not make a system observable: broken attributes, torn context, or over-sampling still leave blind spots.

## Building blocks

- **API**: the instrumentation surface (spans, counters). Libraries should depend on the API so they do not force an SDK or exporter. No-op is the point: unconfigured API calls are free.
- **SDK**: sampling, aggregation, batching, resource, processors, export. Application owners initialize it.
- **Instrumentation**: auto for framework boundaries, manual for business boundaries. Auto intercepts HTTP/SQL/messaging (Java agent, Python monkey-patch; Go is not truly automatic — you still wire contrib libraries). Manual is how a payment amount or VIP flag ever appears. Hybrid is the production default: auto as the floor, manual on the money path.
- **Semantic conventions**: shared names and attributes (`service.name`, HTTP, RPC, database, messaging) so dashboards and rules travel across languages.
- **OTLP**: the wire protocol (gRPC 4317 or HTTP 4318). It is not the storage format.
- **Collector**: an independent pipeline of receivers, processors, exporters, plus connectors and extensions. It is not one of the LGTM letters, but it is usually the ingestion path. Receivers (`otlp`, `prometheus`, `filelog`, `hostmetrics`) convert into pdata. Processors run in order: `memory_limiter` first as the OOM valve, then `attributes`, `filter` (`/healthz` noise), batching. Exporters fan out to LGTM. Sidecar vs DaemonSet vs gateway Deployment is a fan-in question, not a brand preference.

Keep metadata in the right layer. **Resource** is the producing entity (service, pod, cluster, region). **Instrumentation Scope** is which library produced the data. **Record** is the span, metric point, or log line. Stuffing request IDs into Resource, or high-cardinality IDs into Loki labels, is the usual self-inflicted outage.

## Signals, context, and sampling

Traces, metrics, and logs answer different questions. A **span** is the timed atom (operation, attributes, status). A **trace** is the tree of spans keyed by trace ID; context propagation is the invisible thread. Metrics are repeated label-sets over time (RED as the service frame). Logs are ordered evidence with variable bodies — structured beats grep, levels are a noise budget, and microservices shatter them unless the trace ID is on the line. Correlation is a configuration problem, not a default: trace ID in log labels (carefully, as a filter not a stream key), exemplar links from metrics to traces, Grafana data-source links.

The golden incident path: Grafana metric alignment → Tempo span for that second → Loki body on the same ID. Do not restart the payment worker first.

## Close-loop local stack (2026-09-20)

The close-loop notes make correlation a string-equality problem, not a brand choice. SQS → workflow → activity HTTP → hours-later callback is four processes. Grafana being up is not observability if the three backends cannot jump.

```
Go app  --OTLP gRPC:4317-->  Grafana Alloy
                              ├─ docker.sock → parse zap JSON → Loki
                              └─ by trace_id → otel-collector:4317
                                     ├─ traces → Tempo
                                     ├─ spanmetrics → Prometheus :8889
                                     └─ OTLP logs (app SDK does not send; almost empty)
Grafana  three-way jump on trace_id
```

- **One ctx, one tree.** Child spans grow from the ctx you pass. `otelgin` stores the span in `c.Request.Context()`. `context.Background()` in an activity HTTP call becomes an orphan in Tempo. `enabled: false` is a no-op TracerProvider: functions still run, nothing exports, unit tests need no collector.
- **App SDK exports traces only.** Logs are container stdout. RED metrics in Grafana are mostly Collector `spanmetrics` from spans, not hand-written Go counters. Do not expect compose to scrape `GET /metrics`.
- **Alloy is the door.** OTLP receiver on 4317/4318; Docker discovery strips the leading `/` from container names; zap JSON `trace_id` is extracted here. Field name mismatch (`traceId` vs `trace_id`) breaks the whole chain.
- **Collector pipelines are explicit.** Receiver → processor → exporter. A missing pipeline compiles and silently drops that signal. `memory_limiter` before `batch`. Spanmetrics dimensions must match span attribute keys (`case.id`, `case.mail_type`) one character off. 32s histogram buckets will not cover a 30-minute wait — `sqs.process` ends when the message is deleted; the wait lives in Temporal history. See [[durable-execution-temporal]].
- **Grafana glue is regex + exemplars**, not a prebuilt dashboard. `grafana-dashboards.yaml` points at an empty folder; local value is Explore jumps.
- **Two compose entries.** `make observability-up` starts the stack only (app hits `localhost:4317`). `make dev-otel` starts app + mocks + the same stack (app hits `grafana-alloy:4317`).
- **Callback is a new inbound trace** with `trace.Link` back to the original case. SQS span is parent of start-workflow, not of the 48-hour wait.

Production ranking is unchanged: OTel init failure must not block scanning. Observability is additive.

Context propagation is what makes a distributed trace one tree instead of many stubs. Sampling is a production design, not a demo toggle: head sampling is cheap and blind; tail sampling can keep errors and slow traces but needs a collector that sees the whole trace. Wrong sampling looks like "OTel is installed" while incidents stay invisible.

## Why LGTM is four systems

The three signals have different shapes. One database that indexes, compresses, retains, and queries all three will compromise all three. LGTM lets each backend specialize, then Grafana unifies the operator surface. The cost is several distributed systems, separate capacity plans, and explicit correlation config.

- **Loki** indexes labels, not full-text. Filter the stream first with low-cardinality Kubernetes-style labels, then parse the body. `request_id` / `user_id` as labels explode streams.
- **Tempo** stores traces, typically on object storage, and is queried by trace ID or derived indexes. It is not a log search engine.
- **Mimir** is long-term, HA, multi-tenant Prometheus storage. Cardinality still kills it.
- **Grafana** is the control surface: dashboards, Explore, alerting, and jumping between the three backends. It is not the telemetry database.

In Kubernetes, the Collector (or Grafana Alloy) is a DaemonSet or gateway Deployment that scrapes, tails, and receives OTLP, then fans out. Production interview questions are about fan-in cardinality, tail-sampling placement, and what happens when the Collector itself is down: the product path should still serve traffic; observability is additive.

## Four-step interview answer

1. Name the problem: vendor lock-in of instrumentation versus a shared pipeline.
2. Draw OTel → Collector → LGTM, and say what OTel does *not* store.
3. Pick one signal path from the user's close-loop work (a rescan span tree, a scan duration histogram, a callback 5xx log).
4. State the failure mode: torn context, label cardinality, or sampling that dropped the incident.

## Related pages

- [[prometheus-metrics]]: pull model, metric types, Kubernetes scrape, PromQL. OTel does not replace this store.
- [[go-layered-backend]]: why `c.Request.Context()` is the envelope that keeps the tree.
- [[durable-execution-temporal]]: workflow history versus OTel spans; both are needed, neither replaces the other.
- [[eks-platform-engineering]]: Terraform-backed observability pillars on EKS.

## References

