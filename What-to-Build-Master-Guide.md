# What to Build — Master Guide
*Synthesized from all Frontier AI Briefs: April 6–21, 2026*

---

## 🎯 The 3 Highest-Signal Projects

These three appeared across multiple briefs, are backed by the strongest empirical evidence, and are directly actionable today. If you build nothing else, build these.

---

### #1 — APEX-Agents-AA Professional Task Agent
**Why it's #1:** The APEX-Agents-AA benchmark is the most important empirical finding in this entire body of briefs. Every frontier model fails 75%+ on real professional knowledge work tasks. The 24% ceiling is the ground truth. The improvement methodology (File-as-Bus + task budgets + verifier loop) has appeared across 4 separate briefs and is clearly the right architecture. Building and publishing a system that measurably beats this baseline is the single most portfolio-worthy project you can do right now.

**The build:**
Pick one domain — investment banking due diligence, corporate law memos, or consulting analysis. Get the open dataset from `huggingface.co/datasets/mercor/apex-agents`. Establish your baseline score with a naive agent. Then layer in three interventions in sequence, measuring after each:
1. **File-as-Bus workspace** — structured artifact files (plan.md, analysis.md, evidence/) instead of context-window coordination between steps. The AiScientist ablation showed a 31.8% MLE-Bench improvement from this pattern alone.
2. **Task budgets** — Claude Opus 4.7's `task-budgets-2026-03-13` beta header. Set a ceiling, watch how the model prioritizes under constraint.
3. **Verifier pass** — a second Opus 4.7 instance with a rubric-checking prompt reviews the draft before final output.

**Stack:** Claude Opus 4.7 (task budgets + xhigh effort), YAML-defined artifact workspace, separate verifier agent, APEX-Agents Mercor dataset.

**What you'd learn:** How professional-task failure modes manifest empirically. Whether the three interventions compound or specialize. What the right artifact granularity is for multi-document synthesis. This is the most complete single project for understanding where agentic AI actually breaks.

**Signal sources:** April 20 brief, April 21 brief, April 16 brief (File-as-Bus), April 15 brief (verifier loops).

---

### #2 — Isolation-Hardened Agent Evaluation Harness
**Why it's #2:** UC Berkeley's BenchJack finding (April 15) is the most underappreciated story in these briefs: every major agent benchmark — SWE-bench, WebArena, Terminal-Bench, GAIA, OSWorld — can be exploited to achieve near-perfect scores without solving a single task. 10 lines of Python in conftest.py "solves" all of SWE-bench Verified. This means you cannot trust published benchmark scores unless you know the harness is properly isolated. Before you evaluate anything seriously, you need this infrastructure. It also happens to be conspicuously absent from the open-source ecosystem.

**The build:**
Design a mini-benchmark of 15–20 tasks in a domain you care about. The key constraint: architectural isolation between agent and evaluator. Specifically:
- Agent runs in a Docker/gVisor container with no access to the test config or evaluation filesystem
- Scoring is done by a separate verifier process with zero shared state
- Task answers are verified externally — the agent cannot read, write, or inject into the evaluation path
- Each task is designed so "getting the answer right" is verifiably distinct from "manipulating the harness"

**Stack:** Docker/gVisor for sandboxing, any agent framework (LangGraph, Claude Code SDK), a separate verifier process, 15–20 tasks you design from scratch.

**What you'd learn:** Security-oriented thinking for agent evaluation. How to design tasks where capability and cheating are structurally separated. The difference between evaluation validity (does this measure what I think?) and reliability (are the scores consistent?). This skill — building credible evals — is foundational to everything else in this list.

**Signal sources:** April 15 brief (Berkeley BenchJack), April 14 brief (hard benchmark tracker), April 13 brief (CANN/CUDA benchmark).

---

### #3 — Execution-Enabled Research QA Pipeline
**Why it's #3:** The arXiv:2604.12198 finding from April 16 is one of the cleanest empirical results across all the briefs: 97.7% of substantive concerns in computational research papers are *only* surfaceable by running the code — not by reading the paper. A model that reads without executing misses almost all of the real value. This isn't a subtle point. It defines the correct architecture for any AI-assisted research or analysis workflow. No production implementation of this pattern exists for domains outside physics.

**The build:**
A pipeline that takes a computational paper with publicly available code, runs the reproduction loop autonomously, and produces a structured concern report:
1. Parse the paper, identify the core computational claims
2. Generate code to reproduce the key results
3. Execute in a sandboxed environment (E2B or Modal)
4. Compare executed output to reported results
5. Flag discrepancies above a threshold as structured concerns

The benchmark for success: does your concern report match what a human would flag? The Princeton study (111 papers, ~42% flagged concerns) gives you a reference distribution.

**Stack:** Claude Opus 4.7 / GPT-5.4 for paper parsing and code generation, E2B or Modal for sandboxed execution, structured comparison via programmatic diff, JSON concern output format.

**What you'd learn:** How to architect the read-plan-execute-compare loop at scale. Sandboxed execution integration. How to distinguish execution errors from paper errors. The fundamental architecture principle: execution, not language understanding, is where AI creates analytical value.

**Signal sources:** April 16 brief (arXiv:2604.12198 mini research loop), April 15 brief (durable agent execution pattern).

---

## 📋 Complete Project Catalog

All "What to Build" projects across all briefs, organized by theme.

---

### Theme 1: Agent Reliability & Professional Tasks

**APEX-Agents-AA Score Improvement Framework** *(April 20)*
Build a multi-agent system targeting one APEX-Agents domain. Measure baseline, apply File-as-Bus + verifier loop + task budget, document the delta per intervention.
- Stack: Claude Opus 4.7, File-as-Bus YAML workspace, verifier agent, Mercor dataset
- Learn: Professional-task failure modes, how task budgets change prioritization, how to design verifier loops for specific error types

**Domain-Specific Agent for APEX-Agents-AA Investment Banking** *(April 21)*
Specialized agent targeting the investment banking task subset. Apply File-as-Bus, task budgets, and verifier pass. Target 40%+ from the 24% frontier baseline.
- Stack: Opus 4.7 (task budgets), YAML artifact types (data-room-summary.yaml, financial-model.yaml, exec-summary.yaml), verifier agent, Mercor dataset
- Learn: Multi-document financial synthesis, whether the three interventions compound, correct artifact granularity

**Professional-Task Verifier Agent** *(Opportunity, April 20)*
A second-pass agent that reviews, critiques, and flags errors in AI-generated professional outputs before they reach humans. Domain-specific (legal, financial, consulting). The verifier doesn't produce output — just verifies it. Simpler and more reliable than a full agent.

---

### Theme 2: Evaluation Infrastructure

**Isolation-Hardened Agent Evaluation Harness** *(April 15)*
Build a benchmark harness where evaluation is architecturally isolated from agent access. No shared filesystem, no readable task config, scoring via external verifier only.
- Stack: Docker/gVisor, any agent framework, separate verifier process, custom 15–20 task set
- Learn: Security-oriented eval design, evaluation validity vs. reliability, harness exploitation patterns

**Hard Benchmark Tracker** *(April 14)*
A live tracker that runs HLE and ARC-AGI-3 tasks across models you care about, tracking performance over time as models release.
- Stack: Python + model APIs (OpenAI, Anthropic, MiniMax, DeepSeek), 50–100 HLE questions, ARC-AGI-3 public dataset, automated daily runs
- Learn: First-hand capability comparisons without relying on vendor benchmarks, how to design gaming-resistant evals, actual capability delta between open-weight and frontier APIs

**CANN vs. CUDA Inference Benchmark** *(April 13)*
When DeepSeek V4 launches, benchmark it side-by-side with V3.2 and GLM-5.1 on your actual workload: throughput, context recall at 128K, cost per task.
- Stack: Python harness, DeepSeek V4 API, Llama 4 Scout and GLM-5.1 for comparison
- Learn: Whether CANN/Ascend produces meaningfully different model behavior for your use case

**Interface Paradigm Benchmark — Computer Use vs. MCP vs. Tool Use** *(April 21)*
Run the same agentic workflow via three implementations: (a) computer use, (b) structured tool calling, (c) MCP server. Measure success rate, median time, failure mode distribution per 50 runs.
- Stack: Claude Opus 4.7 + computer use API, Codex computer use, custom MCP server, logging harness
- Learn: Empirical reliability profiles for each interface paradigm, when computer use is worth the cost

---

### Theme 3: Research & Scientific AI

**Execution-Enabled Research QA Pipeline** *(April 16)*
Takes a computational paper with available code, autonomously runs the reproduction loop, and produces a structured concern report. Target: surface the 97.7% of concerns that only appear through execution.
- Stack: Claude Opus 4.7 for parsing and code gen, E2B or Modal for sandboxed execution, structured diff comparison, JSON output
- Learn: Read-plan-execute-compare loop at scale, distinguishing execution errors from paper errors, how to measure concern detection recall

**Artifact-Coordinated Multi-Agent Research System** *(April 16)*
A multi-agent ML research system using the File-as-Bus protocol. Orchestrator decomposes ML experiments into stages, spawns specialized agents per stage, coordinates via durable artifact files.
- Stack: LangGraph or Claude Managed Agents, structured YAML/JSON workspace (problem.md, plan.md, code/, results/, analysis.md), specialized subagents per stage
- Learn: Agent coordination at filesystem level, preventing context drift across multi-agent sessions, ablation methodology using MLE-Bench Lite

**ARC-AGI-3 Domain-Specific Agent Harness** *(April 13)*
Build an ARC-AGI-3-style harness for a domain you work in — a series of interactive environments with unknown rules. Implement the orchestrator-subagent program synthesis pattern from Symbolica. Measure against a CoT baseline.
- Stack: Agentica SDK (Symbolica), Claude Opus 4.6 or GPT-5.4 for orchestrator, Python program synthesis as core execution mechanism
- Learn: Orchestrator-subagent architectural tradeoffs, limits of program synthesis for non-rule-governed domains, how to design harnesses that measure adaptive capability

---

### Theme 4: Infrastructure & Training

**Specification Coverage Analyzer** *(April 10)*
Takes a codebase or CLI binary and measures behavioral specification completeness: what fraction of the public API/CLI surface has automated test coverage, and where are the gaps?
- Stack: Python, AST parsing for code coverage analysis, Opus 4.6 or Sonnet 4.6 to generate test cases for uncovered paths, pytest as execution layer
- Learn: Spec-driven AI engineering pipeline, where spec-to-capability gaps exist in practice, the bottleneck MirrorCode identified

**Long-Horizon Coding Benchmark for Your Domain** *(April 10)*
A MirrorCode-style benchmark for a domain you work in. Collect 5–10 real internal tools, write behavioral test suites for each, run frontier models and measure reimplementation success rate and inference cost per task.
- Stack: Any language your domain uses; harness runs the AI's output and compares against a reference binary
- Learn: How to design eval harnesses for long-horizon agentic tasks — the foundational skill as the field moves from HumanEval to real engineering projects

**Scaffold Auto-Optimizer for RL Fine-Tuning** *(April 14)*
Implements the MiniMax self-evolution loop against your own training scaffold. Model runs evals, writes self-feedback, proposes and implements scaffold code changes, loop repeats.
- Stack: M2.7 or Llama 4 Maverick as optimizer agent, your existing eval harness, version-controlled scaffold codebase, CI to run evals
- Learn: Practical limits of model-as-optimizer: when circular optimization occurs, what eval setups prevent it, what ceiling the loop approaches

**Durable Agent Execution Framework** *(April 15)*
A language-agnostic durable agent execution layer using event sourcing: agent state is always a replayable log, compute is always ephemeral, external events are first-class triggers.
- Stack: Temporal or AWS Step Functions, any LLM API, durable message log (Kafka or DynamoDB Streams), Docker for sandboxed containers
- Learn: Event sourcing applied to LLM agents, failure recovery without state loss, async trigger handling, the boundary between "session" and "process" in AI systems

---

### Theme 5: Agent Interface & Integration

**Computer Use → MCP Compiler** *(Opportunity, April 21)*
A tool that records an agent's computer use session, extracts reliable action sequences, and generates an MCP server definition for those interactions. Collapses integration time from weeks to hours. No one has built this.

**Codex + Claude Code Integration Router** *(Opportunity, April 21)*
Orchestration middleware that routes agentic workflows between Codex and Claude Code based on: task type (benchmark data), available integrations (MCP vs. Codex tools), current cost/availability, session continuity.

---

## 📊 Opportunities Across All Briefs

These are product/research gaps identified across all briefs. Organized from most to least validated.

| Opportunity | Signal Strength | First Appeared |
|---|---|---|
| Professional-task verifier agent as a service | ★★★★★ | April 20 |
| Isolation-hardened eval harness tooling | ★★★★★ | April 15 |
| Execution-enabled literature QA as a service | ★★★★☆ | April 16 |
| APEX-Agents vertical specialist agent products (banking, law, consulting) | ★★★★☆ | April 20 |
| Specification generation and verification tooling | ★★★★☆ | April 10 |
| Task budget optimization layer / middleware for API users | ★★★☆☆ | April 20 |
| RL scaffold optimization tooling | ★★★☆☆ | April 14 |
| Computer use → MCP compiler as a service | ★★★☆☆ | April 21 |
| Harness-as-a-service for regulated industries (on-prem) | ★★★☆☆ | April 16 |
| Domain-specific benchmark harness for new vertical models | ★★★☆☆ | April 21 |
| CANN/Ascend inference tooling (vLLM equivalent for Huawei) | ★★★☆☆ | April 13 |
| Competitive intelligence API for AI-competing verticals | ★★☆☆☆ | April 16 |
| GPU procurement advisory for frontier AI | ★★☆☆☆ | April 10 |
| AI governance transition advisory (OpenAI/Anthropic) | ★★☆☆☆ | April 13 |
| Durable agent execution for regulated industries (on-prem) | ★★☆☆☆ | April 15 |
| Agentic vision pipeline for industrial inspection | ★★☆☆☆ | April 15 |

---

## 🔗 Cross-Brief Theme Summary

Three themes recurred across nearly every brief, in order of repetition:

**1. The professional task reliability gap** — APEX-Agents-AA (April 20, 21), AiScientist File-as-Bus (April 14, 15, 16), MirrorCode specification gap (April 10). Every brief touched this theme. The core insight: agents fail long-horizon professional tasks because of orchestration architecture, not model capability. File-as-Bus + verifier loops + task budgets are the architecture that addresses it.

**2. Credible evaluation infrastructure** — BenchJack (April 15), hard benchmark tracker (April 14), APEX-Agents-AA (April 20), interface paradigm benchmark (April 21). The ability to measure agent capability without being gamed is foundational. Most existing benchmarks are exploitable. Building proper evals is a prerequisite for everything else.

**3. Durable execution as the default agent pattern** — Claude Code Routines saga/event-sourcing (April 15), File-as-Bus artifact coordination (April 14–16), task budgets as execution contracts (April 20). The mental model is shifting from "interactive REPL" to "durable background worker." This infrastructure change underlies all serious production agent deployment.

---

*Guide compiled April 21, 2026 from 10 Frontier AI Briefs (April 6–21, 2026)*
