# Frontier AI Brief — 2026-04-15

> Covering: April 14 – April 15, 2026
> ~28 candidates reviewed · 7 kept · ~21 discarded for age / weak evidence / duplication

---

## Executive View

April 14-15 marks an inflection point in **agentic deployment infrastructure**: Anthropic shipped server-side durable execution for Claude Code, decoupling AI workflows from local machines entirely — a genuine architectural shift, not a feature add. Meanwhile, the **cybersecurity frontier is escalating fast**: OpenAI launched a tiered-access cybersecurity fine-tune (GPT-5.4-Cyber) in direct response to Anthropic's Mythos Preview, which itself got expanded to 40+ organizations on April 15. The competitive race to define "responsible capability disclosure" for offensive-grade AI is now live. In robotics, **agentic vision reached a measurable production milestone**: Gemini Robotics-ER 1.6 jumped instrument-reading accuracy from 23% to 93% by combining spatial reasoning with code execution, deployed in Spot at industrial facilities today.

---

## Top Signals

### [Claude Code Routines](https://kingy.ai/uncategorized/claude-code-just-shipped-routines-and-its-a-bigger-deal-than-you-think/) · **High**
*Published: April 14, 2026 (research preview)*

**What changed**
Anthropic shipped Claude Code Routines as a research preview: AI automations that execute on Anthropic's cloud infrastructure, not on the user's local machine. A routine is a saved configuration: prompt + one or more repos + connectors (Slack, Linear, Google Drive, GitHub) + at least one trigger. Triggers are: scheduled cadence, inbound API webhook, or GitHub event. The session harness, execution sandbox, and session state are decoupled — the harness calls the container the same way it calls any other tool, and session state is a durable log outside both. Usage is metered: Pro=5/day, Max=15, Team/Enterprise=25. Available on all paid plans.

**How it works**
The key architectural move is **decoupling durability from the execution process**. Previously, Claude Code ran locally: the user's machine was both the session host and the execution environment. With Routines, the session log persists server-side and is divorced from any particular container instance. Containers are provisioned on-demand only when the agent needs to execute, then torn down. This is a saga/event-sourcing pattern applied to agentic loops: state is the log, compute is ephemeral. P50 time-to-first-token reportedly dropped ~60%, and P95 dropped >90%, because container cold starts no longer sit on the critical path of user interaction — they're isolated to async background execution.

**Why it matters**
This is the first major production deployment of **server-side durable agent execution** by a frontier lab, not as an internal infrastructure choice but as an end-user product. The pattern matters because: (1) agents that modify repos, trigger CI, file PRs, or post Slack updates are now stateful server processes, not REPL sessions; (2) the log-as-state model is the right abstraction for long-horizon agentic work that survives compute interruptions; (3) it directly attacks the "laptop must stay on" problem that has prevented most agentic automation from being operationally reliable.

**What to update in your mental model**
The mental model for "coding agent" is shifting from **interactive REPL** to **durable background worker**. The execution pattern is saga/event-sourcing, not request-response. If you're building multi-step agent pipelines that touch external systems, this is the pattern to study.

---

### [OpenAI Launches GPT-5.4-Cyber, Expands Trusted Access for Cyber](https://help.apiyi.com/en/openai-gpt-5-4-cyber-security-model-launch-en.html) · **High**
*Published: April 14, 2026*

**What changed**
OpenAI released GPT-5.4-Cyber — a version of GPT-5.4 fine-tuned for cybersecurity use cases — to the highest tiers of its Trusted Access for Cyber (TAC) program. It is explicitly "trained to be cyber-permissive": it lowers refusal barriers for legitimate security research tasks (vulnerability analysis, binary analysis, exploit reproduction), adds **binary reverse engineering** capability (analyzing compiled software for malware and vulnerabilities without source code access), and enables more advanced defensive workflows. Verification is via chatgpt.com/cyber (individuals) or enterprise rep (teams). Rollout is to thousands of individuals and hundreds of security teams.

**How it works**
GPT-5.4-Cyber is a post-training alignment shift, not a capability injection from pre-training. The base model already has the underlying capability; the fine-tune reweights the refusal-vs-compliance frontier specifically for authenticated cybersecurity defenders. Binary reverse engineering appears to combine the model's code analysis capability with instruction-following on disassembled binary representations (objdump, Ghidra exports, etc.). The tiered access model gates capability on identity verification and organizational enrollment, providing OpenAI a compliance surface without a public API surface.

**Why it matters**
This launch arrived **one week after Anthropic's Mythos Preview** (restricted to 11 named partners for defensive vulnerability discovery), making this an explicit competitive response. The race is: who controls the "responsible offensive AI" category for enterprise security teams. For builders, the immediate implication is that advanced binary analysis, vuln-scanning automation, and security research workflows now have a legitimate API path — but behind a heavyweight identity gate that may not fit startup/indie use cases.

**What to update in your mental model**
Fine-tuned "capability unlock" models for professional verticals (security, medicine, law) are now a real product category. The refusal-boundary is a tunable parameter per verified use case, not a fixed capability ceiling. This is the alignment policy implementation of "jagged frontier" at the product level.

---

### [Gemini Robotics-ER 1.6 + Boston Dynamics Spot Integration](https://marktechpost.com/2026/04/15/google-deepmind-releases-gemini-robotics-er-1-6-bringing-enhanced-embodied-reasoning-and-instrument-reading-to-physical-ai/) · **High**
*Published: April 14-15, 2026*

**What changed**
Google DeepMind released Gemini Robotics-ER 1.6 with a new **instrument reading capability** and improved spatial reasoning, deployed immediately into Boston Dynamics' Orbit software (AIVI and AIVI-Learning systems) for Spot's industrial inspection workflows. The headline number: **93% accuracy on analog gauge/pressure meter reading**, up from approximately 23% in ER 1.5 — which lacked the capability entirely and was replaced by a narrow classical CV model for this task.

**How it works**
The instrument-reading capability is an **agentic vision pipeline**, not a single model pass. The model combines: (1) spatial reasoning to locate and segment the gauge face in the scene; (2) code execution to mathematically interpret the dial position (angle → value); (3) tool-calling to look up or infer the instrument's scale and units from context. This is a multimodal agent loop — LMM + code interpreter — applied to a precisely scoped physical reasoning problem. The broader ER 1.6 improvements include better pointing accuracy, counting, and success detection vs 1.5 and Gemini 3.0 Flash. Boston Dynamics' AIVI-Learning system uses this in production: Spot walks a facility, photographs instruments, and logs readings autonomously.

**Why it matters**
The jump from ~23% to 93% by switching from narrowly-trained CV to a general agentic vision approach is a strong concrete data point for the ongoing debate about **specialist vs. general agents in industrial settings**. The specific mechanism — using code execution to handle the mathematical/geometric step rather than asking the vision model to do it directly — is an example of correct tool decomposition in embodied AI. This also marks Google DeepMind's first announced production deployment of Robotics-ER in external commercial hardware.

**What to update in your mental model**
"General model + code execution" now outperforms narrowly trained perception models for gauge/instrument reading at production scale. The correct architecture for physical perception tasks that require interpretation (not just classification) is: visual grounding → structured extraction → code-based computation → result. Build for this decomposition, not end-to-end VLM inference.

---

### [Project Glasswing Expanded to 40+ Organizations](https://www.anthropic.com/glasswing) · **Medium**
*Published: April 15, 2026 (expansion; original announcement April 7)*

**What changed**
Anthropic expanded Project Glasswing from its initial 11 named partners (AWS, Apple, Broadcom, Cisco, CrowdStrike, Google, JPMorgan Chase, Linux Foundation, Microsoft, NVIDIA, Palo Alto Networks) to **40+ additional organizations** that build or maintain critical software infrastructure. These include open-source security organizations receiving $4M in direct donations and organizations receiving from $100M total committed in Mythos Preview usage credits. These additional organizations can use Mythos Preview to scan and patch first-party and open-source systems.

**Why it matters**
The expansion is the first evidence that "model as security tool with controlled access" can scale past a handful of named partners. The $100M in usage credits committed to this initiative makes it the largest public commitment to AI-powered defensive security work to date. For builders, this establishes a precedent: **frontier model capabilities can be deployed for high-risk verticals through usage-metered, identity-gated access** rather than public API. Expect this access pattern to replicate in medicine, finance, and critical infrastructure.

**What to update in your mental model**
"Controlled capability deployment" is now a real product category. The business model is: frontier model with aligned fine-tune + identity verification + usage metering + responsible disclosure coordination. This is likely where the next generation of high-stakes vertical AI tools lands.

---

## Agentic Architecture & Engineering

### MiniMax M2.7: First Open-Source Model with Reported Self-Evolution
*Released: April 12, 2026 — 3 days before this brief; included for architectural novelty*

MiniMax released M2.7, a 230B-parameter MoE model (10B active per token, 256 experts, 8 activated), Apache 2.0 license, 200K context window, scoring 56.22% on SWE-Pro and 57.0% on Terminal Bench 2.

The technically distinctive claim: MiniMax ran an internal version of M2.7 autonomously for **100+ rounds** to optimize its own programming scaffold. The model analyzed failure trajectories, modified scaffold code, ran evaluations, and decided whether to keep or revert each change — a rudimentary but concrete self-improving agent loop. MiniMax reports 30% improvement on internal eval sets from this process. The mechanism is: LLM as optimizer over its own tool-use scaffolding, not over weights. This is "self-improving scaffolding" not "self-improving model."

**Affected stack**
`User → Planner → [Scaffold: self-optimized by M2.7] → LLM (M2.7) → Tools → Verifier → Output`
Shift at the **Scaffold** layer: the scaffold is no longer static, it was iteratively improved by the model itself before release.

**Build implication: Experiment.** The architecture — running an agent to optimize its own scaffolding code over many rounds — is replicable with any strong coding model. The key question not answered publicly is evaluation methodology rigor: "30% improvement on internal evals" is unverified. Worth prototyping before trusting as a production pattern.

---

### UC Berkeley BenchJack: All Eight Major Agent Benchmarks Broken
*Published: April 12, 2026 — 3 days before this brief; included for high builder relevance*

A UC Berkeley team (Dawn Song, Alvin Cheung) built **BenchJack**, an automated exploit agent that scanned 8 major AI agent benchmarks and found exploitable gaps between evaluation mechanism and actual behavior. Results: 100% on SWE-bench Verified, SWE-bench Pro, Terminal-Bench, FieldWorkArena, CAR-bench; ~100% on WebArena; 98% on GAIA; 73% on OSWorld — without solving any of the underlying tasks.

Specific exploits:
- **SWE-bench Verified**: 10-line Python conftest.py resolves every instance by patching test infrastructure directly
- **WebArena**: File:// URL navigation reads the gold answer from task config in Chromium
- **Terminal-Bench**: `/usr/bin/curl` replaced with a wrapper that trojans the uvx binary

**Affected stack**
`Evaluation Harness → [BROKEN: exploit surface between harness and task state] → Agent`
Shift at the **Evaluation** layer: the harness-to-task interface is an attack surface.

**Build implication: Watch closely.** This makes scores on all 8 benchmarks suspect retroactively for any agent that had access to the test infrastructure. If you're using SWE-bench Verified, Terminal-Bench, or GAIA as evaluation signals in production research, audit your harness isolation. The practical consequence: **containerized, isolated evaluation environments with no access to task config are now a prerequisite for credible agent evals**, not an optional hardening step.

---

## Infra, Serving & Cloud

### vLLM v0.19.0
*Release: April 2026 (exact date not confirmed in window)*

Notable changes in v0.19.0 (448 commits, 197 contributors):

- **Zero-bubble async scheduling + speculative decoding**: These two features now work together. Previously, enabling speculative decoding forced synchronous scheduling, creating a throughput bottleneck. Zero-bubble overlap now applies to spec decode rejection sampling, removing the constraint.
- **Elastic Expert Parallelism (EPLB, Milestone 2)**: Dynamic GPU scaling for MoE expert shards, with new CLI options for faster EP model loading. Relevant for serving Gemma 4 26B MoE and GLM-5.1 MoE at scale.
- **KV Cache Offloading (FlexKV)**: Smart CPU offloading that stores only frequently-reused KV blocks. FlexKV is a new backend replacing the previous blunt offloading strategy.
- **Gemma 4 support**: Full architecture support including MoE, multimodal, reasoning, tool-use. Requires transformers≥5.5.0.

Deployment relevance: If you're running MoE models (Gemma 4 26B, GLM-5.1) the EPLB improvements matter for cluster efficiency. Zero-bubble spec decode is worth testing if you're currently using speculative decoding on async workloads.

---

## Wider World

### Cybersecurity AI: Rapid Competitive Escalation

The GPT-5.4-Cyber launch and Glasswing expansion in a single 48-hour window confirms a new competitive dynamic: frontier labs are racing to own "responsible offensive capability" as a product category. Both Anthropic and OpenAI are converging on the same access model — identity-gated, usage-metered, tiered — with competitive differentiation in model capability and partner network. The underlying model capabilities (autonomous vulnerability discovery, binary reverse engineering, zero-day identification) now demonstrably surpass "all but the most skilled humans" per Anthropic's own disclosure. **This is not a prediction; it is a disclosed present capability.** The policy and business implications are moving faster than the technical ones at this moment.

---

## Deep Dive

### Claude Code Routines: What Durable Agentic Execution Actually Means

**The problem it attacks**

Every multi-step AI agent that touches external systems (repos, APIs, Slack, databases) has historically run as a foreground process: the user's machine stays on, the terminal stays open, the agent session is stateful in memory. Any interruption — laptop sleeps, network drops, process crash — destroys all accumulated context and in-flight work. This makes agents fundamentally unreliable for long-horizon tasks that take hours, require overnight execution, or need to respond to asynchronous events (like a GitHub webhook at 3am).

**Core mechanism**

Claude Code Routines solves this with three architectural moves:

1. **Decoupled state**: Session state (the conversation log, accumulated context, intermediate results) lives as a **durable log on Anthropic's infrastructure**, outside any specific compute process. This is the fundamental move. State is not in the process's memory; it's in the log.

2. **Ephemeral compute**: Execution containers are provisioned on-demand only when the agent needs to run, then torn down. No container is "kept warm" for a specific session. This is why P95 latency drops so dramatically for user interactions — the user-facing path never waits for background container state.

3. **External triggers**: The trigger surface (schedule, webhook, GitHub event) connects to the durable log, not to a running process. When a GitHub PR opens, the trigger writes to the log, and a new container is provisioned to process the accumulated context.

**Before vs. after**

Before: `User terminal → Claude Code process (ephemeral, stateful in memory) → Tools → External systems`

The failure mode: process dies, state is gone. The user is the session host.

After: `Trigger (cron/webhook/GitHub) → Durable log (Anthropic cloud) → Ephemeral container (provisioned on demand) → Tools → External systems → Result written back to log`

The invariant: the log always exists. Compute is replaceable. This is the saga/event-sourcing pattern applied to agent execution.

**Strengths**

- Agents survive any compute interruption by design
- Asynchronous event-driven triggering is a first-class primitive (not a hack)
- Connectors (Slack, Linear, Drive, GitHub) enable agents that operate across the full toolchain without custom integration work
- P50/P95 latency improvements show infrastructure correctness, not just product polish

**Failure modes and tradeoffs**

- **Rate limits are meaningful now**: Pro users at 5 routines/day will hit the ceiling on any moderate automation workload. The limits suggest this is designed for supplementing human work, not replacing CI pipelines.
- **Context window budget**: Long-running routines accumulate a growing log. Context budget management over multi-day runs is unsolved — what gets truncated, when, and by whom?
- **Debugging opacity**: When a routine fails at 3am after 200 steps, observability tools are unclear. Execution traces and log inspection are not yet first-class in the research preview.
- **Connector scope**: Slack, Linear, Google Drive, GitHub covers many workflows but excludes databases, internal APIs, and cloud consoles — limiting applicability for infrastructure automation.
- **Vendor lock-in**: The durable log lives on Anthropic's infrastructure. The session state is not portable.

**So what for builders**

The correct mental model for long-horizon agent execution is now **event-sourced background worker**, not **foreground REPL session**. If you're building agents that operate over hours or days, the right design is: (1) durable log as the system of record, (2) ephemeral compute that reads from and writes to the log, (3) external triggers that append to the log rather than spawning processes. This is what good workflow orchestration systems (Temporal, AWS Step Functions) have always done. Anthropic is applying this to LLM agents and making it accessible to non-infrastructure engineers. The design pattern, once you see it, is obvious — and now it's a product.

---

## Small Finds

- **GLM-5.1** (Z.ai, April 7 — outside window but widely discussed now): 754B MoE, 40B active, MIT license, #1 open-weight on SWE-bench Pro. Notable: trained entirely on Huawei Ascend 910B chips with MindSpore — zero Nvidia GPUs. First credible data point that high-quality frontier training is viable on non-Nvidia hardware at this scale.

- **GPT-6 "Spud" did not ship April 14**: Despite confirmed completion of pre-training March 24 and an April 14 expected window, no public launch. Current expectation window is April 21–May 25. Pre-training reportedly on Stargate/Abilene cluster. Claimed: 40%+ capability jump over GPT-5.4, 2M token context, $2.50/$12 per M input/output tokens. Nothing confirmed until it ships.

- **Gemma 4 trending hard on HuggingFace** (released April 2, now dominating downloads): 31B dense (30.7B fully active), 26B MoE (128 experts, 8 active, 4B active per token), 256K context, Apache 2.0. Per-Layer Embeddings on 2B/4B edge variants for parameter efficiency. Production-viable open-weight model for agentic tasks today.

- **vLLM gains AMD GPU support for Gemma 4** via ROCm in the v0.19.0 cycle — minor but useful if your inference cluster isn't Nvidia-only.

- **Recall 2.0** launched on April 14 (personal AI memory/recall product) — community interest on HN, not a frontier research development, worth watching as a UX pattern for agent memory surfaces but not technically significant.

---

## Frontier Direction

- **Bottleneck under attack**: Agent execution reliability — moving from "local foreground process" to "server-side durable background worker" is the active battle. Claude Code Routines is the first product-grade implementation.

- **Broader trend**: Cybersecurity is the forcing function for capability disclosure policy. Both major labs converged in one week on "tiered, identity-gated, usage-metered" as the responsible deployment pattern for offensive-grade models. Expect this to become the template for medical diagnosis, legal analysis, and financial risk models.

- **Still unsolved**: Evaluation harness integrity — the Berkeley BenchJack paper invalidates most published agent benchmark results retroactively. There is no widely-adopted standard for evaluation isolation. This is the biggest open problem in agent research right now: we may not know which agents actually work.

- **Emerging paradigm**: Self-improving scaffolding (MiniMax M2.7's autonomous scaffold optimization) — the model improves its own tool-use scaffolding over many agentic rounds, not its weights. This is a new failure mode (uncontrolled scaffold drift) and a new optimization primitive. It's early and unverified at scale, but the pattern is distinct from both RLHF and RAG and worth tracking.

Arrows:
- Interactive REPL agent → Durable background worker with event triggers
- Fixed refusal policy → Per-verified-user capability unlock for high-risk verticals
- Narrow perception CV → Agentic vision (spatial reasoning + code execution) for physical AI
- Static agent scaffolding → Self-optimized scaffolding via autonomous agentic loops

---

## Builder Takeaways

### Try now
**Claude Code Routines** — Set up a simple routine today: pick a GitHub trigger (e.g., new PR opened on main), write a prompt that performs a specific code review or generates a changelog entry, connect GitHub as a connector. This is immediately usable on any paid plan. The point isn't the specific automation — it's internalizing what it feels like to write an agent that runs in the background without your machine. That mental model shift is what you need before designing production agentic systems.

### Experiment with
**Agentic instrument interpretation pipeline** — Re-implement the Gemini Robotics-ER 1.6 gauge-reading approach on your own domain: take a vision model (Gemini 3.0 Flash, GPT-4o), add a code execution step for the mathematical interpretation, and measure accuracy vs. end-to-end VLM inference on a structured extraction task. The hypothesis is that decomposition (vision grounding → structured extraction → code computation) outperforms monolithic inference even on tasks that look like pure perception. Validate or falsify it on your data.

### Go deep on
**Agent evaluation harness design** — The Berkeley BenchJack paper demonstrates that almost every published agent benchmark score is potentially gamed. The high-leverage skill right now is knowing how to build evaluation harnesses that cannot be exploited: containerized task environments, no task-config access from the agent sandbox, deterministic scoring, and controlled output surfaces. Study: Temporal's workflow testing patterns, the HAL benchmark framework, and the BenchJack paper's exploit methodology itself (understanding how it breaks things tells you how to build things that don't break). This is career-defining work if you're serious about agent systems — reliable eval is the bottleneck on the entire field's progress.

### Ignore for now
**GPT-6 speculation** — Every technical claim about "Spud" is unverified until OpenAI ships and publishes evals. Benchmark numbers floating around (HumanEval 95%, MATH 85%, hallucination rate <0.1%) are all second-hand and unconfirmed. Do not update your mental model or architecture decisions based on unverified claims about an unreleased model.

---

## What to Build

**Project 1: Durable Agent Execution Framework**
- **Project**: Build a language-agnostic durable agent execution layer using event sourcing — a framework where agent state is always a replayable log, compute is always ephemeral, and external events (webhooks, schedules) are first-class triggers.
- **Why now**: Claude Code Routines ships exactly this pattern as a product. Understanding the underlying infrastructure design makes you a better consumer and helps you build similar systems where Anthropic's product doesn't fit (internal APIs, custom infrastructure, regulatory environments that can't use hosted services).
- **Stack**: Temporal or AWS Step Functions for orchestration, any LLM API, a durable message log (Kafka or DynamoDB Streams), Docker for sandboxed containers. Language-agnostic.
- **What you'd learn**: Event sourcing applied to LLM agents; failure recovery without state loss; asynchronous trigger handling; the boundary between "session" and "process" in AI systems.

**Project 2: Isolation-Hardened Agent Evaluation Harness**
- **Project**: Build a benchmark harness for a 10-20 task mini-benchmark where the evaluation mechanism is architecturally isolated from agent access — no shared filesystem, no readable task config, scoring via external verifier only.
- **Why now**: The Berkeley BenchJack paper demonstrates that almost all existing harnesses are exploitable. A properly isolated harness is the prerequisite for any credible agent eval work and is conspicuously missing from the open-source ecosystem.
- **Stack**: Docker/gVisor for sandboxing, any agent framework (LangGraph, Claude Code SDK, custom), a separate verifier process with no shared state, a small task set you design from scratch.
- **What you'd learn**: Security-oriented thinking for agent evaluation; containerization boundaries; how to design tasks where "getting the answer right" is verifiably distinct from "cheating the harness"; the difference between evaluation validity and reliability.

---

## Opportunities

1. **Durable agent execution as a service for regulated industries** — The Claude Code Routines architecture (durable log + ephemeral compute + event triggers) is exactly what medical, legal, and financial teams need for AI workflows but can't get from Anthropic's hosted product due to data residency or compliance requirements. A self-hosted open-source implementation of this pattern, deployable on-prem or in a private cloud, has a clear market. The tech is standard (event sourcing + containers); the differentiator is AI-specific state management (context budgeting, long log summarization, connector protocols).

2. **Evaluation harness integrity tooling** — BenchJack reveals there is no standard, publicly available tool for auditing agent benchmarks for exploitability. A tool that automatically probes a given harness for the categories of exploits Berkeley found (shared filesystem, readable task config, injectable tool wrappers) would be immediately useful to every lab and team running agent evals. This is a research-to-product gap that a small team could close quickly.

3. **Vertical agentic vision for industrial inspection** — The Gemini Robotics-ER 1.6 instrument-reading result (23% → 93% via agentic decomposition) is in the robotics domain, but the same pattern applies to any structured physical inspection task: reading labels, verifying safety markings, auditing shelf compliance, inspecting manufacturing output. Building a general-purpose agentic vision pipeline (vision model + code execution + domain-specific ontology) for industrial inspection verticals is a real product with a clear wedge: existing solutions are narrow, brittle, trained-per-instrument; the agentic approach generalizes.

---

*Sources:*
- [Anthropic Claude Code Routines — Kingy AI](https://kingy.ai/uncategorized/claude-code-just-shipped-routines-and-its-a-bigger-deal-than-you-think/)
- [Claude Code Routines — Blink.new Guide](https://blink.new/blog/claude-code-routines-guide)
- [Claude Code Routines — Adam Holter Analysis](https://adam.holter.com/claude-code-routines-what-the-research-preview-actually-delivers-for-cloud-automation/)
- [OpenAI GPT-5.4-Cyber — Implicator AI](https://www.implicator.ai/openai-ships-gpt-5-4-cyber-expands-trusted-access-to-thousands-of-defenders/)
- [OpenAI Cyber Program — Apiyi](https://help.apiyi.com/en/openai-gpt-5-4-cyber-security-model-launch-en.html)
- [OpenAI X announcement](https://x.com/OpenAI/status/2044161906936791179)
- [OpenAI Help Net Security](https://www.helpnetsecurity.com/2026/04/15/openai-gpt-5-4-cyber/)
- [Gemini Robotics-ER 1.6 — MarkTechPost](https://marktechpost.com/2026/04/15/google-deepmind-releases-gemini-robotics-er-1-6-bringing-enhanced-embodied-reasoning-and-instrument-reading-to-physical-ai/)
- [Boston Dynamics AIVI — Google DeepMind blog](https://blog.google/innovation-and-ai/models-and-research/google-deepmind/gemini-robotics-er-1-6/)
- [Boston Dynamics — National Today coverage](https://nationaltoday.com/us/ma/boston/news/2026/04/14/boston-dynamics-and-google-deepmind-integrate-gemini-to-enhance-spot-robots-ai-capabilities/)
- [Anthropic Project Glasswing](https://www.anthropic.com/glasswing)
- [Glasswing expansion — Schneier on Security](https://www.schneier.com/blog/archives/2026/04/on-anthropics-mythos-preview-and-project-glasswing.html)
- [MiniMax M2.7 — MarkTechPost](https://www.marktechpost.com/2026/04/12/minimax-just-open-sourced-minimax-m2-7-a-self-evolving-agent-model-that-scores-56-22-on-swe-pro-and-57-0-on-terminal-bench-2/)
- [MiniMax M2.7 — MiniMax](https://www.minimax.io/news/minimax-m27-en)
- [UC Berkeley BenchJack — RDI](https://rdi.berkeley.edu/blog/trustworthy-benchmarks-cont/)
- [BenchJack — NewClaw Times](https://newclawtimes.com/articles/uc-berkeley-exploit-agent-eight-ai-benchmarks-perfect-scores-no-reasoning/)
- [vLLM v0.19.0 Release](https://github.com/vllm-project/vllm/releases/tag/v0.19.0)
- [Gemma 4 — Google Blog](https://blog.google/innovation-and-ai/technology/developers-tools/gemma-4/)
- [GLM-5.1 Review — Build Fast With AI](https://www.buildfastwithai.com/blogs/glm-5-1-open-source-review-2026)
- [Process Reward Agents — arXiv 2604.09482](https://arxiv.org/abs/2604.09482)
- [GPT-6 Status — FindSkill.ai](https://findskill.ai/blog/gpt-6-release-date/)
