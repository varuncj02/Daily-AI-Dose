# AI Frontier Master Knowledge Base
*Living knowledge graph — updated daily from multi-agent research*
*Last updated: April 8, 2026*

---

## Reasoning & Planning

### Chain-of-Thought → Hierarchical Planning
- CoT remains dominant for single-step reasoning but fails for long-horizon tasks (>50 steps)
- Hierarchical planning (decompose → plan subtasks → execute) is the emerging standard for agent planners
- Cursor 3's parallel Agents Window (April 2026) demonstrates this: task decomposition into parallel sub-agents rather than sequential CoT
- **Open problem:** Agents still fail at coherent replanning after unexpected tool failures mid-chain

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

### Memory Architecture Pattern (Current Standard)
```
Context Window (working memory)
    ↕
External Storage (episodic/semantic/procedural)
    ├── Vector DB (for semantic similarity)
    ├── Graph DB (for relational/temporal)
    └── KV Store (for structured facts)
```
Tiered memory is now standard; distinction is which tier handles which query type.

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

## Capability-Gating (NEW — April 8, 2026)

### Restricted-Access Frontier Models
- **Anthropic Mythos Preview (April 7, 2026):** First frontier model withheld from general release due to specific dual-use capability (autonomous vulnerability exploitation)
- **Pattern:** Capability-gated deployment — not general public API, distributed via institutional partnership only (40 vetted organizations via Project Glasswing)
- **Mechanism for future access:** Planned Cyber Verification Program for individual researchers
- **Quantified capability gap:** Mythos 77.8% SWE-bench Pro vs. best public model GLM-5.1 at 58.4% — 19 point gap is the current "cost of safety gatekeeping"
- **Why it matters architecturally:** The capability (autonomous zero-day discovery + exploit generation) emerged implicitly from general improvements in code reasoning — *not* from explicit security training. Future models will likely have equivalent capability by default.
- **Deployment evolution:**
  - Pre-2026: All frontier models released as general APIs
  - 2026+: Tiered access — general API + restricted-access tiers for high-capability models
- **Link to safety:** Confidence calibration (can't self-assess hacking risk) + autonomous execution = dual-use by construction at frontier scale

---

## Tool Use & Orchestration (MCP, A2A, Function Calling)

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

### Context Window Race
- Llama 4 Scout: 10M tokens (current open-weight leader)
- Gemma 4: 256K tokens
- GPT-4o: 128K tokens
- **Practical ceiling:** Economic crossover between long-context and RAG shifts as context costs fall

---

## Open-Source Landscape

### Model Families (April 8, 2026 Update)
| Family | Best Open Size | Context | License | Multimodal | SWE-bench Pro |
|--------|---------------|---------|---------|-----------|--------------|
| Llama 4 | Maverick (17B/128E) | 10M | Commercial | Native | ~54% est. |
| Gemma 4 | 31B Dense | 256K | Apache 2.0 | Native | — |
| GLM-5.1 | 744B total / 40B active | 200K | Apache 2.0 | No | 58.4% (#1 open) |
| Mistral | Small 4 | 128K | Apache 2.0 | Yes | — |
| DeepSeek | V3 | 128K | MIT | No | — |

Note: Claude Mythos Preview (restricted, not publicly accessible) leads at 77.8% — 19 points above best public model.

### Infrastructure
- **llama.cpp b8664:** CPU-optimized inference, latest CUDA support
- **Ollama v0.20.2:** Native MLX (Apple Silicon), Gemma 4, Llama 4 Scout, Qwen3-VL
- **vLLM / SGLang:** Production serving; waiting for Saguaro integration

### Quantization
- 4-bit and 2-bit quantized weights: Gemma 4 E2B at sub-1.5GB RAM
- **Unsloth:** 70% VRAM reduction for fine-tuning; MoE optimization (12× faster)

---

## Local/Edge Deployment

### Capability (April 2026)
- **Raspberry Pi 5:** Gemma 4 E2B at 7.6 tok/s with NPU acceleration
- **Apple Silicon M5+:** 31 tok/s on Gemma 4 31B with macOS 26.2+ Neural Accelerator
- **Single H100:** Llama 4 Scout (17B active, 10M context)
- **Edge multimodal agents:** Now technically feasible; software stack mostly unbuilt (opportunity)

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

## Business Opportunities & Infra Gaps

### High-Priority Gaps (April 8, 2026 Update)
1. **Agent Memory-as-a-Service:** Managed LLM-curated hierarchical memory for vertical agents
2. **Agent Reliability Monitoring:** 90% failure rate, no mature monitoring tool; confidence calibration unsolved
3. **Long-horizon agent observability (NEW April 8):** No tooling for monitoring agents running 8-hour / 600+ iteration sessions — reasoning drift, strategy revision tracking, context budget management
4. **Edge Multimodal Agent Products:** Hardware ready (Gemma 4 E2B), software stack missing
5. **MCP Server Registry:** 5,800+ servers, no curated quality-graded discovery layer
6. **Multi-Agent Coordination Middleware:** Proven pattern (Cursor 3), missing for non-coding domains
7. **Serving Cost Optimization Tooling:** TurboQuant + Saguaro + T² = 50-70% cost reduction; expertise barrier is commercial opportunity
8. **Agentic security scanning for SMBs (NEW April 8):** Mythos pattern documented; GLM-5.1 capable of agentic code analysis; no accessible service below Glasswing enterprise partnership level
9. **Dual-use AI capability advisory (NEW April 8):** Regulatory/compliance gap between what frontier models can do (autonomous exploit generation) and what governance frameworks cover

---

## Economics & Scale (April 8, 2026 Update)

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
- [April 8]: GLM-5.1 — async RL (Slime) + DSA → first open model sustaining 8-hour / 655-iteration agent sessions; SWE-bench Pro #1 among publicly accessible models
- [April 8]: Anthropic Mythos Preview — capability-gating as deployment pattern; autonomous zero-day discovery at frontier scale; dual-use constraint forces restricted access
- [April 8]: Frontier capability bifurcation — 19-point SWE-bench Pro gap between restricted (77.8%) and open (58.4%) models; gap is structural, not just temporary

---

*This knowledge base is maintained as a living document. Each daily brief contributes:*
- *New concepts → new subsections*
- *Contradictions → flagged with ~~strikethrough~~ and update*
- *Outdated assumptions → marked [OUTDATED as of date]*
- *Evolution tracked: old → limitation → new → tradeoff*
