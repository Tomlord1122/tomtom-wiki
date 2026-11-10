# gRPC service contracts

gRPC is not "JSON but binary." The 2026-11-03 go-backend notes split the choice into transport, contract, and streaming, then show why the same long-lived HTTP/2 connection breaks a dumb load balancer.

## Three axes, not one speed claim

1. **Transport.** gRPC requires HTTP/2. REST APIs usually still sit on HTTP/1.1. Binary framing plus Protobuf is cheaper to parse than text JSON, but the notes treat that as the surface. Multiplexing is the real transport win: many streams share one TCP connection, so you do not open a connection per call and you do not queue them. HPACK shrinks the repeated headers that microservice calls send on every request. See [[http2-multiplexing]].
2. **Contract.** A `.proto` file is contract-first. `protoc` generates the client and server interfaces, so a changed field fails at compile time. OpenAPI is document-first: it describes text, and a generated client can still disagree with what the server actually checks. For a money field, `string` versus `float64` must not be a runtime surprise.
3. **Streaming.** Unary, client-streaming, server-streaming, and bidirectional. A long-lived balance or status sync is a stream, not a poll loop and not a hand-rolled WebSocket.

## Where each belongs

Internal service-to-service prefers gRPC: one contract across many services, generated code, and less connection-pool pressure under concurrency. A public API prefers REST: a browser, `curl`, or Postman can call it without compiling a `.proto`. The notes use a payments company as the example. The split is about who the caller is, not about which protocol is "more modern."

## L4 imbalance

HTTP/2 multiplexing is good for the client and bad for an L4 balancer. An L4 balancer (NLB, LVS) sees IP and port. It picks one backend and sticks the TCP pipe there. Every later stream on that long-lived connection follows the same pipe. New pods stay idle because existing connections do not move.

An L7 balancer (Envoy, NGINX, ALB) terminates the client connection, splits the HTTP/2 frames into requests, and sends each request to a backend by round-robin or least-request. It then opens its own connections to the pods. The cost is that the balancer must understand HTTP/2. The gain is that "one client connection" is no longer "one backend."

## Related pages

- [[http2-multiplexing]]: framing, multiplexing, and HPACK, which this page assumes.
- [[kubernetes-networking]]: where an Ingress or mesh proxy sits in front of Pods.
- [[distributed-systems-course]]: RPC delivery semantics, which start once the call leaves the process.
- [[go-layered-backend]]: the service boundary a generated gRPC server still has to respect.

## References

- [[2026-11-08-heptabase-knowledge-graph]] — notes "gRPC vs REST" and "gRPC 負載平衡圖解：L4 vs L7", edited 2026-11-03.
