# Frontier AI Brief — 2026-04-08

> Covering: April 7 to April 8, 2026 only
> ~35 sources scanned · 3 kept · 32 discarded · 10+ repeats from past briefs skipped

Key repeats discarded: T² scaling, Gemma 4, LiteRT-LM, TurboQuant, ProRL, Colab MCP, Emotion concepts (Anthropic interpretability), FLARE, ByteRover, Harrier embeddings, Cursor 3, Claude Code v2.1.92, LeCun AMI Labs, Simon Willison article, llama.cpp regression.

---

## 🔬 Signal

### [Claude Mythos Preview](https://www.anthropic.com/glasswing) · ✅
*April 7, 2026 · Anthropic / Project Glasswing*

Anthropic released a restricted preview of its most capable model yet — not to the public, but to 40 vetted partner organizations for a single use: cybersecurity defense. The capability that makes it too dangerous to release generally is what makes it so useful here: Mythos can find and exploit zero-day vulnerabilities autonomously across every major OS and browser.

**How the capability works:** Mythos was not explicitly trained for hacking. The capability emerged from general improvements in code understanding, reasoning depth, and autonomous task execution. The operative workflow is an agentic scaffold: an isolated container running a target codebase, a prompt asking the model to find a security vulnerability. From there, Mythos reads source code, forms hypotheses about potential flaws, runs the software, uses debuggers, then produces a bug report with a working proof-of-concept exploit. No human guides the intermediate steps.

**Numbers:** CyberGym vulnerability reproduction: 83.1% (vs. Opus 4.6's 66.6%). Working exploit generation on first attempt: 72.4%. HLE without tools: 56.8% vs. Opus 4.6's 40.0%. HLE with tools: 64.7% vs. Opus 4.6's 53.1%. SWE-bench Pro: 77.8%. This last number is important context: Mythos is the strongest coding model released so far, considerably above GPT-5.4 (57.7%) and Opus 4.6 (57.3%) — yet Anthropic won't ship it.

**What it found:** In pre-deployment testing, Mythos identified thousands of zero-day vulnerabilities across major operating systems and browsers. Many were 10–20 years old. The oldest found: a 27-year-old flaw in OpenBSD. A 16-year-old bug in FFmpeg. These had survived millions of automated test runs.

**Project Glasswing:** 12 partner organizations (Apple, Amazon, Cisco, CrowdStrike, Google, Linux Foundation, Microsoft, Palo Alto Networks, and others) plus a wider 40-organization access pool. Anthropic is committing $100M in usage credits and $4M in direct donations to open-source security organizations. A "Cyber Verification Program" is planned for future vetted individual access. General availability: no timeline given.

**Why it matters:** This is the first frontier model released with an explicit capability ceiling — not because of alignment risk, but because the hacking ability is useful enough to warrant controlled access. It also represents a new deployment pattern: capability-gating by verified use-case, not just by safety classification. If this pattern holds, expect future models to have tiered access by professional context, not just by enterprise contract.

---

### [GLM-5.1: Towards Long-Horizon Tasks](https://github.com/zai-org/GLM-5) · ✅
*April 8, 2026 · Z.AI (Zhipu AI)*

Z.AI released GLM-5.1 today — the first open-weight model to reach #1 on SWE-bench Pro. The 5.1 update is not a general capability improvement; it's a focused upgrade to make the model work effectively on extended autonomous tasks rather than short-horizon coding sprints.

**Architecture:** 744B total parameters, 40B active per inference. MoE with 256 experts and 8 activated per token. Two technical innovations inherited from the GLM-5 paper (arXiv 2602.15763) and refined here:

1. **DeepSeek Sparse Attention (DSA):** Standard multi-head attention over 200K tokens is computationally prohibitive. DSA introduces a lightweight indexer that retrieves the top-k most relevant KV entries per query, then computes attention sparsely over that retrieved subset. A deterministic top-k operator ensures training and inference behavior match exactly. This is how GLM-5.1 gets a 200K context window without quadratic memory cost.

2. **Slime (Async RL):** Traditional RL for LLMs is sequential: generate a batch, evaluate, update weights, repeat. Clusters idle while waiting for evaluation. Slime decouples these stages — generation, evaluation, and training run as overlapping async processes on different GPU groups, eliminating the synchronization bottleneck that makes long-horizon RL expensive. This is the core reason GLM-5.1 can sustain performance over hundreds of iterations: the training infrastructure was designed for long rollouts.

**Benchmarks:** SWE-bench Pro: 58.4% (#1 among publicly accessible models; Mythos at 77.8% is restricted). GLM-5.1 clears GPT-5.4 (57.7%) and Claude Opus 4.6 (57.3%). Also: BrowseComp 68.0, CyberGym 68.7, τ³-Bench 70.6, MCP-Atlas 71.8. Context: 200K tokens. Output: up to 128K tokens.

**The 8-hour execution claim:** In one demonstrated task (optimizing a vector database), the model ran 655 iterations over approximately 8 hours, executed thousands of tool calls, revised strategy multiple times, and delivered 6.9× throughput improvement over the initial production version — autonomously, without human intervention. The key mechanism isn't a time limit; it's sustained problem-solving across hundreds of reasoning/execution loops without performance degradation.

**License:** Apache-2.0.

**Try it:**
```bash
# Via vLLM (Docker)
docker pull zai-org/glm-5.1-vllm
vllm serve zai-org/GLM-5.1 --max-model-len 131072

# Via SGLang
python -m sglang.launch_server --model-path zai-org/GLM-5.1
```

---

## 🌐 Open-Source

*Covered in Signal above — GLM-5.1 is the open-source release of the day.*

**How it compares:** GLM-5.1 (58.4%) vs closest open-weight alternatives: no other open model is within 2 points on SWE-bench Pro. The next open model is Llama 4 Maverick at ~54%. GLM-5.1's edge is specifically on long-horizon tasks — standard single-turn coding benchmarks show narrower margins.

---

## 🤖 Agentic AI & AI Engineering

### GLM-5.1 as a Production Coding Agent · ✅
*April 8, 2026*

The MCP-Atlas benchmark (71.8) measures real multi-tool agent task performance — not synthetic coding questions. GLM-5.1's lead here (vs. GPT-5.4, Opus 4.6) suggests the async RL training specifically improves capability on the kind of tasks that appear in real agent deployments: multi-step tool use, iterative debugging, environment interaction.

**Capability affected:** Planning (long-horizon), Tools (sustained tool-calling across hundreds of steps), Reliability (doesn't degrade over extended sessions).

**Churn or shift?** Shift. The key breakthrough here is not parameter count — it's the training infrastructure (Slime async RL) that allows the model to learn from long agent trajectories efficiently. This is a reproducible architectural pattern, not just a bigger model.

**Do this week:** If you're building a coding agent that needs to run multi-hour autonomous tasks, GLM-5.1 is the first open-weight model worth benchmarking for this use case. Run it against your specific task (not just SWE-bench) — the 200K context and 128K output window matter for repository-scale work.

Agent stack — today's update touches:
**Planner → LLM → Tools/MCP → Verifier** (specifically: planning over extended horizons and sustained tool execution)

---

### Anthropic Mythos Preview as Agentic Security Agent · ✅
*April 7, 2026*

The operative pattern inside Project Glasswing is exactly an agent loop: target codebase → LLM reads and reasons → forms hypothesis → executes code/debugger → evaluates output → revises → produces exploit POC. The entire vulnerability research workflow runs autonomously.

**Do this week:** If you're building security tooling, apply for Project Glasswing access at anthropic.com/glasswing. The model's code-reading + hypothesis + execution loop is a template for any domain requiring expert-level iterative analysis.

---

## ⚡ Infra & Serving

*No major new infra releases from April 7-8. Previous brief covered TurboQuant and LiteRT-LM.*

Note for builders: GLM-5.1's DSA architecture is a serving-relevant innovation — sparsifying attention at inference time reduces KV cache footprint for long-context requests. The technique is architecture-level (built-in), not a post-hoc quantization. Watch for this pattern in other MoE releases.

---

## 🌍 Wider World

### [Anthropic Hits $30B ARR, Signs 3.5GW Compute Deal](https://www.anthropic.com/news/google-broadcom-partnership-compute) · ✅
*April 6-7, 2026 · Anthropic*

Anthropic's annualized revenue run rate crossed $30 billion — up from $9 billion at the end of 2025, a 3.3× increase in roughly one quarter. More than 1,000 business customers now spend over $1M annually; that figure doubled in under two months (from 500 in February).

Alongside the revenue disclosure, Anthropic announced a multi-gigawatt compute deal with Google and Broadcom: access to 3.5 gigawatts of next-generation TPU capacity, expected online starting 2027. Context: Anthropic already has 1GW of Google TPU capacity in 2026. The new deal expands this 4.5× and locks in infrastructure through the 2027-2028 training window.

The compute strategy is multi-platform: Anthropic trains Claude across AWS Trainium, Google TPUs, and NVIDIA GPUs, matching workloads to chips by task type and cost. Broadcom separately confirmed a long-term agreement with Google for future custom TPU chip design through 2031, suggesting the chip supply chain for Anthropic's scaling roadmap is now secured.

**Why it matters:** 3.5GW is a staggering number. For reference, a typical data center runs at 50-100MW. This is 35-70 hyperscale data centers worth of TPU capacity. The compute deal timeline (2027 online) maps to the next training generation after Mythos — meaning Mythos-class capability is being trained on the current infrastructure; whatever comes next will have access to 4.5× more TPU capacity. The revenue trajectory suggests Anthropic has solved the go-to-market problem; the compute deal says they're now solving the infrastructure ceiling.

---

## 🔍 Deep Dive

### Claude Mythos Preview: Capability-Gating as a New Deployment Pattern

**What it does:** Mythos is Anthropic's most capable model. Unlike every prior frontier model, it's being withheld from general release. The stated reason: the same improvements that make it dramatically better at patching code make it dramatically better at exploiting code — and the hacking capability is judged too dangerous to ship broadly without controls.

**How the vulnerability-finding works:**

Standard automated security tools (fuzzing, static analysis, symbolic execution) catch surface-level bugs. They fail on deep logical flaws that require understanding code semantics across large codebases. Mythos works differently:

```
[Isolated container with target codebase]
         ↓
[Mythos reads source code — full context window]
         ↓
[Forms hypothesis about potential vulnerability class]
         ↓
[Executes code, instruments with debugger]
         ↓
[Evaluates runtime behavior vs. hypothesis]
         ↓
[Revises hypothesis, iterates]
         ↓
[Produces bug report + working proof-of-concept exploit]
```

This is not fuzzing. This is semantic code reasoning at frontier scale, combined with autonomous execution and iteration. The model doesn't search for a known vulnerability pattern — it hypothesizes about where logical flaws might exist based on code structure, then tests those hypotheses empirically.

**Why old bugs survive:** The 27-year-old OpenBSD bug and 16-year-old FFmpeg bug were not missed by careless humans. They survived because they require holding the semantics of large codebases in mind across thousands of lines to recognize that a particular interaction creates an exploitable state. Automated tools lack semantic understanding. Human auditors work with finite attention windows. Mythos has neither limitation at the scale it operates.

**Architecture before/after:**

Before Mythos (standard fuzzing + human audit pipeline):
```
Fuzzer → coverage-guided mutation → known crash classes → human triage → CVE
```

After Mythos (agentic semantic analysis):
```
[Model] reads codebase → hypothesizes vulnerability → instruments → executes → validates → POC
Handles: novel vulnerability classes, decade-old logic bugs, cross-component interactions
```

**Tradeoffs:**
- Capability is dual-use by construction. There's no version of "finds zero-days" that doesn't also mean "can write exploits."
- The capability emerged implicitly from general improvements — which means future models may have equivalent or greater capability regardless of deliberate training decisions.
- Controlled access works until it doesn't: vetted partners can be compromised, credentials leaked, or access misused. The 40-organization controlled preview is a mitigation, not a solution.

**Where it breaks:** The approach requires a clean execution environment and access to source code. Binary-only targets (closed firmware, compiled-only distributions) present harder challenges. Also, the model's exploit is often technically correct but may require adaptation for specific kernel versions, ASLR states, or deployment configurations — a skilled human still provides real-world weaponization value.

**So what:** This changes two things. First, vulnerability discovery at scale just became economically viable for defenders — organizations that join Glasswing can now scan their codebases with a system that finds bugs no tool has caught in decades. Second, the deployment pattern itself is new: a frontier model withheld from public API access, distributed via institutional partnership only. Expect more of this as models become capable enough that specific abilities require gatekeeping even when the model as a whole is commercially valuable.

---

## 📌 Small Finds

- **Anthropic revenue context:** At $30B ARR, Anthropic's annualized revenue has exceeded OpenAI's $25B reported earlier this year, making it the highest-revenue AI lab by run rate. Driven by enterprise Claude deployments; 1,000+ customers at $1M+ annually.

- **GLM-5.1 API cost:** Despite the 744B parameter count, the GLM-5.1 API price is roughly $3/M output tokens — well below GPT-5.4 and Opus 4.6 pricing. The MoE architecture (only 40B active) is why: inference cost tracks active parameters, not total parameters.

- **Mythos SWE-bench Pro vs. GLM-5.1:** The gap is stark — 77.8% vs. 58.4%. Mythos is 19 points ahead of the best publicly accessible model on the hardest coding benchmark. This means the gap between frontier-restricted and frontier-accessible capability widened significantly today.

- **Project Glasswing scope:** "Every major operating system and every major web browser" — Anthropic's words on what Mythos scanned. The vulnerabilities it found have been disclosed to vendors; many are now patched. The finding process is ongoing for Glasswing partners.

---

## 🧭 Frontier Direction

Today's developments reveal two overlapping trends:

- **Bottleneck under attack:** Long-horizon autonomous task execution. GLM-5.1's Slime async RL training is a direct attack on the training bottleneck that limits how well models learn from extended agent trajectories. Mythos's vulnerability research loop is a deployed example of what a model looks like when that capability fully matures — hundreds of reasoning/execution cycles, sustained over hours, without degradation.

- **Broader trend:** Capability-gating is becoming infrastructure. Mythos won't be generally released. The SWE-bench Pro scores (Mythos 77.8% restricted vs GLM-5.1 58.4% open) quantify the gap that gatekeeping creates. Expect this bifurcation — open-accessible capability vs. restricted-access capability — to become a permanent feature of the frontier landscape rather than a temporary safety measure.

- **Still unsolved:** How to train open models to match restricted frontier capability on long-horizon agentic tasks. GLM-5.1 (58.4%) is the best open effort, but 19 points behind Mythos. The gap is not just data or parameter count — it's training infrastructure (async RL for long trajectories) and likely post-training techniques that aren't public.

Trend arrows:
- Sequential RL training → Async decoupled RL (Slime) — enables long-horizon agent training
- Chinchilla-optimal pretraining → T²-optimal overtrained models (from prior briefs)
- General public API access → tiered/capability-gated deployment (Mythos pattern)
- Short-horizon coding agents → 8-hour sustained autonomous execution (GLM-5.1)

---

## 🛠️ Builder Takeaways

- ✓ **Try:** GLM-5.1 via vLLM or SGLang for any repository-scale coding task. It's the best open-weight coding agent today, Apache-2.0, and the 200K context handles full codebases. Don't benchmark on single-file tasks — the advantage shows on multi-file, multi-step work.

- → **Experiment:** Use GLM-5.1's MCP-Atlas score (71.8) as evidence that the model handles multi-tool agentic workflows, not just code generation. Run it in your agent framework against a real task before defaulting to Opus 4.6 or GPT-5.4 — at $3/M tokens, the cost-to-capability ratio may make it compelling.

- → **Apply:** If your org does security research on critical infrastructure, apply to Project Glasswing (anthropic.com/glasswing). Even a Cyber Verification Program membership, when available, would give access to a vulnerability discovery capability that outperforms all automated tools and most human auditors on decade-old bugs.

- ✗ **Ignore:** Claims that "Mythos will be available soon." Anthropic explicitly said it won't be made generally available. The Cyber Verification Program is future and invite-only. Plan your security tooling stack around what's accessible — GLM-5.1, Claude Opus 4.6, GPT-5.4.

---

## 💡 Opportunities

**1. Agentic security scanning service (Glasswing-equivalent for SMBs).** Only 40 organizations got Mythos access. Millions of codebases have decade-old bugs that automated tools can't find. A service that wraps GLM-5.1 (or future Opus-class models) in the same agentic scaffold — isolated container, semantic analysis, exploit POC generation — could make professional-grade vulnerability research accessible below enterprise partnership scale. The Mythos workflow is now publicly documented; the model capability is available via GLM-5.1.

**2. Long-horizon agent reliability monitoring.** GLM-5.1 demonstrates 8-hour autonomous runs. No mature monitoring tooling exists for agents running at this timescale: detecting reasoning drift, context window budget management, mid-task strategy revision tracking, output quality over time. The async RL training pattern (Slime) will spread — expect more models trained for long-horizon tasks. None of them ship with production-grade observability.

**3. Dual-use AI capability advisory.** Mythos establishes that frontier models will henceforth have capabilities that require institutional gatekeeping. Legal, compliance, and risk teams at enterprises have no frameworks for "AI that can autonomously exploit software vulnerabilities." The regulatory gap between what these models can do and what governance frameworks cover is months to years wide. Technical advisory bridging AI capability assessment and legal/compliance frameworks is a real commercial need today.

---

*Sources: [Project Glasswing — Anthropic](https://www.anthropic.com/glasswing) · [Claude Mythos Preview — red.anthropic.com](https://red.anthropic.com/2026/mythos-preview/) · [TechCrunch — Mythos](https://techcrunch.com/2026/04/07/anthropic-mythos-ai-model-preview-security/) · [CNBC — Mythos withheld](https://www.cnbc.com/2026/04/07/anthropic-claude-mythos-ai-hackers-cyberattacks.html) · [Help Net Security](https://www.helpnetsecurity.com/2026/04/08/anthropic-claude-mythos-preview-identify-vulnerabilities/) · [The Hacker News](https://thehackernews.com/2026/04/anthropics-claude-mythos-finds.html) · [Axios — Mythos](https://www.axios.com/2026/04/07/anthropic-mythos-preview-cybersecurity-risks) · [GLM-5.1 GitHub](https://github.com/zai-org/GLM-5) · [Z.AI Docs](https://docs.z.ai/guides/llm/glm-5.1) · [GLM-5 paper arXiv:2602.15763](https://arxiv.org/abs/2602.15763) · [Analytics India Mag — GLM-5.1](https://analyticsindiamag.com/ai-news/zhipu-ais-glm-51-becomes-top-model-on-swe-bench-pro-beats-gpt-54-claude-opus-46) · [Dataconomy — GLM-5.1](https://dataconomy.com/2026/04/08/z-ais-glm-5-1-tops-swe-bench-pro-beating-major-ai-rivals/) · [Anthropic compute deal](https://www.anthropic.com/news/google-broadcom-partnership-compute) · [The Register — Broadcom](https://www.theregister.com/2026/04/07/broadcom_google_chip_deal_anthropic_customer/) · [BenchLM SWE-bench Pro](https://benchlm.ai/benchmarks/swePro)*
