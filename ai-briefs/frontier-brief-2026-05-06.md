# Frontier AI Brief — 2026-05-06

> Covering: May 5–6, 2026
> ~20 candidates reviewed · 5 kept · 15 discarded (outside window / prior coverage / weak signal / no new mechanism)

---

## Executive View

Two parallel channels defined the last 48 hours. The first is enterprise agent productization: Anthropic's finance agent release on May 5 is the most detailed public description yet of a deployable vertical agent architecture — skills + connectors + subagents packaged as an installable unit — and reveals the template other verticals will follow. The second is inference acceleration: Google's MTP drafters for Gemma 4, shipping May 6 under Apache 2.0, deliver up to 3× speedup at no quality cost via a speculative decoding mechanism that shares KV cache with the target model, dropping directly into vLLM, SGLang, and Ollama. Both channels are simultaneously validated by an alarming counterpoint: a May 5 scan of 1 million exposed AI services found that rapid LLM deployment is producing massive unauthenticated attack surface — 31% of queried Ollama servers answered without authentication. The industry is shipping faster than it is securing.

---

## Top Signals

### [Anthropic Finance Agents: 10 Reference Architectures for Vertical Agent Deployment](https://www.anthropic.com/news/finance-agents) · **High**
*Published: May 5, 2026*

**What changed**

Anthropic released 10 pre-built agent templates for financial services: Pitch builder, Meeting preparer, Earnings reviewer, Model builder, Market researcher, Valuation reviewer, General ledger reconciler, Month-end closer, Statement auditor, and KYC screener. Each ships as a plugin for Claude Cowork and Claude Code, and as a cookbook for Claude Managed Agents. Simultaneously: Claude MS365 add-ins now carry context across Excel, PowerPoint, Word, and Outlook (coming soon) in a single agent session. Moody's launched an MCP app bringing credit ratings and risk data for 600M+ companies directly into Claude.

**How it works**

The architecture of each template packages three layers:
1. **Skills:** domain instructions and task knowledge (e.g., pitchbook construction methodology, KYC screening logic)
2. **Connectors:** governed access to the data the task runs on (Bloomberg, Moody's, internal ledgers via MCP)
3. **Subagents:** additional Claude model instances called for specific sub-tasks (comparables selection, methodology double-check, reconciliation verification)

The deployment path: plugin in Claude Cowork / Claude Code for human-in-the-loop workflows → Claude Managed Agents cookbook for programmatic/automated use. A team can take a template from installation to running on real financial data in days rather than months, because the task decomposition, domain knowledge, and data access patterns are pre-assembled.

**Why it matters**

This is the clearest public description of a repeatable enterprise vertical agent product architecture to date. The skills + connectors + subagents template separates concerns that most teams conflate: what the agent knows (skills), what data it can touch (connectors), and how it decomposes work (subagents). The MS365 cross-application context is genuinely novel — one agent session that reads an Excel model and writes a PowerPoint deck without re-prompting removes the biggest friction point in document-heavy professional workflows.

The Moody's MCP integration is architecturally significant: a proprietary data vendor embedding their full platform into an AI as a native app, not a third-party integration. This is the MCP adoption pattern at enterprise scale — data vendors becoming first-class AI connectors rather than API providers.

**What to update in your mental model**

The vertical agent product template is now public and repeatable: skills (domain knowledge) + connectors (data governance) + subagents (task decomposition) = deployable enterprise agent. Anthropic has demonstrated it for finance; legal, clinical, and engineering verticals will follow the same template. If you're building in any professional vertical, this is your reference architecture.

---

### [Gemma 4 MTP Drafters: 3× Inference Speedup via Shared-KV Speculative Decoding](https://blog.google/innovation-and-ai/technology/developers-tools/multi-token-prediction-gemma-4/) · **High**
*Published: May 6, 2026*

**What changed**

Google released Multi-Token Prediction (MTP) draft models for the full Gemma 4 family under Apache 2.0, delivering up to 3× inference speedup without measurable quality degradation. Available on Hugging Face and Kaggle, with native support in transformers, MLX, vLLM, SGLang, and Ollama — no new serving infrastructure required.

**How it works**

MTP drafters implement speculative decoding with a key architectural twist: the draft model shares the target model's KV cache and activations directly. Standard speculative decoding runs a small draft model independently and then verifies with the large model — two separate forward passes with separate KV caches. Gemma 4's MTP drafters instead:

1. Access the target model's already-computed KV cache at each decode step
2. Use the target model's layer activations as input to the draft head — no redundant context recomputation
3. Predict multiple tokens ahead; the target model verifies in a single forward pass
4. On acceptance: output the full drafted sequence plus one additional token in the time it normally takes to generate one

For edge models (E2B, E4B), an efficient clustering technique in the embedder accelerates generation further.

The 3× speedup is realized on the Dense (non-MoE) Gemma 4 models at batch size 1. The 26B MoE model at batch size 1 on Apple Silicon shows limited gains due to expert routing overhead, but batch sizes of 4–8 unlock ~2.2× speedup locally.

**Why it matters**

This is the most broadly accessible speculative decoding implementation yet released — Apache 2.0, drop-in for every major inference engine, covering the Gemma 4 model family that already has 60M+ downloads. The shared-KV-cache trick is architecturally cleaner than standard speculative decoding: it eliminates the draft model's context recomputation overhead, which is the primary cost that limits standard speculative decoding gains at long context lengths. At 3× speedup with zero quality loss, this changes the economics of interactive Gemma 4 deployments: latency and cost per generated token both drop by a factor of 3 for throughput-unconstrained workflows.

Combined with Gemma 4's existing edge inference capability (E2B on Raspberry Pi), MTP drafters make locally-hosted reasoning-capable models practical for latency-sensitive applications on commodity hardware.

**What to update in your mental model**

Speculative decoding via MTP with shared KV cache is now the most efficient available form of speculative inference for open models. The bottleneck is no longer the mechanism — it's the draft model quality and acceptance rate. If you're using Gemma 4 in any serving configuration, enabling MTP drafters should be the first infrastructure change you make. The technique generalizes: any model family that releases MTP drafters with shared-KV conditioning achieves similar gains. Watch for this pattern to propagate to other open model families.

---

### [MolmoAct2: Open Action Reasoning Model for Real-World Robot Deployment](https://allenai.org/blog/molmoact2) · **High**
*Published: May 5, 2026 — arXiv:2605.02881*

**What changed**

AI2 released MolmoAct2, a fully open action reasoning model for physical robot deployment: open weights, training code, action tokenizer, and three new datasets (MolmoAct2-BimanualYAM: 720 hours of bimanual trajectories; DROID; SO100/101). The model runs 37× faster than the original MolmoAct. On a Franka arm: 100% success on apple-to-plate task, 86.7% on pipette-to-tray, 62% on the longest-horizon multi-object sequence tasks — beating Physical Intelligence's π0.5 on every evaluated task.

**How it works**

MolmoAct2 solves a core architectural tension in robotics AI: discrete planning and continuous motor control require fundamentally different representations. Discrete token models are good at reasoning; continuous action experts are good at precise motor control. MolmoAct2 grafts these together via:

- **Molmo2-ER backbone:** A VLM specialized for spatial and embodied reasoning, trained on 3.3M examples with a specialize-then-rehearse recipe — it learns spatial understanding first, then rehearses general vision-language capabilities to avoid forgetting
- **Flow-matching continuous action expert:** Attached to the VLM via per-layer KV-cache conditioning. The action expert reads the VLM's internal representations at each layer (not just the final output), giving it rich spatial and semantic context for continuous action prediction
- **MolmoAct2-Think (adaptive reasoning):** Re-predicts depth tokens only for scene regions that change between timesteps — rather than re-reasoning about the full scene at every step. This achieves geometric grounding at a fraction of prior latency
- **OpenFAST Tokenizer:** An open-weight action tokenizer trained on millions of trajectories across five different robot embodiments — enabling transfer across robot types

**Why it matters**

This is the clearest open alternative to proprietary robotics foundation models (π0.5, etc.) released to date. The architecture — VLM backbone with KV-cache-conditioned continuous action expert — is the most principled approach yet to combining language-level reasoning with motor control, and the full release (weights + training code + data + tokenizer) makes it reproducible. The 37× speed improvement over MolmoAct makes interactive control cycles practical for real-time deployment.

The spatial VLM + flow-matching pattern is increasingly the consensus for embodied AI. MolmoAct2 is the first fully open implementation of it with competitive real-world results. For agent architecture researchers: the per-layer KV-cache conditioning mechanism (action expert reads intermediate LLM representations, not just output tokens) is an architectural idea that may generalize beyond robotics to any task requiring tight coupling between reasoning and action generation.

**What to update in your mental model**

Embodied AI is no longer behind a proprietary wall. MolmoAct2 is competitive with Physical Intelligence's π0.5 on real hardware with full open release. The flow-matching + VLM backbone pattern is the current frontier in action generation. For builders interested in physical AI or tool-using agents that operate in physical environments, the architecture is now studyable and buildable.

---

### [Automated Agent Red Teaming: Weeks → Hours](https://arxiv.org/html/2605.04019v1) · **Medium**
*Published: May 5, 2026 — arXiv:2605.04019 (Dreadnode)*

**What changed**

Dreadnode (Raja Sekhar Rao Dheekonda, Will Pearce, Nick Landers) published a paper demonstrating an agentic red teaming system that compresses manual AI red teaming workflows from weeks to hours. The agent accepts natural-language red teaming objectives via a TUI, autonomously selects attacks (from 45+ adversarial attack types), composes transforms (450+ available), selects scorers (130+ available), executes the full workflow, and produces reports — without operators needing to know which specific attacks apply to which targets.

**How it works**

The core shift is from library-oriented to goal-oriented red teaming:

- **Before:** Operator reads attack library, manually selects applicable attacks, hand-assembles workflows, runs each, aggregates results
- **After:** Operator states goal in natural language ("find prompt injection vulnerabilities in this multi-agent pipeline targeting medical records access"); the agent reasons over the attack/transform/scorer library, assembles a workflow, executes it, and reports results

The system targets multi-agent systems, multilingual targets, and multimodal targets — which are poorly covered by current manual red teaming practice because the attack surface grows combinatorially.

**Why it matters**

The security evaluation gap for agent systems is severe and growing. As production agents connect to more tools, operate with more autonomy, and handle more sensitive data, manual red teaming cannot scale. An automated red teaming agent that can probe multi-agent pipelines (where vulnerabilities often emerge from agent-to-agent interactions, not single-agent behavior) addresses the highest-priority security gap in deployed agent infrastructure.

**Build implication:** Watch. No production-ready open-source implementation is available yet, but the Dreadnode SDK is open. The pattern — agent that reasons over an attack library to compose custom workflows — will likely become the standard for security evaluation of agent systems within 6–12 months.

---

## Agentic Architecture & Engineering

### The Skills + Connectors + Subagents Pattern as Deployable Unit

The Anthropic finance agents release makes explicit what had previously been implicit in agent deployment discussions: the three-layer separation that makes a vertical agent deployable.

**Affected stack:**
```
User → [Skills Layer] → [Connectors Layer] → [LLM + Subagents] → [Verifier] → Output
```

Shift at the **Skills** and **Connectors** layers — both are now pre-assembled rather than hand-built per deployment.

**The three-layer separation matters because each layer has different ownership:**

| Layer | What it packages | Who owns it | Update frequency |
|---|---|---|---|
| Skills | Task knowledge, instructions, decomposition logic | Domain experts / Anthropic | Slow (weekly/monthly) |
| Connectors | Data access governance, MCP integrations | IT / security / vendor | Medium (per data change) |
| Subagents | Specialized model calls for sub-tasks | ML engineers | Fast (per model update) |

Conflating these three in a single system prompt produces brittle agents that break whenever any one layer changes. Separating them makes each independently updateable and testable.

**The Moody's MCP pattern:** A proprietary data vendor shipping their full platform as an MCP app (not an API wrapper, not a search tool — a native Claude app) is the enterprise data integration model the MCP ecosystem was designed to enable. This is what "data connectors" look like at production scale: vendors self-hosting MCP servers that surface their proprietary data into agent sessions without data leaving the vendor's environment.

**Build implication: Adopt.** When designing any vertical agent, separate skills, connectors, and subagents into distinct layers from the start. The Anthropic finance templates are the clearest public reference architecture for this; review the official template structures at [anthropic.com/news/finance-agents](https://www.anthropic.com/news/finance-agents) before building your own vertical agent.

### MS365 Cross-Application Context: The Implication for Document-Heavy Agents

The Claude MS365 integration ships something architecturally non-trivial: a single agent session that maintains context across Excel, PowerPoint, Word, and Outlook simultaneously. The implication for document-heavy professional workflows:

- An agent that reads a financial model (Excel) can write a pitchbook (PowerPoint) that cites the correct figures, without the user re-explaining the model structure
- Context doesn't reset at application boundaries — the agent's understanding of the task persists as the work medium shifts

This is a session-persistence pattern applied across application contexts rather than across conversation turns. The same multi-turn reliability concerns from the "LLMs Get Lost" finding (last brief) apply: the longer and more complex the cross-application session, the higher the reliability risk. But the productivity gain — eliminating the "explain the context again" step at each application boundary — is immediately practical.

---

## Infra, Serving & Cloud

### Gemma 4 MTP Drafters: Deployment Details

The shared-KV-cache speculative decoding mechanism is available today across:
- `transformers` (HuggingFace): drop-in via `AssistantModel` parameter
- `vLLM`: `--speculative-model` flag
- `SGLang`: `--draft-model` flag
- `Ollama`: speculative decoding model pair configuration
- `MLX`: native support for Apple Silicon batch-1 workloads

**Performance numbers to set expectations:**
- Dense Gemma 4 models at batch 1: up to 3×
- 26B MoE at batch 1 on Apple Silicon: limited gains due to expert routing overhead
- 26B MoE at batch 4–8: ~2.2×

**Migration note:** The MTP drafters use the same Apache 2.0 license as Gemma 4 and require no separate serving infrastructure. The only deployment change is specifying the draft model alongside the target model. There is no accuracy loss to evaluate or accept — quality is mathematically guaranteed to match the target model output distribution. This is a zero-risk, high-upside deployment change for any existing Gemma 4 setup.

### Agent Infrastructure Security: What the 1M Exposed Services Scan Reveals

A scan of 1 million exposed AI services (published May 5) found:
- 31% of 5,200+ queried Ollama servers answered without authentication, with a model loaded and ready to query
- Exposed instances of n8n and Flowise agent management platforms accessible without login
- Claude-powered deployments with API keys disclosed in plaintext
- Generic chatbots hosting multimodal LLMs, freely accessible

**The architectural implication:** Self-hosted LLM inference is being deployed at scale by teams with no enterprise security baseline. The ease of `ollama pull` + `ollama serve` means models land in production before anyone has asked about authentication, rate limiting, network exposure, or audit logging. This is the exact attack surface that the Agent Gateway pattern (covered in prior brief) addresses — but that pattern requires deliberate engineering investment that most self-hosting teams aren't making.

**For teams self-hosting inference:** Ollama's default configuration is unauthenticated. Add authentication via a reverse proxy (Nginx + basic auth, Traefik + middleware, or Caddy + basicauth directive) before any Ollama instance is accessible outside localhost. This is a 15-minute configuration change with no performance cost.

---

## Wider World

### AI Reaches 53% Adoption in Three Years — Faster Than PC or Internet

The Stanford AI Index 2026 (released April 13, now widely covered) documents: generative AI reached 53% population adoption globally within three years of mainstream availability — faster than the personal computer or the internet. Estimated value to US consumers: $172B/year, with median value per user tripling between 2025 and 2026.

Three findings directly relevant to builders:

1. **US and China are trading places at the top of performance rankings.** As of March 2026, Anthropic's best model leads by only 2.7% on leading benchmarks — the gap that was 20+ points two years ago has nearly closed.
2. **Model transparency scores dropped from 58 to 40** on the Foundation Model Transparency Index — labs are disclosing less about training data, compute, and risk evaluations even as capabilities grow.
3. **AI scholars moving to the US dropped 89% since 2017**, with 80% of that decline in the last year. Switzerland now ranks first for AI researchers per capita.

---

## Deep Dive

### Gemma 4 MTP Drafters: How Shared-KV Speculative Decoding Works and Why It's Different

**The problem it attacks**

Standard LLM inference is memory-bandwidth bound, not compute bound, at batch size 1. The GPU is fast enough to compute; it's limited by how quickly it can load model weights and KV cache from HBM (high-bandwidth memory) on each forward pass. Each token generation requires one full forward pass — loading billions of parameters for a single token output is deeply inefficient.

Speculative decoding was designed to address this: draft many tokens cheaply with a small model, verify them in batch with the large model, accept as many as match the large model's distribution. The speedup comes from the batch verification step — verifying N drafted tokens costs roughly the same as generating 1 token from the large model, so if even a few tokens are accepted, you win.

**Standard speculative decoding's bottleneck**

The draft model is run independently. It has its own forward pass, its own KV cache, its own context computation. For long contexts, the draft model's KV cache is large, its context computation is non-trivial, and the benefit is partially offset by the cost of running the draft model itself.

**What MTP shared-KV conditioning changes**

Gemma 4's MTP draft heads don't run as separate models. They:

1. **Read the target model's activations directly** at each layer — meaning the "draft model" is effectively a lightweight head attached to the target model's computational graph, not a separate model
2. **Share the target model's KV cache** — no duplicate context storage, no separate attention computation for the drafted tokens
3. **Predict multiple tokens ahead from the same forward pass** — the MTP head outputs a distribution over next-token, next+1-token, next+2-token simultaneously

The result: the "cost" of draft generation is nearly zero relative to the target model's forward pass — just the MTP head computation on top of activations that are already being computed. Verification is a free check against the target model's output distribution, not an additional forward pass.

**Before vs. after architecture**

```
Standard autoregressive (before):
Step 1: Load model weights + KV cache → compute forward pass → generate 1 token
Step 2: Load model weights + updated KV cache → compute forward pass → generate 1 token
...
Cost per token: full model load + forward pass

Standard speculative decoding:
Draft phase: small model → generate N tokens, O(N) cost
Verify phase: large model → verify N tokens in batch, O(1) cost
Accept: ~3-4 tokens per verification cycle on average
Effective cost per token: (O(N) draft + O(1) verify) / 3-4 tokens

MTP shared-KV speculative decoding (Gemma 4):
Forward pass of large model: computes activations as always
MTP head: uses activations to draft N tokens, negligible additional cost
Verification: already happened — draft is conditioned on exact model activations
Accept: verified by construction; output distribution matches target
Effective cost per token: O(1) / N — full forward pass amortized over N tokens
```

**Why 3× at batch 1 but only 2.2× at higher batch sizes**

At batch size 1, memory bandwidth is the binding constraint. Each forward pass loads model weights once; with MTP, that load is amortized over 3 tokens instead of 1 — direct 3× reduction in effective weight load per output token.

At higher batch sizes, compute starts to matter (multiple requests process in parallel). The MTP drafters add compute overhead (the MTP heads) which starts to compete with the weight-loading savings. The optimal batch size for maximum throughput without MTP is different from the optimal batch size with MTP — at high batch sizes, standard inference may be equally efficient.

**The 26B MoE case: why it's different**

Mixture-of-Experts models route each token through different expert subsets. The MTP head, which predicts multiple tokens ahead simultaneously, cannot easily predict which experts will be activated for future tokens — expert routing is itself data-dependent and input-specific. This is the fundamental challenge: MTP assumes the model's internal computation path is relatively stable across the drafted tokens, but MoE routing makes the computation path input-dependent in a way that complicates multi-step prediction.

At higher batch sizes, the per-request expert routing averages out across the batch, reducing the prediction error in the MTP head — explaining why 2.2× is achievable at batch 4–8 where batch 1 is limited.

**Failure modes and tradeoffs**

- **Domain distribution mismatch:** MTP drafters are trained on Gemma 4's pre-training distribution. For heavily out-of-distribution content (specialized medical jargon, uncommon languages, highly structured formats), acceptance rate drops — reducing the speedup proportionally
- **MoE routing at batch 1:** As noted, limited gains for the 26B MoE at batch 1 on Apple Silicon due to expert routing overhead
- **Temperature sensitivity:** High-temperature sampling (creative generation) increases token variability, reducing acceptance rate of drafted tokens. Low-temperature (factual, structured output) maximizes acceptance rate and speedup

**So what for builders**

For any Gemma 4 deployment at batch size 1 (typical for interactive chat, single-user agents, real-time applications): enable MTP drafters immediately. The quality is guaranteed identical; the speedup is 3×; the implementation is a single flag change. This is not an experiment — it's a production optimization with zero downside. For batch-constrained high-throughput serving (high batch sizes, many concurrent requests), benchmark your specific workload — the gains are present but smaller. For MoE deployments at batch 1, the gains are modest; prioritize higher batch utilization first.

---

## Small Finds

- **Cloudflare Workers AI "Infire" + Large Models:** Cloudflare published a blog (late April / early May) detailing their custom inference engine Infire — shared across their global PoP network for open-weight model hosting. Key stats: sub-20-second cold start for Kimi K2.5 (1T+ parameter sparse MoE), 20% higher throughput than standard vLLM on unconstrained systems via custom CUDA kernels, plus "Unweight" weight compression (15–22% model size reduction without accuracy loss). The pattern: Cloudflare commoditizing large open-model serving at the edge. Relevant for teams wanting globally-distributed inference without GPU cluster management. ([blog.cloudflare.com](https://blog.cloudflare.com/workers-ai-large-models/))

- **Dreadnode TUI Red Teaming SDK:** The Dreadnode open-source SDK underlying the red teaming agent paper (2605.04019) is available at [dreadnode.io/ai-red-team](https://dreadnode.io/ai-red-team). Contains 45+ attack primitives, 450+ transforms, 130+ scorers usable independently or orchestrated by the agent. Worth examining for teams building security evaluation into their CI pipelines.

- **InfiAgent file-centric state (arXiv:2601.03204, January 2026):** Getting renewed community attention this week after the AiScientist File-as-Bus result. Core claim: bounding the agent's reasoning context by externalizing all state to a file workspace — reading curated workspace snapshot + recent actions at each step rather than full history — enables 80-paper literature reviews without context degradation. Competitive with larger proprietary systems on DeepResearch tasks using a 20B open model. Now has multiple community implementations. The File-as-Bus + InfiAgent state pattern is converging as the production standard for long-horizon agent execution.

- **Mistral Medium 3.5 + Vibe Remote Agents (April 29):** Just outside the window but relevant context: Mistral's 128B dense model (77.6% SWE-bench Verified) with async cloud coding sessions and a `/teleport` command to move a live local session into cloud execution. No equivalent exists in Claude Code or Codex. The async session migration pattern (local → cloud without session restart) is an unexplored UX surface in coding agents.

- **MolmoAct2-BimanualYAM Dataset:** AI2's 720-hour open bimanual robotics demonstration dataset — the largest open bimanual manipulation dataset released to date. Training data scarcity was the primary bottleneck for open robotics foundation models; this release removes it for tabletop manipulation tasks. ([github.com/allenai/molmoact2](https://github.com/allenai/molmoact2))

---

## Frontier Direction

- **Bottleneck under attack:** Inference cost and latency for open models at batch 1. Gemma 4 MTP drafters achieve 3× via shared-KV speculative decoding; TurboQuant achieves 6× KV compression; DeepSeek V4 CSA/HCA achieves 90% KV reduction via architecture. Three orthogonal attack vectors converging on the same target: make long-context inference on open models cheap enough for production agent loops.
- **Broader trend:** Vertical agent productization is accelerating. Anthropic's finance templates reveal the pattern; the skills+connectors+subagents architecture is now the template for professional vertical deployments. Expect legal, clinical, and engineering verticals to follow within Q2–Q3 2026.
- **Still unsolved:** Multi-turn agent reliability (see April 27 brief) — no production primitive for session-state management, commitment tracking, or context reset exists in any SDK. Anthropic's finance templates don't address this; they are single-task workflows, not long-horizon sessions.
- **Emerging paradigm:** Physical AI reaching open-model parity. MolmoAct2 beating π0.5 with full open release changes the embodied AI calculus — the gap between open and proprietary is closing in robotics as it did in language two years ago.

Arrows:
- Separate inference engines per use case → unified serving with speculative decoding (Gemma 4 MTP drops into all engines)
- Proprietary robotics foundation models → open action reasoning models (MolmoAct2 → π0.5 parity)
- Hand-assembled vertical agents → packaged skills+connectors+subagents templates (Anthropic finance → repeatable pattern)
- Manual AI red teaming → agentic automated red teaming with attack orchestration

---

## Builder Takeaways

### Try now
**Enable Gemma 4 MTP drafters in your inference setup.** If you are serving any Gemma 4 dense model at batch size 1 (interactive agents, real-time chat, single-user workloads), add the MTP draft model today. In vLLM: `--speculative-model google/gemma-4-mtp-draft`. In Ollama: speculative decoding model pair. In transformers: `AssistantModel` parameter. Zero downside — output distribution is identical to target model. 3× token generation speedup. This is a 30-minute configuration change with production-ready results.

### Experiment with
**Build one agent using the skills+connectors+subagents template.** Take any professional workflow you currently handle with a single-prompt agent. Separate it into: (1) a skills YAML with domain instructions, (2) a connectors config that governs data access, (3) a subagents spec that defines specialized sub-task models. Measure: does separability make the agent easier to update when domain knowledge changes? When data sources change? When you swap models? The hypothesis is yes — and quantifying the development velocity difference is a useful portfolio result. Use one of the Anthropic finance templates as a structural reference.

### Go deep on
**Speculative decoding mechanisms — shared-KV MTP is the new standard for single-request inference.** Today's Gemma 4 result makes speculative decoding a production-default technique rather than an experimental one. Go deep here by understanding: (1) the theoretical guarantee (output distribution identical to target — why?), (2) the KV sharing mechanism (how does the draft head condition on target activations?), (3) the acceptance rate dynamics (what workloads maximize/minimize acceptance?). Read: the Gemma 4 MTP blog post (technical section), the original Speculative Decoding paper (arXiv:2211.17192), and the EAGLE speculative decoding paper (arXiv:2401.15077) for the related approach. This technique will propagate to every major open model family — understanding it deeply now means you can apply it to any new model release.

### Ignore for now
**The exposed AI services security findings as a research area.** The 1M exposed services scan is an important operational warning, but it doesn't reveal new attack techniques or novel vulnerabilities — it documents a hygiene failure at deployment scale. The fix (authentication on Ollama + reverse proxy) is immediate and well-documented. Don't treat this as research to explore; treat it as an ops checklist item to close immediately.

---

## What to Build

**Project 1: Vertical Agent Template Packager**
- **What to build:** A CLI tool that takes three inputs — a skills YAML (domain instructions), a connectors config (data source specs), and a subagents spec (task decomposition) — and packages them into a deployable agent template compatible with Claude Managed Agents API and OpenAI Agents SDK.
- **Why now:** Anthropic just published the reference architecture for vertical agent templates. No open-source tooling exists to parameterize and instantiate this pattern. A generalized template packager would work across verticals (finance → legal → clinical) and backends (Claude → GPT-5.5 → Gemini).
- **Stack:** Python CLI, Claude Managed Agents API (beta), OpenAI Agents SDK (for multi-backend testing), YAML schema for skills/connectors/subagents spec.
- **What you'd learn:** The precise semantics of skills vs. connectors vs. subagents separation; what breaks when you conflate layers; how domain knowledge encoding (skills) interacts with data governance (connectors) in practice.

**Project 2: MTP Acceptance Rate Monitor**
- **What to build:** A lightweight observability wrapper for speculative decoding deployments that tracks per-request acceptance rate, speedup ratio, and workload distribution — surfacing when MTP drafters are providing good gains vs. when they're not and why.
- **Why now:** Gemma 4 MTP drafters are now production-available but teams will apply them blindly without understanding the workload-dependent acceptance rate dynamics. A monitor that makes this visible enables informed serving configuration decisions.
- **Stack:** Python, vLLM metrics API, Grafana or simple HTML dashboard, Gemma 4 + MTP draft model pair.
- **What you'd learn:** The empirical distribution of acceptance rates across workloads; the correlation between temperature, domain, and speedup; how to tune serving configurations based on observed acceptance rates.

---

## Opportunities

1. **Vertical agent template library for non-finance domains:** Anthropic shipped finance. Legal (contract review, due diligence, compliance screening), clinical (SOAP note generation, differential diagnosis support, prior auth drafting), and engineering (incident response, architecture review, tech debt triage) are all un-addressed. Each vertical needs the same three-layer template (skills + connectors + subagents) with domain-specific knowledge baked in. First-mover advantage is real — whoever ships a high-quality open-source legal agent template before Anthropic's official legal vertical will own community mindshare.

2. **Speculative decoding benchmark suite for open models:** With Gemma 4 MTP drafters shipping and similar techniques expected for Llama 4, Qwen3, and Mistral 3.5, there is no standardized benchmark suite for measuring actual acceptance rates across workload types (creative, factual, code, math, structured output) across model families. A public benchmark suite with leaderboard would be immediately useful for the inference engine community and would position its authors as the reference point for speculative decoding evaluation.

3. **Agent deployment security scanner:** The 1M exposed services scan reveals a massive unsealed gap between AI deployment velocity and security practice. An automated scanner that checks a deployed agent stack for: unauthenticated endpoints, API key exposure, missing rate limiting, unvalidated tool call inputs, and absent audit logging — and produces a prioritized remediation checklist — would serve every team moving production agents to self-hosted infrastructure. The audit need is immediate; the tooling doesn't exist.

---

*Sources:*
- [Anthropic Finance Agents announcement](https://www.anthropic.com/news/finance-agents)
- [Fortune: Anthropic deepens push into Wall Street](https://fortune.com/2026/05/05/anthropic-wall-street-financial-services-agents-jamie-dimon/)
- [The Register: Anthropic unleashes finance agents for Claude](https://www.theregister.com/2026/05/05/anthropic_unleashes_finance_agents_claude/)
- [Google Blog: Multi-Token Prediction Gemma 4](https://blog.google/innovation-and-ai/technology/developers-tools/multi-token-prediction-gemma-4/)
- [Google AI for Developers: MTP Gemma 4 docs](https://ai.google.dev/gemma/docs/mtp/mtp)
- [MarkTechPost: Google AI Releases MTP Drafters for Gemma 4](https://www.marktechpost.com/2026/05/06/google-ai-releases-multi-token-prediction-mtp-drafters-for-gemma-4-delivering-up-to-3x-faster-inference-without-quality-loss/)
- [AI2 MolmoAct2 blog](https://allenai.org/blog/molmoact2)
- [MolmoAct2 arXiv:2605.02881](https://arxiv.org/html/2605.02881v1)
- [MolmoAct2 HuggingFace paper page](https://huggingface.co/papers/2605.02881)
- [GitHub allenai/molmoact2](https://github.com/allenai/molmoact2)
- [arXiv:2605.04019 Redefining AI Red Teaming in the Agentic Era](https://arxiv.org/html/2605.04019v1)
- [Dreadnode AI Red Teaming platform](https://dreadnode.io/ai-red-team)
- [Hacker News / The Hacker News: 1M Exposed AI Services](https://thehackernews.com/2026/05/we-scanned-1-million-exposed-ai.html)
- [Cloudflare Workers AI Large Models blog](https://blog.cloudflare.com/workers-ai-large-models/)
- [Mistral Vibe Remote Agents announcement](https://mistral.ai/news/vibe-remote-agents-mistral-medium-3-5)
- [InfiAgent arXiv:2601.03204](https://arxiv.org/abs/2601.03204)
- [Stanford AI Index 2026](https://hai.stanford.edu/ai-index/2026-ai-index-report)
