# Frontier AI Brief — 2026-05-13

> Covering: May 12–13, 2026
> ~20 candidates reviewed · 5 kept · 15 discarded (outside window / weak evidence / no new mechanism / prior coverage / pure business/marketing)

---

## Executive View

A moderate signal day between two high-density events: yesterday's Android Show and next week's Google I/O keynote (May 19). The single strongest artifact is Shepherd (arXiv:2605.10913, Stanford), which treats agent execution as a first-class version-controlled artifact — enabling runtime intervention, counterfactual optimization, and Tree-RL training over the same infrastructure. Its numbers are the kind that force architectural reconsideration: 28.8%→54.7% on CooperBench for supervised agent pairs, up to 11 benchmark points with counterfactual meta-optimization, 58% wall-clock reduction. Google's Gemini Intelligence made the agentic OS layer concrete with published security architecture details — particularly the prompt injection defense model built into Android. OpenAI's Deployment Company launch ($4B, 150 FDEs) is the largest structural signal: OpenAI is entering the enterprise AI implementation services market directly, which reshapes the build-vs-partner calculus for enterprise teams.

---

## Top Signals

### [Shepherd: A Runtime Substrate Empowering Meta-Agents with a Formalized Execution Trace](https://arxiv.org/abs/2605.10913) · **High**
*Published: May 11, 2026 (appeared on HuggingFace Papers May 12) · Stanford · Open-source*

**What changed**

Stanford researchers released Shepherd, a functional programming model and runtime substrate that formalizes every agent–environment interaction as a typed event in a Git-like execution trace. The key primitive: any past agent state can be forked and replayed at 5× the speed of Docker, with >95% prompt-cache reuse on replay. Three demonstrated applications:

1. **Runtime intervention:** A live supervisor meta-agent monitors a coding agent's execution trace in real time and intervenes when it detects a mistake. Pair coding pass rate: 28.8% → 54.7% on CooperBench.
2. **Counterfactual meta-optimization:** At a decision point, fork the agent into multiple branches, explore counterfactual choices, and commit the best path. Outperforms baselines across four benchmarks by up to 11 points; reduces wall-clock time by up to 58%.
3. **Tree-RL training:** Fork rollouts at selected turns during RL training to increase trajectory diversity without re-running from scratch. TerminalBench-2: 34.2% → 39.4%.

Core operations are mechanized in Lean. Open-sourced.

**How it works**

The architecture has two layers:

- **Execution trace layer:** Every tool call, LLM response, filesystem write, and environment observation is recorded as a typed event with a content-addressed hash. This creates a Merkle-like DAG of agent state — analogous to Git commits. Any state in the DAG can be forked (COW filesystem snapshot + prompt cache checkpoint) in ~200ms rather than container boot time.

- **Meta-agent layer:** A meta-agent is a function that takes a target agent's execution trace as input and returns an intervention (inject a message, redirect a tool call, restart from a prior fork point). The functional model means meta-agents are composable — you can stack supervisors, reviewers, and optimizers without coupling them.

The 5× Docker speedup comes from using filesystem copy-on-write (like `rclone` or ZFS snapshots) rather than full container rebuilds, plus KV-cache checkpointing tied to trace state.

**Why it matters**

Current agent frameworks treat execution as a black box: a sequence of events that happens, and then either succeeds or fails. Shepherd treats execution as a versioned artifact — something you can inspect, branch, rewind, and replay, exactly like source code. This unlocks three things that were previously impractical at runtime:

- **Live supervision:** A meta-agent can watch execution as it happens and intervene before failure propagates. The 28.8%→54.7% improvement is not from a better base model — it is from a supervisor that catches recoverable mistakes the agent makes mid-execution.
- **Counterfactual search at inference time:** Instead of running the agent once and hoping it picks the right path, fork at decision junctions and explore alternatives in parallel. This is MCTS-style search applied to full agent execution trees.
- **RL data amplification:** For Tree-RL, forking at selected turns generates more diverse training trajectories from fewer full runs — directly reducing the compute cost of RL training on long-horizon agent tasks.

The 58% wall-clock reduction on counterfactual optimization is especially significant: it means the search overhead is more than compensated by the path quality improvement.

**What to update in your mental model**

Agent execution is not a single-shot process — it is a tree of possible executions. Shepherd makes this tree cheap enough to explore at runtime. The correct mental model is no longer "run agent → evaluate outcome → log"; it is "run agent → fork at key decision points → compare branches → commit best path → log execution DAG." For teams building agent infrastructure: the Git analogy is exact. Just as version control transformed software development from "write and hope" to "write, branch, merge, bisect", Shepherd does the same for agent execution.

---

### [Gemini Intelligence: OS-Level Agentic Layer + Security Architecture for Android](https://blog.google/security/android-gemini-intelligence-security-privacy/) · **Medium**
*Announced: May 12, 2026 (Android Show: I/O Edition) · Coming summer 2026*

**What changed**

Google announced Gemini Intelligence as the new brand for its production agentic AI integration in Android — not a new model, but a named OS-level layer that integrates Gemini 3.1 into the core of Android for cross-app task automation. Key concrete capabilities announced:

- **Gemini in Chrome / auto-browse (Android):** Agent navigates web pages and completes multi-step tasks (booking, ordering, form submission) on behalf of the user. Available to AI Pro and Ultra subscribers. Requires Android 12+, ≥4GB RAM.
- **Cross-app task automation (Personal Intelligence):** Takes data from one app and performs multi-step actions across apps — e.g., photograph event flyer → find event on Expedia → add to calendar. Opt-in. Draws from personal data in connected apps.
- **AI-generated widgets (Create My Widget):** Describe a widget in natural language; Gemini generates it. On-device Gemini model generates the widget's logic and layout.
- **Rambler (Gboard):** Gemini-powered multilingual dictation that removes filler words, handles mid-sentence corrections ("let's meet at 3, um, 2 PM" → "let's meet at 2 PM"), and supports code-switching across languages.

The security architecture blog post is the most technically substantive artifact: it describes the prompt injection defense model built into Android for agentic actions. Key design: sensitive agentic actions (form submission, booking, purchase) require explicit confirmation before execution; content encountered by the agent (from web pages, notifications, other apps) is treated as untrusted data and cannot override user-issued instructions. The post states that key parts of the security infrastructure are open-source and binary-transparent with third-party audit.

**How it works**

The OS-level integration means Gemini Intelligence has a privileged system channel to cross-app state — not a screen-reading accessibility hack, but actual API-level integration with Android's intents system. This is the architectural difference from prior Android AI assistants (Google Assistant, early Bard): cross-app data access is happening at the OS layer with platform-level permission controls, not through a sandboxed app scraping what it can see.

The widget generation model is on-device — described as running via Personal Intelligence infrastructure to keep data local.

**Why it matters**

This is the production deployment of what Orbit (Anthropic), ChatGPT Pulse (OpenAI), and Gemini contextual updates have been converging on: proactive AI integrated at the OS level. Google's version has the advantage of controlling the OS, the browser, and the identity layer simultaneously — which eliminates the permission friction that limits third-party apps. Google I/O (May 19) is expected to announce Gemini 4 with significantly extended capabilities on top of this same infrastructure.

The prompt injection defense architecture is worth studying specifically because it is the first published OS-level defense model for agentic AI at consumer scale. The core principle (untrusted content from the environment cannot override user instructions) is sound; the implementation details (how trust boundaries are enforced across app contexts) will be in the open-source components.

**What to update in your mental model**

The agentic OS layer is no longer a research concept — it is a named product shipping to Samsung Galaxy and Pixel this summer. The security architecture principle (environment content is untrusted data; user instructions have higher trust tier; sensitive actions require confirmation) is the correct default for any agentic system with environmental tool use.

---

### [OpenAI Deployment Company: $4B Enterprise AI Implementation Firm + Tomoro Acquisition](https://openai.com/index/openai-launches-the-deployment-company/) · **Watch**
*Announced: May 11–12, 2026 · $4B raised from TPG, Bain, Advent · Majority-owned by OpenAI*

**What changed**

OpenAI launched the OpenAI Deployment Company (internally: "DeployCo"), a majority-owned separate entity backed by $4B from a 19-firm PE consortium led by TPG, with Bain Capital and Advent as co-leads. Pre-money valuation: $10B. Simultaneously acquiring Tomoro, an applied AI consulting firm, which brings ~150 Forward Deployed Engineers (FDEs) to the new entity from day one.

The entity's function: embed FDE teams inside enterprise organizations to identify high-ROI AI deployment opportunities, redesign workflows around AI, and build bespoke AI systems that satisfy enterprise constraints (security, compliance, governance, legacy infrastructure). Explicitly compared to Palantir's deployment model. Accenture stock fell ~3% on the announcement.

**Why it matters for builders**

This is not a technical architectural development, but it is a structural signal with direct implications for how AI capability reaches enterprise organizations:

1. **OpenAI is moving down the stack into implementation services** — historically the province of SIs like Accenture, Deloitte, and Palantir. The $4B war chest and existing enterprise relationships (ChatGPT Enterprise) give DeployCo a credible runway.

2. **FDE specialization is becoming a job category.** The model — engineers who embed in client organizations, know frontier AI capabilities deeply, and can redesign workflows — is the professional archetype that creates the most leverage on AI deployment at scale. Understanding how to be an effective FDE (or hire them) is a near-term career consideration.

3. **For independent builders:** DeployCo targets complex multi-department enterprise transformation projects. The opportunity below that tier — small business, mid-market, domain-specific verticals — remains fully open and is less likely to be addressed by a $10B SI model.

**What to update in your mental model**

The enterprise AI implementation market is bifurcating: OpenAI + large SIs handling complex multi-system enterprise transformation at the top; independent builders, boutique agencies, and platform integrators handling the rest. The gap between frontier model capability and enterprise deployment rate is now the market being competed for.

---

## Agentic Architecture & Engineering

### Shepherd: What Git Semantics Mean for Agent Infrastructure

Shepherd is the most architecturally significant paper in the May window because it formalizes something the field has been describing informally for two years: agent execution as a tree, not a sequence.

**Affected stack:**
```
User → [Agent Planner] → [Tool Calls] → [Environment State]
              ↑ (fork)              ↑ (trace record)
     [Meta-Agent Supervisor] ← [Execution Trace DAG]
              |
     (intervene / branch / replay)
```

The trace DAG sits *alongside* the execution path, not inside it. The meta-agent reads the DAG and can inject into the execution stream at any point, rewind to a prior fork point, or start a parallel branch.

**Three use cases and their stack positions:**

1. **Runtime supervision** — Meta-agent at `Verifier` position, but active during execution rather than after: watches the trace stream, detects errors in progress, injects corrections before failure propagates. This is fundamentally different from the Outcomes evaluator (May 11 brief) which evaluates *after* completion. Shepherd-style supervision catches recoverable failures *before* they become unrecoverable.

2. **Counterfactual optimization** — Meta-agent at `Planner` position, implemented as branch search: fork at decision junctions, run branches in parallel, prune bad paths early, commit best. This is inference-time MCTS applied to full agent execution trees, not to token generation.

3. **Tree-RL training** — Meta-agent as training data generator: fork at selected turns to multiply diverse training trajectories without full re-runs. Reduces the compute cost of RL on long-horizon tasks.

**Build implication:** Watch → Experiment. Shepherd is open-sourced. For teams building agent supervisors or quality gates: the immediate experiment is whether a live supervisor (meta-agent watching execution trace in real time) catches more recoverable failures than a post-hoc evaluator. The 28.8%→54.7% number on CooperBench suggests the answer is yes for coding tasks with detectable mid-execution mistakes.

For teams doing agent RL training: the Tree-RL forking approach (fork at selected turns instead of full re-runs) is worth evaluating for any long-horizon task where trajectory diversity is the bottleneck.

### Gemini Intelligence Security Architecture: Prompt Injection Defense at OS Scale

Google published the security architecture for Gemini Intelligence — the first large-scale public description of how an agentic OS layer defends against prompt injection.

**The threat model:** An agent browsing the web, reading notifications, or processing data from apps can encounter content (a web page, a notification, an email) that contains instructions designed to hijack the agent's task. This is prompt injection at scale: millions of devices, always-on agents, adversarially crafted content.

**The defense architecture:**
- **Trust hierarchy:** User instructions have the highest trust tier; environmental content (web, notifications, app data) has a lower trust tier; actions triggered by environmental content cannot override user instructions.
- **Sensitive action confirmation:** Any action that is consequential (form submission, purchase, booking) requires explicit user confirmation before execution. This is a circuit breaker: even a successfully injected instruction cannot complete a consequential action without user intervention.
- **Open-source security components:** Key parts binary-transparent + third-party audited — stated but specific components not yet disclosed. Expected to appear in Android 17 release.

**Affected stack:**
```
User → [Gemini Intelligence layer] → [Cross-app API / Chrome auto-browse] → [Environment]
                                              ↑
                                    [Content from environment: UNTRUSTED]
                                    [User instructions: TRUSTED]
                                    Sensitive actions → CONFIRMATION GATE
```

**Build implication:** The two-tier trust model (user = trusted, environment = untrusted) and the sensitive-action confirmation gate are the correct defaults for any agentic system with environmental tool use. Apply this architecture to any agent that reads from external sources (web, email, files) and takes consequential actions. The confirmation gate adds latency but removes the most critical risk class.

---

## Infra, Serving & Cloud

No major new releases in this window. Google I/O (May 19) is the next expected concentration of infrastructure announcements — Gemini 4, updated Vertex AI Agent Engine, and likely new model cards.

---

## Wider World

**Google I/O preview (May 19):** The Android Show was a precursor event. I/O proper is expected to announce: Gemini 4 (new frontier model with integrated image/video generation, extended context, improved agentic reasoning), Android 17 GA, Remy (Google's proactive AI assistant — the counterpart to Anthropic's Orbit and OpenAI's Pulse), and potential Android XR updates. Signal will be dense. The May 14 brief window should be quiet; May 19–20 will be a high-signal run.

---

## Deep Dive

### Shepherd: Why Versioned Agent Execution Is the Missing Infrastructure Primitive

**The problem**

Agent systems have a fundamental observability gap. When an agent succeeds, you have output. When it fails, you have an error message and a log of what happened — but you cannot answer the most useful debugging question: "At which point in execution would a different decision have led to success?"

This gap exists because current agent runtimes treat execution as a stream: events flow forward, states are discarded or logged but not indexed, and there is no mechanism to fork from an intermediate state. The agent either runs to completion or it crashes. There is no "git bisect" for agent behavior.

**What Shepherd actually provides**

Shepherd provides three primitives:

1. **`record(event)`** — Append a typed event (tool call, LLM response, environment observation) to the execution trace DAG. Every event has a content-addressed hash. The DAG represents the full history of execution as a structured, traversable artifact.

2. **`fork(state)`** — Create a branch from any past state. The system takes a COW filesystem snapshot + a KV-cache checkpoint. Cost: ~200ms and ~200MB for a typical coding agent state. This is 5× faster than Docker because it avoids container startup and image layers — only the agent's filesystem delta and KV state need to be copied.

3. **`replay(trace, from=t)`** — Re-run execution from state `t` forward, potentially with a different context injection (a different instruction, a corrected tool result, an intervention from a supervisor). KV-cache reuse means >95% of the prompt does not need to be re-computed — only the suffix after the fork point.

**Why the numbers are large**

The 28.8%→54.7% improvement on CooperBench (pair coding) reflects a qualitative difference in what the supervisor can do. Without Shepherd: the supervisor sees completed outputs and evaluates them — it is a post-hoc reviewer. With Shepherd: the supervisor watches the execution trace as it builds and can intervene when it detects a recoverable mistake — a wrong function call, an incorrect assumption stated mid-reasoning, a plan that will provably fail based on a tool result already in the trace.

The class of recoverable mistakes is large for coding tasks: assuming a variable exists before checking, using the wrong API method, writing a loop condition that diverges. These are visible in the trace before the agent reaches a failure state — but only if someone is watching. Shepherd makes the watching cheap.

The 58% wall-clock reduction on counterfactual meta-optimization is a consequence of pruning. A naive approach to branching exploration would run all branches to completion before comparing. Shepherd's forking mechanism allows a supervisor to detect that a branch is diverging from a correct path early (by observing intermediate trace events against expected patterns) and terminate it — exactly like alpha-beta pruning in game tree search, applied to agent execution trees.

**Before vs. after architecture**

Before:
```
[Agent execution] → [Output] → [Evaluator: pass/fail]
                                     ↓
                            [Retry from scratch if fail]
```

After (Shepherd):
```
[Agent execution] → [Execution trace DAG]
       ↑ forks              ↓ meta-agent reads
[Branches at decision points] → [Commit best path]
                                       ↓
                              [Output] → [Evaluator: pass/fail]
                                               ↓
                              [Replay from divergence point if fail]
```

The difference: failure handling goes from "restart from scratch" to "replay from the point of divergence" — which is exactly what you want if most of the work before the failure point was correct.

**Strengths**

- Open-source, designed for composability
- The functional programming model (meta-agent = function on trace → intervention) is clean and composable — you can stack multiple meta-agents (supervisor, optimizer, auditor) without coupling them
- KV-cache reuse makes replay genuinely cheap, not just theoretically cheap
- Three working applications shipped in the same paper, with concrete benchmark numbers across all three

**Failure modes and tradeoffs**

- **Fork explosion:** Counterfactual exploration branches geometrically. Without a pruning policy, the number of live branches can exceed available compute. The paper uses a supervisor to decide when to fork and when to prune — this policy is task-specific and must be designed.
- **Cache invalidation:** The >95% KV-cache reuse depends on the fork point being in the early-to-middle of the execution trace. Forking near the beginning (where the prompt diverges maximally from the base) reduces cache reuse substantially.
- **Storage cost:** Storing the full execution trace DAG for long-horizon tasks (thousands of events, large filesystems) adds up. The paper doesn't characterize storage scaling behavior for >100-step tasks.
- **Meta-agent design:** The quality of runtime intervention depends on the meta-agent's ability to detect recoverable mistakes in partial execution traces. This is not a general capability — it requires either task-specific detection logic or a powerful general-purpose meta-agent model that is itself expensive.

**So what for builders**

The immediate applications are:

1. **Live supervision for high-stakes coding agents:** If you have a coding agent where failure cost is high (production deployment agents, automated refactoring), add a Shepherd-style supervisor that watches the execution trace and intervenes on detected mistake patterns. Start with a whitelist of recoverable mistake signatures (wrong assumption mid-reasoning, tool call that will provably fail given trace history) rather than general supervision.

2. **Counterfactual debugging:** When an agent fails, don't restart from scratch. Identify the divergence point in the trace (last state before the first wrong action), fork from there, inject a correction, and replay forward. This is dramatically faster than full re-runs and gives you clean signal about the root cause.

3. **RL data amplification:** For teams doing agent RL, forking at selected turns multiplies trajectory diversity at a fraction of full run cost. This directly reduces the compute budget required for RL on long-horizon agent tasks.

The Git analogy is the correct intuition pump: just as you don't re-write your entire codebase when you discover a bug (you `git checkout` to the last good state), you shouldn't re-run your entire agent from scratch when it fails mid-execution. Shepherd makes the "checkout" primitive available for agents.

---

## Small Finds

- **BoundaryRouter (arXiv:2605.07180, May 8):** Training-free routing between direct LLM inference and full agent execution. Key mechanism: runs both systems on a shared seed set of queries → builds an experience memory of which query types needed agent vs. LLM → uses retrieval at inference time to route new queries. Results: 60.6% inference time reduction vs. always-agent, 28.6% accuracy improvement vs. always-LLM. Introduces RouteBench (in-domain, paraphrased, out-of-domain routing scenarios). Slightly outside the 48h window (May 8) but the practical impact — essentially a production-ready LLM/agent router with no training required — is high enough to include. ([arxiv.org/abs/2605.07180](https://arxiv.org/abs/2605.07180))

- **Googlebooks laptops:** Google announced new Googlebook laptops (Acer, Asus, Dell, HP, Lenovo OEM partners, launching fall 2026) built around Gemini Intelligence. Features include "Magic Pointer" (cursor with Gemini built in), native Android phone app integration, custom widget creation, and Gemini Intelligence processing with personal data handling. Architectural note: these are the first consumer laptops with a named AI intelligence tier as a first-class hardware/software co-design constraint — the pattern Anthropic is pursuing with Orbit and OpenAI with DeployCo. ([techcrunch.com](https://techcrunch.com/2026/05/12/everything-google-announced-at-its-android-show-from-googlebooks-to-vibe-coded-widgets/))

- **Accenture -3% on DeployCo:** The market's immediate reaction to OpenAI's Deployment Company launch confirms that the SI community reads this as direct competition. Accenture has ~400K practitioners globally; DeployCo starts with 150 FDEs. The competitive dynamics are asymmetric in headcount but OpenAI has model-level access advantages (pre-release capabilities, direct API routing, FDE training on frontier capabilities) that SIs cannot replicate. Structural tension to watch.

- **Google I/O May 19 — expect Gemini 4:** The Android Show was explicitly the precursor event. The main I/O keynote (May 19, 10AM PT) is expected to announce Gemini 4, updated Deep Research, Remy (Google's proactive AI assistant), and potentially new Vertex AI Agent Engine capabilities. Defer coverage of Google's AI stack until post-keynote.

---

## Frontier Direction

- **Bottleneck under attack:** Agent execution observability and recovery. Shepherd attacks the specific failure mode where agents can't be debugged at the execution level — you can evaluate outputs but you can't inspect or branch the execution tree. Git semantics for agent execution is the mechanism.
- **Broader trend:** Agent infrastructure is evolving from "run and evaluate" to "version, branch, and replay." The same week Anthropic shipped Dreaming (memory consolidation between sessions), Stanford shipped Shepherd (execution versioning within and across runs). Together they address the two most ignored infrastructure gaps: session-to-session learning and within-session error recovery.
- **Still unsolved:** Pruning policy design for counterfactual exploration. Shepherd provides the forking mechanism; it does not provide a general solution to "when should the meta-agent fork, and when should it let the agent proceed?" This is task-specific and must be hand-designed or learned — the paper uses a supervised supervisor for coding tasks, but the general case is open.
- **Emerging paradigm:** Meta-agent as infrastructure layer. Shepherd, MAGE shadow memory, Anthropic's Outcomes evaluator, and cotomi Act's verbal-diff compression all share a structural pattern: a parallel process that monitors, compresses, or evaluates the primary agent's execution without being part of that execution. The meta-agent layer is solidifying as a first-class architectural tier.

Arrows:
- Single-shot agent execution (run → evaluate → retry from scratch) → Versioned agent execution (fork → branch → replay from divergence point) (Shepherd)
- Post-hoc quality evaluation (Outcomes evaluator, clean-context) → Live runtime supervision (meta-agent watching execution trace in progress) (Shepherd)
- Prompt-based LLM/agent routing (static classification) → Experience-memory routing (BoundaryRouter: retrieval over behavioral seed set) (arXiv 2605.07180)
- Reactive AI assistant (prompt → respond) → Agentic OS layer (Gemini Intelligence: cross-app, always-on, OS-level permissions) (Google Android Show)

---

## Builder Takeaways

### Try now
**Read and run the Shepherd examples on GitHub.** The paper is open-sourced. The most immediately applicable experiment: implement a Shepherd-style runtime supervisor for your most failure-prone agent. Start narrow — identify 3–5 recoverable mistake patterns specific to your task (wrong assumption mid-reasoning, tool call that conflicts with a prior tool result, plan that requires a file that doesn't exist). Build a meta-agent that watches for these patterns in the execution trace and injects a correction message when detected. Measure: does early intervention change the pass rate? What fraction of failures are recoverable vs. fundamental?

### Experiment with
**BoundaryRouter on your own query distribution.** The training-free routing approach (seed set → experience memory → retrieval-based routing) is implementable in an afternoon. Pick 50 representative queries from your workload. Run both direct LLM and full agent on all 50. Build the experience memory. Then route your next 100 queries and measure: what fraction correctly routes to LLM (fast, cheap) vs. agent (capable, expensive)? This gives you empirical data on your specific LLM/agent boundary before committing to a routing architecture.

### Go deep on
**Execution trace design and meta-agent architecture.** Shepherd formalizes a pattern — typed event traces, fork/replay semantics, meta-agents as functions on traces — that will be the foundation of production agent debugging infrastructure. The specific skills to build: (1) designing typed event schemas for agent traces (what events matter, what metadata to capture); (2) building meta-agents that reason over partial traces rather than final outputs; (3) fork/replay infrastructure — either using Shepherd directly or implementing the COW + KV-cache-checkpoint pattern. These skills apply across agent frameworks and compound as agent systems get longer-horizon. Read: Shepherd paper + source, the MAGE shadow memory paper (arXiv:2605.03228 from May 7), and the systems security literature on shadow stacks (the conceptual inspiration for both).

### Ignore for now
**OpenAI Deployment Company as a competitive threat if you're building software.** DeployCo is targeting complex multi-department enterprise transformation projects that require embedded engineers. If you're building AI-native software products or developer tools, this is not your competitive surface. The relevant signal is the FDE archetype — understand what Forward Deployed Engineers do and whether that skill set creates value on your roadmap.

---

## What to Build

**Project: Agent Execution Tracer + Replay Tool**
- **What to build:** A lightweight, framework-agnostic execution trace recorder for LLM agents — records every tool call, LLM response, and state transition as a typed event; provides replay from any checkpoint; integrates with LangGraph, OpenAI Agents SDK, and CrewAI via adapters; includes a CLI for browsing and diffing traces.
- **Why now:** Shepherd just demonstrated that typed execution traces + replay unlock runtime supervision (28.8%→54.7% CooperBench), counterfactual debugging (58% wall-clock reduction), and RL data amplification. The Shepherd system is designed for their specific use case; a framework-agnostic tracer that exposes these capabilities for the broader agent ecosystem doesn't exist yet.
- **Stack:** Python, adapters for LangGraph / OpenAI Agents SDK / CrewAI, SQLite for trace storage (WAL mode for concurrent writes during execution), subprocess-level fork via Python's `os.fork()` + filesystem snapshot, Redis for KV-cache state serialization (works with vLLM's KV-cache export API), CLI using Rich.
- **What you'd learn:** Typed event schema design for heterogeneous agent actions; COW filesystem snapshotting at the Python process level; KV-cache serialization and replay with vLLM/SGLang; how often in practice a live supervisor finds recoverable failures vs. fundamental failures. The last data point is the most valuable — it tells you whether runtime supervision is worth the overhead for your task type.

**Project: LLM/Agent Boundary Profiler**
- **What to build:** A profiling tool that runs any task set through both direct LLM and full agent, records behavioral signals (token count, tool call count, response latency, pass/fail), and builds a BoundaryRouter-compatible experience memory. Outputs a routing policy (decision boundary + confidence regions) and a cost-quality frontier chart showing where routing saves money without sacrificing quality.
- **Why now:** BoundaryRouter showed 60.6% inference time reduction with no training. Most teams route naively (always use agent for complex tasks, always use LLM for simple ones) because they don't have empirical data on their specific query distribution's LLM/agent boundary. This tool generates that data automatically.
- **Stack:** Python, any LLM API (call both direct and agent on same queries), FAISS for experience memory retrieval, matplotlib for cost-quality frontier visualization, JSON schema for portable experience memory format.
- **What you'd learn:** Where your specific workload's LLM/agent boundary actually sits vs. where you assumed it was; how stable the boundary is across query paraphrases (RouteBench's key finding: in-domain routing is easier than out-of-domain); the actual cost multiplier of agent vs. LLM for your task type.

---

## Opportunities

1. **Framework-agnostic Shepherd adapter:** Shepherd is open-source but designed for a specific Stanford research setup. Building adapters that make Shepherd's fork/replay primitives available inside LangGraph, OpenAI Agents SDK, and CrewAI — without requiring teams to rewrite their agent infrastructure — would unlock immediate adoption. The critical engineering challenge is KV-cache portability: each inference framework (vLLM, SGLang, OpenAI API) handles KV state differently. A unified KV-cache snapshot interface is the hardest and most valuable piece.

2. **Prompt injection detection library for agentic Chrome/Android apps:** Google's security architecture blog confirms that Gemini Intelligence's prompt injection defense is open-source and auditable. A developer library that implements the same two-tier trust model (user = trusted, environment = untrusted) + sensitive-action confirmation gate — packaged for web extensions, Android apps, and browser agents — would fill an immediate gap as Chrome auto-browse and Gemini Intelligence spawn a wave of third-party agentic app development.

3. **Agent execution trace analytics:** Teams running agent workloads accumulate execution logs, but have no tooling to analyze them at the trace level (where in execution do failures occur most? which decision points have highest counterfactual impact? which tool calls precede failure most often?). An analytics layer over Shepherd-format execution traces — similar to what Weights & Biases does for training runs — would make trace data actionable without requiring engineers to write custom analysis each time.

---

*Sources:*
- [Shepherd: arXiv:2605.10913](https://arxiv.org/abs/2605.10913)
- [Shepherd on Hugging Face Papers](https://huggingface.co/papers/2605.10913)
- [BoundaryRouter: arXiv:2605.07180](https://arxiv.org/abs/2605.07180)
- [TechCrunch: Everything Google announced at Android Show](https://techcrunch.com/2026/05/12/everything-google-announced-at-its-android-show-from-googlebooks-to-vibe-coded-widgets/)
- [Google blog: Gemini Intelligence for Android](https://blog.google/products-and-platforms/platforms/android/gemini-intelligence/)
- [Google blog: Android Show I/O Edition 2026](https://blog.google/products-and-platforms/platforms/android/android-show-io-edition-2026/)
- [Google Security blog: Android's Agentic Future — Security & Privacy](https://blog.google/security/android-gemini-intelligence-security-privacy/)
- [Google blog: Gemini in Chrome auto browse for Android](https://blog.google/products-and-platforms/products/chrome/bringing-chrome-ai-to-android/)
- [OpenAI: OpenAI Deployment Company launch](https://openai.com/index/openai-launches-the-deployment-company/)
- [CIO Dive: OpenAI spins up standalone consulting business](https://www.ciodive.com/news/openai-deployment-company-4-billion-ai-consulting-integration/819942/)
- [CoinCentral: Accenture -3% on DeployCo](https://coincentral.com/accenture-acn-stock-falls-3-after-openai-launches-deployment-company/)
- [9to5Google: Everything announced at The Android Show](https://9to5google.com/2026/05/12/the-android-show-2026/)
- [TechCrunch: Google brings agentic AI and vibe-coded widgets to Android](https://techcrunch.com/2026/05/12/google-brings-agentic-ai-and-vibe-coded-widgets-to-android/)
