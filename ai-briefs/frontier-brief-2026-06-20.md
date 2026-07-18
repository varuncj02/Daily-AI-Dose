# Frontier AI Brief — 2026-06-20

> Covering: 2026-06-18 to 2026-06-20 (48h window, weekend catch-up)
> 9 candidates reviewed · 5 kept · 4 discarded (age/weak evidence/duplication)

---

## Executive View

The talent layer moved more than the model layer this window. Two co-architects of the current frontier — Noam Shazeer (Transformer co-author, Gemini co-lead) and John Jumper (AlphaFold co-creator, Nobel laureate) — left Google for OpenAI and Anthropic respectively within 24 hours of each other, both during the run-up to OpenAI's and Anthropic's IPOs. Separately, the MCP trust-boundary problem flagged in the last several briefs got two concrete, dated developments pulling in opposite directions: Microsoft disclosed a real RCE exploit chain (AutoJack) that crosses the localhost boundary via an agent's own browsing tool, while Anthropic shipped Enterprise-Managed Authorization to remove the per-user OAuth friction that has been the main blocker to enterprise MCP adoption. Both are evidence that the "MCP needs a trust/identity layer" thesis is now being actively built out, from both the attack and the defense side, in production systems rather than papers.

---

## Top Signals

### [Noam Shazeer leaves Google for OpenAI; John Jumper leaves Google DeepMind for Anthropic](https://www.techtimes.com/articles/318613/20260618/transformer-architect-behind-gemini-jumps-openai-after-google-spent-27b.htm) · High
*Published: 2026-06-18 (Shazeer) / 2026-06-19 (Jumper)*

**What changed**
Two of Google's most consequential AI researchers departed within about 24 hours of each other. Noam Shazeer — co-author of "Attention Is All You Need," co-lead of Gemini, and the researcher Google paid $2.7B to reacquire (via the Character.AI deal) in 2024 — announced on June 18 that he is leaving Google for OpenAI; this is his second exit from Google in five years. John Jumper — 2024 Nobel laureate in Chemistry and co-creator of AlphaFold, Google DeepMind's protein-structure-prediction system — announced on June 19 that he is leaving after nearly nine years to join Anthropic. Sam Altman publicly welcomed Shazeer as someone he'd "wanted to work with since OpenAI's early days." Both moves land in the run-up to OpenAI's and Anthropic's respective IPOs (both filed confidential S-1s earlier in June).

**How it works**
This isn't a technical development, but it has structural consequences for where frontier capability work concentrates. Shazeer's specialty is pretraining and model architecture — exactly the area OpenAI is racing to lead before its public-market debut. Jumper's specialty is applying transformer-style architectures to a non-language domain (protein structure) — exactly the area Anthropic has been building toward all year: wet labs, a life-sciences benchmark (LifeSciBench, covered in the 2026-06-18 brief), and named partnerships with the Allen Institute and HHMI. Jumper joining isn't a generic hire; it's Anthropic acquiring the single most credentialed individual in AI-for-biology, directly ahead of its IPO roadshow.

**Why it matters**
For builders, the signal isn't the headline — it's the pattern. Frontier labs are now in a phase where the scarcest resource is not compute or data but specific individuals who hold tacit knowledge of how a particular architecture class actually works (Shazeer on transformers/MoE pretraining internals, Jumper on adapting attention-based architectures to physical/biological structure prediction). Both moves happened in the same week each lab is trying to maximize its growth story for public investors. Watch for Anthropic's life-sciences agent work (Claude + Allen Institute/HHMI) to accelerate noticeably over the next 1–2 quarters now that it has Jumper, and watch OpenAI's pretraining roadmap for any architectural moves that look like Shazeer's prior work (MoE routing, multi-query attention variants).

**What to update in your mental model**
"Frontier AI talent" is no longer a single fungible pool — labs are now recruiting for specific sub-specialties (pretraining architecture, domain-adapted scientific modeling) the way sports teams recruit position players. If you're tracking competitive dynamics between labs, track named researchers and their specialty, not just headcount or funding.

---

### [AutoJack: Microsoft discloses RCE chain that uses an AI browsing agent to cross the localhost trust boundary](https://www.microsoft.com/en-us/security/blog/2026/06/18/autojack-single-page-rce-host-running-ai-agent/) · High
*Published: 2026-06-18*

**What changed**
Microsoft's Defender Security Research Team disclosed "AutoJack," a three-vulnerability exploit chain in AutoGen Studio (Microsoft Research's open-source UI for prototyping multi-agent systems) that lets a single malicious webpage achieve remote code execution on the developer's machine — with no credentials and no user interaction beyond the agent rendering the page. The chain was identified and fixed during development; it never shipped in a published PyPI release, so users who installed via `pip` were never exposed. The fix landed in commit `b047730` on the AutoGen `main` branch.

**How it works**
Three independent weaknesses chain together. First, AutoGen Studio's MCP WebSocket only accepts connections from `Origin: localhost` — a reasonable defense against a human's browser visiting a malicious site, but meaningless against an agent's own headless browser, which *is* localhost and inherits that trust regardless of what page it's rendering. Second, the app's auth middleware explicitly skipped `/api/mcp/*` paths on the assumption the MCP handler would do its own authentication — it never did, so the MCP WebSocket accepted unauthenticated connections under every configured auth mode (none, GitHub, MSAL, Firebase all failed identically). Third, the WebSocket endpoint read a `server_params` query parameter, base64-decoded it into `StdioServerParams`, and passed the resulting `command`/`args` straight to `stdio_client()` with no allowlist — so `calc.exe`, `powershell.exe -enc ...`, or `bash -c '...'` were all accepted as valid "MCP servers." An attacker's page just needs JavaScript that opens `ws://localhost:8081/api/mcp/ws/?server_params=<payload>`; if the agent ever browses to that page (via direct prompt, prompt injection in earlier content, or a URL field), the host process spawns the attacker's command under the developer's account.

**Why it matters**
This is the concrete pattern the last several briefs' "MCP provenance gap" and "agentjacking" entries predicted: any agent that can both browse untrusted content *and* talk to a privileged local control plane dissolves localhost as a trust boundary, because the agent's browsing tool inherits the localhost identity. AutoJack is architecturally distinct from the earlier Sentry MCP "agentjacking" disclosure (data-provenance confusion — fake bug reports treated as trusted tool output) — this is missing authentication plus unsanitized command execution on a local control-plane socket. Same root cause class (no standard trust model for what an agent's tool calls are allowed to reach), different specific bug. Microsoft frames the pattern as generalizable: "a development tool exposes a powerful local control plane → that control plane assumes localhost-only access is sufficient authentication → the user runs an agent that browses untrusted content on the same machine." Expect more disclosures with this exact shape across other agent frameworks (LangGraph, CrewAI, custom internal tooling) as researchers go looking for it.

**What to update in your mental model**
If you run any agent framework locally with both browsing/code-execution tools *and* an MCP server bound to localhost, audit whether that MCP endpoint actually authenticates every connection — "it's only reachable from localhost" is not a security boundary once an agent with a headless browser is running on that same host. Concretely: bind control-plane sockets to loopback *and* require a token; never accept executable parameters from a URL/query string without an allowlist; treat any tool parameter reachable from model output, indirect prompt injection, or rendered web content as attacker-controlled.

---

### [Anthropic ships Enterprise-Managed Authorization for MCP connectors, starting with Okta](https://claude.com/blog/enterprise-managed-auth) · Medium-High
*Published: 2026-06-18*

**What changed**
Anthropic launched Enterprise-Managed Authorization (EMA) for Claude's MCP connectors. IT admins authorize a connector once at the org level through their identity provider (Okta at launch); every employee then inherits access automatically on first login through their existing IdP groups/roles, with no individual OAuth consent screen. Supported connectors at launch: Asana, Atlassian, Canva, Figma, Granola, Linear, and Supabase, with Slack "coming soon." Centralized audit and policy now live in the IdP admin console rather than being scattered across each user's individual connector grants.

**How it works**
Standard MCP connector auth today is per-user: each employee individually OAuths into each third-party tool (Asana, Figma, etc.) the first time they use it inside Claude, and IT has no central visibility into who granted what. EMA inverts this: the admin pre-authorizes the connector at the organization level, and individual user access becomes a function of IdP group membership rather than a one-off consent click. This is architecturally the same pattern as SSO/SCIM for SaaS apps generally, applied specifically to the MCP connector-authorization step. The companion post on the Model Context Protocol blog frames this as closing what the MCP community had identified as the single largest blocker to enterprise-scale MCP adoption: security teams could not centrally govern per-user OAuth grants at scale.

**Why it matters**
This is a direct, dated counterpoint to the AutoJack story above: one development this window is agents creating new unauthenticated attack surface (AutoJack), and the other is a lab proactively closing an authentication/governance gap at the connector level (EMA). Together they sketch the actual shape of the current MCP security problem: individual connectors and control planes are being hardened piecemeal (auth at the connector-grant layer here, auth at the control-plane-socket layer in AutoJack) rather than via any single MCP-spec-level trust primitive. For builders deploying MCP connectors at company scale, EMA removes a real adoption blocker — but note it's currently Okta-only (Azure AD, Google Workspace "on the roadmap"), and it governs *connector-level* access, not the deeper tool-output-provenance gap (still no standard exists for an agent to distinguish first-party application state from externally-writable data surfaced through a connector — see the Sentry "agentjacking" entry in the knowledge base).

**What to update in your mental model**
"MCP connector authentication" and "MCP tool-output trust/provenance" are two separate unsolved problems, and they're being solved on different timelines by different parties. EMA addresses the former (who can authorize a connector, and how) but does nothing for the latter (once authorized, can the agent tell trusted application data from attacker-injected content flowing through that same connector). Don't conflate "connector access is now centrally governed" with "tool output from that connector is now trustworthy."

---

## Agentic Architecture & Engineering

**Affected stack:** `User → Planner → Memory → Retriever → LLM → Tools → Verifier → Output`

**AutoJack and EMA both land at `Tools`** — but at different sub-layers. AutoJack is a failure at the *control-plane authentication* sub-layer (the socket an agent's tool calls reach should authenticate regardless of apparent origin). EMA is a fix at the *connector-grant* sub-layer (who is allowed to authorize a connector for an organization, and how individual users inherit that grant). Neither touches the still-unsolved *tool-output provenance* sub-layer (can the agent tell first-party data from externally-injected data once a connector call returns). **Build implication**: if you're building or auditing an agent's tool-calling surface, separate these three sub-layers explicitly — connector authorization, control-plane authentication, and output provenance — because they require different controls and are being solved independently across the ecosystem, not as one unified "MCP security" fix.

**Claude Code's June 19 changelog adds destructive-action guardrails to auto mode** — Anthropic's Claude Code update (version released 2026-06-19) blocks `git reset --hard`, `git checkout -- .`, `git clean -fd`, and `git stash drop` when the user didn't explicitly ask to discard local work; blocks `git commit --amend` when the commit wasn't made by the agent in the current session; and blocks `terraform destroy`/`pulumi destroy`/`cdk destroy` unless the specific stack was requested. This is a narrow, concrete instance of the broader "agent reliability" problem: rather than a general verifier/critic loop, Anthropic is hard-coding an allowlist/denylist of specific destructive commands that require explicit confirmation regardless of what the agent's own reasoning concluded. **Build implication**: adopt the pattern, not just the tool — if you're building any agentic coding tool with shell/infra access, maintain an explicit denylist of irreversible commands gated on "did the user/agent's own prior action create the state being destroyed," independent of whatever planning or verification logic the agent itself runs. Models can be talked into bad tool calls; a hard-coded guardrail at the tool-execution layer can't.

---

## Wider World

**EU selects EUROPA consortium to build a sovereign 400B+ parameter open-source frontier model (2026-06-19)** — The European Commission named EUROPA (led by Italian AI company Domyn) as the winner of its Frontier AI Grand Challenge, launched in February 2026. The mandate: an open-source model exceeding 400B parameters, natively covering all 24 official EU languages, built on European infrastructure. The winning project gets access to up to 2.5% of total EuroHPC JU compute capacity for one year, plus services from EU "AI Factories." This is explicitly a sovereignty play — the Commission's framing is about ensuring European businesses, researchers, and public institutions aren't permanently dependent on US or Chinese frontier labs for advanced AI access, not about leading on raw capability. Watch for whether a 400B-parameter dense or MoE model trained primarily for multilingual EU-language coverage produces benchmarks comparable to the open-weight frontier (GLM-5.2, MiniMax M3) on English-centric coding/reasoning evals, or whether it trades raw capability for language coverage and regulatory/data-residency guarantees that matter more to EU enterprise buyers than leaderboard position.

**Talent migration ahead of dual IPOs** — covered in Top Signals above. Background note: this is the second and third major lab-to-lab departure disclosed this month (following the SpaceX/Cursor consolidation and ongoing Fable/Mythos export-control story from prior briefs), reinforcing that 2026's AI competitive dynamics are being shaped as much by personnel and corporate-structure moves as by model releases.

---

## Deep Dive: AutoJack — Why "Localhost" Stopped Being a Trust Boundary

**What problem it attacks**
Developer tools have relied on a decades-old assumption: a service that only accepts connections from `127.0.0.1`/`localhost` is safe, because reaching it requires code already running on that machine — and code already running on your machine is (the assumption goes) code you trust. AutoJack shows this assumption breaks the moment an AI agent on that machine can be steered, by content it merely *renders*, into making the very localhost connection the developer never intended.

**Core mechanism**
The chain has three independent links, each individually a "reasonable shortcut in a research-grade prototype" (Microsoft's own framing), but lethal in combination:

1. **Origin-allowlist defeated by the agent's own browser.** AutoGen Studio's MCP WebSocket checks that the connecting `Origin` header is `http://127.0.0.1` or `http://localhost`. That stops a human's browser from being redirected to the local service by a malicious site (classic cross-site WebSocket hijacking defense). It does not stop a *headless browser the agent itself controls* from making that exact connection — because that headless browser is a process on the workstation, and any JavaScript it executes (including JavaScript served by an attacker's page the agent merely navigated to) inherits the localhost identity when it opens a WebSocket.

2. **Auth middleware that assumed someone else would check.** The app's auth middleware explicitly excluded `/api/mcp/*` from its checks, with a code comment indicating the MCP handler was expected to authenticate independently. It never did. The result: regardless of which auth mode (`none`, `github`, `msal`, `firebase`) the operator configured for the rest of the app, the MCP WebSocket itself accepted unauthenticated connections in every case.

3. **A URL parameter became a command line.** The WebSocket endpoint read a base64-encoded `server_params` parameter from the URL, decoded it into a `StdioServerParams` object, and passed `command`/`args` straight into `stdio_client()` — the function that spawns the actual MCP server subprocess. There was no allowlist restricting which executables this could invoke. Any string was accepted as a "command."

**Before vs. after**
*Before (the vulnerable design)*: developer builds an AutoGen agent with web-browsing tooling on their laptop → agent browses to a URL (direct request, prompt injection, or planted link) → attacker's page JavaScript opens a WebSocket to the local MCP endpoint with a malicious base64 payload → AutoGen Studio decodes it and spawns the attacker's process under the developer's account, no credentials or further interaction required.

*After (the fix, commit `b047730`)*: `server_params` are no longer accepted from the URL at all — a separate authenticated `POST` endpoint stores parameters server-side keyed by a UUID, and the WebSocket handler only accepts a session ID, refusing unknown IDs. The MCP path was removed from the auth-middleware skip-list, so it now flows through normal authentication like every other route.

**Strengths of the disclosure**
Microsoft frames this explicitly as a pattern to recognize elsewhere, not just a bug to patch — they name the generalizable triangle (powerful local control plane + localhost-only assumption + an agent that browses untrusted content) and predict it will recur across the ecosystem. The fix was caught and shipped before any PyPI release, meaning the actual blast radius of this specific instance was limited to developers building from the `main` branch during a narrow window.

**Failure modes and what's still unresolved**
The deeper issue Microsoft's writeup surfaces but doesn't solve: there is still no standard for agent identity that's distinct from developer identity. The recommended mitigation — "separate the agent's browsing identity from the developer's identity (different OS user, container, or VM)" — is sound but entirely manual; no agent framework ships this separation by default today. Until that changes, every new agent framework that adds both browsing tools and a local control plane is a candidate for the same chain under a different name.

**So what for builders**
If you operate any agent framework with local MCP/control-plane sockets: authenticate every control-plane endpoint regardless of apparent origin, allowlist which executables can be spawned as "MCP servers" rather than accepting arbitrary command/args, and run agent processes under a separate, lower-privilege identity from your own developer account. If you're evaluating which agent framework to standardize on internally, ask the vendor directly whether their local control-plane endpoints authenticate independent of origin checks — AutoJack shows that's not a given even from a major research lab's own prototyping tool.

---

## Small Finds

- **Claude Code (2026-06-19) blocks destructive git/infra commands in auto mode by default** — see Agentic Architecture section above. A concrete, narrow agent-reliability fix worth adopting as a pattern in any tool with shell/infra access.
- **EU's EUROPA consortium (Domyn-led) selected to build a 400B+ parameter, 24-language sovereign open model** — policy/infrastructure story, not yet a capability result; watch for training progress updates over coming months.
- **Anthropic's life-sciences push gets its most credentialed hire yet** — John Jumper's move to Anthropic (see Top Signals) directly follows the Allen Institute/HHMI partnerships and February's wet-lab buildout; expect concrete agent-in-biology output to accelerate.

---

## Frontier Direction

- **Bottleneck under attack**: enterprise MCP adoption friction (per-user OAuth at scale) — Anthropic's EMA is a direct fix, though scoped to connector-grant governance only.
- **Broader trend**: the "MCP needs a trust layer" thesis flagged across several recent briefs is now visibly bifurcating into separately-solved sub-problems (connector authorization, control-plane authentication, tool-output provenance) rather than one unified spec-level fix — see Agentic Architecture section.
- **Still unsolved**: agent identity distinct from developer/operator identity. AutoJack's root cause — an agent's browsing tool inherits the human operator's localhost trust — has no standard mitigation; "run it in a separate container" is advice, not infrastructure.
- **Emerging paradigm**: frontier capability competition is now visibly a personnel-acquisition race tied to specific architectural sub-specialties (pretraining internals, domain-adapted attention architectures) rather than a generic "more researchers" race, timed to coincide with IPO positioning.

Arrows:
- Localhost-as-trust-boundary → localhost-is-an-attack-surface-once-an-agent-browses-untrusted-content
- Per-user OAuth friction blocking enterprise MCP adoption → admin-provisioned, IdP-inherited connector access
- Generic AI talent competition → specialty-specific talent acquisition (pretraining architects, domain-science architects) ahead of IPO milestones
- Single unified "MCP security" framing → three distinct, independently-solved sub-problems (authorization, authentication, provenance)

---

## Builder Takeaways

### Try now
**Audit your own local agent tooling against the AutoJack pattern today.** If you run any agent (AutoGen, LangGraph, a custom framework) with both web-browsing/code-execution tools and a local MCP or control-plane socket on the same machine, check: (1) does that socket authenticate regardless of `Origin`, (2) is there an allowlist on what can be spawned as a subprocess, (3) is the agent's process identity distinct from your own login. This is a 30-minute audit with a concrete, recently-demonstrated exploit class as the motivation.

### Experiment with
**Prototype agent-identity isolation.** Microsoft's stated mitigation for AutoJack-class issues — running an agent's browsing/execution identity under a separate OS user, container, or VM from the developer's own account — is sound advice with no off-the-shelf tooling. Build a minimal version: a container image that runs an agent framework (AutoGen, your own harness) under a non-privileged user, with explicit volume mounts for only the files the agent needs, and measure what breaks versus what's actually unnecessary access the agent had been silently relying on.

### Go deep on
**MCP's three unsolved trust sub-problems** — authorization (who can grant a connector, addressed partially by EMA), authentication (does the control-plane endpoint verify every caller, the gap AutoJack exploited), and provenance (can the agent tell first-party from externally-injected data once a connector call returns, the gap underlying the earlier Sentry "agentjacking" disclosure). Understanding which sub-problem a given fix addresses — and which two it doesn't — is now a core skill for anyone deploying agents with real tool access. Study: the MCP spec's current auth extensions, Anthropic's EMA documentation, and the AutoJack and Sentry agentjacking writeups side by side; map each onto the three-layer model above.

### Ignore for now
**Treating EUROPA as a near-term capability competitor.** The project just selected its lead consortium; training hasn't started, and "400B+ parameters, 24 languages" is a mandate, not a benchmark result. Revisit when there's an actual model or even a training-progress disclosure — not before.

---

## What to Build

**Project:** A local-agent security linter that scans an agent framework's codebase (or a running instance's open ports/sockets) for the AutoJack pattern: localhost-bound control-plane endpoints that skip authentication, accept executable parameters from request data, or rely on `Origin` checks as their only access control.
**Why now:** AutoJack is a concrete, recently-disclosed, well-documented instance of a pattern Microsoft explicitly predicts will recur across other agent frameworks. A linter that checks for these three specific anti-patterns (origin-only auth, middleware skip-lists for WS/MCP paths, unsanitized command construction from request parameters) would be immediately useful and is scoped tightly enough to build in a weekend.
**Stack:** Static analysis (AST parsing for Python/Node agent frameworks) to flag missing-auth middleware patterns and unsanitized subprocess calls; a runtime component that probes open localhost ports for missing-auth WebSocket/HTTP control planes. Test against the documented AutoJack chain in AutoGen Studio's pre-fix commit as a known-positive case.
**What you'd learn:** How agent framework control planes are actually wired (auth middleware ordering, WebSocket handshake authentication, subprocess parameter handling) — directly transferable to building or auditing any agent system with local tool access.

**Project:** A reference implementation of agent-identity isolation as a reusable container/sandbox template (the gap Microsoft's AutoJack mitigation guidance names but doesn't provide tooling for).
**Why now:** "Run the agent under a separate identity" is now explicit, named guidance from a major lab's security research team with no corresponding off-the-shelf tool. Building the reference implementation now means shipping into a gap that's been freshly, explicitly identified rather than guessing at demand.
**Stack:** A minimal container/VM template (Docker or Firecracker-based) with a non-privileged user, explicit allowlisted file-system mounts, network egress restricted to declared domains, and no access to the host's other local sockets/ports by default. Benchmark against AutoGen Studio and at least one other agent framework to confirm broad applicability.
**What you'd learn:** Practical sandboxing/isolation techniques for agentic systems, and where the real friction points are when an agent's normal operation legitimately needs broader access than a naive sandbox would allow — this tension is the actual hard problem, not the sandboxing mechanics themselves.

---

## Opportunities

- **Agent-framework security linting-as-a-service**: AutoJack is unlikely to be the last disclosure of this exact pattern. A continuously-updated scanner (static + runtime) that checks popular agent frameworks (AutoGen, LangGraph, CrewAI, custom MCP servers) against the growing catalog of agent-specific vulnerability classes (agentjacking-style provenance confusion, AutoJack-style control-plane auth gaps, prompt-injection-driven tool misuse) is a defensible, immediately useful product with a clear and growing reference catalog of real disclosed exploits to validate against.

- **Enterprise MCP governance beyond Okta**: Anthropic's EMA launches Okta-only, with Azure AD and Google Workspace "on the roadmap" and unspecified timeline. A third-party governance layer that sits in front of MCP connectors and works across whatever IdP an enterprise already uses (rather than waiting for each connector vendor to build first-party IdP integrations one at a time) addresses the same adoption blocker with broader and faster IdP coverage.

- **Domain-specialist AI talent intelligence**: the Jumper/Shazeer moves underscore that competitive signal in frontier AI increasingly lives in *which specific researchers* move where, not just funding or model benchmarks. A tracking product mapping named researchers to their technical sub-specialty and lab affiliation over time — essentially a talent-flow analog to a patent-citation graph — would give builders and investors a genuinely differentiated signal that current funding-round or benchmark-leaderboard tracking doesn't capture.

---

*Sources:*
- [TechTimes — Transformer Architect Behind Gemini Jumps to OpenAI After Google Spent $2.7B](https://www.techtimes.com/articles/318613/20260618/transformer-architect-behind-gemini-jumps-openai-after-google-spent-27b.htm)
- [CNBC — John Jumper to leave Google DeepMind for Anthropic](https://www.cnbc.com/2026/06/19/john-jumper-to-leave-google-deepmind-for-anthropic.html)
- [GuruFocus — John Jumper Leaves Google DeepMind for Anthropic Amid AI Talent Wars](https://www.gurufocus.com/news/8924392/john-jumper-leaves-google-deepmind-for-anthropic-amid-ai-talent-wars)
- [Bloomberg — Nobel Winner John Jumper to Leave Google DeepMind for Anthropic](https://www.bloomberg.com/news/articles/2026-06-19/nobel-winner-john-jumper-to-leave-google-deepmind-for-anthropic)
- [Yahoo Finance — Top AI researcher leaves Google for OpenAI](https://finance.yahoo.com/technology/ai/articles/top-ai-researcher-leaves-google-141950344.html)
- [Microsoft Security Blog — AutoJack: How a single page can RCE the host running your AI agent](https://www.microsoft.com/en-us/security/blog/2026/06/18/autojack-single-page-rce-host-running-ai-agent/)
- [The Hacker News — AutoJack Attack Lets One Web Page Hijack AI Agent for Host Code Execution](https://thehackernews.com/2026/06/autojack-attack-lets-one-web-page.html)
- [CSO Online — Microsoft says web-enabled AI agents can trigger host-level RCE](https://www.csoonline.com/article/4187155/microsoft-says-web-enabled-ai-agents-can-trigger-host-level-rce.html)
- [Claude — Centrally manage authorization for MCP connectors](https://claude.com/blog/enterprise-managed-auth)
- [Model Context Protocol Blog — Enterprise-Managed Authorization: Zero-touch OAuth for MCP](https://blog.modelcontextprotocol.io/posts/enterprise-managed-auth/)
- [TechTimes — Claude MCP Connectors Now Provision Through Okta](https://www.techtimes.com/articles/318704/20260619/claude-mcp-connectors-now-provision-through-okta-employees-inherit-access-login.htm)
- [Okta Newsroom — Okta becomes a featured identity provider powering secure AI agent connections for Claude](https://www.okta.com/en-au/newsroom/press-releases/okta-becomes-a-featured-identity-provider-powering-secure-ai-agent-connections-for-claude-enterprise/)
- [European Commission — Commission selects EUROPA consortium as winner of the Frontier AI Grande Challenge](https://digital-strategy.ec.europa.eu/en/news/commission-selects-europa-consortium-winner-frontier-ai-grande-challenge-project-build-european)
- [European Express — Europe Chooses Its Own Frontier AI Builder](https://www.european.express/2026/06/19/europe-chooses-its-own-frontier-ai-builder/)
- [Claude Code Changelog](https://code.claude.com/docs/en/changelog)
