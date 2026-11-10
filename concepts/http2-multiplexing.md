# HTTP/1.1 vs HTTP/2

Why the twenty-year-old plaintext HTTP/1.1 design became a bottleneck, and how HTTP/2's binary framing, multiplexing, and header compression fix it. The mental model is a single-checkout supermarket becoming a modern logistics center.

## HTTP/1.1's three problems

1. **Layer-7 head-of-line blocking** — HTTP/1.1 is request-response serial on one TCP connection: the next request waits until the previous response is fully received. One slow large file stalls the 49 resources behind it, even if they are tiny.
2. **Browser connection limits and hacks** — browsers open ~6 parallel TCP connections per domain to compensate. Each connection pays a handshake tax (TCP SYN/SYN-ACK/ACK plus TLS handshake over HTTPS) and re-enters TCP slow start, so fresh connections start slow. Engineers patched the limit with domain sharding (split assets across subdomains), image sprites (one big image + CSS cropping), and JS concatenation — all workarounds for a protocol defect.
3. **Header bloat** — HTTP is stateless, so every request repeats near-identical plaintext, uncompressed headers (User-Agent, Cookies, Referer). Requesting a 10-byte favicon can ship ~1000 bytes of headers; wasteful, especially on mobile.

## HTTP/2's revolution

Born from Google's SPDY experiment:

- **Binary framing** — HTTP/2 splits requests/responses into small binary frames: `HEADERS` frames carry headers, `DATA` frames carry the body. Machines parse bits far faster than text lines; this is the foundation for everything else.
- **Multiplexing** — one TCP connection carries many concurrent streams, each with a unique ID. Fragments of different responses interleave in the same pipe, and the client reassembles by Stream ID like a jigsaw. This ends L7 HOL blocking, removes the 6-connection tax, and lets the single connection pass slow start quickly.
- **HPACK header compression** — a static table encodes well-known headers as single numbers, and a dynamic table caches repeated headers ("same User-Agent as request 1"). Headers shrink 80–90%.
- **Server push** — the server can predict and pre-send assets (e.g., `main.css` alongside `index.html`). Largely deprecated in practice (cache-invalidation complexity; Chrome has dropped it), but a notable attempt.

## Why this matters in Kubernetes

Protocols are for microservices now, not just people:

- **gRPC requires HTTP/2** — service-to-service RPC in Kubernetes rides on HTTP/2 for low latency, long-lived connections, and bidirectional streaming. Understanding multiplexing explains why gRPC performs the way it does at scale. The same long-lived connection is why an L4 balancer sticks every stream to one pod. That imbalance, and the contract-versus-REST choice, is on [[grpc-service-contracts]].

## Related pages

- [[https-tls-web-security]] — HTTP/2 runs over TLS in practice; the security layer beneath.
- [[computer-networking-tcp-ip]] — where HTTP sits in the layered model, and TCP slow start's role in the connection tax.
- [[kubernetes-networking]] — east-west traffic and the Ingress L7 layer that speaks HTTP.

## References
