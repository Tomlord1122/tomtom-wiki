# AI Gateways

An AI Gateway is gateway infrastructure for AI traffic, not necessarily a separate product category. The Kubernetes AI Gateway Working Group describes capabilities such as token-based rate limiting, fine-grained inference access control, payload inspection, intelligent routing, caching, guardrails, and AI-specific protocols.

This shifts part of AI platform design into the network layer. A reliable deployment needs policy and observability around model calls, not only a model endpoint. Gateway behavior can also become a control point for cost budgets, tenant isolation, model selection, and safety checks.

The concept connects [[cloud-native-ai-infrastructure]] with [[agentic-reasoning]], because agents increase the number and variety of model/tool interactions that need policy and limits.

## References
- [[2026-07-16-kubernetes-ai-gateway-working-group.html]]
