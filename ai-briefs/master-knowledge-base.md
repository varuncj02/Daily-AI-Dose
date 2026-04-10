# AI Frontier Master Knowledge Base
*Living knowledge graph — updated daily from multi-agent research*
*Last updated: April 10, 2026*

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

### Pattern Consolidation
- Agent frameworks (LangGraph, AutoGen, Agent Framework) converging on graph-based orchestration
- MCP as lingua franca for tool definition (ecosystem moving toward this)
- Cloud-managed agents + self-hosted frameworks coexisting; market bifurcating on deployment preference

---

## Agent Evaluation & Reliability

### Current Benchmarks (April 8, 2026 Update)
- **SWE-bench Verified** (500 tasks): GPT-5.4 Pro leads at 88.3%, Claude Opus 4.6 at 79.3%
- **SWE-bench Pro** (hardest tasks — UPDATED April 8):
  - Claude Mythos Preview: 77.8% (restricted access only)
  - GLM-5.1: 58.4% (#1 among publicly accessible models)
  - GPT-5.4: 57.7%
  - Claude Opus 4.6: 57.3%
  - ~~Previous top at 23.3% — this was the April 6 state; SWE-bench Pro has been updated~~
- **SWE-bench Live:** monthly updates prevent contamination
- **CyberGym:** Mythos 83.1% / Opus 4.6 66.6% — new security-specific benchmark now tracking capability gaps
- **MCP-Atlas:** GLM-5.1 71.8 — multi-tool agentic workflow benchmark, now relevant as real-world agent proxy

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

## AI in Security & Cybersecurity (NEW — April 8, 2026)

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
- **Current access:** Restricted to 40 vetted organizations via Project Glasswing; $100M usage credits + $4M open-source security donations

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

### Model Families (April 9, 2026 Update)
| Family | Best Open Size | Context | License | Multimodal | SWE-bench Pro |
|--------|---------------|---------|---------|-----------|--------------|
| Llama 4 | Maverick (17B/128E) | 10M | Commercial | Native | ~54% est. |
| Gemma 4 | 31B Dense | 256K | Apache 2.0 | Native | — |
| GLM-5.1 | 744B total / 40B active | 200K | Apache 2.0 | No | 58.4% (#1 open) |
| Mistral | Small 4 | 128K | Apache 2.0 | Yes | — |
| DeepSeek | V3 | 128K | MIT | No | — |

Note: Claude Mythos Preview (restricted, not publicly accessible) leads at 77.8% — 19 points above best public model.

### Closed-Source Non-Llama Models (NEW — April 9, 2026)
- **Meta Muse Spark:** Closed-source proprietary model from Meta Superintelligence Labs. Not Llama. Private API preview only. Multi-agent "Contemplating" mode. Ranks 4th on AAII v4.0.
- **Watch:** DeepSeek V4 (expected late April 2026) — 1T MoE parameters, 1M context, Engram conditional memory, $0.30/MTok. V4-Lite live on API nodes since early April. Third-party verification pending.

### Infrastructure
- **llama.cpp b8664:** CPU-optimized inference, latest CUDA support
- **Ollama v0.20.2:** Native MLX (Apple Silicon), Gemma 4, Llama 4 Scout, Qwen3-VL
- **vLLM / SGLang:** Production serving; waiting for Saguaro integration

### Quantization
- 4-bit and 2-bit quantized weights: Gemma 4 E2B at sub-1.5GB RAM
- **Unsloth:** 70% VRAM reduction for fine-tuning; MoE optimization (12× faster)

---

## Embodied AI & Robotics (NEW — April 2–9, 2026)

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
- **SWE-bench Pro (hardest):** 23.3% ceiling — hard limit of current autonomous coding
- **Cursor 3 (April 2, 2026):** Parallel Agents Window, Design Mode, 5-hour RL refresh
- **Claude Code v2.1.92 (April 4, 2026):** 500K MCP results, 60% faster diffs, /cost breakdown

### Trend
- IDE architecture shifting from code-completion to agent orchestration
- Real-time RL feedback (5-hour checkpoint refresh in Cursor 3) enables continuous improvement without redeploy

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

### High-Priority Gaps (April 9–10, 2026 Update)
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

---

*This knowledge base is maintained as a living document. Each daily brief contributes:*
- *New concepts → new subsections*
- *Contradictions → flagged with ~~strikethrough~~ and update*
- *Outdated assumptions → marked [OUTDATED as of date]*
- *Evolution tracked: old → limitation → new → tradeoff*
