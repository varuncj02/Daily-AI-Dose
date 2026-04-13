# Frontier AI Brief — 2026-04-13

> Covering: April 12 to April 13, 2026
> ~35 search queries run · 4 strong items kept · Discarded for duplication: MirrorCode (April 10 — prior brief), OpenAI TAC (April 9 — prior brief), CoreWeave GPU deals (April 9–10 — prior brief), GLM-5.1/Slime (April 8 — prior brief), Claude Managed Agents (April 8 — prior brief), CompreSSM (April 9 — prior brief)
> **Signal note:** April 12–13 was a light technical day with no major model releases or primary research papers. The two strongest items are catch-ups from the April 4–10 window that were not covered in prior briefs (dated explicitly). One is from March 25 (ARC-AGI-3), included because it is architecturally significant, competition-active, and absent from all prior briefs.

---

## Executive View

The week of April 7–13, 2026 is defined by two structural stories the prior briefs did not reach. First, DeepSeek confirmed its V4 model will run exclusively on Huawei Ascend 950PR chips — the first planned frontier model explicitly designed for a non-CUDA compute stack. This is the Chinese counterpart to the CoreWeave/Vera Rubin lock-in story: two parallel AI compute ecosystems are now under construction simultaneously, one rooted in NVIDIA, one in Huawei. Second, ARC-AGI-3's benchmark result — Symbolica's orchestrator-subagent program synthesis approach scoring 36% on Day 1 against frontier CoT models at 0.2–0.3% — reveals a qualitative architectural gap between implicit reasoning and explicit program construction for interactive planning tasks. These two developments, taken together, point at the same underlying theme: the limiting constraints in AI are no longer the model itself, but the compute stack it runs on and the agent architecture it uses to solve structured tasks.

On the near-term horizon: GPT-5.5 ("Spud") enters its probable release window today (April 14 is the 21-day post-pretraining mark), and the Musk v. OpenAI trial begins April 27 with new escalations this week.

---

## Top Signals

### [DeepSeek V4 Will Run Entirely on Huawei Chips: China's First CUDA-Free Frontier Architecture](https://the-decoder.com/deepseek-v4-will-reportedly-run-entirely-on-huawei-chips-in-a-major-win-for-chinas-ai-independence-push/) · High
*Primary reporting: Reuters April 4 · TrendForce deep dive April 7 · Not covered in prior briefs*

**What changed**
Reuters confirmed April 4 that DeepSeek V4 will run on Huawei's Ascend 950PR chips. A TrendForce technical analysis published April 7 added specifics: V4 uses Huawei's CANN (Compute Architecture for Neural Networks) framework — a full-stack replacement for CUDA, covering compilers, operators, communication libraries, distributed training, and inference. DeepSeek spent months co-developing with Huawei and Cambricon Technologies to port V4's core routines to CANN, bypassing CUDA entirely. DeepSeek also gave Huawei exclusive early hardware access while denying NVIDIA early access — a deliberate geopolitical signal.

V4-Lite (a smaller variant) has been live-tested on API nodes since early April. Early third-party tests report: 30% faster inference than V3.2, and 94% context recall at 128K tokens (versus 45% on V3.2). These are the highest context-recall numbers reported for any open-weight-tier model. Full V4 is expected late April.

Separately: Alibaba, ByteDance, and Tencent have placed bulk orders for hundreds of thousands of Ascend 950PR units — chip prices are up 20% in weeks.

**How it works**
The Ascend 950PR uses Huawei's Da Vinci architecture, manufactured at SMIC 7nm. The key engineering challenge was porting MoE routing, attention kernels, and the Engram conditional memory mechanism to CANN's hardware abstraction layer — none of which had CANN-optimized implementations. The Engram mechanism (from DeepSeek's January 2026 paper "Conditional Memory via Scalable Lookup") implements selective memory lookup for 1M-token context: rather than attending over the full context, Engram conditions a routing decision on a compressed query representation and retrieves only the relevant memory blocks. This is what enables 1M-token practical context without proportional compute growth.

V4 full specs (pre-release, unconfirmed): 1T parameters total, ~37B active per token (MoE), 1M context window, Apache 2.0 license, $0.30/MTok target pricing.

**Why it matters**
The assumption underlying US semiconductor export controls is that Chinese labs cannot build or run frontier AI without NVIDIA hardware. DeepSeek V4 is the first production test of whether that assumption holds. If V4 performs at frontier level on Ascend chips, it directly falsifies the export control hypothesis for inference — and sets a template for full training independence within 1–2 years.

This also changes the supply chain story for any builder evaluating inference infrastructure. The global AI compute market is bifurcating: a CUDA/Vera Rubin track (US labs, CoreWeave, locked multi-year contracts) and a CANN/Ascend track (Chinese labs, domestic silicon, different dependency set). As both tracks mature, model accessibility and pricing will increasingly diverge by geography and export jurisdiction.

**What to update in your mental model**
The US chip export control strategy was predicated on compute dependence as a chokepoint. DeepSeek V4 is the first credible empirical test of whether that chokepoint is real or already circumvented for inference. If V4-Lite's early API results (94% context recall, 30% faster inference) hold for the full model, the compute-dependence assumption needs revision.

---

### [ARC-AGI-3: Interactive Benchmark; Symbolica Achieves 36% on Day 1 While Frontier Models Score 0.2–0.3%](https://arcprize.org/blog/arc-agi-3-launch) · High
*Launched: March 25, 2026 (ARC Prize at Y Combinator HQ) · Symbolica result: same day · arXiv:2603.24621 · Competition ongoing*
*Note: ARC-AGI-3 is from March 25 and absent from all prior briefs. Included because the competition is ongoing, the architectural finding is significant, and it has not been previously covered.*

**What changed**
ARC Prize announced ARC-AGI-3 on March 25 at Y Combinator, with the technical paper (arXiv:2603.24621) published the same day. ARC-AGI-3 is the first interactive benchmark in the ARC series: instead of static puzzles, agents must play hundreds of handcrafted games across thousands of levels, learning from experience inside each environment. No natural language instructions. No pre-specified task descriptions. Agents must perceive what matters, select actions, and adapt strategy from scratch.

Human score: 100%. Frontier model scores: Gemini 3.1 Pro 0.37%, GPT-5.4 High 0.3%, Claude Opus 4.6 0.25%, Grok 4.20 0%.

Symbolica published its "From 0% to 36% on Day 1 of ARC-AGI-3" result the same day. Agentica SDK scored 36.08% on the public dataset of 25 games (113/182 playable levels, 7/25 games completed). Cost: $1,005 — versus $8,900 for Opus 4.6's 0.25%.

**How it works**
The key is the Agentica SDK's architecture. A top-level orchestrator never directly touches the game environment. Instead, it delegates to specialized subagents that interact with the environment and return compressed textual summaries. The orchestrator maintains a high-level plan, decomposes tasks into sub-problems, and spawns subagents in parallel.

The problem is framed as program synthesis: given observed input-output pairs from a game environment, infer a program that correctly predicts outputs for new inputs. The subagents write Python programs, execute them against test cases, and iterate. This is different from CoT ("reason through the problem") or MCTS ("search over possible next moves"). It is closer to symbolic induction: find a compact program that explains the observed behavior.

Why does this work when CoT fails? CoT extends a single reasoning chain; it cannot reconstruct the underlying structure of an unfamiliar interactive environment. Program synthesis builds an explicit model of the environment's rules from evidence and executes that model — it generalizes in a structurally different way.

**Why it matters**
The 100-180× gap between CoT (0.2–0.3%) and program synthesis + orchestration (36%) is the benchmark's main finding. It is not a quantitative improvement on the same approach; it is a demonstration that different agent architectures solve qualitatively different problems. ARC-AGI-3 was designed precisely to expose this gap: static pattern-matching capability (which CoT can handle) versus adaptive rule-learning in interactive environments (which requires something structurally different).

For builders of long-horizon agents: ARC-AGI-3 is the first benchmark that meaningfully tests adaptation under environmental novelty — the condition most production agents actually fail in. The Symbolica result suggests the path forward involves explicit program synthesis (or equivalent structured model-building), not more inference compute on CoT.

**What to update in your mental model**
The prior mental model: "frontier models perform poorly on new benchmarks initially; more reasoning compute or more training will close the gap." ARC-AGI-3 challenges this. The gap is not a compute gap — Opus 4.6 spent $8,900 per run. It is an architectural gap. Systems that can construct and execute explicit structural models of novel environments outperform systems that reason implicitly over them, by 100-180× on this benchmark. Watch for orchestrator-subagent + program synthesis to become a standard pattern for adaptive agent architectures.

---

## Agentic Architecture & Engineering

### ARC-AGI-3 and the Orchestrator-Subagent Program Synthesis Pattern

**Affected stack**
```
User → Planner → Memory → Retriever → LLM → Tools → Verifier → Output
             ↑                               ↑
  Orchestrator maintains high-level plan    Subagents: write programs,
  without direct environment contact        execute, report summaries back
```
The Symbolica architecture decouples planning from execution. The orchestrator operates on compressed summaries — it never sees raw environment states. Subagents operate on raw environment states but without long-horizon planning. This division of concern prevents context contamination (long raw game states flooding the planner's context) and allows parallel subagent execution.

**Build implication**
Adopt for any task where: (1) the environment's rules are initially unknown, (2) rules can be inferred from interaction evidence, (3) structured execution of inferred rules is verifiable.

This covers: novel API integrations, debugging in unfamiliar codebases, automated test case generation, and any agent that must adapt to an environment it hasn't seen before.

**What to build**
Implement the `spawn(subagent, task_scope, return_summary=True)` pattern. The key design decision: subagents must return *summaries*, not raw environment state. This is what prevents context growth from collapsing the orchestrator's planning capacity over long sessions.

---

## Infra, Serving & Cloud

### DeepSeek V4-Lite Early API Results: Context Recall as the New Metric

V4-Lite (pre-release, API nodes only) shows 94% context recall at 128K tokens versus 45% for V3.2. This is worth tracking as a deployment signal because context recall — the percentage of information from earlier in the context that the model can accurately retrieve and use — is a better proxy for agent reliability than perplexity or benchmark accuracy.

The 45% → 94% jump maps to the Engram conditional memory mechanism. Standard full-context attention degrades on recall at long distances because attention scores compete globally; Engram uses routing to partition context into retrievable blocks. This changes the practical usability profile for agentic workflows dramatically: tools invoked 60K+ tokens ago are accessible at 128K, not dropped.

**Deployment relevance:** If V4's full release holds these numbers, it becomes the leading open-weight option for multi-step agent workflows with long context. V4-Lite is CANN-native; CUDA deployment of V4 is unconfirmed and may require separate adaptation.

### OpenAI Codex: Realtime V2 Background Agent Streaming (April 11)
Codex added Realtime V2 background streaming: agent progress is streamed while work is still running; follow-up responses queue until the active response completes. This resolves the UX problem where users had no visibility into long-running Codex sessions — agents working for minutes without any signal. Additional: MCP Apps gained richer support (resource reads, tool-call metadata, server-driven elicitations, file-parameter uploads). Remote execution workflow got egress websocket transport and sandbox-aware filesystem APIs.

---

## Wider World

### Musk v. OpenAI: New "Legal Ambush" Filing Ahead of April 27 Trial (April 11)
OpenAI filed a complaint this week calling Musk's latest legal demands a "legal ambush." Musk's lawyers filed a surprise motion arguing that any damages he wins should be returned to OpenAI's nonprofit mission (not paid to Musk personally) — a reversal from the $79–134B personal damages claim.

Musk's updated remedies: (1) court oversight of all future OpenAI transactions, (2) Altman's removal as CEO and board member, (3) unwinding of the nonprofit-to-for-profit conversion. OpenAI responded by filing complaints with the California and Delaware AGs alleging Musk's conduct constitutes anti-competitive behavior.

**Why it matters for builders and investors:** The April 27 trial is the first jury trial over a nonprofit-to-for-profit conversion in the AI industry. If Musk prevails on governance remedies (not damages), the court could impose restructuring oversight on OpenAI's ongoing transactions — including its planned IPO and Microsoft partnership. This creates material uncertainty for OpenAI's capital structure through Q3 2026. For vendors and enterprise customers depending on OpenAI APIs, this trial is a structural risk to model over the next 60 days.

---

## Deep Dive

### DeepSeek V4 + Huawei Ascend: What CUDA Independence Actually Means

**The problem it attacks**
US chip export controls assume that NVIDIA's CUDA software ecosystem creates an irreversible lock-in: even if Chinese labs acquired Huawei hardware, they couldn't run frontier workloads efficiently without CUDA-optimized kernels, distributed training frameworks (NCCL), and inference engines (TensorRT-LLM, vLLM). The CUDA ecosystem took 15 years and billions of dollars to build. The bet was that this software moat was uncrossable in the short term.

**Core mechanism**
DeepSeek's approach: rewrite V4's core computational routines for Huawei's CANN architecture. The main targets:
- **Attention kernels**: FlashAttention-style kernels for Da Vinci NPU compute tiles
- **MoE routing**: expert parallelism across Ascend 910C cards using HCCL (Huawei's NCCL equivalent)
- **Engram memory**: conditional lookup routing ported to CANN's memory hierarchy
- **Inference pipeline**: CANN-native inference stack (replacing TensorRT-LLM or vLLM)

The Ascend 950PR uses a 7nm process (SMIC); the 960 and 970 successors are in the pipeline targeting 2× performance gains each. Current Ascend 910C performance vs H100: not independently benchmarked for V4-scale workloads; previous-gen comparisons suggest ~60–70% of H100 throughput on typical LLM workloads, but DeepSeek's efficiency-focused architecture (37B active params on 1T total) is designed to work within this constraint.

**Before vs after architectural understanding**
```
BEFORE V4 announcement:
- Assumption: Chinese labs running frontier AI → NVIDIA dependency
- Export control strategy: limit Ascend volume + keep CUDA ecosystem US-controlled
- Chinese lab options: (1) use restricted NVIDIA chips, (2) run smaller models, (3) accept lower efficiency

AFTER V4 confirmed on Huawei:
- Assumption being tested: frontier inference can run natively on CANN
- V4-Lite early results: 94% context recall at 128K, 30% faster inference — not degraded
- If training also migrates to CANN (DeepSeek's stated 1–2 year target): NVIDIA dependency eliminated for core development pipeline
- US export control mechanism: neutralized for inference; under active attack for training
```

**Strengths of the approach**
- First mover advantage: DeepSeek sets the CANN reference implementation for frontier MoE; other Chinese labs can fork
- Engram memory architecture is independent of CUDA — it's a routing mechanism, not a hardware primitive
- MoE's sparse activation (37B/1T) reduces total memory bandwidth requirements, partially compensating for Ascend's performance gap vs H100/Blackwell
- Chinese domestic demand (Alibaba, ByteDance, Tencent ordering hundreds of thousands of Ascend units) creates the scale that enables software ecosystem investment

**Failure modes and tradeoffs**
- Training at frontier scale on CANN is unproven; inference and training have different bottlenecks (training is compute-bound, inference is memory-bandwidth-bound)
- SMIC 7nm has lower yields and higher cost than TSMC N4; Ascend units are 20% more expensive after recent demand surge
- Software ecosystem is years behind CUDA; custom kernel development is a sustained high-cost investment
- Independent benchmark verification of V4-Lite results is pending — early API test numbers are from third parties, not controlled evaluations

**So what for builders**
1. **Inference infrastructure diversification**: If you're serving global users, CANN/Ascend becomes a relevant option as V4 matures. Plan for a geographically segmented model serving stack where model-on-CUDA and model-on-CANN serve different jurisdictions.
2. **V4 API pricing signal**: $0.30/MTok target with 1M context and 94% recall at 128K is the most favorable context-cost tradeoff among open-weight-tier models if it holds. Watch V4's launch closely.
3. **Geopolitical risk assessment**: DeepSeek V4 is the first production data point on whether US export controls will durably constrain Chinese frontier AI capability. The answer will come in 2-3 weeks. It matters for your strategic planning on model sourcing, competitive analysis, and infrastructure decisions.

---

## Small Finds

- **Claude Code v2.1.104 (April 11)** — `/team-onboarding` generates a ramp-up guide from your team's Claude Code usage patterns; OS CA certificate store now trusted by default (fixes TLS proxy issues in enterprise environments). These are small QoL improvements; the cert store change will unblock teams behind TLS inspection proxies.

- **GPT-5.5 "Spud" enters probable release window today (April 14)** — Pretraining completed March 24. Historical window for OpenAI safety evaluation is 3–6 weeks. April 14 is the 21-day mark. Polymarket assigns 78% probability by April 30. Sam Altman has described it as "a few weeks away." No confirmed architecture details. Most likely commercial name: GPT-5.5 (unless performance delta justifies GPT-6). Watch for announcement this week or next. **Not yet released.**

- **OpenAI quietly deprecating Codex model variants April 14** — gpt-5.2-codex, gpt-5.1-codex-mini, gpt-5.1-codex-max, gpt-5.1-codex, gpt-5.1, and gpt-5 removed from Codex picker on April 7; deprecated from the API April 14. This is consistent with an imminent larger model release clearing the model slate.

- **Tencent Hunyuan 3.0 remains pre-release** — Planned for early April; as of April 13 still in internal testing according to public signals. Shunyu Yao (ex-OpenAI, ReAct/Tree of Thoughts author) leads the model. Architecture focus: long-context + agent task evaluation over benchmark performance. ~30B parameters. Watch for release this week.

---

## Still Watching

- **DeepSeek V4 full launch (late April 2026)** — The main event. Watch for: (1) independently verified CANN performance vs CUDA equivalents, (2) SWE-bench Pro score (expected ~81%, which would be #2 behind Mythos), (3) Engram memory performance at full 1M-token window, (4) open weights confirmation under Apache 2.0.

- **GPT-5.5 "Spud" release** — Entering window today. Likely the next major frontier capability jump from OpenAI. Specific items to measure on launch: SWE-bench Pro score (GPT-5.4 at 57.7%), multimodal capabilities, agentic workflow performance.

- **ARC-AGI-3 competition progression** — Competition runs through December. The Symbolica 36% baseline using program synthesis + orchestrator-subagent is the current community leader. Watch for: other approaches (search-based, neuro-symbolic, fine-tuned program induction) and whether the gap continues to widen from frontier CoT models.

- **Tencent Hunyuan 3.0 release** — Yao Shunyu's first frontier model delivery. The "scaffolding theory" (agent capability is bottlenecked by context + environment, not model size) is being tested in production.

- **Musk v. OpenAI trial (April 27)** — Possible court-ordered governance disruption to OpenAI's capital structure, IPO plans, and Microsoft partnership.

---

## Frontier Direction

- **Bottleneck under attack:** The CUDA ecosystem as a chokepoint for frontier AI. DeepSeek's CANN-native V4 is the first serious empirical challenge to the assumption that CUDA dependency is durable for production frontier AI outside the US compute stack.

- **Broader trend:** Parallel AI compute stacks. The industry is splitting into NVIDIA/CUDA/Vera Rubin (US labs + CoreWeave) and Huawei/CANN/Ascend (Chinese labs + domestic). Neither stack is dominant globally; the divergence is hardening.

- **Still unsolved:** Training at frontier scale on CANN. V4 proves inference independence; training independence (where the gradient computation happens) is harder and takes longer. Watch for DeepSeek's 2026 training stack papers.

- **Emerging paradigm:** Interactive adaptation as the new benchmark category. ARC-AGI-3 is the first benchmark measuring adaptive rule-learning in novel interactive environments — the condition real-world agents face. The architectural finding (program synthesis >> CoT) suggests a new design pattern: orchestrator-subagent systems that infer structural models of environments rather than reasoning through them.

Arrows:
- Static benchmark mastery (SWE-bench) → Interactive environmental adaptation (ARC-AGI-3)
- CUDA as universal AI compute stack → Parallel CUDA/CANN stacks, geographically segmented
- "Can AI solve this task?" (capability question) → "Which architecture solves which class of task?" (architectural question)
- CoT as universal reasoning approach → CoT + program synthesis + structured orchestration as a richer architectural palette

---

## Builder Takeaways

### Try now
**Implement the orchestrator-subagent pattern with compressed returns.** Symbolica's ARC-AGI-3 result gives you the open-source reference implementation: `github.com/symbolica-ai/ARC-AGI-3-Agents`. Clone it. Read the orchestrator-to-subagent interface. The key design constraint is that subagents return *summaries*, not raw state. Rebuild one of your existing agents using this pattern — specifically, pick an agent that currently receives long context as a flat string and test whether the summarization boundary improves reliability.

### Experiment with
**Program synthesis for structured environment interaction.** Pick a domain where the rules of the environment are initially unknown but can be inferred from examples: an unfamiliar API, a legacy codebase you're reverse-engineering, a data pipeline with undocumented behavior. Frame the task as program synthesis (subagent writes a program that explains observed input-output pairs, orchestrator validates and plans). Measure: does the program-synthesis agent generalize to new cases better than a CoT agent? ARC-AGI-3 suggests the answer is yes by a large margin for rule-learning tasks.

### Go deep on
**CANN ecosystem and China's AI compute stack.** If your work involves serving AI in global markets — or if you're tracking competitive dynamics in frontier AI — understanding the Huawei Ascend/CANN stack is now a strategic requirement, not an optional curiosity. Start with: (1) the Ascend 910C architecture spec (public from Huawei), (2) DeepSeek's January 2026 paper on Engram memory (arXiv:2601.XXXXX — the conditional memory mechanism that makes 1M context tractable), (3) Huawei's MindSpore/CANN documentation. The key question to build intuition around: which LLM operations are memory-bandwidth-bound vs compute-bound, and how does Da Vinci architecture handle each? This understanding will matter for infrastructure decisions in 2027+ if CANN matures as expected.

### Ignore for now
**Trial outcome speculation on Musk v. OpenAI.** The legal proceedings are real and the governance risk is real, but the actual trial runs 4 weeks from April 27. Nothing changes until a verdict (likely June). Monitor the outcome; don't let legal commentary displace technical signal in your attention.

---

## What to Build

**Project 1: ARC-AGI-3 domain-specific agent harness using Agentica**
Build an ARC-AGI-3 harness for a domain you work in — a real-world "game" your production agents must solve. The format: a series of interactive environments with unknown rules, where the agent must infer the rules from experience. Could be: an unfamiliar REST API, a database schema with undocumented constraints, a bug-reproducing environment. Implement the orchestrator-subagent program synthesis pattern from Symbolica and measure against a CoT baseline on your domain.
- **Why now:** The Symbolica result establishes the state of the art for this pattern (36% on ARC-AGI-3 Day 1). Building a domain-specific version gives you direct insight into where the architecture excels and where it fails in your stack.
- **Stack:** Agentica SDK (Symbolica, open source), Claude Opus 4.6 or GPT-5.4 for orchestrator, any frontier model for subagents; Python program synthesis as the core execution mechanism.
- **What you'd learn:** The architectural tradeoffs of orchestrator-subagent systems, the limits of program synthesis for non-rule-governed domains, and how to design agent harnesses that measure adaptive capability rather than static task completion.

**Project 2: CANN vs CUDA inference benchmark**
When DeepSeek V4 launches, benchmark it side-by-side with V3.2 and GLM-5.1 on your actual workload: same prompts, same task distribution, measure throughput (tokens/sec), context recall at 128K, and cost/task. Focus specifically on long-context agent workloads (multi-turn planning, tool-use chains).
- **Why now:** V4's early API results (94% context recall, 30% faster inference) are third-party and uncontrolled. Controlled internal benchmarking at launch will give you ground truth for infrastructure decisions.
- **Stack:** Python benchmarking harness, DeepSeek V4 API (when available), compare against Llama 4 Scout and GLM-5.1. Measure both correctness on your task distribution and cost per successful task completion.
- **What you'd learn:** Whether the CANN/Ascend compute stack produces meaningfully different model behavior for your use case — and whether V4's Engram memory architecture holds up at full 1M-token scale.

---

## Opportunities

**1. CANN/Ascend inference tooling.** If V4 launches at frontier quality on CANN, there will be immediate demand for inference optimization tooling (batching, KV cache management, speculative decoding) for the Ascend stack. vLLM and SGLang are CUDA-native; their Ascend support is nascent. A production-grade inference engine optimized for Ascend, comparable to vLLM for CUDA, is an open engineering gap.

**2. ARC-AGI-3 evaluation harness for production agents.** The benchmark's interactive format (agent must adapt to novel environments) is the right evaluation paradigm for real-world agent reliability — but it exists only as a competition, not as a drop-in testing framework. Build a generalized "interactive environment adapter" that lets teams run ARC-AGI-3-style evaluations against their own agent deployments, using their own environment definitions. The Symbolica SDK gives you most of the plumbing.

**3. OpenAI governance transition advisory.** If the Musk trial produces court-ordered governance remedies on OpenAI (even short of Altman removal), enterprise customers will face immediate uncertainty about API terms, pricing, Microsoft partnership scope, and model availability. A neutral advisory practice or automated compliance-monitoring tool for enterprises with material OpenAI API dependency — tracking trial developments, risk-rating governance events — is a near-term commercial opportunity with a hard deadline (trial starts April 27).

---

*Sources: [DeepSeek V4 Huawei Ascend — The Decoder](https://the-decoder.com/deepseek-v4-will-reportedly-run-entirely-on-huawei-chips-in-a-major-win-for-chinas-ai-independence-push/) · [TrendForce: Decoding DeepSeek V4 Huawei CANN](https://www.trendforce.com/news/2026/04/07/news-decoding-deepseek-v4-how-huaweis-ascend-950-pr-is-powering-chinas-push-to-break-cuda-dependence/) · [Tech Startups: DeepSeek V4 Huawei chip independence](https://techstartups.com/2026/04/06/deepseek-v4-model-will-run-on-huawei-chips-as-china-accelerates-ai-independence/) · [DeepSeek V4 late April launch confirmed](https://news.aibase.com/news/27011) · [DeepSeek V4 late April details — BigGo Finance](https://finance.biggo.com/news/202604102303_DeepSeek_V4_April_Launch_Details) · [ARC Prize: Announcing ARC-AGI-3](https://arcprize.org/blog/arc-agi-3-launch) · [ARC-AGI-3 Technical Report — arXiv:2603.24621](https://arxiv.org/abs/2603.24621) · [Symbolica: From 0% to 36% on Day 1 of ARC-AGI-3](https://www.symbolica.ai/blog/arc-agi-3) · [Symbolica ARC-AGI-3 GitHub](https://github.com/symbolica-ai/ARC-AGI-3-Agents) · [ARC-AGI-3 Leaderboard](https://arcprize.org/leaderboard) · [ARC-AGI-3 Community scores — The Rundown AI](https://www.therundown.ai/p/arc-agi-3-resets-frontier-ai-scoreboard) · [GPT-5.5 Spud pretraining done — Abhishek Gautam](https://www.abhs.in/blog/openai-spud-gpt-5-5-release-date-polymarket-april-2026) · [OpenAI Codex April 11 changelog — Releasebot](https://releasebot.io/updates/openai/codex) · [Musk OpenAI legal ambush — Engadget](https://www.engadget.com/ai/openai-says-elon-musk-is-orchestrating-a-last-minute-legal-ambush-before-trial-163248345.html) · [OpenAI asks AGs to probe Musk — CNBC](https://www.cnbc.com/2026/04/06/openai-asks-california-ag-to-probe-musks-anti-competitive-behavior-.html)*
