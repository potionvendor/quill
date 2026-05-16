# AuthorChain — Discovery Context Document
### Gaps, Assumptions, Risks, and How to Attack This Problem
*Version 1.0 — May 2026*

---

## Purpose of This Document

This document is the honest reckoning. Before writing a line of production code, we need to surface what we don't know, what we're assuming, and where this can fail. This is the working document for the discovery phase. It should be updated as assumptions are validated or invalidated.

---

## Critical Gaps (Things We Do Not Know Yet)

### 1. The Legal Standard Is Still Fuzzy
**Gap:** The Copyright Office has not defined a bright line for "sufficient human contribution." The *Allen v. Perlmutter* case (600+ prompts used to refine AI-generated art) is still pending and could shift the bar. We are building toward a legal standard that is directionally clear but not precisely specified.

**Implication:** We cannot guarantee that what AuthorChain captures will meet the threshold in every case. We are building the best available evidence system, not a guarantee of protection.

**What to do:** Engage 2–3 IP attorneys specializing in software copyright to advise on what a "defensible" authorship record looks like in practice. This shapes the data model.

---

### 2. Developer Workflow Integration is Unvalidated
**Gap:** We don't know how much friction developers will tolerate, or how much passive capture is actually possible across the fragmented IDE/AI tool ecosystem. Copilot, Cursor, Claude Code, and Windsurf all have different APIs, extension models, and data access levels.

**Implication:** The "passive capture" design principle may require active developer participation in ways we haven't accounted for.

**What to do:** Prototype in one IDE (VS Code) with one AI tool (GitHub Copilot via its extension API) before assuming portability. Validate the passive capture model before committing to it architecturally.

---

### 3. The Buyer / User Disconnect
**Gap:** The pain is felt by GCs and IP counsel (buyers). The product lives in the developer workflow (users). These two populations have different incentives, different vocabularies, and rarely coordinate proactively on tooling decisions.

**Implication:** Standard PLG (product-led growth) may not work here. We may need a top-down enterprise sale that mandates developer adoption — which is harder and slower.

**What to do:** During discovery interviews, explicitly map the internal org chart at target companies. Who currently owns AI coding policy? Is it legal, engineering, security, or nobody? The answer shapes GTM.

---

### 4. We Don't Know the Actual Pain Intensity Yet
**Gap:** The legal risk is real, but we don't know how viscerally companies are feeling it today vs. treating it as a background concern. Pain becomes acute at M&A/diligence events — but those are episodic, not continuous.

**Implication:** Selling this as continuous SaaS may be hard if companies only feel the pain at deal moments. The product may need to serve both the ongoing workflow and the episodic diligence event.

**What to do:** First 10 discovery calls should explicitly probe: "Have you had a deal, fundraise, or audit where AI code provenance was raised? What happened?" If the answer is consistently "not yet," urgency is thinner than the legal landscape suggests.

---

### 5. What Exactly Goes in the Authorship Artifact?
**Gap:** We have a directional sense of what should be captured (prompts, edits, decisions, timestamps). We do not have a validated data schema or output format that legal/IP teams would actually accept and use.

**Implication:** We could build a logging system that produces an artifact nobody trusts or knows how to use.

**What to do:** Before building the capture layer, design the output artifact first. Work backward from what a copyright registration attorney would need, what an M&A diligence team would want, and what the Copyright Office's disclosure requirements actually specify. Validate the artifact format with legal professionals before building the pipeline that generates it.

---

## Key Assumptions (Ordered by Risk)

| # | Assumption | Risk if Wrong | How to Validate |
|---|---|---|---|
| 1 | Companies will pay for continuous authorship logging, not just point-in-time reports | Product becomes a services business, not SaaS | Discovery interviews + pricing test |
| 2 | Passive capture is technically feasible across major AI coding tools | Core design principle fails; UX becomes burdensome | Technical spike in VS Code + Copilot |
| 3 | GCs/IP counsel are already aware of and worried about this problem | Sales cycle is 2x longer; must build the category first | First 10 discovery calls |
| 4 | The authorship artifact we produce will be accepted as credible by legal/IP teams | Product produces output nobody trusts | Legal advisor input on artifact design |
| 5 | No large player (GitHub, Cursor) will absorb this feature in the next 18 months | Market closed before we get to scale | Competitive monitoring; build moat in depth of legal integration |
| 6 | The legal standard will remain fuzzy long enough for us to establish a market position | Standard clarifies, reducing urgency | Monitor *Allen v. Perlmutter* and Copyright Office guidance |
| 7 | Developers will not actively resist or route around the tool | Adoption fails inside the team that matters most | Developer interviews separate from buyer interviews |

---

## Risk Register

### High Severity

**Legal standard shifts unfavorably.** If courts or Congress create broad safe harbors for AI-generated works (or new sui generis rights), the urgency for authorship documentation drops. Timeline: 3–5 year risk. Mitigation: move fast; establish market position before this resolves.

**GitHub/Microsoft ships this natively.** Copilot Enterprise already has IP indemnification. Provenance tracking is a natural next feature. If they ship it, the TAM narrows significantly. Mitigation: compete on depth of legal integration and artifact quality, not just logging. Become the "gold standard" that legal teams trust.

**Developers hate it and route around it.** If the tool creates friction or feels like surveillance, adoption inside engineering teams fails. Buyers mandate; users resist. This is the death spiral for any developer tool. Mitigation: developer experience is a first-class product value, not an afterthought. Ship to developers first, validate they'll tolerate it before selling to legal buyers.

### Medium Severity

**M&A / fundraising market softens.** If the volume of deals requiring IP diligence drops, one of our key acute pain triggers weakens. Mitigation: build toward the compliance use case (EU AI Act) as the parallel forcing function.

**EU AI Act enforcement is delayed or weakened.** Article 50 (August 2026) is the strongest near-term regulatory trigger. If enforcement is soft or delayed, compliance urgency decreases. Mitigation: US-facing customers have enough non-regulatory drivers; treat EU as accelerant, not foundation.

**Open source license contamination tools (Black Duck, FOSSA) expand into authorship.** Adjacent players could reframe their scan products to address authorship. Mitigation: they are backward-looking (scanning existing code); we are forward-looking (capturing during creation). Different motion, different data model.

### Low Severity (Monitor)

**Privacy / surveillance concerns from developers.** Capturing prompts and edit histories raises questions about what happens to that data, especially prompts that may contain proprietary business logic. Mitigation: on-premise / self-hosted option; clear data handling policies; no cloud transmission of sensitive prompts without explicit opt-in.

---

## How to Attack This: Discovery Phase Plan

### Phase 0 — Legal Foundation (Weeks 1–2)
- Engage 2–3 IP attorneys to define the authorship artifact requirements
- Map the Copyright Office's current disclosure requirements for AI-assisted works
- Map EU AI Act Article 50 technical requirements
- Output: **Authorship Artifact Specification v0.1**

### Phase 1 — Customer Discovery (Weeks 2–5)
- 10 interviews with GCs / IP counsel at Series B+ software companies
- 10 interviews with senior developers / engineering leads at same companies
- Questions to answer: Is the pain real and felt? Who owns the problem internally? What would they pay? What would make developers adopt this?
- Output: **Discovery Findings Report** + validated/invalidated assumptions table

### Phase 2 — Technical Spike (Weeks 3–6, parallel)
- Can we passively capture prompt + edit delta in VS Code + GitHub Copilot?
- What does the data look like? Is it structurable into a legally-oriented format?
- What are the API limitations of each major AI coding tool?
- Output: **Technical Feasibility Report** + prototype data model

### Phase 3 — Artifact Validation (Weeks 6–8)
- Show the Authorship Artifact v0.1 to 5 IP attorneys
- "Would you present this as supporting evidence for a copyright claim?"
- "What's missing? What would you never use?"
- Output: **Validated Artifact Specification v1.0**

### Phase 4 — MVP Definition (Week 8–10)
- Combine legal requirements, customer discovery findings, and technical constraints
- Define the smallest thing we can build that delivers real value
- Output: **MVP Spec** + go/no-go decision point

---

## For Claude Code: Architecture Considerations

When we get to building, the key technical questions to resolve early:

**Data capture layer:**
- How do we hook into IDE events (file saves, copilot suggestion accept/reject, manual edits)?
- What's the data format for prompt capture across different AI tools?
- How do we create tamper-evident timestamps? (Cryptographic signing? Third-party timestamp authority?)

**Data storage:**
- On-premise vs. cloud? Many customers will require on-prem for IP sensitivity reasons.
- What's the retention model? How long does authorship evidence need to live?

**Artifact generation:**
- What format does the output artifact take? (Structured JSON? PDF report? Both?)
- How do we summarize thousands of micro-decisions into a human-readable authorship narrative?
- Can AI (irony noted) help synthesize the log into a coherent authorship statement?

**Integration surface:**
- VS Code extension (priority 1)
- JetBrains plugin (priority 2)
- Git hook layer (priority 3 — tool-agnostic fallback)
- CI/CD integration for org-wide policy enforcement (enterprise tier)

---

## The Decision We Need to Make First

Before anything else: **Do we design around the buyer (legal/GC) or the user (developer)?**

- Design for the buyer → enterprise sales motion, long cycles, high ACV, slow growth
- Design for the user → PLG motion, faster adoption, lower ACV, needs a path to legal buyer

The answer shapes everything: pricing, GTM, product surface, and what we build first. This is the first question discovery interviews must answer.
