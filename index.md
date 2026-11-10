# Wiki Index

Catalog of all wiki pages. Updated on every ingest.

## Entities

- [[cometline-project]] — Cometline: local-first AI companion desktop application.

## Concepts

### AI / Engineering
- [[reasoning-language-models]] — Modular reasoning structures, supervision, tools, test-time compute, and KV-cache inference.
- [[agentic-reasoning]] — Planning, tool use, memory, feedback, and multi-agent interaction.
- [[hermes-agent-memory]] — Nous Research Hermes: bounded MEMORY.md/USER.md, skill lifecycle, session search. Memory and skill review share one fork and one whitelist.
- [[ai-gateways]] — Gateway policies for AI traffic, including routing, limits, caching, and guardrails.
- [[ai-assisted-software-development]] — DORA's systems view of AI-augmented engineering.
- [[cloud-native-ai-infrastructure]] — Platform substrate and operational concerns for AI workloads.
- [[computer-networking-tcp-ip]] — TCP/IP layers, request tracing, CIDR/subnet capacity, Kubernetes Pod address planning, special IPv4 ranges, and troubleshooting.
- [[https-tls-web-security]] — HTTPS security model: certificates, CA trust, hybrid crypto, and TLS handshakes.
- [[http2-multiplexing]] — HTTP/1.1 bottlenecks and HTTP/2 binary framing, multiplexing, and HPACK.
- [[grpc-service-contracts]] — gRPC versus REST on transport, `.proto` contracts, and streaming. L4 balancers stick a long-lived HTTP/2 connection to one pod; L7 splits streams.
- [[distributed-systems-course]] — MIT 6.824-style course: why distribute, replication's consistency tax, CAP as C-versus-A under partition, RPC timeout ambiguity, and Go lab concurrency.
- [[container-and-kubernetes-foundations]] — Container & Network Foundation board: process isolation, image layers, control loop, Deployment contract, Service/Ingress, ConfigMap/Secret/RBAC.
- [[kubernetes-networking]] — Flat Pod networks, CoreDNS, Ingress/TLS, NetworkPolicy, EKS ALB/External DNS path, and request tracing.
- [[helm-kubernetes-package-manager]] — Charts, Values, Releases, template structure, Helmfile delivery controls, dual pipeline, and safe operations.
- [[terraform-infrastructure-as-code]]: Terraform desired-state workflow, two-repo split, account isolation, GitHub OIDC, and stack-output contracts.
- [[aws-vpc-subnet-design]]: AWS VPC boundaries, route-table and CIDR design, EKS address capacity, multi-AZ egress, Security Groups, discovery tags, and Terraform implementation.
- [[eks-platform-engineering]] — EKS identity (IRSA/OIDC), secrets chain, Ingress/ALB, HPA/CA, observability, and six-layer defense.
- [[go-memory-model]] — Go arrays, three-word slice headers, strings, maps (`hmap`/tophash), addressability, and unsafe.Pointer versus uintptr.
- [[go-runtime-anatomy]] — Anatomy of Go chapters 6–8: function frames/defer/panic, heap/GC/PACER, MPG scheduler/channels/select.
- [[go-layered-backend]] — Caller-owned interfaces: handler never SQL, service never Gin, compile-time `var _ Interface`.
- [[system-design-interview]] — 6-step interview funnel, NFR pillars, hardware-first BOE, QPS tiers, tail latency, request-lifecycle narration, CAP/consistency spectrum, fan-out, and payment idempotency.
- [[prometheus-metrics]] — Prometheus pull model, four metric types, Kubernetes scrape/discovery, PromQL instant vs range, rate vs irate.
- [[opentelemetry-lgtm]] — OpenTelemetry as the telemetry pipeline; Loki/Grafana/Tempo/Mimir as specialized backends; API/SDK/Collector, Alloy local stack, trace_id joins.
- [[durable-execution-temporal]] — Workflows-as-code, event history, deterministic replay, activities vs workflow tasks, Signal/Selector.
- [[native-feel-cross-platform-desktop]] — Tauri 2 and the native-feel cross-platform tradeoff.
- [[vercel-skills-cli-discovery]] — `npx skills add` discovers by `SKILL.md` depth, not family folder. `--skill` filters by name; a GitHub tree URL can scope one directory. `skills@1.7.4` installed one skill from `tree/master/skills/<name>` into `~/.agents/skills`.

### Cometline
- [[cometline-architecture]] — Three-layer architecture: desktop shell → agent runtime → LLM SDK.
- [[cometline-agent-runtime]] — Multi-step agent loop, built-in tools, coding-harness delegation.
- [[cometline-agent-internals]] — Runner, turn host, context window, compaction, token budget.
- [[cometline-tool-system]] — Complete built-in tool catalog: files, commands, web, memory, jobs, skills, MCP.
- [[cometline-database]] — Full SQLite schema: sessions, messages, memories, jobs, inbox, gateway.
- [[cometline-api]] — OpenAPI contract: all endpoints, SSE events, code generation pipeline.
- [[cometline-sdk-internals]] — comet-sdk provider architecture, StreamMessage, retry, typed errors.
- [[cometline-provider-sdk]] — Provider-agnostic streaming LLM I/O with retries and typed errors.
- [[cometline-coding-delegation]] — ACP: OpenCode/Claude Code/Codex delegation, subagent lifecycle.
- [[cometline-semantic-memory]] — Auto-retrieve, auto-extract, and compact persistent memory across sessions.
- [[cometline-mcp-integration]] — MCP client with OAuth. Remote tools are searched and dispatched, not registered one-for-one.
- [[mcp-apps-host-alignment]] — MCP Apps host contract. Cometline now fetches `ui://` HTML into a sandboxed panel; the postMessage bridge is still missing.
- [[cometline-jobs-scheduling]] — Durable work queue, cron scheduling, and autonomous job execution.
- [[cometline-frontend]] — SvelteKit + Electron stores, controllers, routes, settings pipeline.
- [[cometline-desktop-shell]] — SvelteKit + Electron native shell with mini-window and jobs board. Expanded git-graph cards must push later rails; a file click opens that commit's diff, not the worktree diff. Each graph row is pane width so a long message cannot slide the SHA.
- [[cometline-inbox-gateway]] — Inbox plus Discord gateway. The gateway is already a remote companion: iOS would be another adapter, and the `tomlord.io` Supabase project is not the agent database.
- [[cometline-inbox-system]] — Persistent inbox, replies, background processing, API, and desktop drawer.
- [[cometline-web-panel-and-wiki]] — Workspace file tree, previews, @mentions, wikilinks, backlinks, and panel context.
- [[cometline-model-capability-learning]] — Seven-day learned negative cache for provider/model capabilities.
- [[cometline-turn-reliability]] — Incomplete tool calls, user nudges, Codex streaming, and 10-minute idle timeout.
- [[cometline-ollama-integration]] — Ollama local LLM: catalog, pull, OpenAI compat chat, native embedder, setup wizard.
- [[cometline-terminal-panel]] — In-app terminal via xterm.js + Electron PTY, scoped per session, selection-as-context.
- [[cometline-model-roles]] — Default model pair and atomic helper-role resolution. Turn review does not fall back to Default when the extraction pin is empty.
- [[cometline-agent-controls]] — Persisted auto and research-only plan modes, provider-validated reasoning effort, and bounded execution.
- [[cometline-reasoning-replay]] — DeepSeek family reasoning_content replay contract, empty-field preservation, and the cross-provider continuation landscape.
- [[cometline-learning-path]] — Reading order through the repo's own docs/learning series for becoming fluent in the Cometline codebase.
- [[cometline-session-lifecycle]] — Startup restoration, disposable-session cleanup, unread output, navigation history, and mini-window coordination.

### 投資理財（from letf.com.tw）

- [[personal-investing-leverage]] — 用人力資本、金融資本與風險餘裕判斷投資槓桿。
- [[portfolio-concentration]] — 從整體資產而非單一 ETF 判斷集中風險。
- [[leverage-etf]] — 正二、波動耗損、50/50 策略與台灣市場特性。
- [[rebalancing]] — 再平衡機制、流動性失效案例與個人資產再平衡。
- [[lifecycle-investing]] — 生命週期投資法：年輕時提高曝險，隨年齡逐步去槓桿。
- [[contango-backwardation]] — 期貨正逆價差機制與對槓桿 ETF 的影響。
- [[cashflow-management]] — 借貸現金流：透過貸款年期調整改善現金流。
- [[financial-literacy]] — 財商知識：貨幣幻覺、槓桿的本質。
- [[investment-mindset]] — 投資心態：韭菜思維、複利拐點、長期實證。
- [[dividend-myths]] — 股息迷思：殖利率誤導、總報酬才是重點。
- [[opportunity-cost]] — 機會成本：選擇的代價是你放棄了什麼。
- [[insurance-priority]] — 保險優先順序：醫療實支 > 失能/長照 > 重大傷病 > 意外 > 壽險。
- [[income-growth]] — 增加收入：原始財富、餘裕、中產陷阱與收入增長策略。
- [[vehicle-insurance]] — 汽機車保險與投保時機。

## Syntheses

- [[heptabase-learning-focus]] — Where Heptabase study actually is: distributed-systems course, AWS Terraform platform, and gRPC. Quiet boards are listed so they are not treated as current work.
- [[cometline-current-platform-state]] — Current platform snapshot: one turn-review child at a time. Wiki review uses ten user turns; skill review uses ten tool-calling iterations. Query-wiki and the weekly skill curator remain.

- [[2026-ai-engineering-systems]] — Cross-topic synthesis: AI capability depends on the surrounding system.
- [[letf-author-knowledge-map]] — letf.com.tw 知識地圖（更新中）：收入與保險餘裕 → 資本階段 → 槓桿工具，正二不是兩倍報酬。
- [[cometline-as-ai-system]] — Cometline as an AI engineering system: how the three-layer architecture embodies AI systems principles.

## Sources

- [[2026-11-08-heptabase-knowledge-graph]] — First Heptabase graph snapshot: 731 cards, 16 whiteboards, and the text of the 93 newest notes.
- [[2026-04-02-reasoning-language-models-blueprint.html]] — arXiv, *Reasoning Language Models: A Blueprint*.
- [[2026-04-02-dora-state-ai-assisted-software-development.html]] — DORA, *State of AI-assisted Software Development 2025*.
- [[2026-04-02-cncf-annual-cloud-native-survey.html]] — CNCF, *Annual Cloud Native Survey*.
- [[2026-04-02-tauri-2.html]] — Tauri, *Tauri 2.0*.
- [[2026-07-16-agentic-reasoning-for-llms.html]] — arXiv, *Agentic Reasoning for Large Language Models*.
- [[2026-10-08-hermes-agent-memory-skills]] — Hermes Agent memory, skills, curator, and background-review cadence. Official docs plus two GitHub issues; disagreements marked.
- [[2026-10-08-hermes-background-review-fork]] — Hermes review fork: separate counters and prompts, shared memory+skills whitelist. Issue #82708 (open) plus two secondary writeups.
- [[2026-10-08-combined-turn-review]] — Cometline `a446a708`: one turn-review child after every tenth user turn. Later merged through PR 194. Old cooldown and burst gates not confirmed removed.
- [[2026-10-08-turn-review-single-flight]] — Cometline `69b134e5` on `main` via merge `3206abcb`: one review child per process; a later eligible turn replaces the single pending job.
- [[2026-07-16-kubernetes-ai-gateway-working-group.html]] — Kubernetes Contributors, *Announcing the AI Gateway Working Group*.
- [[2026-07-16-letf-leverage-investing-mindset.html]] — 大仁，〈信貸還是質押？哪種槓桿方式比較好〉。
- [[2026-07-16-letf-leverage-etf-batch1.md]] — 槓桿 ETF、正二、再平衡、正價差文章摘要。
- [[2026-07-16-letf-inventory.md]] — letf.com.tw 公開 WordPress API 的分類與文章數量盤點。
- [[2026-07-16-letf-concentration-0050.md]] — 0050 集中度相關文章摘要。
- [[2026-07-16-letf-contango-backwardation.md]] — 正價差現象與正價差難題：正2的風險。
- [[2026-07-16-letf-taiwan-leverage-edge.md]] — 全世界最適合槓桿ETF的國家：台灣。
- [[2026-07-16-letf-dividend-myths.md]] — ETF 股息與殖利率迷思、0056 隱性成本。
- [[2026-07-16-letf-decay-analysis.md]] — 正2報酬下降的元兇：盤整耗損還是大跌？
- [[2026-07-16-letf-lifecycle-investing.md]] — 生命週期投資法：改變定期定額的風險。
- [[2026-07-16-letf-loan-cashflow.md]] — 借貸新思維：提昇現金流的實用技巧。
- [[2026-07-16-letf-opportunity-cost.md]] — 機會成本：投資者一定要知道的經濟學概念。
- [[2026-07-16-letf-insurance-concepts.md]] — 醫療險 vs 壽險、實支實付、重大傷病險。
- [[2026-07-16-letf-disability-insurance.md]] — 失能險與長照險的區別。
- [[2026-07-16-letf-financial-literacy.md]] — 財商知識：貨幣幻覺與槓桿的本質。
- [[2026-07-16-letf-investment-mindset.md]] — 投資心態：韭菜思維、複利拐點與長期投資。
- [[2026-07-16-letf-insurance-details.md]] — 汽機車保險與健康檢查投保時機。
- [[2026-07-16-letf-income-growth.md]] — 增加收入：原始財富、餘裕、中產陷阱。

- [[2026-07-16-cometline-agents.md]] — Cometline AGENTS.md: development rules, commands, and architecture reference.
- [[2026-07-18-cometline-update-v0.2.76.md]] — Git capture for v0.2.76: inbox, capability learning, web panel/file tree, and UX updates.
- [[2026-08-09-cometline-session-lifecycle-update.md]] — Git capture through `26c9d92`: session restoration, startup cleanup, unread output, navigation focus, and mini-window reliability.
- [[2026-07-21-cometline-update-default-roles-tooltips.md]] — Git capture through `099dc20`: Default model roles, transition polish, thinking variants, and shortcut tooltips.
- [[2026-08-14-vercel-skills-cli-discovery]] — vercel-labs/skills README excerpts plus issues #1506 and #413: discovery depth, `--skill`, and path-scoped installs.
- [[2026-11-02-skills-cli-tree-url-install]] — `skills@1.7.4` tree-URL `--list` and global install of one skill under `skills/`, plus the PromptScript global-install failure.
## Other

- `click-test.md` — Existing manual click-path test page; not a schema page.
