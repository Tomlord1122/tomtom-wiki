# Cometline Learned Model Capability Compatibility

Cometline now learns model-specific support for optional capabilities instead of hard-coding provider quirks into the agent loop.

## Problem

Providers and models may reject optional features differently. A capability can be valid for one model, unsupported by another, or exposed differently through an endpoint. Retrying the same unsupported request wastes time and produces confusing user-visible failures.

## Resolver Design

`cometmind/internal/modelcompat/resolver.go` provides two interfaces to the SDK:

- `CapabilityResolver` — resolves a policy for a provider/endpoint/model scope
- `CapabilityPolicy` — answers whether a capability is disabled and can mark it unsupported

The agent loop asks for a policy before building a request. Provider-specific rules stay behind the resolver boundary.

## Negative Cache

When a provider explicitly rejects an optional capability, the policy records a negative cache entry keyed by:

```text
provider_id + endpoint + model_id + feature
```

Entries live for **7 days** (`negativeTTL`). On later turns:

1. Read active negative entries from SQLite
2. Disable those capabilities in the resolved policy
3. Build a compatible request without them
4. Remove expired entries during cleanup

The cache is deliberately temporary: provider/model support may change, and a permanent disable would become stale configuration.

## Persistence

The `model_capability_negatives` table stores:

- `provider_id`
- `endpoint`
- `model_id`
- `feature`
- `expires_at`

The table uses the four identifying columns as its primary key and has an expiry index.

## Architectural Value

This is a capability-negotiation layer between the provider SDK and agent orchestration. It makes the system adaptive while preserving the one-way boundary:

```text
provider rejection → capability policy → normalized request
```

The agent loop does not need provider-specific conditionals.

## Relationship to static compatibility rules

The negative cache adapts to live rejections. Some contracts are known ahead of time and stay static instead, for example the DeepSeek family rule that preserves an empty `reasoning_content` on OpenCode's OpenAI-compatible path. See [[cometline-reasoning-replay]]. Both layers keep provider quirks out of the agent loop.

## References

- [[2026-07-18-cometline-update-v0.2.76.md]]
- [[cometline-sdk-internals]]
- [[cometline-agent-internals]]
- [[cometline-database]]
- [[cometline-reasoning-replay]]
- [[cometline-project]]
