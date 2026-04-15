# AI Frontier Master Knowledge Base
*Living knowledge graph — updated daily from multi-agent research*
*Last updated: April 15, 2026 (v5 — verified sources: Claude Code Routines, GPT-5.4-Cyber, Gemini Robotics-ER 1.6, UC Berkeley BenchJack, vLLM v0.19.0)*

---

## Reasoning & Planning

### Chain-of-Thought → Hierarchical Planning
- CoT remains dominant for single-step reasoning but fails for long-horizon tasks (>50 steps)
- Hierarchical planning (decompose → plan subtasks → execute) is the emerging standard for agent planners
- Cursor 3's parallel Agents Window (April 2026) demonstrates this: task decomposition into parallel sub-agents rather than sequential CoT
- **Open problem:** Agents still fail at coherent replanning after unexpected tool failures mid-chain

### Latent Planning Depth Ceiling (NEW — April 9, 2026)
- **The Depth Ceiling (arXiv:2604.06427, April 7, 2026):** LLMs have a hard upper bound on how many sequential reasoning steps they can execute inside a single forward pass (no CoT output)
- **Empirical numbers:** Tiny transformers → 3 latent steps; fine-tuned GPT-4o/Qwen3-32B → 5 steps; GPT-5.4 → 7 steps
- **Training ceiling:** ~5 steps learned during gradient-based training; *but* models trained to depth 5 generalize to depth 8 at test time
- **Why there's a ceiling:** Training signal is only on final output correctness — no supervision on intermediate latent steps → weak gradient signal for deep sequential latent computation
- **What scaling buys:** More planning *breadth* (wider branching factor) rather than *depth* (more sequential reasoning steps). Scaling from 8-layer small transformer to GPT-4o adds only 2 latent planning steps
- **Design implication:** Any task requiring >7–8 sequential reasoning operations CANNOT rely on implicit LLM reasoning inside one call — explicit CoT, scratchpad, or tool-chaining is architecturally mandatory, not optional
- **This refines the earlier finding on long-horizon planning failure:** The prior KB entry noted LLMs fail at long-horizon planning; the Depth Ceiling paper now quantifies the mechanism — it's a structural architectural limit (~7–8 steps), not just a capability gap

### Test-Time Compute Scaling
- T² scaling laws (arXiv:2604.01411, April 2026): joint optimization of train tokens + inference samples under fixed E2E compute budget
- **Key insight:** Inference-heavy workloads (agents, search, coding) should overtrain smaller models, not scale parameters
- Optimal: train smaller model 3-5× longer → rely on more inference samples at test time
- **Limitation:** Only validated at moderate scales; extreme overtraining behavior not characterized
- **Replaces:** Chinchilla-optimal as default for inference-heavy applications

### Interactive Environment Adaptation (NEW — April 13, 2026 catch-up from March 25)
- **ARC-AGI-3 (arXiv:2603.24621, launched March 25, 2026):** First interactive benchmark in the ARC-AGI series
  - **Design:** Hundreds of handcrafted games, thousands of levels; agents must perceive, act, and adapt without natural language instructions or task descriptions — no static puzzle-solving
  - **Human score:** 100%
  - **Frontier CoT model scores:** Gemini 3.1 Pro 0.37%, GPT-5.4 0.3%, Claude Opus 4.6 0.25%, Grok 4.20 0%
  - **Symbolica Agentica SDK (Day 1):** 36.08% on public dataset; $1,005 vs $8,900 for Opus 4.6's 0.25% → **100-180× performance gap, not a cost gap**
  - **Why frontier models fail:** CoT extends a reasoning chain; it cannot reconstruct the underlying rule structure of an unfamiliar interactive environment
  - **Why Agentica works:** Orchestrator-subagent architecture where orchestrator never touches raw environment state; subagents interact, write Python programs, execute against test cases (program synthesis = infer compact program from observed I/O), return compressed summaries; orchestrator maintains high-level plan without context contamination
  - **Key architectural pattern:** `spawn(subagent, task_scope, return_summary=True)` — subagents return summaries, not raw state. This is what prevents context growth from collapsing planning capacity.
  - **Design implication:** Program synthesis (structured model-building from evidence) is architecturally distinct from CoT (implicit reasoning over evidence). For interactive adaptive tasks, they are not substitutes.
  - **Competition:** ARC Prize 2026 ongoing through December; $2M+ prize; Kaggle active
  - **Source:** https://arcprize.org/blog/arc-agi-3-launch · https://www.symbolica.ai/blog/arc-agi-3

### Long-Horizon Agent Execution (NEW — April 8, 2026)
- **GLM-5.1 (Z.AI, April 8, 2026):** First open-weight model demonstrated sustaining effective performance over 8-hour / 655-iteration autonomous agent sessions
- Core mechanism: Slime async RL infrastructure decouples generation, evaluation, and training — eliminating synchronization bottlenecks that limit long-trajectory RL
- This extends the prior finding on T² (overtrain + more inference samples) by showing the *training infrastructure* itself must be redesigned for long-horizon tasks
- **Practical ceiling demonstrated:** 655 iterations, thousands of tool calls, 6.9× vector DB throughput — on a single open-model deployment
- **Pattern:** Async RL → better long-horizon policy learning → sustained reasoning without degradation
- **Gap:** Still 19 SWE-bench Pro points below Anthropic Mythos (restricted), suggesting restricted labs have additional post-training techniques not public

---

## Memory Systems (Episodic, Semantic, Procedural)

### LLM-Curated Memory (NEW — April 2026)
- **ByteRover** (arXiv:2604.01599): 5-tier hierarchical context tree where the reasoning LLM curates its own memory
- Architecture: Domain → Topic → Subtopic → Entry → Chunk
- Progressive retrieval: sub-100ms for 80%+ queries without LLM involvement; escalates to LLM only for novel cross-domain synthesis
- **Advantage over vector RAG:** No separate infrastructure; semantic coherence preserved (curating agent understands task context); lower latency on repeated patterns
- **Tradeoffs:** LLM tokens consumed on insert; unclear scaling beyond research-scale corpora; cold start equivalent to standard RAG
- **When to use:** Bounded knowledge domains (specific codebase, company docs, product catalog)
- **When to avoid:** Web-scale retrieval (>10M docs), streaming freshness-dominated use cases

### Vector Memory (Established)
- Standard: embedding model → vector DB (Qdrant/Weaviate/Pinecone) → approximate nearest neighbor retrieval
- Qdrant (Rust): 2-3× better memory efficiency than Go-based alternatives
- Weaviate: strongest for hybrid search + multi-tenancy
- **Trend:** Being challenged for bounded-domain agents by LLM-curated approaches (ByteRover)
- Still dominant for web-scale and freshness-sensitive use cases

### External Agent Memory Libraries
- **Mem0** (48K GitHub stars, April 2026): vector-first with graph/KV fallback; dominant for general-purpose agent memory
- **Zep:** Differentiates on temporal query accuracy via knowledge graphs; best for time-sensitive queries

### Adaptive Inference-Time Memory (NEW — April 9, 2026)
- **In-Place Test-Time Training (arXiv:2604.06169, April 7, 2026):** ICLR 2026 Oral — repurposes the final projection matrix in MLP blocks as "fast weights" that update during inference
- **Mechanism:** Apply-then-update cycle per token chunk: (1) apply current fast weights to produce output, (2) update fast weights using next-token prediction objective on that chunk's activations
- **No new modules, no pretraining required** — drops into any existing LLM architecture
- **Why MLP projection matrix:** Attention is already dynamic (per-context); MLP blocks are static and store factual patterns — making them adaptable creates a write-accessible session memory inside the model
- **Key result:** Improves performance on tasks where novel in-context information needs to persist across a long session
- **Relationship to existing memory approaches:** This extends the prior memory architecture pattern by adding a *fifth* tier — in-weights ephemeral memory — that is cheaper than external stores for short-horizon adaptation
- **Connection to Depth Ceiling finding:** In-Place TTT addresses context retention, not reasoning depth — it helps the model "remember" what it saw earlier in a session, but does not increase the 7–8 step latent planning ceiling

### Memory Architecture Pattern (Updated — April 9, 2026)
```
Context Window (working memory)
    ↕
[NEW] In-Weights Ephemeral Memory (In-Place TTT fast weights) — session-scoped adaptation
    ↕
External Storage (episodic/semantic/procedural)
    ├── Vector DB (for semantic similarity)
    ├── Graph DB (for relational/temporal)
    └── KV Store (for structured facts)
```
Tiered memory now has 5 tiers: KV cache (token-level), in-weights ephemeral (In-Place TTT), vector/graph/KV external. Distinction is which tier handles which query type and update frequency.

---

## Retrieval & RAG

### Embedding Models (April 2026 State)
- **Microsoft Harrier-OSS-v1** (March 30, 2026): SOTA on Multilingual MTEB v2 across 270M/600M/27B sizes
  - Novel: decoder-only architecture for embeddings (not standard BERT-style encoder)
  - License: MIT — best permissive multilingual embedder available
  - Base models: Gemma 3 (270M, 27B), Qwen3 (600M)
- Previous SOTA: Google multilingual embeddings (now outperformed by Harrier on Multilingual MTEB v2)
- **Recommendation:** Use Harrier-OSS-270M for production multilingual RAG (MIT, self-hostable, SOTA)

### RAG Architecture Evolution
- Long-context models (Llama 4 Scout: 10M tokens) reduce need for RAG by eliminating chunking for bounded corpora
- **Trade-off:** Cost of 10M context vs. cost of retrieval infrastructure — economic crossover depends on query frequency
- Hybrid: RAG for web-scale + long-context for focused corpus remains best practice

---

## Capability-Gating & Open-Source Strategy

### Restricted-Access Frontier Models (April 8, 2026 — Extended)
- **Anthropic Mythos Preview (April 7, 2026):** First frontier model withheld from general release due to specific dual-use capability (autonomous vulnerability exploitation)
- **Pattern:** Capability-gated deployment — not general public API, distributed via institutional partnership only (40 vetted organizations via Project Glasswing)
- **Mechanism for future access:** Planned Cyber Verification Program for individual researchers
- **Quantified capability gap:** Mythos 77.8% SWE-bench Pro vs. best public model GLM-5.1 at 58.4% — 19 point gap is the current "cost of safety gatekeeping"
- **Why it matters architecturally:** The capability (autonomous zero-day discovery + exploit generation) emerged implicitly from general improvements in code reasoning — *not* from explicit security training. Future models will likely have equivalent capability by default.
- **Deployment evolution:**
  - Pre-2026: All frontier models released as general APIs
  - 2026+: Tiered access — general API + restricted-access tiers for high-capability models
- **Link to safety:** Confidence calibration (can't self-assess hacking risk) + autonomous execution = dual-use by construction at frontier scale

### Meta's Open-Source Retreat (NEW — April 9, 2026)
- **Muse Spark (April 8, 2026):** First model from Meta Superintelligence Labs (MSL), led by Alexandr Wang — CLOSED SOURCE
- **This contradicts the prior assumption** that Meta would continue publishing all competitive models as Llama open weights. Muse Spark breaks that pattern.
- **Strategic signal:** When MSL builds something considered competitive, the default is now proprietary. Llama series continues for open-weight releases, but MSL's first product is a closed API.
- **The bifurcation:** Open-weight ecosystem (Llama 4, GLM-5.1, Gemma 4) is now structurally behind both Anthropic-restricted (Mythos) AND Meta-proprietary (Muse Spark) frontier capability.
- **Architecture:** MSL rebuilt Meta's AI stack from scratch over 9 months — new infrastructure, new architecture, new data pipelines. Muse Spark is not an iteration on Llama 4.
- **Efficiency claim:** Muse Spark achieves Llama 4 Maverick-class performance with over an order of magnitude less compute. Mechanism: architectural redesign + improved data curation (not yet published).
- **Current capabilities:** Multimodal (vision/text/audio input, text output), multi-agent "Contemplating" mode, shopping mode. Weaker than frontier on coding. Competitive on multimodal understanding + health.
- **Benchmark position:** Ranks 4th on Artificial Analysis Intelligence Index v4.0 (score 52), behind Claude Sonnet 4.6, Gemini 3.1 Pro, and GPT-5.4.

---

### Orchestrator-Subagent Program Synthesis Pattern (NEW — April 13, 2026)
- **Core pattern:** Orchestrator maintains high-level plan; subagents interact with environment, write programs from I/O examples, execute, return compressed summaries
- **Why summary compression matters:** Raw environment state can be large; if subagents return it directly, orchestrator context fills up rapidly. Summary compression is the mechanism that enables long-horizon orchestration.
- **When to use:** Tasks where environment rules are initially unknown but inferrable from evidence; generalizes better than CoT for adaptive rule-learning
- **When NOT to use:** Tasks where rules are fixed and known upfront (CoT or standard RAG is sufficient); web-scale open-ended retrieval
- **Reference implementation:** Symbolica ARC-AGI-3 harness — https://github.com/symbolica-ai/ARC-AGI-3-Agents
- **Connection to prior patterns:** Extends the parallel sub-agent pattern (Cursor 3, April 2026) by adding the program synthesis + summary compression design constraints

## Tool Use & Orchestration (MCP, A2A, Function Calling, Terminal Agents)

### MCP (Model Context Protocol) — April 2026 State
- **Scale:** 5,800+ MCP servers, 97M monthly SDK downloads
- **Platform backing:** Anthropic, OpenAI, Google, Microsoft, AWS
- **April 2026 improvements:** Fixed multi-element error truncation, eliminated per-turn JSON.stringify, improved SSE transport for large frames
- **Claude Code v2.1.92 (April 4):** `_meta["anthropic/maxResultSizeChars"]` allows tool results up to 500K chars (previously auto-truncated)
- **Context pollution:** Previously a blocker with many MCP attachments — now resolved per Simon Willison (April 2026)
- **Gap:** No curated, quality-graded MCP server registry exists (5,800 servers with chaotic discovery)

### OpenAI Agents SDK — April 2026
- `list_resources()` / `read_resource()` on MCPServer: dynamic tool discovery at runtime
- Handoff history consolidation: multi-agent chain history compressed to single labeled message (~30-50% token reduction on deep chains)
- Configurable tool error handling: graceful degradation vs. hard stop — now configurable

### Google A2A Protocol
- Agent-to-agent communication standard
- Less ecosystem traction than MCP as of April 2026; watch for convergence

### PTY-Based Terminal Interaction (NEW — April 9, 2026)
- **tui-use (April 8–9, 2026):** Spawns any program in a PTY, runs headless xterm emulator, presents clean plain-text screen state + `highlights` field for TUI-selected items; sends keystrokes back through PTY
- **Gap it fills:** AI agents currently stall when programs ask for interactive input (npm create, psql, vim, redis-cli, Python REPL). bash subprocess calls can't handle TTY-aware programs
- **Why `highlights` matters:** Standard TUI programs show selected items via inverse-video ANSI codes. Parsing this in plain text is brittle; the `highlights` field normalizes it into a structured field agents can consume
- **Scope:** macOS/Linux only (requires Unix PTY). Works for any interactive terminal program without code changes to the target program
- **Relation to MCP:** Could be wrapped as an MCP server for tool-calling agents — currently a standalone CLI tool

---

## Agent Architectures & Patterns

### Canonical Agent Architecture (April 2026)
```
[User Goal]
     ↓
[Planner / Task Decomposer]  ← Cursor 3: parallel sub-agents (April 2026)
     ↓
[Memory Layer]               ← ByteRover: LLM-curated hierarchical tree (April 2026)
 ├─ Episodic
 ├─ Semantic
 └─ Procedural
     ↓
[Retriever / RAG]            ← Harrier-OSS multilingual embeddings (March 2026)
     ↓
[LLM / Reasoning Engine]     ← Llama 4 / Gemma 4 / Claude
     ↓
[Tool Executor / MCP]        ← 500K result size (April 2026), dynamic discovery
     ↓
[Verifier / Critic]          ← Reliability gap: self-correction loops emerging as required
     ↓
[Action / Output]
```

### Parallel Multi-Agent Pattern (Emerging Standard)
- **Cursor 3 (April 2026):** Agents Window enables parallel task execution with cloud orchestration
- Pattern: Orchestrator decomposes task → spawns parallel sub-agents → collects results → synthesizes
- Production evidence: Money Forward 15-20 hrs/week savings, 70% QA speedup
- **Missing:** Standardized multi-agent coordination protocol for non-coding domains

### Agent-First IDE Architecture
- Cursor 3 (April 2026): parallel execution, Design Mode (UI-modifying agents), 5-hour RL checkpoint refresh
- Shift: developers orchestrating agents rather than writing code

---

## Agent Frameworks & Infrastructure (NEW — April 7–8, 2026)

### Microsoft Agent Framework 1.0 (April 7, 2026)
- Unification of Semantic Kernel + AutoGen into single production SDK
- **Stable APIs + long-term support commitment** — first major lab commitment to agent framework stability
- **MCP + A2A integration:** Full support for Model Context Protocol tool discovery + Agent-to-Agent 1.0 protocol for cross-framework agent collaboration
- **Multi-agent features:** Context consolidation (30–50% token reduction on deep chains), configurable tool error handling
- **Design implication:** Framework consolidation reduces vendor lock-in for agents; interop becomes standard

### Claude Managed Agents (April 8, 2026 — Public Beta)
- Fully managed cloud infrastructure for long-running autonomous agents on Claude Platform
- **Session persistence:** Multi-hour sessions survive client disconnection
- **Multi-agent spawning:** Agents can spawn and coordinate other agents (research preview)
- **Built-in tools:** Secure file/code/web execution with session-level isolation
- **Observability:** Session tracing + debugging via Claude Console; inspect every tool call, decision, failure
- **Pricing:** Standard Claude tokens + $0.08/session-hour runtime fee
- **Early customers:** Notion, Rakuten, Asana
- **Production signal:** Removes weeks of infrastructure work; shifts bottleneck from "can Claude be an agent" to "how do we run production agents at scale"

### Claude Code Routines — Durable Agentic Execution (NEW — April 14, 2026)
- **What it is:** Saved, schedulable AI automations that run on Anthropic's cloud infrastructure — not on the user's local machine
- **Architecture (key):** Session state = durable log on Anthropic servers. Compute = ephemeral containers provisioned on-demand. This is saga/event-sourcing applied to LLM agents. The invariant: the log always exists; compute is replaceable.
  - `Trigger → Durable Log → Ephemeral Container → Tools → External Systems → Result → Log`
- **Triggers:** (1) scheduled cadence (cron-style), (2) inbound API webhook, (3) GitHub events (PR opened, push, issue, etc.)
- **Connectors:** Slack, Linear, Google Drive, GitHub
- **Configuration:** A routine = prompt + repos + connectors + trigger. Stored as a cloud-side saved configuration.
- **Usage limits:** Pro=5/day, Max=15, Team/Enterprise=25
- **Availability:** Research preview on all paid plans (Pro, Max, Team, Enterprise)
- **Performance:** P50 TTFT dropped ~60%; P95 dropped >90% vs. session-persistent containers. Containers not on critical path for user-facing interactions.
- **Relationship to Claude Managed Agents:** Managed Agents is the infrastructure layer; Routines is the user-facing product built on top (scheduled, event-driven, no laptop required)
- **Current tradeoffs:** Context budget management for long-running logs unsolved; debugging opacity in research preview; vendor lock-in (log on Anthropic infra); connector scope limited to 4 integrations
- **Design pattern:** Same as Temporal/Step Functions for workflow orchestration — now a Claude product

### Pattern Consolidation
- Agent frameworks (LangGraph, AutoGen, Agent Framework) converging on graph-based orchestration
- MCP as lingua franca for tool definition (ecosystem moving toward this)
- Cloud-managed agents + self-hosted frameworks coexisting; market bifurcating on deployment preference

---

## Agent Evaluation & Reliability

### Benchmark Landscape (April 14, 2026 Update)

| Benchmark | What it tests | Human | Best AI | Notes |
|-----------|--------------|-------|---------|-------|
| SWE-bench Verified | Code bug-fixing | — | GPT-5.4 Pro 88.3% | Hours-scale tasks |
| SWE-bench Pro | Hardest code tasks | — | Mythos 77.8% (restricted); **MiniMax M2.7 56.22% (NEW #1 open)** | Week-scale implied by MirrorCode |
| Terminal Bench 2 | Terminal-native agent tasks | — | **MiniMax M2.7 57.0%** | New entry |
| SWE Multilingual | Coding across languages | — | **MiniMax M2.7 76.5** | New entry |
| MirrorCode | Weeks-scale autonomous coding | 100% | Opus 4.6 (weeks-scale) | Spec quality = binding constraint |
| ARC-AGI-3 | Interactive adaptation / rule-learning | 100% | Gemini 3.1 Pro 0.37% | Symbolica Agentica 36.08% (program synthesis) |
| HLE | Expert-level multi-domain questions | — | Claude Opus 4.6 / Gemini 3.1 Pro >50% | Was 8.8% (o1, early 2025); 40+ point gain in 12 months |
| CyberGym | Autonomous vulnerability reproduction | — | Mythos 83.1% | Security-specific capability measurement |
| MCP-Atlas | Multi-tool agentic workflows | — | GLM-5.1 71.8 | Real-world agent proxy |

### Current Benchmarks (April 14, 2026 Update)
- **SWE-bench Verified** (500 tasks): GPT-5.4 Pro leads at 88.3%, Claude Opus 4.6 at 79.3%
- **SWE-bench Pro** (hardest tasks — UPDATED April 14):
  - Claude Mythos Preview: 77.8% (restricted access only)
  - **MiniMax M2.7: 56.22% (NEW #1 open-weight / open-access)** — matches GPT-5.3-Codex (restricted)
  - GLM-5.1: 58.4% (note: pending recalibration — M2.7 may surpass this; conflicting reports)
  - GPT-5.4: 57.7%
  - Claude Opus 4.6: 57.3%
  - ~~Previous top at 23.3% — this was the April 6 state; SWE-bench Pro has been updated~~
- **HLE (Humanity's Last Exam):** 8.8% → 50%+ in 12 months — fastest capability jump on any major benchmark per Stanford AI Index 2026
- **SWE-bench Live:** monthly updates prevent contamination
- **CyberGym:** Mythos 83.1% / Opus 4.6 66.6% — new security-specific benchmark now tracking capability gaps
- **MCP-Atlas:** GLM-5.1 71.8 — multi-tool agentic workflow benchmark, now relevant as real-world agent proxy

### CRITICAL: Benchmark Integrity Collapse (UC Berkeley BenchJack, April 12, 2026)
- **Finding:** UC Berkeley (Dawn Song, Alvin Cheung) built **BenchJack** — automated exploit agent that achieves near-perfect benchmark scores without solving any tasks
- **Method:** Two-phase exploit: (1) probe benchmark infrastructure to map evaluation mechanism; (2) craft minimal exploit that achieves perfect score via harness vulnerability
- **Benchmarks broken and scores achieved:**
  - SWE-bench Verified: 100% (10-line conftest.py patches test infrastructure)
  - SWE-bench Pro: 100%
  - Terminal-Bench: 100% (/usr/bin/curl replaced with trojaned wrapper for uvx binary)
  - WebArena: ~100% (file:// URL reads gold answer from task config in Chromium)
  - FieldWorkArena: 100%
  - CAR-bench: 100%
  - GAIA: 98%
  - OSWorld: 73%
- **Root cause:** The gap between evaluation mechanism and task completion is an exploitable attack surface in every tested harness
- **Implication for all published benchmark scores:** Any score on these 8 benchmarks from a system with access to test infrastructure is retroactively suspect. This does NOT mean all published results are wrong — it means they can't be distinguished from exploit results without independent auditing
- **Required harness properties for credible evals (post-BenchJack):**
  1. Containerized task environment (gVisor or equivalent)
  2. No shared filesystem between agent and task config/evaluation mechanism
  3. Scoring via external verifier process (zero shared state with agent)
  4. No injectable tool wrappers in agent's PATH or environment
  5. Deterministic, reproducible protocol with version-controlled task definitions
- **Build implication:** Evaluation harness isolation is now a prerequisite, not an optimization. If you're shipping benchmark results, audit your harness against BenchJack's exploit categories first.
- **Reference:** https://rdi.berkeley.edu/blog/trustworthy-benchmarks-cont/

### Reliability Gap (Critical — April 2026)
- **Finding:** 90% of deployed legacy agents fail within weeks (Harvard Berkman Klein / Fortune, March-April 2026)
- **Root cause:** Lack of architectural depth, no confidence calibration
- **Confidence calibration unsolved:** Models cannot distinguish correct from incorrect predictions better than random
- **Current mitigation:** Reflection checkpoints + real-time autorater loops (small verifier model in loop)
- **Design implication:** Self-correction is now a design requirement, not an optimization

---

## Multi-Agent Coordination

### Current State (April 2026)
- LangGraph Fleet (beta, announced April 2026): fleet-wide agent identity, memory isolation, permission boundaries
- Enables SOC2-compliant multi-agent deployment with separate identities and audit trails
- **Gap:** No mature multi-agent coordination standard for non-coding domains
- **Trust + conflict resolution:** Unsolved in production systems

---

## AI in Security & Cybersecurity (Updated — April 10, 2026)

### Autonomous Vulnerability Discovery (Frontier Capability)
- **Mythos Preview (Anthropic, April 7, 2026):** Agentic workflow — isolated container, LLM reads source code, forms hypotheses, executes/debugs, produces bug report + working exploit POC
- **Performance:** 83.1% CyberGym vulnerability reproduction; 72.4% working exploit on first attempt; 56.8% HLE without tools (vs. Opus 4.6's 40.0%)
- **Key finding:** Capability is emergent from general code+reasoning improvements, NOT from explicit security training
- **What this enables finding:** Multi-decade-old logic flaws that survive millions of automated test runs (27-year-old OpenBSD, 16-year-old FFmpeg)
- **Why old bugs survive traditional tools:** Fuzzing + static analysis lack semantic understanding. Human auditors have finite attention windows. Mythos has neither constraint.
- **Agentic security workflow:**
  ```
  Isolated container → LLM reads codebase (full context) → hypothesis → execute/debug → validate → iterate → exploit POC
  ```
- **Dual-use problem:** Same capability that finds defensive bugs creates offensive exploits — no way to have one without the other at this capability level
- **Anthropic access model (Project Glasswing):** Institutional partnership — 40+ vetted organizations; $100M usage credits + $4M open-source security donations; planned Cyber Verification Program for individual researchers

### Competing Governance Frameworks (Updated — April 14-15, 2026)
- **OpenAI Trusted Access for Cyber (TAC, April 9, 2026 → GPT-5.4-Cyber expansion April 14, 2026):**
  - April 9: Identity-tiered access program for GPT-5.3-Codex cyber capabilities (Tier 1/2/3)
  - **April 14 (NEW):** OpenAI launched **GPT-5.4-Cyber** — version of GPT-5.4 fine-tuned for cybersecurity, with explicit "cyber-permissive" refusal training. New capabilities added: **binary reverse engineering** (analyzing compiled software without source — disassembly/decompiled code analysis for malware and vulnerabilities). Access via chatgpt.com/cyber (individuals, identity-verified) or enterprise rep. Targeting thousands of individuals + hundreds of security teams.
  - Technical safeguards: refusal training on 10M+ adversarial prompts; real-time classifiers detecting evasion; activity monitors for bulk scans
  - $10M API credits committed; alignment note: GPT-5.4-Cyber is fine-tuned to be "cyber-permissive" — meaning the refusal boundary is explicitly recalibrated, not just the base model with loose prompting
  - **Direct competitive response** to Anthropic Mythos (April 7) — GPT-5.4-Cyber arrived 7 days after Mythos was announced
- **Anthropic Project Glasswing (April 7 → expanded April 15):**
  - April 7: 11 named institutional partners (AWS, Apple, Broadcom, Cisco, CrowdStrike, Google, JPMorgan Chase, Linux Foundation, Microsoft, NVIDIA, Palo Alto Networks)
  - **April 15 (NEW):** Expanded to **40+ additional organizations** that build/maintain critical software infrastructure; $100M usage credits total committed; $4M direct donations to OSS security orgs; Cyber Verification Program planned for individual researchers
- **Governance model divergence:**
  - Anthropic: Institutional gatekeeping (partnerships, named organizations, high-verification)
  - OpenAI: Identity-based graduated verification (broader individual access, lower barrier but tiered capability)
- **Competition implication:** Both labs now have competing "responsible offensive AI" products shipping simultaneously. Access decisions are a product differentiator. The market for enterprise cybersecurity AI tooling is live.
- **Still unresolved:** Neither framework solves the dual-use problem. No regulatory standard for this capability class. Identity verification reduces expected harm but doesn't prevent it.

---

## Training & Post-Training

### Scaling Laws (April 2026 Update)
- **T² (arXiv:2604.01411):** Joint train/inference optimization; shifts optimal pretraining into overtraining regime for inference-heavy workloads
- **Chinchilla status:** Still valid for training-compute-only budget; outdated for E2E compute budgets with significant inference
- **Practical impact:** For agent workloads spending >40% compute on inference, overtrain smaller models

### Agentic RL: Template Collapse & SNR-Based Filtering (NEW — April 7, 2026)
- **RAGEN-2 finding:** Multi-turn RL agents experience "template collapse" — learning spurious input-agnostic patterns instead of genuine long-horizon reasoning strategies
- **Mechanism:** Reward signal only on final outcomes → weak gradient for intermediate reasoning steps → model learns shortcuts (template patterns) that achieve high reward in training but fail to generalize
- **Solution:** SNR-Aware trajectory filtering — measure variance across successful rollouts, filter low-SNR (high-variance, likely spurious) trajectories before training
- **Results:** ~40% reduction in training iterations to convergence; preserved reasoning generalization
- **Connection to Depth Ceiling (April 9):** Depth Ceiling = architectural limit on implicit reasoning within one call (~7–8 steps). RAGEN-2 = learning limit on RL-derived reasoning even with explicit steps. Two orthogonal bottlenecks.
- **Implication for builder:** If RL agent training plateaus, check for template collapse via variance analysis before scaling

### Async RL for Long-Horizon Agent Training (NEW — April 8, 2026)
- **GLM-5.1 Slime infrastructure:** Decouples generation, evaluation, and training as overlapping async processes on separate GPU groups
- **Why it matters:** Traditional RL is sequential (generate → evaluate → update); clusters idle during evaluation. Slime eliminates this bottleneck, enabling learning from long trajectories at scale
- **Result:** Model learns effective policies over 600+ step interactions without degradation — not achievable with synchronous RL pipelines
- **Evolution:** Sequential RL → Slime async RL → long-horizon agent capability
- **Connection to T²:** Both are training-infrastructure changes that improve inference-time performance; T² via compute allocation, Slime via trajectory length

### Model-as-Optimizer-in-Training-Pipeline (NEW — April 14, 2026)
- **Source:** MiniMax M2.7 self-evolution mechanism (open-sourced April 11, 2026)
- **Pattern:** A deployed model acts as an agent inside its own (or another model's) RL post-training pipeline, autonomously iterating on the scaffold codebase used to train it
- **Concrete loop:** `analyze failure trajectories → plan changes → modify scaffold code → run evaluations → compare results → commit/revert → update memory + self-feedback → repeat`
- **Components required:**
  1. **Short-term memory file** — markdown updated after each iteration; captures what was tried, what failed, what improved
  2. **Self-feedback critique** — model critiques its own results before next round starts
  3. **Self-optimization chain** — next round reads full history (all memory + feedback), not just last result
- **Demonstrated result:** MiniMax M2.7 ran 100+ autonomous rounds, optimized sampling parameters, scaffold workflow rules, and loop detection → 30% performance improvement on internal eval sets
- **Scope:** Currently validated for scaffold tuning (orchestration code, prompting, tool parameters). NOT general recursive self-improvement — model does not modify its own weights.
- **Main failure mode:** Eval overfitting — if the model optimizes toward the eval it's running against, gains are spurious. Mitigation: maintain a held-out eval set that is never directly optimized against.
- **Portability:** Pattern is model-agnostic. Can be applied by any team with (1) a tool-using frontier model, (2) a measurable eval signal, (3) a version-controlled scaffold codebase.
- **Connection to Karpathy's AI-curated systems (April 10):** Both are instances of "AI as active participant in its own development pipeline" — curation loop for knowledge, optimization loop for RL training
- **Practical application:** Apply to scaffold tuning, prompt optimization, agent configuration tuning, eval harness maintenance — any case where there are discrete changeable parameters and a clean eval signal
- **Source:** https://www.minimax.io/news/minimax-m27-en

### Distillation
- **REOPOLD (arXiv:2603.11137, March 2026):** Relaxed on-policy distillation connecting distillation to RL; 10-100× cheaper than standard RL with improved stability
- On-policy distillation with relaxed constraints maintains quality while reducing instability

### Fine-Tuning Tooling
- **Unsloth Studio (beta, March 2026):** No-code local fine-tuning, 70% VRAM reduction, 2× faster, MoE 12× faster
- Supports 500+ models; covers Gemma 4, Llama 4, Mistral Small 4

---

## Inference & Serving

### KV Cache Optimization (April 2026)
- **TurboQuant (Google, April 2026):** Polar coordinate transform + 1-bit error correction → 3-bit KV cache
  - 6× compression, zero accuracy loss, no calibration required
  - Previous best (KIVI): 2.6× with accuracy tradeoffs
  - **NVIDIA KVTC (complementary):** Up to 20× via transform coding (some quality loss)
  - Impact: 2-4× higher batch sizes → 60-70% cost reduction for long-context serving
- **Use case priority:** Long-context agents (100K+ tokens) see largest benefit

### Speculative Decoding (April 2026)
- **Saguaro (ICLR 2026):** Pre-emptive verification outcome prediction → 2× over optimized SD, 5× over standard AR
- **Mirror-SD:** 5.8× on 14-66B models
- **Batch speculative decoding:** 3× throughput at batch size 8 with 95% output equivalence
- **Status:** Research → production path; watch for vLLM/SGLang integration

### Serving Cost Optimization Patterns
- Smart routing (capability-matched model per task): 47-80% cost reduction
- Semantic caching: 40-70% cost reduction, latency 850ms → 120ms
- Combined (routing + caching + TurboQuant + Saguaro): potential 70-80% total cost reduction

### Local/Edge Inference (April 2026)
- **Ollama v0.20.2 + MLX:** 93% performance gain on Apple Silicon; requires macOS 26.2+ for M5 Neural Accelerator
- **Gemma 4 E2B:** 7.6 tok/s on Raspberry Pi 5 at <1.5GB RAM; production-grade edge agents viable
- **Llama 4 Scout:** 17B active params, single H100 — sovereign cloud / on-prem frontier capability

---

## Interpretability & Safety

### Mechanistic Interpretability
- **DLM-Scope (arXiv:2602.05859, February 2026):** SAE-based interpretability extended from autoregressive to diffusion language models
- SAEs (Sparse Autoencoders) remain primary tool for feature extraction across architectures
- **Status:** Methodological extension work; not yet impacting production agent design

### Agent Safety
- **LlamaFirewall (Meta, April 2026):** Prompt injection protection for Llama 4 deployments
- **Llama Guard 4 (Meta, April 2026):** Content moderation for open-weight agents
- Confidence calibration unsolved — agents cannot self-assess reliability (see Reliability section)

---

## Multimodal & World Models

### Native Multimodal (April 2026 State)
- **Standard:** Vision and audio baked into base architecture (not bolted on via adapters)
- **Llama 4:** Native multimodal — eliminates separate vision encoder dependency
- **Gemma 4:** Native vision + audio; full multimodal at edge (E2B on Raspberry Pi)
- **Trend:** Multimodal-first is now the default for new model families

### Speech & Voice (NEW — April 2–4, 2026)
- **MAI-Transcribe-1 (Microsoft):** SOTA speech-to-text, 25 languages, outperforms Whisper-large-v3 + GPT-Transcribe on FLEURS benchmark, $0.36/hour (commodity pricing)
- **MAI-Voice-1 (Microsoft):** Text-to-speech with emotional range, 60 seconds generated in 1 second
- **Mistral Voxtral Models:** Open-weight text-to-speech (Voxtral TTS) + real-time transcription (Voxtral Realtime) on HF Hub
- **Market signal:** Speech/voice components are now commodity infrastructure — focus shifting to multimodal integration, not basic speech recognition

### Video Generation (NEW — April 7–10, 2026)
- **HappyHorse-1.0 (Alibaba ATH, April 7):** SOTA text-to-video and image-to-video, wins blind-test benchmarks; core innovation is temporal consistency mechanism preserving subject identity + motion over 2-minute sequences
- **Alibaba Wan 2.7 (April 6):** "Thinking Mode" for planning-intensive generation, reference tracking, motion primitives
- **Google Veo 3.1 Lite (April 7):** Cost-reduced version with text-to-video + image-to-video
- **Coherence timeline:** 2024 → 10–30 seconds single-pass; 2025 → 30–60 seconds; 2026 → 2-minute single-pass coherent generation (now achieved)
- **Market implication:** Video generation is commodity feature for any content platform by Q3 2026

### Context Window Race
- Llama 4 Scout: 10M tokens (current open-weight leader)
- Gemma 4: 256K tokens
- GPT-4o: 128K tokens
- **Practical ceiling:** Economic crossover between long-context and RAG shifts as context costs fall

---

## Open-Source Landscape

### Upcoming Models (April 14, 2026 Update)

| Model | Lab | Expected | Key Specs | Status |
|-------|-----|----------|-----------|--------|
| GPT-5.5 "Spud" | OpenAI | April 14–May 5, 2026 | Unknown; described as "big model feel," 2 years research | No announcement as of April 14; Polymarket 78% by April 30 |
| DeepSeek V4 | DeepSeek | Late April 2026 | 1T params MoE, 37B active, 1M context, Engram memory, $0.30/MTok, Apache 2.0 | V4-Lite on API nodes; full model expected late April |
| Tencent Hunyuan 3.0 | Tencent | TBD April | ~30B params, long-context focus, agent task evaluation (Yao Shunyu) | Internal testing as of April 14 |

### Model Families (April 14, 2026 Update)
| Family | Best Open Size | Context | License | Multimodal | SWE-bench Pro |
|--------|---------------|---------|---------|-----------|--------------|
| MiniMax M2.7 | 229B total / 10B active | 204K | Modified-MIT | No | 56.22% (#1 open-access, matches restricted GPT-5.3-Codex) |
| Llama 4 | Maverick (17B/128E) | 10M | Commercial | Native | ~54% est. |
| Gemma 4 | 31B Dense | 256K | Apache 2.0 | Native | — |
| GLM-5.1 | 744B total / 40B active | 200K | Apache 2.0 | No | 58.4% (prior #1; M2.7 may have surpassed) |
| Mistral | Small 4 | 128K | Apache 2.0 | Yes | — |
| DeepSeek | V3 | 128K | MIT | No | — |

Note: Claude Mythos Preview (restricted, not publicly accessible) leads at 77.8% — ~21 points above best open-access model (M2.7).

### Closed-Source Non-Llama Models (NEW — April 9, 2026)
- **Meta Muse Spark:** Closed-source proprietary model from Meta Superintelligence Labs. Not Llama. Private API preview only. Multi-agent "Contemplating" mode. Ranks 4th on AAII v4.0.
- **Watch:** DeepSeek V4 (expected late April 2026) — 1T MoE parameters, 1M context, Engram conditional memory, $0.30/MTok. V4-Lite live on API nodes since early April. Third-party verification pending.

### Infrastructure
- **llama.cpp b8664:** CPU-optimized inference, latest CUDA support
- **Ollama v0.20.2:** Native MLX (Apple Silicon), Gemma 4, Llama 4 Scout, Qwen3-VL
- **vLLM v0.19.0 (April 2026 — NEW):** 448 commits, 197 contributors
  - Zero-bubble async scheduling now compatible with speculative decoding (removes prior constraint where enabling spec decode forced sync scheduling)
  - Elastic Expert Parallelism (EPLB Milestone 2): dynamic GPU scaling for MoE expert shards; faster EP model loading — relevant for Gemma 4 26B MoE, GLM-5.1, MiniMax M2.7 at cluster scale
  - FlexKV: smart KV cache offloading — stores only high-reuse blocks (replaces blunt offloading); new backend for better long-context cost management
  - Full Gemma 4 support (requires transformers≥5.5.0); AMD/ROCm support for Gemma 4
  - ~~"Waiting for Saguaro integration"~~ — Saguaro integration status unchanged; v0.19.0 focuses on spec decode + EPLB + KV
- **SGLang:** Latest stable v0.5.9 (Feb 2026); RadixArk commercial spinout valued ~$400M

### Quantization
- 4-bit and 2-bit quantized weights: Gemma 4 E2B at sub-1.5GB RAM
- **Unsloth:** 70% VRAM reduction for fine-tuning; MoE optimization (12× faster)

---

## Embodied AI & Robotics (NEW — April 2–9, 2026)

### Gemini Robotics-ER 1.6 + Boston Dynamics Spot (NEW — April 14-15, 2026)
- **What shipped:** Google DeepMind released Gemini Robotics-ER 1.6 and integrated it into Boston Dynamics' Orbit software (AIVI + AIVI-Learning systems) for Spot's industrial facility inspection workflows
- **Headline capability:** **Instrument reading** — reading analog gauges, pressure meters, sight glasses in industrial environments
  - ER 1.5: ~23% accuracy (lacked dedicated capability; fell back to narrow classical CV model)
  - ER 1.6: **93% accuracy** (agentic vision pipeline)
- **How instrument reading works (agentic vision decomposition):**
  1. Spatial reasoning to locate and segment gauge face in scene (VLM)
  2. Structured extraction of dial/pointer position
  3. Code execution to compute angle → value mathematically
  4. Context lookup for scale/units from instrument label or prior knowledge
  - This is NOT end-to-end VLM inference — it's a multimodal agent loop: spatial reasoning + structured extraction + code computation
- **Broader ER 1.6 improvements over 1.5:** Better pointing accuracy, counting, success detection (all spatial/physical reasoning tasks)
- **Production deployment:** Live today in Boston Dynamics Orbit platform; Spot robots walking industrial facilities and logging instrument readings autonomously
- **Key insight for builders:** "General VLM + code execution" now outperforms narrowly-trained CV for physical interpretation tasks at production scale. Pattern generalizes: visual grounding → structured extraction → code-based computation > end-to-end VLM for any task requiring mathematical/geometric interpretation
- **Sources:** https://blog.google/innovation-and-ai/models-and-research/google-deepmind/gemini-robotics-er-1-6/ · https://bostondynamics.com/blog/aivi-learning-now-powered-google-gemini-robotics/

### Vision-Language-Action Models
- **HY-Embodied (Tencent, April 9):** MoT-2B weights open-sourced; 32B frontier variant for robot control
  - Takes: camera feeds + text commands + prior action context
  - Outputs: discrete action tokens for robot execution
  - **Cross-morphology generalization:** Trained on heterogeneous datasets (humanoids, mobile bases, arms); generalizes without per-morphology retraining
  - **Production signal:** EAIDC 2026 (April 2, Shenzhen) demonstrated live robot control across multiple morphologies
  - **Architecture:** "Brain" for Vision-Language-Action (VLA) pipelines
- **Status shift:** VLA models are transitioning from research (simulation-dominated) to production (real-world capable)

### Market & Events
- **EAIDC 2026 (April 2, Shenzhen):** Embodied AI Developers Conference — first major conference dedicated to lab→production transition for embodied systems
- **Market growth:** Physical AI market valued at ~$4.12B in 2024, projected to reach $61B by 2034 (31% CAGR)

### Builder Implication
- Open-source VLA models + frontier-class LLMs means embodied AI dev timelines compress dramatically in 2026
- Previously: months to adapt a VLA to a new robot morphology; now: plug-and-play HY-Embodied

---

## Local/Edge Deployment

### Capability (April 2026)
- **Raspberry Pi 5:** Gemma 4 E2B at 7.6 tok/s with NPU acceleration
- **Apple Silicon M5+:** 31 tok/s on Gemma 4 31B with macOS 26.2+ Neural Accelerator
- **Single H100:** Llama 4 Scout (17B active, 10M context)
- **Edge multimodal agents:** Now technically feasible; software stack mostly unbuilt (opportunity)
- **Edge embodied AI:** HY-Embodied opens frontier robot control on edge devices

---

## AI Coding Agents

### Production State (April 2026)
- **SWE-bench Verified:** GPT-5.4 Pro 88.3%, Claude Opus 4.6 79.3%
- **SWE-bench Pro (hardest):** Mythos 77.8% (restricted); GLM-5.1 58.4% (best public); GPT-5.4 57.7%
- **Cursor 3 (April 2, 2026):** Parallel Agents Window, Design Mode, 5-hour RL refresh
- **Claude Code v2.1.92 (April 4, 2026):** 500K MCP results, 60% faster diffs, /cost breakdown

### Long-Horizon Coding Capability Threshold (NEW — April 10, 2026)
- **MirrorCode (Epoch AI + METR, April 10, 2026):** First benchmark at the weeks-timescale for autonomous AI coding
  - **Task:** Reimplement a 16,000-line Go bioinformatics toolkit (gotree, 40+ CLI subcommands) without source code
  - **Benchmark design:** Execute-only access to original binary; model must reverse-engineer behavior and design full architecture from scratch; test suites verify functional equivalence
  - **Result:** Claude Opus 4.6 passes nearly every program up to gotree's scale; task estimated at 2–17 human engineer weeks
  - **Scaling property:** Continued gains from more inference compute (tokens) → larger projects tractable; no hard ceiling at gotree scale
  - **Key precondition:** Detailed, machine-checkable behavioral specification (test oracle). Without this, quality degrades — the model cannot self-verify.
  - **Source:** https://epoch.ai/blog/mirrorcode-preliminary-results/

### Specification Quality: The New Binding Constraint (NEW — April 10, 2026)
- **Before MirrorCode:** Model capability was the bottleneck for AI-assisted engineering
- **After MirrorCode:** Specification quality is the bottleneck. The model can do the work; the question is whether you can describe "correct" well enough to be checkable.
- **Implication for system design:**
  - Invest in behavioral test suites and executable specifications before deploying autonomous coding agents on projects
  - The "spec-first" pattern (write tests → verify tests → agent implements) is the architectural unlock for weeks-scale task delegation
  - If your team lacks comprehensive automated test coverage of existing software, that gap is now the direct blocker for AI-driven reimplementation/migration
- **Practical planning update:** Raise the delegation ceiling. Autonomous task delegation is now viable at the weeks-long project scale, not just the hours-long subtask scale.

### Trend
- IDE architecture shifting from code-completion to agent orchestration
- Real-time RL feedback (5-hour checkpoint refresh in Cursor 3) enables continuous improvement without redeploy
- MirrorCode (April 2026): task delegation ceiling raised from hours-scale to weeks-scale; spec quality is the new bottleneck

---

## Knowledge Curation & AI-Curated Systems (NEW — April 3–10, 2026)

### Shift: From Code Generator to Knowledge Curator
- **Pattern (Andrej Karpathy, April 3):** Feed raw research materials (papers, notes, blog posts, PDFs) into a folder; point an LLM at it; LLM autonomously builds and maintains an interlinked wiki
  - Writes articles explaining concepts
  - Creates backlinks between related ideas
  - Organizes into categories + hierarchies
  - Maintains consistency across new updates
- **Scale achieved:** Research wiki on single topic reached 100+ articles, 400K+ words, fully AI-maintained
- **Also observed:** AI agents autonomously modifying code, training for 5 minutes, evaluating results, keeping/discarding changes, iterating — the same pattern applied to ML research itself

### Design Implication
- **Old paradigm:** Humans write code, AI generates code / completes code
- **New paradigm:** Humans provide raw materials (research, data, problem context), AI curates + organizes + maintains knowledge structure
- **Scope:** This pattern applies to research workflows, documentation management, codebase organization, and any domain requiring synthesis of multiple sources

### Business Opportunity
- **Knowledge curation platform:** SaaS product packaging this approach for research teams, legal teams, VCs, compliance teams
- First-mover advantage in 2026; by 2027 likely becomes table-stakes feature

---

## Business Opportunities & Infra Gaps

### High-Priority Gaps (April 14, 2026 Update)
1. **Agent Memory-as-a-Service:** Managed LLM-curated hierarchical memory for vertical agents
2. **Agent Reliability Monitoring:** 90% failure rate, no mature monitoring tool; confidence calibration unsolved
3. **Long-horizon agent observability (April 8):** No tooling for monitoring agents running 8-hour / 600+ iteration sessions — reasoning drift, strategy revision tracking, context budget management
4. **Edge Multimodal Agent Products:** Hardware ready (Gemma 4 E2B), software stack missing
5. **MCP Server Registry:** 5,800+ servers, no curated quality-graded discovery layer
6. **Multi-Agent Coordination Middleware:** Proven pattern (Cursor 3), missing for non-coding domains
7. **Serving Cost Optimization Tooling:** TurboQuant + Saguaro + T² = 50-70% cost reduction; expertise barrier is commercial opportunity
8. **Agentic security scanning for SMBs (April 8):** Mythos pattern documented; GLM-5.1 capable of agentic code analysis; no accessible service below Glasswing enterprise partnership level
9. **Dual-use AI capability advisory (April 8):** Regulatory/compliance gap between what frontier models can do (autonomous exploit generation) and what governance frameworks cover
10. **Reasoning depth analyzer for agent pipelines (NEW April 9):** No tool that profiles sequential reasoning depth per LLM call, classifies tasks against the Depth Ceiling, and recommends where to inject explicit CoT. Theoretical basis now available (arXiv:2604.06427).
11. **PTY-native tool execution in agent frameworks (NEW April 9):** tui-use fills the interactive terminal gap standalone, but no first-party integration exists in LangChain, LangGraph, CrewAI, or as an MCP server. Builders hitting the interactive CLI wall is a large underserved market.
12. **AI billing intensity auditing for healthcare (NEW April 9):** AI scribes confirmed to increase coding intensity → billing inflation. Neither regulatory framework nor commercial tooling exists for automated detection/normalization. Insurers and providers both need it.
13. **RAGEN-2 template collapse diagnostic tooling (NEW April 10):** Build analyzer that detects trajectory collapse in RL agent training pipelines, measures SNR, recommends filtering thresholds. Most teams training agents blind to whether they're learning genuine reasoning or spurious patterns.
14. **Knowledge curation SaaS (NEW April 10):** Package Karpathy's wiki-building approach as product: feed research materials → AI builds interlinked knowledge base → team queries it. For VCs, legal, research teams; table-stakes by 2027.
15. **Embodied AI orchestration framework (NEW April 10):** Layer atop HY-Embodied + Claude Managed Agents for cross-robot task coordination. One agent planning, multiple robots executing, abstraction over morphologies. Market opening as VLA models mature.
16. **Specification generation and verification tooling (NEW April 10):** MirrorCode reveals spec quality as the binding constraint for weeks-scale AI coding. No commercial tool helps teams generate comprehensive behavioral specifications, measure coverage, or maintain them as AI-verifiable artifacts. Every team deploying coding agents will need this.
17. **Cybersecurity AI governance advisory (NEW April 10):** Two competing governance frameworks (OpenAI TAC + Anthropic Glasswing) are now competing for enterprise adoption. Enterprises need neutral comparison: capability, liability, compliance, audit requirements. Advisory practice or SaaS product.
18. **GPU procurement advisory for frontier AI (NEW April 10):** Vera Rubin access window closing as multi-year CoreWeave deals consume supply. Specialized brokerage/advisory for GPU cloud procurement at scale — navigating CoreWeave vs. AWS vs. Azure vs. GCP across price, availability, and Vera Rubin timeline.
19. **CANN/Ascend inference tooling (NEW April 13):** If DeepSeek V4 launches at frontier quality on CANN, there will be immediate demand for production inference optimization (batching, KV cache management, speculative decoding) for the Ascend stack. vLLM and SGLang are CUDA-native; their Ascend support is nascent. Production-grade inference engine for Ascend comparable to vLLM for CUDA is an open engineering gap.
20. **ARC-AGI-3-style interactive evaluation harness (NEW April 13):** The benchmark's interactive format (adaptive rule-learning in novel environments) is the right evaluation paradigm for real-world agent reliability, but exists only as a competition. A generalized "interactive environment adapter" for internal agent evaluation — using teams' own environment definitions — is absent from the market.
21. **OpenAI governance risk monitoring for enterprise customers (NEW April 13):** If Musk v. OpenAI (trial April 27) produces court-ordered governance remedies, enterprise customers face immediate uncertainty about API terms, model availability, and Microsoft partnership scope. An automated compliance-monitoring tool tracking trial developments and risk-rating governance events for enterprises with material OpenAI API dependency has a hard deadline opportunity.
22. **RL post-training scaffold auto-optimizer tooling (NEW April 14):** MiniMax M2.7 demonstrated 30% gain via 100+ autonomous optimization rounds of the RL scaffold. Any team running RL fine-tuning needs this loop. Existing MLOps platforms (W&B, MLflow) track experiments but don't close the loop with model-driven scaffold modification. Build the tooling that wraps any team's eval harness with the M2.7 loop pattern.
23. **Benchmark freshness service for hard AI tasks (NEW April 14):** Stanford AI Index shows HLE crossed 50% in one year (faster than designed), and model transparency is declining. There is a gap between what the benchmark community knows and what practitioners know. A continuously updated capability dashboard — running HLE subsets, ARC-AGI-3, SWE-bench Pro, Terminal Bench weekly across all major models — would be used by every serious AI team. Data product, not a model product.
24. **Transparent independent model evaluation for enterprise procurement (NEW April 14):** Foundation Model Transparency Index average dropped from 58 to 40. Frontier models are now the least transparent in the Index's history. Enterprises making model procurement decisions need independent, methodology-disclosed capability and reliability evaluation. Commercial opportunity as voluntary lab transparency declines.
25. **Self-hosted durable agentic execution for compliance-sensitive industries (NEW April 15):** Claude Code Routines delivers the saga/event-sourcing pattern for LLM agents as a hosted product. Banks, healthcare providers, and legal firms need the same architecture but can't use Anthropic-hosted infrastructure (data residency, compliance). An open-source, self-hostable implementation of: durable event log + ephemeral containers + trigger system + connector framework is a clear infra product gap. Tech: Temporal/Step Functions for orchestration, any LLM API, Docker/Kubernetes for sandboxed containers.
26. **Evaluation harness integrity auditor (NEW April 15):** BenchJack (UC Berkeley) demonstrates no standard tool exists for auditing agent benchmarks for exploitability. A tool that automatically probes a harness for BenchJack-class exploits (shared filesystem access, readable task config, injectable tool wrappers) would immediately serve every lab and team running agent evals. This is a research-to-product gap closeable by a small team using BenchJack's documented methodology.
27. **Agentic vision for industrial inspection (NEW April 15):** Gemini Robotics-ER 1.6 shows that VLM + code execution outperforms narrow CV for gauge/instrument reading at 93% accuracy. The same decomposition pattern (spatial grounding → structured extraction → code computation) applies to: manufacturing quality inspection, energy facility dashboards, warehouse shelf compliance, pharmaceutical label verification. Building domain-specific pipelines using frontier VLMs + code execution for vertical inspection use cases is an immediately accessible product opportunity.

---

## AI Compute: Geopolitics & Hardware Stacks (NEW — April 13, 2026)

### CUDA/NVIDIA Stack vs CANN/Huawei Stack
- **As of April 2026:** The AI compute market is bifurcating into two parallel stacks, not a single global market
- **Stack 1 — NVIDIA/CUDA/Vera Rubin:** US labs (Anthropic $6.8B CoreWeave, Meta $35B CoreWeave), TSMC manufacturing, well-established software ecosystem (NCCL, TensorRT-LLM, vLLM, SGLang). Next-gen hardware: Vera Rubin (~2× Blackwell performance), accessible late 2026–2027 to locked-in partners.
- **Stack 2 — Huawei/CANN/Ascend 950PR:** Chinese labs (DeepSeek, Alibaba, ByteDance, Tencent), SMIC 7nm manufacturing, maturing software ecosystem (CANN, HCCL). DeepSeek V4 is the reference implementation for frontier MoE on CANN.

### DeepSeek V4 + Huawei Ascend: CUDA Independence (NEW — April 13, 2026)
- **First frontier model designed for non-CUDA compute stack** (confirmed April 4, 2026; reporting April 6–10)
- **Architecture:** 1T parameter MoE, ~37B active per token, 1M context window (Engram conditional memory), Engram's routing mechanism partitions context into retrievable blocks — enabling 94% recall at 128K vs 45% on V3.2
- **Hardware:** Huawei Ascend 950PR (Da Vinci architecture, SMIC 7nm). DeepSeek rewrote attention kernels, MoE routing, Engram memory, and inference pipeline for CANN, bypassing CUDA entirely
- **V4-Lite early API results (unconfirmed, third-party):** 30% faster inference, 94% context recall at 128K tokens (vs 45% on V3.2). If confirmed, best context recall among open-weight-tier models.
- **Strategic signal:** DeepSeek gave Huawei exclusive early hardware access to V4 while denying NVIDIA access. Chinese labs (Alibaba, ByteDance, Tencent) ordering hundreds of thousands of Ascend 950PR units; chip prices up 20%.
- **Geopolitical significance:** First credible empirical test of the export control assumption (CUDA dependency = frontier AI bottleneck for China). If V4 performs at frontier quality: inference independence demonstrated, training independence (~1–2 years further development) under active attack.
- **Expected release:** Late April 2026. Apache 2.0 license. Target pricing: $0.30/MTok.
- **SWE-bench Pro expected score:** ~81% (would be #2 behind Anthropic Mythos at 77.8%)
- **What to watch for at launch:** (1) independently verified performance on CANN vs CUDA equivalents; (2) SWE-bench Pro score; (3) Engram memory at full 1M context; (4) open weights confirmation
- **Sources:** Reuters April 4 · TrendForce April 7 · The Decoder April 6

### Engram Conditional Memory (NEW — April 13, 2026)
- **DeepSeek paper (January 2026): "Conditional Memory via Scalable Lookup"**
- **Mechanism:** Rather than full-context attention over 1M tokens (prohibitively expensive), Engram computes a compressed query representation, routes to relevant memory blocks, and attends only within those blocks
- **Effect:** Near-constant inference cost as context grows (within blocks), rather than quadratic growth
- **V4-Lite early evidence:** 94% context recall at 128K vs 45% for standard attention — this suggests Engram's routing correctly prioritizes relevant blocks
- **Design implication:** For builders using long-context models for agentic workflows, Engram-style architectures (selective block retrieval) may be a viable middle ground between RAG (explicit retrieval) and full-context attention
- **Connection to memory systems:** Adds a new row to the memory architecture pattern — in-context conditional lookup as an alternative to both full attention and external vector DB

## Capability Measurement & Transparency (NEW — April 14, 2026)

### Stanford HAI 2026 AI Index — Key Quantitative Findings
- **HLE (Humanity's Last Exam):** 8.8% (o1, early 2025) → 50%+ (Claude Opus 4.6, Gemini 3.1 Pro, April 2026). Designed to last years; exceeded 50% in ~12 months. Fastest single-year gain on any major benchmark in Index history.
- **US-China gap:** Down to 2.7% overall capability composite. Specific marker: Claude Opus 4.6 vs. ByteDance Dola-Seed 2.0 Preview = 39 Elo points on LMArena. One year ago: 150+ Elo. Note: DeepSeek V4 not yet included — gap may narrow further in next edition.
- **Frontier model cluster:** Top US models now within ~2 points of each other on most benchmarks. Differentiation shifts to cost, reliability, deployment profile — not raw capability.
- **Foundation Model Transparency Index (FMTI):** Average score dropped 58 → 40 out of 100. Trend reversed from 2022-2024 (when transparency was increasing). Frontier models are now the *least* transparent in the Index's history. Implication: vendor-provided benchmark claims and training disclosures are less reliable than ever.
- **AI adoption:** >50% of world population uses AI (faster adoption rate than PC or internet); 88% of organizations; 4 in 5 university students.
- **US private investment vs. China:** $285.9B (US) vs $12.4B (China) in 2025. China leads on publications, patents, industrial robotics. US leads on frontier model count and private capital.
- **US AI talent inflows:** Down 89% since 2017; 80% decline in the last year alone. Long-term strategic risk signal.
- **Source:** https://hai.stanford.edu/ai-index/2026-ai-index-report

### Workforce Impact — First Measurable Entry-Level Displacement
- **Pattern confirmed:** 22-25 year olds in high-AI-exposure fields (software engineering, customer service) — employment declining. Workers 30+ in same fields — growing 6-12%.
- **Interpretation:** AI is eliminating the on-ramp, not the expert tier. Senior engineers are more productive with AI; entry-level slots are being reduced before being filled.
- **Key nuance:** Effect is concentrated in the youngest workers, not general workforce. Senior roles growing in productivity. Mid-career roles largely unaffected so far.
- **Connection to prior AI labor data (April 9):** Q1 2026 saw 40% YoY jump in tech layoffs; 25% attributed to AI in March. Stanford Index adds the age-stratified view: these layoffs are disproportionately hitting entry-level.

---

## Economics & Scale (April 9, 2026 Update)

### Frontier Lab Revenue Race
- **Anthropic:** $30B ARR (April 2026), up from $9B end of 2025 — 3.3× in one quarter
- **OpenAI:** ~$25B ARR (reported earlier April 2026)
- **Anthropic now leads OpenAI in annualized revenue run rate** — a significant competitive shift
- **Anthropic enterprise footprint:** 1,000+ customers at $1M+/year, doubled in under 2 months
- **Infrastructure secured:** 3.5GW TPU deal with Google/Broadcom for 2027+ training
- **Training strategy:** Multi-platform (AWS Trainium, Google TPUs, NVIDIA GPUs) — workload-matched by chip type

### Compute Infrastructure Trajectory
- Anthropic currently has 1GW of Google TPU capacity (2026)
- 3.5GW additional coming online 2027 — 4.5× expansion
- Context: Single hyperscale data center ≈ 50-100MW; 3.5GW ≈ 35-70 hyperscale data centers
- Revenue-to-compute deal correlation: as demand tripled, compute secured for next 2-3 training generations

### GPU Supply Chain Concentration (NEW — April 9–10, 2026)
- **CoreWeave dual deals (April 9–10, 2026):**
  - Meta $21B new commitment (April 9) → total Meta/CoreWeave: $35B through 2032
  - Anthropic $6.8B multi-year (April 10) → provides NVIDIA Vera Rubin GPU access starting late 2026
  - CoreWeave total revenue backlog: >$66B
- **Vera Rubin GPU:** NVIDIA next-gen after Blackwell; ~2× performance. Both deals include phased Vera Rubin access. Production deployment of models trained on Vera Rubin: mid-to-late 2027 estimate.
- **Structural implication:** Frontier AI compute is transitioning from a spot/shared market to a multi-year locked-in supply chain. A small number of GPU cloud providers (CoreWeave, AWS, Azure, GCP) now hold exclusive capacity agreements with frontier labs through 2030+. Independent researchers and smaller labs face structural disadvantage in securing next-gen compute.
- **For builders:** Vera Rubin spot access will be constrained through 2026–2027. Blackwell remains the accessible path. Plan inference infrastructure procurement timelines accordingly.
- **Anthropic custom silicon exploration (April 9–10, 2026):** Internally evaluating whether to design proprietary AI chips (not committed; no formal team or design finalized). Context: $30B run rate + Broadcom/Google TPU deal → scale may justify vertical integration. If initiated, timeline: 3–5 years to production silicon. Would follow Google (TPUs), Amazon (Trainium), Microsoft (Maia).

### AI Labor Market Impacts (NEW — April 9, 2026)
- **Q1 2026 actuals:** U.S. tech sector cut 52,050 jobs — 40% jump from Q1 2025
- **March 2026:** AI was the stated reason for 25% of tech layoffs (vs. 10% in February 2026) — fastest month-over-month acceleration observed
- **NY Fed workplace AI study (April 8 advisory):** Central bank-level dataset on who uses AI at work, productivity claims, unemployment expectations, and training access value — first major study of this kind
- **Goldman Sachs:** Calls AI "the big story in 2026 in labor"; entry-level knowledge workers and content creation most at risk near-term
- **Concentration:** Impact concentrated in knowledge/content work — not yet general workforce displacement

### AI Scribes & Healthcare Cost Inflation (NEW — April 9, 2026)
- **STAT News / Peterson Health Technology Institute finding (April 8, 2026):** Both insurers AND providers agree AI scribes are increasing healthcare billing intensity
- **Mechanism:** AI scribes produce more complete clinical notes → more billable conditions documented → higher coded claims → higher healthcare costs
- **This is not fraud:** The codes are accurate; the AI is documenting things that were previously underdocumented. The result is structurally higher billing
- **No consensus on solution:** Providers argue underbilling was the pre-AI baseline. Insurers argue costs are rising regardless. No regulatory framework exists for "AI-caused billing inflation through completeness"
- **Pattern:** First clear production example of AI causing economic disruption through unintended second-order effects (not automation of existing tasks, but improvement of documentation quality changing cost structures)

---

## Concept Evolution Log

- [April 6]: T² scaling — pretraining should account for inference cost; overtrain smaller models for agent workloads
- [April 6]: ByteRover — LLM-curated hierarchical memory as alternative to vector RAG for bounded domains
- [April 6]: Harrier-OSS — decoder-only architecture outperforms BERT-style encoders for multilingual embeddings
- [April 6]: Gemma 4 — local/global attention alternation + MoE makes frontier-class models edge-deployable
- [April 6]: TurboQuant — 6× KV cache compression, zero accuracy loss, training-free
- [April 7]: LiteRT-LM — Google's production edge inference framework, Gemma 4 E2B first-party support
- [April 7]: T² validated through post-training — overtraining benefit survives RLHF/SFT stage
- [April 7]: Long-horizon planning failure confirmed empirically — LLMs fail at constraint satisfaction over long horizons regardless of reasoning strength
- [April 7]: RAGEN-2 — template collapse in agentic RL; spurious pattern learning dominates genuine reasoning; SNR-based trajectory filtering solution
- [April 7]: Microsoft Agent Framework 1.0 — Semantic Kernel + AutoGen unification; stable APIs; MCP + A2A integration; first major framework stability commitment
- [April 7]: HappyHorse-1.0 — temporal consistency solved for text-to-video; 2-minute coherent generation achieved; video generation moves to commodity feature
- [April 8]: GLM-5.1 — async RL (Slime) + DSA → first open model sustaining 8-hour / 655-iteration agent sessions; SWE-bench Pro #1 among publicly accessible models
- [April 8]: Anthropic Mythos Preview — capability-gating as deployment pattern; autonomous zero-day discovery at frontier scale; dual-use constraint forces restricted access
- [April 8]: Frontier capability bifurcation — 19-point SWE-bench Pro gap between restricted (77.8%) and open (58.4%) models; gap is structural, not just temporary
- [April 8]: Claude Managed Agents — cloud infrastructure for production agent scaling; session persistence + multi-agent coordination; removes months of infrastructure work
- [April 8]: Microsoft MAI models — speech/voice/image SOTA; speech-to-text commodity pricing ($0.36/hour); first Microsoft leadership position on infrastructure-tier AI
- [April 9]: Latent planning depth ceiling — empirically measured: ~5 steps learnable in training, ~7-8 at test time for frontier models; scale improves breadth not depth; CoT/tools mandatory for >8 sequential steps
- [April 9]: In-Place TTT — MLP projection matrix as fast-weight session memory; no new modules; adaptive inference without external memory; adds 5th tier to memory architecture
- [April 9]: Meta's open-source retreat — Muse Spark (MSL) is closed-source; contradicts prior assumption that Meta would continue publishing all competitive models as Llama weights
- [April 9]: AI-induced healthcare billing inflation — AI scribes increase coding completeness → higher billed amounts; first confirmed second-order economic effect of AI deployment at scale
- [April 9]: AI as primary tech layoff driver — 25% of March 2026 tech layoffs attributed to AI (up from 10% in Feb); Q1 2026 tech sector cuts up 40% YoY
- [April 9]: tui-use — PTY + headless xterm unlocks interactive terminal programs for AI agents; closes the "interactive CLI stall" gap that bash subprocess calls can't handle
- [April 10]: RAGEN-2 template collapse diagnosis — RL agents plateau from learning spurious patterns; SNR filtering enables genuine reasoning to emerge; ~40% training reduction
- [April 10]: HY-Embodied (Tencent) — open-source VLA for robot control; cross-morphology generalization; production demo at EAIDC 2026; embodied AI moves lab→production
- [April 10]: Knowledge curation as new AI primitive — AI building/maintaining wikis from raw research materials (Karpathy pattern); shifts paradigm from code generation to knowledge organization
- [April 10]: Video generation temporal consistency solved — HappyHorse-1.0 SOTA; 2-minute single-pass coherent generation; market consolidation phase beginning
- [April 10]: MirrorCode — AI can do weeks-long autonomous coding if given checkable specs; capability ceiling raised from hours-scale to weeks-scale; specification quality is now the binding constraint, not model capability
- [April 10]: OpenAI TAC vs. Anthropic Glasswing — cybersecurity AI has two competing governance frameworks; identity-tiered (OpenAI) vs. institutional-partnership (Anthropic); competitive market for highest-risk AI capability class now forming
- [April 10]: CoreWeave GPU lock-in — $21B Meta + $6.8B Anthropic deals in 48 hours; total CoreWeave backlog >$66B; Vera Rubin access secured by top labs through multi-year contracts; spot market supply constrained through 2027
- [April 10]: Anthropic custom silicon exploration — $30B run rate + 3.5GW TPU deal may justify vertical chip integration; currently exploratory; 3–5 year production timeline if pursued
- [April 13 — catch-up March 25]: ARC-AGI-3 launched — first interactive adaptation benchmark; frontier CoT models score 0.2–0.3%; Symbolica orchestrator-subagent program synthesis scores 36% Day 1; 100-180× gap is architectural, not a compute gap; interactive rule-learning is a qualitatively different capability class from static task completion
- [April 13 — catch-up April 4–10]: DeepSeek V4 confirmed on Huawei Ascend 950PR — first planned frontier model designed for non-CUDA compute stack; V4-Lite early API: 94% context recall at 128K (vs 45% V3.2), 30% faster inference; full V4 expected late April; Chinese labs ordering hundreds of thousands of Ascend units; prices up 20%; this is the first credible empirical test of the export control assumption
- [April 13]: GPT-5.5 "Spud" entering release window — pretraining done March 24; April 14 earliest expected date; Polymarket 78% by April 30; no confirmed architecture details
- [April 13]: AI compute stack bifurcation formalized — NVIDIA/CUDA/Vera Rubin track (US labs, CoreWeave) and Huawei/CANN/Ascend track (Chinese labs) now simultaneously under construction; both hardening in 2026
- [April 13]: Musk v. OpenAI trial (April 27) — Musk seeking Altman's ouster + governance oversight as remedies; OpenAI filing AG complaints for anti-competitive behavior; first jury trial over nonprofit-to-for-profit conversion in AI; structural governance risk to OpenAI through Q3 2026
- [April 13]: Orchestrator-subagent program synthesis pattern documented — ARC-AGI-3 finding; subagents return summaries (not raw state) to prevent context growth; orchestrator maintains high-level plan; framed as program synthesis (infer program from I/O examples); extends parallel sub-agent pattern with summary compression design constraint
- [April 14]: MiniMax M2.7 open-sourced — 229B/10B-active MoE, Modified-MIT, 56.22% SWE-bench Pro (matches restricted GPT-5.3-Codex, #1 open-access); self-evolution mechanism validated in production: 100+ autonomous scaffold optimization rounds → 30% performance improvement; model-as-optimizer-in-training-pipeline is now a demonstrated practice, not a speculation; scope limit: scaffold code only, not weights
- [April 14]: Stanford AI Index 2026 published — US-China frontier gap: 39 Elo points (2.7%); HLE: 8.8% → 50%+ in 12 months (fastest benchmark gain in Index history); FMTI transparency: 58 → 40 (frontier models now least transparent in Index history); entry-level job displacement confirmed: 22-25 yo workers in software engineering and customer service declining; open-source model tier now in range of restricted frontier coding benchmarks (M2.7 = GPT-5.3-Codex on SWE-Pro)
- [April 14]: GPT-5.5 "Spud" — still not announced as of April 14; Polymarket 78% by April 30; any-day release expected
- [April 15]: Claude Code Routines — first production implementation of saga/event-sourcing pattern for LLM agent execution as a consumer product; session state = durable log on Anthropic servers; compute = ephemeral containers; triggers (schedule/webhook/GitHub) append to log, not spawn processes; mental model for "coding agent" shifts from interactive REPL to durable background worker
- [April 15]: UC Berkeley BenchJack — 8 major agent benchmarks proven exploitable to achieve near-perfect scores without solving tasks; all published scores on SWE-bench Verified, SWE-bench Pro, WebArena, Terminal-Bench, GAIA, OSWorld, FieldWorkArena, CAR-bench are retroactively suspect without independent harness audit; containerized isolation now prerequisite for credible evaluation
- [April 15]: GPT-5.4-Cyber — new "cyber-permissive" fine-tune category established; binary reverse engineering added to LLM capability surface; refusal boundary treated as a tunable parameter per verified professional use case; direct competitive response to Anthropic Mythos within 7 days
- [April 15]: Project Glasswing expanded — initial 11 partners → 40+ organizations with $100M usage credits; establishes tiered identity-gated deployment as the template for all high-risk AI capability releases
- [April 15]: Gemini Robotics-ER 1.6 production deployment — 23% → 93% instrument reading accuracy via agentic vision (VLM + code execution decomposition); confirms general model + tool-use outperforms narrow CV in production physical AI; Boston Dynamics Spot integration is the first announced external production deployment of Robotics-ER
- [April 15]: vLLM v0.19.0 — zero-bubble speculative decoding now compatible with async scheduling; Elastic Expert Parallelism Milestone 2 for MoE clusters; FlexKV smarter KV cache offloading; Gemma 4 full support
- [April 15]: GPT-6 "Spud" did not ship April 14; expected window moved to April 21–May 25; pre-training complete March 24; all capability claims (40%+ gap, HumanEval 95%, MATH 85%, 2M context) remain unverified until public release

*Last updated: April 15, 2026*

---

*This knowledge base is maintained as a living document. Each daily brief contributes:*
- *New concepts → new subsections*
- *Contradictions → flagged with ~~strikethrough~~ and update*
- *Outdated assumptions → marked [OUTDATED as of date]*
- *Evolution tracked: old → limitation → new → tradeoff*
