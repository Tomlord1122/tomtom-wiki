# Prometheus Metrics, Scrape, and PromQL

Prometheus is the pull-model metrics system the 2026-09-12 Heptabase DevOp notes treat as Kubernetes' de facto monitor. It is not Grafana, not OpenTelemetry, and not a log store. OTel can emit Prometheus-readable metrics; Prometheus still scrapes, stores time series, and answers PromQL. See [[opentelemetry-lgtm]] for the pipeline that often sits in front.

The notes use an ML inference pipeline as the running example. The durable claims are the data model, pull philosophy, Kubernetes discovery, and query types — not that particular workload.

## What Prometheus actually is

Four pieces, one binary at the center:

- **Prometheus Server** — Retrieval (the puller), TSDB (Gorilla-style compressed samples), HTTP API + PromQL.
- **Exporters** — translators for things that do not speak `/metrics` (Node, Blackbox, a thin adapter over a C++ engine).
- **Pushgateway** — a holding pen for jobs shorter than the scrape interval. Not a general collector. It is a SPOF and does not forget dead jobs; batch jobs only.
- **Alertmanager** — grouping, dedup, routing. Prometheus decides the threshold; Alertmanager decides who gets paged.

Pull is a control-plane choice. The server owns cadence and backpressure. A flood of application traffic does not also flood the monitor. In Kubernetes, the server watches the API instead of asking every Pod for the monitor's address.

## Time series, not events

Prometheus records changing numbers, not event text. A series is:

```
<metric name>{label set} <value> <timestamp>
```

The application increments in memory. Every scrape interval the server snapshots `/metrics`. Logs would store every request; Prometheus stores the latest count (or gauge, or bucket) at scrape time.

Four metric types:

| Type | Shape | Use |
|---|---|---|
| Counter | only goes up (resets on restart) | request totals, bytes, errors — always `rate()` / `increase()` |
| Gauge | up and down | queue depth, memory, in-flight |
| Histogram | cumulative buckets + `_count` / `_sum` | latency, size; P99 from buckets, not from an average |
| Summary | client-side quantiles | precomputed quantiles; less aggregable across instances |

RED (Rate, Errors, Duration) is the service-level frame. USE (Utilization, Saturation, Errors) is the resource-level frame. Averages lie; histogram buckets are how you talk about tail latency.

High-cardinality labels (`user_id`, `trace_id`, `request_id` as a series key) explode the TSDB. The same cardinality rule Loki has for stream labels. See [[opentelemetry-lgtm]].

## Scrape and Kubernetes discovery

`scrape_configs` names a `job`, interval, path (`/metrics`), and targets. `static_configs` is for addresses that never move. Pods move.

`kubernetes_sd_configs` watches the API. Roles: `node` (host metrics), `pod`, `service`, and usually **`endpoints`** for application scrapes — the Service's live backends, not the ClusterIP.

Relabeling is the filter after discovery: keep only annotated Pods, map Kubernetes labels onto Prometheus labels, drop `/healthz`. Prometheus Operator (ServiceMonitor / PodMonitor) is the modern YAML for that mapping so the server itself is not a snowflake.

When a target disappears, Prometheus marks it stale. Chasing old Pod IPs is the failure mode of static configs under HPA.

## PromQL: instant vs range

Every query is about time.

- **Instant vector** — one sample per series at a timestamp. A snapshot. Gauges plot directly: `http_requests_total{status="200"}`.
- **Range vector** — all samples in a window: `http_requests_total{status="200"}[5m]`. Cannot plot as-is. It is input to functions.

Grafana's pretty line is many instant queries, one per pixel.

`rate()` is average per-second increase over the window — the alerting and trend function. `irate()` uses the last two points — sensitive, noisy, not for alerts. Counters need these; a raw counter only grows.

Aggregation (`sum by`, `avg by`) is how you survive many replicas. Histogram quantiles (`histogram_quantile`) are computed from buckets, then you still `sum by` the right labels or you mix incompatible shapes.

## Related pages

- [[opentelemetry-lgtm]]: OTel API/SDK/Collector; LGTM backends; Prometheus often sits behind a Collector `prometheus` receiver or beside Mimir.
- [[kubernetes-networking]]: why Pod IPs are not scrape targets you type by hand.
- [[eks-platform-engineering]]: platform observability layer on EKS.

## References

