# Reasoning Language Models

Reasoning language models extend ordinary LLM inference with explicit search, reinforcement learning, process/outcome supervision, retrieval, tools, and additional test-time compute. The blueprint frames chains, trees, graphs, and nested reasoning as modular structures rather than one fixed architecture.

**Working implications:** reasoning quality is an inference-and-systems property, not only a parameter-count property. Evaluation should track answer quality, process reliability, latency, and compute cost. Modular policy/value components and tool use make experimentation easier, but deployment needs budget controls and familiar training distributions.

## KV cache (inference bottleneck)

A 2026-08-23 Heptabase note on the KV cache makes the same point at the transformer layer. Naive autoregressive generation recomputes Key and Value tensors for the whole prefix on every new token, so attention work explodes with sequence length. Caching K/V in GPU VRAM turns decode into roughly linear work: compute Q/K/V for the new token only, append K/V, attend the new query against the cached past. Queries are not cached. Prefill still processes the prompt in parallel and fills the cache; decode is memory-bandwidth bound as the cache grows with layers × tokens. This is why long-context agents are a VRAM and latency problem, not only a model-quality problem.

## References
- [[2026-04-02-reasoning-language-models-blueprint.html]]
