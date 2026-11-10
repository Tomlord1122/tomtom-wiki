# Agentic Reasoning

Agentic reasoning treats an LLM as a system that can plan, act through tools, and learn from interaction rather than only answer in a closed-world prompt. A 2026 survey groups the space into foundational capabilities, self-evolving behavior through feedback and memory, and collective multi-agent reasoning.

The useful distinction for system design is between **in-context orchestration**, which spends test-time interaction and structured tool use, and **post-training optimization**, which changes behavior through supervised fine-tuning or reinforcement learning. This makes agent quality depend on more than the base model: memory, tools, feedback loops, coordination, evaluation, and governance all become part of the product.

For a personal AI companion or coding agent, the practical questions are what actions are permitted, how long-horizon work is checkpointed, how memory is selected, and how failures are surfaced to the user. These concerns connect [[reasoning-language-models]] to [[ai-assisted-software-development]] and [[2026-ai-engineering-systems]].

[[hermes-agent-memory]] is one concrete self-evolving design: a char-capped frozen snapshot for facts, skills for procedures, and FTS session search for everything else. The review that writes those stores is a counter-gated background fork, not a fine-tune.

## References
- [[2026-07-16-agentic-reasoning-for-llms.html]]
- [[hermes-agent-memory]]
