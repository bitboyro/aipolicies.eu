# Council Transcript — eu-regulations MCP Server Pre-Launch Evaluation
**Date:** 2026-06-13  
**Session:** 5 advisors · 5 peer reviewers · 1 chairman synthesis

---

## Original Question
Evaluate the eu-regulations MCP server before launch. It is meant to help professionals use their agents for EU AI regulations navigation, citations, document creation etc.

## Framed Question
The `eu-regulations` MCP server is a free, read-only, no-auth HTTP MCP server built for professionals using AI agents (Claude Code, Claude Desktop, VS Code Copilot) to navigate EU regulations: EU AI Act (2024/1689), GDPR (2016/679), and Data Act (2023/2854). It exposes 12 tools: discovery (TOC, dimensions, document/node listing, KB health), semantic + full-text + dimension-guided search, batch node/document retrieval, and error reporting. The corpus has 3 documents indexed, 1427 graph edges, 36 dimensions (652+ domain-concept nodes), and a cross-regulation graph linking all three acts. A companion SKILL.md tells Claude when and how to use it, with a mandatory citation rule (every claim must cite EUR-Lex). Target users are compliance professionals, legal ops teams, and developers building regulatory AI agents. **Is this ready to launch? What are the critical gaps, risks, or opportunities before it goes public?**

---

## Advisor Responses

### The Contrarian

Three critical problems before this goes live.

**The citation rule is unenforceable and creates liability, not protection.** "Every claim must cite EUR-Lex" is a prompt instruction in a SKILL.md file. Claude can and will hallucinate citations, cite the wrong article, or paraphrase incorrectly while attaching a real-looking EUR-Lex URL. Compliance professionals who rely on this for actual regulatory work and get a wrong citation face real consequences. The server has no mechanism to verify that what Claude says matches what the tool returned. The citation rule gives false confidence.

**1427 edges and 36 dimensions sounds impressive until you ask: validated by whom?** The knowledge graph was built programmatically. Who confirmed the cross-regulation edges are legally meaningful? A dimension-mediated edge between GDPR Article 22 and AI Act Article 9 might be spurious pattern-matching. Legitimacy of cross-references needs legal validation. Lawyers using this for compliance work will find one bad connection and distrust the entire corpus.

**"No-auth, free, read-only" with no rate limiting is an operational timebomb.** The first time someone builds a scraper or a bulk embedding job against it, the server goes down. There's no mention of rate limits, abuse controls, or SLA. "Free" is fine; "unprotected" is not. When it's down, the SKILL.md tells agents to call it — they'll fail silently or with unhelpful errors.

The opportunity hiding in these gaps: a disclaimer layer that surfaces tool output verbatim alongside any synthesis, plus a confidence/provenance field per result. That's the difference between a toy and something compliance teams can actually cite in an audit.

Not ready. Fix the liability surface first.

---

### The First Principles Thinker

The question assumes "launch readiness" is the right frame. It isn't. The real question is: **what job does this server do, and does anything prevent it from doing that job?**

Strip away the feature list. What does a compliance professional actually need?

1. Trust that the information is current and authoritative
2. Ability to act on what they find (not just read it)
3. Confidence the tool won't silently mislead them

**Gap 1: Corpus staleness is existential, not operational.** You have 3 documents. The EU AI Act has implementing acts, delegated acts, and guidance trickling out continuously. A stale corpus doesn't just reduce utility — it creates liability. The mandatory EUR-Lex citation rule is defensive documentation, not a solution. When did these documents last sync? Is there a sync mechanism at all?

**Gap 2: The 12-tool surface area solves a problem users don't have.** Compliance professionals don't want to orchestrate 12 tools. They want answers. The complexity lives in your system, not theirs. This matters at launch because first impressions determine adoption patterns.

**Gap 3: "Read-only, no-auth" is a feature statement, not a trust statement.** Who maintains this? What's the SLA? When it returns wrong information (it will), what's the correction path? The SKILL.md citation rule implies Claude is responsible for accuracy. That's wrong. The server is responsible.

**The actual launch blocker:** You haven't defined what "wrong" looks like and how you'll catch it. Until you have a correctness monitoring mechanism, you're not launching a compliance tool — you're launching a confidence trap.

---

### The Expansionist

This is undervalued as a *platform*, not just a tool.

You're launching a free MCP server for three regulations. That's the seed. What you're actually sitting on is the infrastructure for a **living regulatory intelligence layer** — and if this works even modestly, the expansion path is enormous.

**The corpus is the moat, not the server.** 1427 edges and 652+ domain-concept nodes across three acts is non-trivial curation work. That graph is an asset. The question before launch isn't "is the server ready" — it's "how fast can you index Regulation (EU) 2024/903 (Chips Act), NIS2, CRA, DORA, DSA, DMA?" Every regulation you add increases the value exponentially because cross-regulation edges compound. Users don't want three regulations — they want *the EU regulatory graph*.

**The no-auth, free model is a distribution weapon.** Don't second-guess it. Every AI agent builder who integrates this becomes a dependent. Once you're in their agent stack, switching cost is high. This is land-and-expand at the infrastructure layer.

**The SKILL.md companion is underrated.** You're shipping a cognitive interface contract, not just an API. That's the pattern for how MCP servers become standards. Formalize it — version the SKILL.md alongside the server, publish a changelog, make it something agent builders can pin.

The critical opportunity before launch: **publish a public roadmap of which regulations are coming next.** Give compliance teams and agent builders a reason to bet on you. The server being ready is table stakes. The roadmap is what turns a tool into a platform.

---

### The Outsider

Looking at this purely from the outside, with no prior knowledge of the builder or their ecosystem:

The product description is written for someone who already knows what it is. "12 tools," "1427 graph edges," "652+ domain-concept nodes" — these numbers mean nothing to a compliance professional who just wants to know if the EU AI Act applies to their product. The framing is engineer-to-engineer, not engineer-to-buyer.

**The corpus gap is invisible.** You say 3 documents indexed. A first-time user has no way to know what's missing until they ask a question and get a confident-sounding answer about something you haven't indexed. That's not a small trust risk — that's a professional liability for the compliance officer citing your output to their board.

**"No-auth, free, read-only" sounds like a prototype.** That combination signals "not production-ready" to enterprise buyers, not "accessible." Legal ops teams expect rate limiting, SLAs, and audit trails. The absence of auth is a feature to a developer and a red flag to a compliance team's IT security.

**The citation rule is load-bearing but unverifiable.** SKILL.md mandates EUR-Lex citations, but nothing in the description says what happens when the MCP server returns a result that can't be cited, or when the citation is stale. If a regulation was amended after indexing, the citation rule creates false confidence rather than preventing it.

The biggest gap: there's no failure mode described anywhere. What does the system say when it doesn't know? Until that's answered, launching to compliance professionals is premature.

---

### The Executor

Three concrete things blocking Monday launch, ranked by how fast they'll hurt you:

**1. Rate limiting is not mentioned anywhere.** A free, no-auth HTTP endpoint with no rate limiting is a free DDOS target. Before you push the URL publicly, you need nginx/Traefik rate limiting in place. This takes two hours. Do it first.

**2. You have no error baseline.** The `report_error` tool collects errors from agents — but right now you have zero traffic, so you have no idea if the 12 tools actually work end-to-end under real agent usage. Before launch, run Claude Desktop against every single tool with adversarial inputs (empty queries, malformed node IDs, cross-regulation edge cases). Log every failure. Fix what breaks. This is a half-day of work.

**3. The SKILL.md citation rule has no enforcement path.** You say "every claim must cite EUR-Lex" but the MCP server can't enforce that — it's guidance only. If an agent ignores it and a compliance professional acts on an uncited claim, that's your reputational problem. Add EUR-Lex URLs directly into every tool response payload so the citation is automatic, not optional. That's a one-sprint fix but it's the difference between a professional tool and a liability.

What's actually ready: the corpus scope (3 acts, cross-regulation graph) is reasonable for a v1. The 12-tool surface area is coherent.

First step Monday: enable rate limiting on the Traefik route for the MCP server. Everything else can follow, but that one can't wait.

---

## Anonymization Mapping (for reference)
- Response A = The Expansionist  
- Response B = The Executor  
- Response C = The Contrarian  
- Response D = The Outsider  
- Response E = The First Principles Thinker  

---

## Peer Reviews

### Review 1
**Strongest: B (Executor)** — Only response with time-bounded, actionable fixes. Names a concrete first step for Monday. The others diagnose; B prescribes.

**Biggest blind spot: A (Expansionist)** — Pure strategy, ignores operational floor. Strategy is worthless if the v1 is a liability trap.

**What all five missed:** Corpus versioning. The regulations have version identifiers (OJ publication dates, corrigenda, amendment acts). If the server returns content from a superseded version of an article without surfacing that provenance, every other safeguard is undermined. A compliance professional citing Article 9 of the AI Act needs to know *which version* they are citing.

---

### Review 2
**Strongest: E (First Principles)** — Reframes around user jobs-to-be-done. "Corpus staleness is existential" and "confidence trap" are the most precise framings in the set.

**Biggest blind spot: A (Expansionist)** — Treats graph curation as a moat without asking whether the curation is correct. The entire "platform play" argument collapses if the first compliance professional finds a bad edge and publishes about it.

**What all five missed:** MCP transport layer. None asked what happens when Claude's MCP client misparses a tool response, retries a failed call, or silently drops a result mid-reasoning chain. A compliance professional's agent could silently skip a result and still produce a confident, EUR-Lex-cited answer.

---

### Review 3
**Strongest: E (First Principles)** — "Confidence trap" is the most precise diagnosis of the citation-rule problem. Subsumes the best points from C and D without their repetition.

**Biggest blind spot: A (Expansionist)** — Completely ignores every risk the others identified. Optimism without risk calibration is not strategy, it's cheerleading.

**What all five missed:** The MCP protocol itself is the trust boundary, not the SKILL.md. Tool results get silently summarized, truncated, or mixed with other context before reaching the user. The actual risk is laundered hallucination, not uncited responses.

---

### Review 4
**Strongest: E (First Principles)** — Forces the builder to answer what "wrong" looks like before shipping.

**Biggest blind spot: A (Expansionist)** — "Don't second-guess the no-auth model" without mentioning rate limits is dangerous advice that could accelerate a bad launch.

**What all five missed:** MCP tool schema quality. None examined whether the 12 tools are correctly typed, return consistent schemas, and degrade gracefully when called out of order or with missing context. A compliance professional never touches the MCP directly; an agent does. If the agent gets a malformed response or ambiguous tool description, the failure is invisible to the user and untraceable.

---

### Review 5
**Strongest: E (First Principles)** — The three-foundation test (trust, actionability, no silent misleading) is the most useful launch filter in the set. Correctly identifies that 12-tool complexity is a supply-side artifact, not a demand-side feature.

**Biggest blind spot: A (Expansionist)** — Entirely growth/positioning strategy with zero operational risk assessment. Optimism about distribution without acknowledging liability is the most dangerous blind spot in this set.

**What all five missed:** MCP protocol differences across clients. Claude Code, Claude Desktop, and VS Code Copilot each implement MCP slightly differently, update independently, and can break tool contracts silently. A free no-auth server with no versioning, no protocol compatibility matrix, and no client-specific testing is fragile against the very distribution channel it depends on. The SKILL.md companion is not a substitute for protocol stability testing across clients.

---

## Chairman Synthesis

### Where the Council Agrees

Five out of five advisors flagged the same three problems independently: the citation rule is unenforceable, rate limiting is absent, and the corpus is unvalidated. When every perspective — including the one explicitly assigned to find upside — converges on the same failure modes, those are real blockers, not matters of opinion.

The Executor, Contrarian, Outsider, and First Principles Thinker all said the same thing in different words: a compliance professional who acts on wrong, stale, or hallucinated-but-cited information faces real consequences, and the current architecture provides no protection against that. The SKILL.md citation rule is a disclaimer dressed up as a safeguard.

### Where the Council Clashes

The Expansionist argues the no-auth, free model is a distribution weapon and you should launch now and expand. The rest of the council argues that same model is an operational timebomb and a liability surface. Both are right — but they are not in tension. The Expansionist is describing a six-month outcome; the others are describing a day-one risk. The disagreement dissolves when you sequence it: fix the floor, then exploit the moat.

The other live tension: the Expansionist treats the 1427-edge knowledge graph as the moat. The Contrarian and First Principles Thinker treat it as the biggest unvalidated assumption. Who confirmed those cross-regulation edges are legally meaningful rather than pattern-matched? This is not a philosophical disagreement — it has a concrete answer, and the builder needs to produce it before launch.

### Blind Spots the Council Caught

**Corpus versioning and provenance.** The regulations have version identifiers — OJ publication dates, corrigenda, amendment acts. If the server returns content from a superseded article version without surfacing that, the EUR-Lex citation requirement becomes actively dangerous. A compliance professional citing "Article 9 of the AI Act" needs to know which version they are citing. This is not addressed anywhere in the current description.

**MCP transport layer as a silent failure mode.** None of the five advisors asked what happens when Claude's MCP client misparses a tool response, retries a failed call, or silently drops a result mid-reasoning chain. The actual risk is not uncited responses — it is laundered hallucination: an agent skips a result, fills the gap from training data, and still produces a confident, EUR-Lex-cited answer the user cannot distinguish from a real retrieval.

**MCP client compatibility fragmentation.** Claude Code, Claude Desktop, and VS Code Copilot each implement MCP differently and update independently. A free no-auth server with no versioning, no protocol compatibility matrix, and no client-specific testing is fragile against the exact distribution channel it depends on. Tool schema quality — correct typing, consistent response shapes, graceful degradation on out-of-order calls — determines whether real agents integrate cleanly or fail invisibly.

### The Recommendation

Not ready to launch to compliance professionals as the primary target user. Ready to launch to developers as a beta with explicit caveats.

The gap is not technical capability — the corpus scope, 12-tool surface area, and cross-regulation graph are credible for a v1. The gap is the trust infrastructure required to serve the stated user.

Launching to developers first is not a retreat — it is the correct sequencing. Developers will surface MCP client compatibility bugs, malformed tool responses, and edge case failures before a compliance professional ever sees them. That is free QA on the hardest problem.

Three things before any public launch:
1. Add corpus version metadata (OJ publication date, last sync timestamp) to every tool response payload — not as documentation, but as a returned field.
2. Enable rate limiting on the Traefik route. Two hours of work. No argument.
3. Run every tool against adversarial inputs across all three MCP clients (Claude Code, Claude Desktop, VS Code Copilot) and document what breaks. Publish the compatibility matrix.

The Expansionist is right that the roadmap is what turns a tool into a platform. Publish it. But publish it after the above — a roadmap attached to a liability is not an asset.

### The One Thing to Do First

Put the corpus version and last-sync timestamp into every single tool response payload as a returned field — not in documentation, not in SKILL.md, not as a README note. In the payload.

Everything else on this list — rate limiting, MCP client testing, edge validation, the disclaimer layer — is important. But if you do only one thing before any human being uses this for real regulatory work, make it impossible for the tool to return information without also returning when it was last verified. Every other safeguard depends on the user knowing whether what they are reading is current. Right now they cannot know that. That is the confidence trap. Close it first.
