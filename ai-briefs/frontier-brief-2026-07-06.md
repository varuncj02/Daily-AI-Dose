# Frontier AI Brief — 2026-07-06

> Covering: 2026-06-30 to 2026-07-06 (catching up from 2026-06-23 gap)
> 12 candidates reviewed · 5 kept as Top Signals + 3 Agentic/Infra items + 3 Small Finds · 4 discarded (thin evidence / no published artifact / Grok 4.5 private beta only)

---

## Executive View

The six-day window from June 30 to July 6 resolved several threads this brief series had been tracking. Fable 5's 19-day export-control shutdown ended July 1 via a reviewed single-classifier fix — the resolution anatomy matters more than the restoration itself, because it establishes the first documented template for how a government/lab "model recall" dispute gets technically resolved. Claude Sonnet 5 launched June 30 as the most architecturally significant mid-tier model release in months: adaptive thinking is now always-on and inference-time reasoning effort becomes a slider, not a toggle. GPT-5.6 Sol/Terra/Luna previewed under an unprecedented government-gated release structure, and Cerebras is preparing 750 tok/s frontier inference — an order-of-magnitude jump over current GPU deployments. Claude Code's latest release ships two genuinely new agentic loop patterns: autonomous draft-PR creation by background agents, and real Chrome browser access in the agent loop. Together these releases represent a step-function in the default operational capability of everyday agentic coding workflows — not just benchmark numbers.

---

## Top Signals

### [Introducing Claude Sonnet 5](https://www.anthropic.com/news/claude-sonnet-5) · High
*Published: 2026-06-30*

**What changed**
Claude Sonnet 5 launched June 30 as Anthropic's new default model for Free, Pro, and Max plans — and the new default for Claude Code. Key changes from Sonnet 4.6: (1) adaptive thinking is **on by default** on every request, replacing the opt-in extended thinking toggle; (2) manual extended thinking is removed and returns a 400 error; (3) sampling parameter customization now returns a 400 error; (4) a new tokenizer produces ~30% more tokens for the same text; (5) 1M token context window is now the standard, not an opt-in; (6) max output tokens is 128k. Pricing at launch: $2/$10 per M input/output through August 31, then $3/$15.

**How it works**
Adaptive thinking lets the model decide *how much* to reason per request based on complexity, replacing the binary "thinking on/off" switch. Exposed effort tiers: low, medium, high, max, xhigh — builders can pin a tier or leave it fully adaptive. Critically, adaptive thinking automatically enables interleaved thinking: Claude can reason *between* tool calls, not just before the first one. This changes the execution shape of tool-heavy agentic workflows — the model can reconsider its plan after each tool result rather than committing before any tool fires.

The new tokenizer produces ~30% more tokens for identical input text. Pricing per token is unchanged ($3/$15), but **effective per-request cost increases ~30%** unless you're in the introductory period. Builders pricing out Sonnet 5 workloads vs. Sonnet 4.6 need to factor this in.

Setting non-default sampling parameters (temperature, top_p, etc.) now throws a 400 error — the model's behavior is now defined by effort tier, not by caller-controlled sampling. This is a breaking API change for any code that passed custom sampling params to Sonnet 4.x.

**Why it matters**
Interleaved adaptive thinking is the mechanism that makes tool-heavy agents qualitatively different. Prior models reasoned once (or at fixed interleaved points); Sonnet 5's adaptive interleaved reasoning means the agent builds a richer internal model of the task state after each tool result. Reported gains are largest on coding and multi-step agentic tasks.

The 400 error on custom sampling parameters signals Anthropic's direction: the model, not the caller, manages its own inference behavior. This trades flexibility for reliability in long-horizon agentic runs (fewer hallucination modes from unusual sampling configs), but removes levers that some advanced callers use deliberately (low temperature for determinism, high temperature for creative diversity).

**What to update in your mental model**
"Extended thinking" as a feature you turn on is dead. The new model is: reasoning is always active; you steer *effort level*, not *presence*. Any orchestration layer that conditionally enabled thinking for hard subtasks needs to be rearchitected around effort tiers instead.

---

### [Claude Fable 5 / Mythos 5 Restored — Classifier Fix Anatomy](https://www.tomshardware.com/tech-industry/artificial-intelligence/anthropic-restores-claude-fable-5-as-us-lifts-export-controls) · High
*Published: 2026-07-01*

**What changed**
The US Commerce Dept. lifted export controls on Fable 5 and Mythos 5 on June 30. Fable 5 came back online globally July 1 — 19 days after the June 12 shutdown. The resolution mechanism: Anthropic trained a new cybersecurity classifier that runs alongside Fable 5 in real time, targets the specific jailbreak technique Amazon researchers flagged (a prompt that got the model to identify software vulnerabilities and write functional exploit code), and blocks it in >99% of cases, rerouting flagged requests to Opus 4.8. The classifier was reviewed and cleared by Commerce's Center for AI Standards and Innovation (CISA) before controls came off.

**How it works**
The fix is architecturally a **guard model** pattern: a smaller, faster classifier runs on every request before it reaches Fable 5. If the classifier fires, the request is rerouted to a less capable model (Opus 4.8), not rejected outright — this preserves user experience while preventing the flagged capability surface. The fix does not alter Fable 5's weights; the model retains the underlying capability. The classifier is tuned to detect the *technique*, not the capability itself.

Disclosed side effect: the classifier also flags some benign coding and security debugging requests. This is the canonical tradeoff of a technique-targeting classifier vs. a capability-removing fine-tune — the former is faster to deploy but has higher false-positive rate on adjacent legitimate tasks.

Fable 5 is available via Pro, Max, Team, and selected Enterprise plans at up to 50% of weekly usage limits through July 7, then reverts to the standard credit model ($10/M input, $50/M output).

**Why it matters**
The 19-day shutdown is the first fully resolved "model recall" event in commercial AI history. The resolution anatomy — (1) lab trains classifier fix, (2) classifier submitted to government review, (3) government clears fix, (4) controls lifted — is now a documented template. Every frontier lab now knows what a realistic resolution path for a similar event looks like, including: ~18-day negotiation timeline, classifier-based mitigation (not weight surgery), and mandatory government review before restoration.

The ECRA "deemed export" mechanism that triggered the shutdown (15 CFR 734.13 — providing API access to a foreign national inside the US counts as a regulated export) is still standing law and applies to any frontier model with similar capabilities. The Fable 5 precedent doesn't make future shutdowns less likely; it shows they can be resolved faster once labs have the classifier-training playbook.

**What to update in your mental model**
"Classifier-as-runtime-guard reviewed by government body" is the viable path for resolving a model recall — not weight retraining, not capability removal, not full model replacement. The CISA review step means any lab with a frontier model worth exporting needs an internal process for rapidly building, evaluating, and submitting safety classifiers for government review. "How fast can you ship a targeted safety classifier to a government auditor?" is now a real organizational capability question.

---

### [Previewing GPT-5.6 Sol, Terra, and Luna](https://openai.com/index/previewing-gpt-5-6-sol/) · High
*Published: 2026-06-26 (accessed/reported this window)*

**What changed**
OpenAI previewed GPT-5.6 as a three-tier model family under the June 2 EO's voluntary pre-release-access framework: Sol (flagship, $5/$30 per M tokens), Terra (balanced, $2.50/$15), Luna (fast, $1/$6). Access is restricted to ~20 government-vetted organizations; no public enrollment or waitlist. OpenAI explicitly states it provided the government 30 days' advance access before releasing to any trusted partners. Separately, GPT-5.6 Sol is launching on Cerebras wafer-scale hardware in July at up to **750 tokens per second**.

**How it works — Cerebras deployment**
Cerebras runs Sol on its WSE-3 wafer-scale chips. The 750 tok/s figure is approximately 10x faster than frontier-model GPU deployments in production (typical GPU range: 40–120 tok/s for streaming completions). Mechanism: the WSE-3 keeps the entire model's activation state on-chip (800,000 cores, 44 GB on-chip SRAM, no HBM round-trips), eliminating the memory bandwidth bottleneck that constrains GPU inference at large model scales. At 750 tok/s, an agentic workflow that takes 30s on a standard GPU cluster completes in under 3s — within human-interactive latency thresholds for complex reasoning tasks.

**Why it matters**
Two independent signals here, both important:

*Government-gated release structure*: GPT-5.6 is the first frontier release to follow the EO Section 3 voluntary pre-brief framework in full — 30-day government access, then staged rollout to vetted partners, then general availability in "coming weeks." OpenAI complied; Anthropic did not (Fable 5 launched 7 days post-EO without a pre-brief, which contributed to the shutdown). This creates implicit pressure: labs that skip the pre-brief process now have a clear precedent for the consequence (shutdown risk) vs. compliance (staged rollout with intact access).

*750 tok/s at frontier quality*: This is not an open-model optimization trick — it's a closed frontier model at sub-3s end-to-end latency for complex multi-step chains. The practical implication for agentic systems: inference speed has been the dominant bottleneck in user-facing agentic loops. At 750 tok/s, the bottleneck shifts entirely to tool execution (API calls, browser operations, code execution) — a structurally different system design problem.

**What to update in your mental model**
The government pre-brief framework is becoming the de facto compliance mode for US frontier labs releasing new capability tiers. Labs that don't participate face shutdown risk; labs that do participate get staged rollouts rather than full immediate availability. General availability timelines will now have a policy-gating variable that didn't exist pre-June 2026. For builders: plan for top-ranked models to be unavailable for 30-60 days post-announcement, and maintain capable fallback tiers.

---

### [Claude Code v2.1.198: Background Agents Auto-Commit, Push, and Open Draft PRs + Claude in Chrome GA](https://code.claude.com/docs/en/changelog) · High
*Published: 2026-07-01*

**What changed**
Claude Code v2.1.198 shipped two significant agentic loop advances:
1. **Background agents complete a full write-test-review cycle autonomously**: after finishing code work in a worktree, agents now automatically commit changes, push to a remote branch, and open a draft PR — without stopping to ask. They then load the PR's live preview URL in a real Chrome session to run a smoke test.
2. **Claude in Chrome reaches general availability**: Claude Code can now drive a real Chrome browser (not a headless test environment) to test web apps, capture console logs, fill forms, and extract page data. The integration uses the Claude in Chrome extension's DOM tools, not screenshot-based computer use.

Additional: background agent notifications now fire the `Notification` hook (`agent_needs_input` / `agent_completed`), enabling teams to integrate completion signals into Slack, PagerDuty, or custom workflows. The `AskUserQuestion` dialog no longer auto-continues by default.

**How it works**
The draft-PR handoff pattern closes what was previously an open loop in background agents: the agent did work, then stopped at a "should I commit?" prompt. Now the agentic loop extends to: edit → commit → push → open draft PR → smoke test in real browser → complete. The human reviewer sees a PR that has already been self-tested against a live preview. The worktree isolation (introduced previously) means the agent's uncommitted work was already contained; the commit/push/PR sequence just surfaces that work in a form the review workflow can act on.

Claude in Chrome GA replaces the prior screenshot-based approach with DOM-aware tools (`find`, `form_input`, `get_page_text`, `javascript_tool`). DOM-aware browsing is faster, less brittle against layout changes, and produces more reliable structured extraction than vision-based click-at-pixel approaches.

**Why it matters**
The draft-PR-with-smoke-test loop is the first fully autonomous write-test-review cycle in a shipping coding agent. Prior background agents wrote code; this one writes, tests (against a real browser), and hands off a review-ready artifact. The human's first interaction is a PR review, not a "should I commit?" dialog.

The DOM-aware Chrome integration means agents can reliably handle web-app testing tasks that were previously fragile (screenshot-based clicking fails on any layout change; DOM queries are stable against style changes). This closes a real gap in frontend development workflows.

**What to update in your mental model**
"Background agent" is no longer a synonym for "long-running code editor." It's now a minimal autonomous software delivery unit: it writes code, validates it in a real browser, and opens a deliverable for human review. The human is downstream of the test cycle, not upstream.

---

### [EO Section 3 AI Cybersecurity Clearinghouse Established (Treasury/NSA/CISA)](https://www.whitehouse.gov/presidential-actions/2026/06/promoting-advanced-artificial-intelligence-innovation-and-security/) · Watch
*Published/operative: 2026-07-02*

**What changed**
The AI Cybersecurity Clearinghouse mandated by the June 2 EO was established July 2, its statutory deadline. Treasury leads; NSA and CISA co-participate. The clearinghouse's stated function: coordinate AI-assisted scanning for software vulnerabilities across critical infrastructure, validate findings, and manage disclosure and patching timelines — effectively a centralized deconfliction layer between AI vulnerability-discovery tools and the organizations they scan.

**Why it matters**
This operationalizes the EO's most concrete near-term infrastructure: a government body that can receive vulnerability findings from AI tools (including frontier models like Mythos 5, which NSA red-teamed), vet them, and coordinate disclosure. Combined with the voluntary pre-release review process (which also routes through this infrastructure), the clearinghouse makes the US government a node in the vulnerability-discovery and disclosure pipeline for AI-assisted security work — not just a regulator of model releases, but an active participant in what AI models find and when it's disclosed.

For builders deploying AI agents that do any security scanning or vulnerability discovery, this creates a new question: does your tool's output need to go through the clearinghouse disclosure process? The EO is voluntary, but "voluntary" has been shown to carry significant implicit pressure (see GPT-5.6 pre-brief compliance vs. Fable 5 shutdown).

**What to update in your mental model**
AI-assisted vulnerability discovery is now a policy-regulated activity at the federal level, with a specific government body to route findings through. If you're building security scanning agents or coding agents with vulnerability-identification capability, the clearinghouse is the relevant institutional contact — not just CISA's standard CVD process.

---

## Agentic Architecture & Engineering

### Claude Sonnet 5: Adaptive Thinking as the New Default Architecture

**Affected stack**
`User → Planner → [Reasoning] → Memory → Retriever → LLM → [Interleaved Reasoning] → Tools → [Interleaved Reasoning] → Verifier → Output`

The shift: reasoning now happens *at every Reasoning node in the stack*, not just before the first LLM call. Adaptive interleaved thinking means the agent can reassess its plan after each tool result without the caller having to explicitly enable it.

**Build implication: Adopt.** Redesign any orchestration layer that conditionally enabled "thinking" for hard subtasks. Replace with effort tier steering (`thinking_effort: "high"` or `"max"` for hard subtasks, `"low"` or omitting the param for simple ones). Remove any code that passes custom sampling parameters — these now throw 400.

Also: account for the ~30% tokenizer-induced cost increase in all budget estimates. If you were caching frequently seen Sonnet 4.6 prompt prefixes, those caches may be cache-miss-heavy until re-warmed with Sonnet 5's tokenization.

### Background Agent Autonomy Increase in Claude Code

**Affected stack**
`Planner → Code Execution (worktree) → [NEW: commit + push + PR open] → [NEW: real browser smoke test] → Human Review Queue`

The agentic loop now closes all the way to a PR-ready artifact with empirical validation. The human's role shifts from "approve commit?" to "review diff after automated test passes."

**Build implication: Experiment.** If you run background Claude Code agents for code generation, upgrade to v2.1.198 and wire up the `agent_completed` notification hook to your review workflow (Slack/PagerDuty/email). Measure the reduction in back-and-forth per PR and the false-positive rate on draft PRs that the agent's own smoke test catches vs. misses.

---

## Infra, Serving & Cloud

**GPT-5.6 Sol on Cerebras at 750 tok/s — quantifying the wafer-scale inference gap:** The 750 tok/s figure is roughly 10x the production GPU frontier for streaming completions (40–120 tok/s typical). The mechanism is SRAM-based: WSE-3 avoids HBM bandwidth bottlenecks entirely by keeping the model's activation state on-chip. At this throughput, the latency bottleneck in agentic loops shifts from model inference to tool execution. This doesn't change agent design today (access is gated to ~20 orgs) but sets the floor for what production frontier inference will cost once wafer-scale supply scales. Cerebras IPO is planned for 2026; this deployment is a proof point.

**Fable 5 / Mythos 5 back on credits ($10/$50 per M tokens) as of July 7:** The introductory period (50% of weekly limits included for Pro/Max/Team) ends July 7. Any pipeline budgeted against free Fable 5 access needs to be re-costed today.

**Sonnet 5 tokenizer: ~30% token inflation vs. Sonnet 4.6.** All cached cost estimates, token budget calculations, and context window planning for Sonnet 4.6 workflows are stale. Re-run with Sonnet 5's tokenizer before migrating high-volume pipelines. Introductory pricing ($2/$10) holds through August 31; standard is $3/$15.

---

## Wider World

**White House voluntary pre-release AI framework is now operational, not just a policy:** Both the Fable 5 resolution (classifier reviewed by CISA before controls lifted) and GPT-5.6's staged rollout (30-day government pre-brief honored by OpenAI) confirm the June 2 EO's framework is functioning as a real gate, not a paper requirement. Labs that engage get staged access; labs that skip risk shutdown. OpenAI is the first lab to fully comply with the pre-brief requirement for a new frontier release. **Anthropic's biometric ID verification (July 8 deadline)** lands in two days — that policy's interaction with the export-control framework (ID verification as a nationality-check mechanism to satisfy ECRA requirements) will be the next data point on how labs operationalize compliance.

**EU AI Act transparency rules: 27 days to enforcement (August 2).** From June 23's brief: Article 50 obligations apply August 2. The window is now under a month. If you have EU-facing AI deployments without disclosure labeling, the clock is running.

---

## Deep Dive

### Claude Sonnet 5: What Adaptive Interleaved Thinking Actually Changes

**The problem it attacks**
Standard LLM-in-agent-loop architectures front-load all reasoning before the first tool call. This means: the model commits to a plan before seeing any tool output. When tool outputs surprise it (API returns an error, file doesn't exist, web page structure differs from expected), the model must either (a) fail and retry from scratch, (b) continue with a stale plan, or (c) call for help. None of these are good for long-horizon tasks.

**Core mechanism: before vs. after**

*Before (Sonnet 4.6 with optional thinking):*
```
System prompt → User message → [thinking: block] → tool_use call → tool_result → tool_use call → tool_result → ...
```
Thinking happened once, upfront. Subsequent tool calls were made with fixed priors from the initial reasoning block.

*After (Sonnet 5 with adaptive interleaved thinking):*
```
System prompt → User message → [think] → tool_use → tool_result → [think] → tool_use → tool_result → [think] → response
```
The model has a reasoning step after *each* tool result. It can revise its world model, detect plan failures, and choose a different next tool — without the caller orchestrating this.

The effort tier parameter (`thinking_effort: low/medium/high/max/xhigh`) controls how much compute the model allocates to each thinking step, not whether thinking occurs at all.

**What this changes architecturally**
Interleaved thinking means the agent loop can be simpler on the caller side. Previously, robust agent orchestration required the caller to: detect tool-call errors, inject error context back into the prompt, re-invoke the model, and check for plan divergence. With interleaved thinking, the model can handle lightweight versions of all these internally — it sees the tool_result, thinks about what it means, and chooses accordingly.

This doesn't eliminate the need for caller-side retry logic (tool timeouts, auth failures, and network errors still need handling), but it shifts the model from a "plan-execute" architecture to a "plan-execute-observe-replan" architecture at the token level.

**The tokenizer tradeoff**
The new tokenizer's ~30% inflation is a direct cost of the adaptive thinking architecture: more tokens are needed to represent the reasoning blocks between tool calls. The model is doing more per-token work; the caller pays for it. At the $3/$15 price point, a Sonnet 4.6 workflow that cost $10 now costs ~$13 in steady state. For high-volume pipelines with many tool calls, this cost increase can be significant — but the rework/retry cost reduction from better mid-loop reasoning often offsets it.

**Failure modes and tradeoffs**
- **Effort over-allocation**: leaving effort fully adaptive means the model may spend `xhigh`-level reasoning on trivial subtasks. Builders with cost constraints should profile effort level per task type and hard-pin tiers for known-simple operations.
- **400 on sampling params**: any existing code that passes temperature, top_p, etc. to Sonnet-class models breaks immediately. This is a hard API-level change with no backward-compatibility shim.
- **Cache invalidation**: prompt caches tuned to Sonnet 4.6's tokenizer will have different cache keys. Re-warm before depending on cached prefixes for latency or cost savings.
- **Reasoning not visible by default**: adaptive thinking's internal reasoning is not exposed to the caller unless you explicitly enable thinking output. The model is making plan revisions the caller can't inspect in most default configurations.

**So what for builders**
Adopt Sonnet 5 for any multi-tool agentic workflow where today you're seeing plan divergence after tool failures. The interleaved reasoning is the primary value add. For simple single-turn or few-tool tasks, Sonnet 4.6 or Haiku 4.5 remain cheaper with no meaningful quality loss. Fix the 400-on-sampling-params breakage immediately if you have it — it's a hard failure, not a degradation.

---

## Small Finds

- **AGI Maze benchmark (arXiv:2607.00627, July 2026)**: A new benchmark framework for world-modeling agents — tests agents on navigation and planning tasks that require building a coherent world model rather than pattern-matching on seen examples. Early-stage paper; worth watching if it gets eval harness releases. World-model evaluation is genuinely underserved vs. coding/reasoning benchmarks.

- **HARC: Coupling Harmfulness and Refusal Directions (arXiv:2607.00572)**: New safety-alignment paper addresses the mismatch between harmfulness detection and refusal behavior in safety-trained models — specifically the failure mode where models refuse benign requests (false positive) or accept harmful ones (false negative) because the two learned directions aren't properly coupled. Mechanism-level paper, not production ready, but relevant to the Fable 5 classifier tradeoff (false positives on benign security requests) discussed in Top Signals.

- **Fable 5 19-day shutdown post-mortem is the most important enterprise AI infrastructure lesson of 2026 so far.** MarketScale coverage framed it clearly: a single government action suspended the #1-ranked model for every enterprise customer simultaneously, with no per-customer opt-out and ~90 minutes notice. Every multi-provider fallback architecture built since the GPT-4 era was validated this month. If you don't have at least one capable fallback routing to a non-restricted alternative, you have infrastructure debt.

---

## Frontier Direction

- **Bottleneck under attack**: Agentic loop latency — the Cerebras/GPT-5.6 Sol deployment and Sonnet 5's interleaved reasoning both attack the inference-side contribution to end-to-end agent latency. At 750 tok/s, the model stops being the bottleneck; tool execution becomes the constraint.
- **Broader trend**: Model-managed inference behavior (adaptive thinking, effort tiers) is replacing caller-managed inference behavior (temperature, sampling params, explicit thinking toggles). The model decides how to think; the caller decides how much effort to apply. This is a direction, not just a Sonnet 5 quirk — expect it to propagate to other models.
- **Still unsolved**: The 30% tokenizer cost inflation from interleaved reasoning is a real tax on token-efficient pipeline design. No technique for "reasoning-compressed" output (summary of thinking steps instead of full token rendering) has landed in production yet. Also still unsolved: agentjacking provenance gap in MCP — still no spec-level fix.
- **Emerging paradigm**: Government as a node in the model-release pipeline — not just a regulator of model capabilities post-launch, but a participant in the pre-release review, vulnerability discovery, and classifier-fix review process. The Fable 5 and GPT-5.6 cases together define a new "government-gated release" paradigm that's now operational, not theoretical.

Arrows:
- Extended thinking as optional toggle → Adaptive interleaved reasoning as always-on with effort steering
- Agent loop as plan-then-execute → Agent loop as plan-execute-observe-replan at the token level
- Government as post-hoc AI regulator → Government as pre-release access node and classifier-fix reviewer
- Human review of agent output → Human review of agent-created-and-tested PR artifact

---

## Builder Takeaways

### Try now
**Upgrade to Claude Code v2.1.198 and wire the `agent_completed` Notification hook** — background agents now auto-commit, push, and open draft PRs with smoke tests. Set up a webhook from the hook to your Slack or ticketing system and run a real background agent on a bounded coding task (a specific bug fix or isolated feature). The loop now completes to a PR; measure whether the agent's self-generated smoke test catches issues you'd previously only catch in code review.

### Experiment with
**Profile Sonnet 5 effort tiers on your specific workload.** Run the same multi-tool agent task at `low`, `medium`, `high`, and `max` effort. Measure: (1) task success rate, (2) total tokens used, (3) latency to completion, (4) number of tool calls before completion. Map the tradeoff curve for your specific task distribution. Most workloads have a "good enough" effort tier well below `max` — but you can't know which without the profile. The introductory pricing window ($2/$10 through August 31) makes this the cheapest time to run the experiment.

### Go deep on
**Inference-time compute scaling as a system design axis.** The Sonnet 5 release (effort tiers as a compute dial) and the Cerebras/GPT-5.6 deployment (750 tok/s frontier inference) together point to the same emerging reality: inference compute is now a first-class variable in system design, not a fixed cost. Understanding how to allocate inference compute across subtasks in an agent pipeline — when to use `xhigh` effort, when to route to a cheap model, when the bottleneck is actually tool latency not model latency — is the skill that separates a well-tuned agentic system from a na¨ıvely assembled one. Study: Anthropic's adaptive thinking docs, the SemiAnalysis inference economics data (posted in prior brief), and build a small benchmark that varies model tier × effort level × task complexity for your domain.

### Ignore for now
**Grok 4.5** — 1.5 trillion parameters confirmed in private beta at SpaceX/Tesla, no public access, no published benchmarks. Revisit when weights or API access land. The parameter count alone tells you nothing useful about capability distribution.

---

## What to Build

**Project:** A multi-provider agent router that dynamically selects model + effort tier based on task complexity classification, with automatic failover across Sonnet 5, Fable 5, Opus 4.8, and an open-weight fallback (GLM-5.2 or Cohere North Mini Code), logging which tier was actually used and the task success outcome.
**Why now:** The Fable 5 shutdown demonstrated that single-model agent pipelines fail at the government-action level. Sonnet 5's effort tiers create a new optimization axis. The GPT-5.6 gated release suggests more government-gated tiers are coming. A router that handles all three concerns (failover, effort optimization, government-availability flags) is a real infrastructure need that no off-the-shelf tool handles end-to-end yet.
**Stack:** Claude Code (Sonnet 5 default), LiteLLM for unified routing (patched to 1.83.7+), a lightweight task complexity classifier (Haiku 4.5 for classification, Sonnet 5 for execution), GLM-5.2 via vLLM for the open-weight failover tier.
**What you'd learn:** Agentic infrastructure design: provider abstraction, real failover routing, inference cost optimization, and production observability for multi-model pipelines — all in one concrete system.

---

**Project:** A Sonnet 5 effort-tier profiler: a test harness that runs a representative sample of your team's actual agent tasks at all five effort tiers, collects success rate, token count, and latency, and outputs a per-task-type tier recommendation with cost savings estimate.
**Why now:** Sonnet 5's introductory pricing ($2/$10) ends August 31. Running this profiler now, before prices normalize, gives you the full tradeoff curve at minimal cost. The recommendation output directly translates into `thinking_effort` configuration in your production prompts.
**Stack:** Python, Claude SDK, a fixed corpus of 20-50 representative tasks from your actual workload, simple SQLite for result logging, matplotlib for the tradeoff curve visualization.
**What you'd learn:** Empirical reasoning about inference-time compute allocation — a skill that generalizes to every future model release that exposes compute-steering controls.

---

## Opportunities

- **Effort-tier configuration and tuning tooling**: Sonnet 5 exposes five effort tiers with no tooling for profiling which tier is appropriate per task type. A SaaS or open-source profiler that runs a task sample across tiers and outputs optimal tier configs is an immediate gap with a clear first customer: any team migrating from Sonnet 4.6 to Sonnet 5.

- **Multi-provider AI availability monitoring**: The Fable 5 shutdown demonstrated that model availability is now a regulatory-risk variable, not just an uptime variable. A monitoring service that tracks model availability across providers *including* government-action-driven outages (not just HTTP 5xx), with alerting and failover trigger signals, is an unmet need for enterprise AI teams.

- **Government pre-release framework compliance tooling**: The EO Section 3 voluntary pre-brief process (30-day government access before launch, classifier-fix review, CISA submission) is now an operational reality for frontier labs. Labs need internal tooling to: manage the pre-brief timeline, prepare classifier submissions, and document their government engagement. A workflow/compliance tool for this specific process doesn't exist yet.

---

*Sources:*
- [Introducing Claude Sonnet 5 — Anthropic](https://www.anthropic.com/news/claude-sonnet-5)
- [What's new in Claude Sonnet 5 — Claude Platform Docs](https://platform.claude.com/docs/en/about-claude/models/whats-new-sonnet-5)
- [Adaptive thinking — Claude Platform Docs](https://platform.claude.com/docs/en/build-with-claude/adaptive-thinking)
- [Anthropic restores Claude Fable 5 as US lifts export controls — Tom's Hardware](https://www.tomshardware.com/tech-industry/artificial-intelligence/anthropic-restores-claude-fable-5-as-us-lifts-export-controls)
- [Anthropic Redeploys Claude Fable 5 on July 1 After US Export Controls Lift — MarkTechPost](https://www.marktechpost.com/2026/07/01/anthropic-redeploys-claude-fable-5-on-july-1-after-us-export-controls-lift-adds-new-cybersecurity-classifier/)
- [Fable 5 and Mythos 5 Are Back — MarketScale](https://www.marketscale.com/industries/software-and-technology/fable-5-and-mythos-5-are-back-what-the-19-day-shutdown-taught-every-enterprise-about-ai-as-infrastructure/)
- [Previewing GPT-5.6 Sol — OpenAI](https://openai.com/index/previewing-gpt-5-6-sol/)
- [OpenAI unveils GPT-5.6 Sol, Terra and Luna — VentureBeat](https://venturebeat.com/technology/openai-unveils-gpt-5-6-sol-terra-and-luna-models-but-only-accessible-to-limited-preview-partners-for-now-per-us-gov)
- [Cerebras Runs OpenAI GPT-5.6 Sol at 750 Tokens per Second — Value Add Pulse](https://valueaddvc.com/pulse/cerebras-openai-gpt-5-6-sol-750-tokens-2026)
- [Claude Code v2.1.198: Background Agents Now Commit, Push, and Open Draft PRs — AI Agents First](https://aiagentsfirst.com/claude-code-v2-1-198-background-agents)
- [Claude Code changelog — Claude Code Docs](https://code.claude.com/docs/en/changelog)
- [Claude Code Updates — Releasebot](https://releasebot.io/updates/anthropic/claude-code)
- [White House EO on Promoting Advanced AI Innovation and Security](https://www.whitehouse.gov/presidential-actions/2026/06/promoting-advanced-artificial-intelligence-innovation-and-security/)
- [AGI Maze Benchmark — arXiv:2607.00627](https://arxiv.org/abs/2607.00627)
- [HARC: Coupling Harmfulness and Refusal Directions — arXiv:2607.00572](https://arxiv.org/abs/2607.00572)
