# Quill — Working Plan
*Single source of truth for iteration sequencing, open questions, and working principles.*
*Last updated: May 2026*

---

## Current State

Discovery phase. No code exists yet. **Next step: Technical Spike — VS Code + Passive Capture Feasibility** (Iteration 2).

**What exists right now:**
- `PROJECT_DESCRIPTION.md` — product framing and how to work with Claude on this project
- `SEED_DOC.md` — problem, opportunity, competitive landscape, revenue model hypotheses
- `CONTEXT.md` — discovery gaps, assumptions ranked by risk, phased attack plan
- `ARTIFACT_SPEC.md` — Authorship Artifact Specification v0.1 (four-layer schema, JSON-LD + PDF format decision, 10 open questions for attorneys)
- `PLAN.md` — this document

**What does not exist yet:**
- Any code
- Attorney validation of the artifact spec
- Customer discovery interviews
- A technical spike on passive capture feasibility

---

## Open Questions

### Blocking

**[B1] Buyer vs. user design — UNDECIDED**
- *Question:* Do we design first for the legal/GC buyer (enterprise top-down motion) or the developer user (PLG motion)?
- *Why it matters:* This decision shapes pricing, GTM, product surface, and what we build first. It is the first question the CONTEXT.md identified, and it is still open.
- *Owner:* Founder decision — discovery interviews are the primary input, but we may be able to reason toward a working hypothesis before they happen
- *Decide before:* MVP Spec (Iteration 4 output)

**[B2] Authorship artifact format — DRAFT COMPLETE, ATTORNEY VALIDATION PENDING**
- *Decision so far:* Four-layer schema (session envelope, interaction log, human decision record, authorship summary). JSON-LD machine-readable + PDF human-readable. Required/recommended/excluded field breakdown complete. See `ARTIFACT_SPEC.md`.
- *Remaining:* IP attorney validation of the schema before the capture layer is built. 10 specific questions documented in `ARTIFACT_SPEC.md` Section 8.
- *Owner:* Validation (IP attorneys); Founder must initiate attorney engagement
- *Decide before:* Any production code on the capture layer

### Important

**[I1] IP attorney engagement**
- *Question:* Which 2–3 IP attorneys do we engage to validate the artifact spec and advise on what a "defensible" authorship record looks like in practice?
- *Why it matters:* The artifact spec (Iteration 1 output) is a research draft. It is not validated until attorneys with actual copyright registration and M&A diligence experience have reviewed it and told us what's missing or unusable.
- *Owner:* Founder (identifying and reaching attorneys); Claude can draft the ask and briefing materials
- *Decide before:* Iteration 3

**[I2] Passive capture technical feasibility**
- *Question:* Can we passively hook into IDE events and AI tool suggestion flows (accept/reject) across VS Code + Copilot without requiring active developer participation?
- *Why it matters:* "Passive by default" is a core design principle. If it's not technically feasible, the UX burden shifts significantly and the product motion changes.
- *Owner:* Research + build (Claude designs and runs the spike)
- *Decide before:* MVP Spec (Iteration 4 output)

**[I3] Claude Code extension API access**
- *Question:* What does Claude Code expose via its VS Code extension API that would allow us to capture prompt/response/accept events? (We are literally using Claude Code right now — this is both a research question and a dog-food opportunity.)
- *Why it matters:* Claude Code is a priority target for the capture layer. Understanding its API surface early shapes the generalizability of our approach.
- *Owner:* Research (Claude can investigate)
- *Decide before:* Iteration 2

### Monitor

**[M1] Allen v. Perlmutter / legal standard clarification**
- The "sufficient human contribution" threshold remains fuzzy. If it clarifies — in either direction — urgency shifts. Monitor for Copyright Office guidance and case outcomes.
- *Owner:* Research when relevant

**[M2] GitHub / Microsoft shipping native provenance tracking**
- Copilot Enterprise has IP indemnification. Provenance logging is a natural next feature. If they ship it, TAM narrows significantly.
- *Owner:* Competitive monitoring — revisit quarterly

**[M3] EU AI Act Article 50 enforcement (August 2026)**
- **Reframed (Iteration 1 finding):** Article 50 likely does not apply directly to AI-assisted code development. Scope is public-facing deception scenarios (deep fakes, AI-generated public-interest text, synthetic media) — not internal software development workflows. The compliance hook for Quill's customers is more accurately: US copyright registration requirements, M&A rep & warranty language, EU Cyber Resilience Act SBOM requirements, and sector-specific AI regulations. Quill aligns with regulatory direction of travel; it does not satisfy a specific Article 50 mandate for software development.
- *Owner:* Monitor; update product positioning language in `SEED_DOC.md` once attorney validation confirms this reading

**[M4] Developer privacy / surveillance concerns**
- Capturing prompts and edit histories raises questions about what happens to proprietary business logic in those prompts. On-prem / self-hosted option may be required for enterprise.
- *Owner:* Defer to MVP design — design the data model to support self-hosted from day one

---

## Queued Iterations

### ✅ COMPLETE: Iteration 1 — Authorship Artifact v0.1 Schema

### ⬇️ NEXT: Iteration 2 — Technical Spike: VS Code + Passive Capture Feasibility
**Theme:** Design the output before building the pipeline that generates it. Work backward from what copyright registration attorneys, M&A diligence teams, and the Copyright Office actually need.
**Scope:**
- Research Copyright Office disclosure requirements for AI-assisted works (current guidance, registration forms, stated expectations)
- Research what M&A diligence teams are currently asking about AI code provenance (what questions appear in IP schedules, what documentation is being requested)
- Research EU AI Act Article 50 technical specification for provenance disclosure
- Draft a structured artifact schema: what fields, in what format (JSON? structured report? both?), at what level of granularity
- Draft the human-readable summary layer: what gets generated from the raw schema for a non-technical legal audience
- Identify minimum required fields vs. nice-to-have vs. actively harmful (fields that overclaim and create legal liability)
- Flag open questions the artifact spec cannot resolve without attorney input

**Acceptance criteria:**
- Schema covers the four authorship evidence categories from the seed doc: what AI generated, what the human decided, what the human added, the sequence
- Format decision is explicit and justified (not just "JSON" but why JSON and what schema)
- Human-readable summary format is drafted alongside the structured schema — not deferred
- A clear list of "questions for IP attorneys" is appended so the Iteration 3 attorney briefing has a concrete agenda
- Document is formatted to match the style and depth of the existing project docs

**Output:** `ARTIFACT_SPEC.md` — Authorship Artifact Specification v0.1
**Blocking questions:** None — this is pure research and drafting
**Depends on:** Nothing — this is the foundation

---

**Theme:** Determine whether the passive capture model is technically feasible before committing to it architecturally. Prototype in one IDE with one AI tool first.
**Scope:**
- Map the VS Code extension API: what events fire during development, what data is accessible, what requires user permission
- Map the GitHub Copilot extension API: can we hook into suggestion accept/reject events? What prompt/response data is exposed?
- Map Claude Code's extension API surface (see [I3])
- Investigate Cursor and Windsurf API access models — document what's possible vs. closed
- Identify the minimum viable data capture flow: what does a single session's raw capture look like?
- Draft a prototype data model: how does raw capture data map to the artifact schema fields from Iteration 1?
- Document API limitations, closed surfaces, and where active developer participation would be required as a fallback

**Acceptance criteria:**
- Each major AI coding tool (Copilot, Cursor, Claude Code, Windsurf) has a documented feasibility verdict: passive capture possible / partial / not possible / unknown
- A prototype data model maps raw events to artifact schema fields — gaps are explicitly flagged
- The "passive by default" design principle is either confirmed or revised with a specific alternative
- If full passivity is not feasible, a concrete proposal for the minimum required developer action is documented

**Output:** `TECH_SPIKE.md` — Technical Feasibility Report + prototype data model
**Blocking questions:** [B2] must be resolved (artifact spec needed to know what we're capturing toward)
**Depends on:** Iteration 1

---

### Iteration 3 — Legal Foundation + Doc Sharpening
**Theme:** Sharpen the existing docs based on what we've learned, lay the groundwork for attorney engagement, and resolve gaps the first two iterations surface.
**Scope:**
- Update `CONTEXT.md` assumptions table based on Iteration 1 and 2 findings (which assumptions are now stronger or weaker?)
- Research EU AI Act Article 50 in full technical detail — what does machine-readable provenance disclosure actually require?
- Draft the IP attorney briefing package: a 1-page summary of the project + the artifact spec + the list of open questions from Iteration 1 that require legal input
- Identify specific IP attorney candidates (firms and individuals with AI copyright / M&A IP diligence experience)
- Sharpen `PROJECT_DESCRIPTION.md` if the artifact spec or tech spike changed the product framing
- Draft Quill's privacy commitment language for the data handling policy (captures prompts — what are we committing to about storage, transmission, access?)

**Acceptance criteria:**
- `CONTEXT.md` assumptions table is updated with current validation status for each assumption
- Attorney briefing package is ready to send — no placeholders
- Attorney candidate list has at least 5 names with firm, specialty, and why they're relevant
- Privacy commitment language exists in a draft form — not final, but not deferred

**Output:** Updated project docs + `ATTORNEY_BRIEF.md` + `PRIVACY_COMMITMENT_DRAFT.md`
**Blocking questions:** [I1] IP attorney identification (Founder task — Claude can draft the brief, Founder must identify and contact)
**Depends on:** Iterations 1 + 2

---

### Iteration 4 — Discovery Interview Guides
**Theme:** Write the interview guides for customer discovery, grounded in the artifact spec and technical feasibility findings. By this point we know what we're building — the interviews validate that someone will pay for it.
**Scope:**
- GC/IP counsel interview guide: ~10 questions, hypothesis-testing format, sequenced to build trust before probing pain
- Developer/engineering lead interview guide: ~10 questions, separate from the legal guide — different vocabulary, different concerns
- Screener criteria for finding the right interviewees (what does a qualified GC interviewee look like? what does a qualified developer interviewee look like?)
- Question ordering logic and interviewer notes (what follow-ups to probe, what to listen for, how to handle "we haven't thought about this yet")
- A shared synthesis template: what do we capture from each interview to enable cross-interview pattern matching?

**Acceptance criteria:**
- Both guides stand alone — an interviewer with no prior context on Quill could run them
- Each guide explicitly tests the core assumptions from `CONTEXT.md` (continuous SaaS vs. point-in-time, pain intensity, buyer/user dynamic)
- Developer guide includes explicit probes on friction tolerance and surveillance concern
- Screener criteria are specific enough to qualify or disqualify a candidate in under 5 minutes

**Output:** `INTERVIEW_GUIDE_LEGAL.md` + `INTERVIEW_GUIDE_DEVELOPER.md` + `INTERVIEW_SYNTHESIS_TEMPLATE.md`
**Blocking questions:** None — these can be drafted without attorney input, informed by what Iterations 1–3 surface
**Depends on:** Iterations 1–3 (so the questions are grounded in what we actually learned, not just what we assumed going in)

---

## Parking Lot
*Explicitly deferred. Not in scope until stated.*

| Item | Reason deferred | Revisit when |
|---|---|---|
| Any production code | Artifact spec and technical feasibility must be validated first | After Iteration 2 + attorney input |
| JetBrains plugin | Priority 2 after VS Code — don't build before VS Code is proven | After VS Code spike succeeds |
| Git hook layer (tool-agnostic fallback) | Priority 3 — useful if IDE hooks are infeasible | After Iteration 2 feasibility findings |
| CI/CD integration | Enterprise tier feature | After core capture layer is built |
| Pricing model validation | Requires knowing what we're selling | After MVP spec |
| Revenue model choice (SaaS vs. enterprise vs. compliance module) | Depends on buyer/user design decision [B1] | After [B1] is resolved |
| On-prem / self-hosted option | Required for enterprise IP sensitivity — design the data model to support it | Before first enterprise customer |
| AI-synthesized authorship narrative | Using AI to summarize the log into a coherent authorship statement | After artifact spec is validated |

---

## Working Principles

### What Claude decides autonomously
- Research findings and synthesis within a planned iteration
- Document structure and formatting consistent with the existing project docs
- Drafting artifacts (specs, guides, briefs) that don't require founder decisions to complete
- Flagging open questions that arise mid-iteration and noting them without blocking progress

### What Claude surfaces before acting
- Any change to the product framing in `PROJECT_DESCRIPTION.md` or `SEED_DOC.md` — these are the source of truth and changes need explicit agreement
- Scope additions mid-iteration — once an iteration starts, new ideas go into Queued Iterations or the Parking Lot
- Assumptions embedded in a draft that contradict the existing docs — surface the conflict, don't silently resolve it
- Anything that touches the buyer/user design question [B1] before it's decided — flag it, don't pre-decide it in the artifact

### What Claude always asks about
- Changes to the sequencing of Queued Iterations
- Moving something out of the Parking Lot
- Resolving a Blocking open question — this requires explicit founder decision, not Claude inference
- Any recommendation to engage a specific named person or firm externally
- Anything that would commit to a data model, schema, or API surface before attorney input is received

### Iteration discipline
- Each iteration has a named theme, explicit scope, and acceptance criteria written before work starts
- Scope is frozen when work starts — new ideas go into Queued Iterations or the Parking Lot
- An iteration's output document is committed to the repo when the iteration closes
- An iteration doesn't get queued until its Blocking questions are resolved or explicitly deferred with a stated reason
- Completed iterations get an honest summary: what was produced, what diverged from the plan, and why

### Open question handling
- **Blocking** — must be resolved before the next iteration that depends on it. Claude will not build past a blocking question without an explicit decision.
- **Important** — affects the next 2 iterations. Claude includes it in the session brief so the founder can weigh in before work starts.
- **Monitor** — real but not yet actionable. Claude tracks it and revisits when it becomes relevant.
- Each question has an owner: **Founder** (requires your judgment), **Research** (Claude can investigate and recommend), or **Defer** (wait for later learnings).

---

## Iteration Log
*What was produced, in order. Reference only — look up, not forward.*

### Iteration 0 — Project Initialization (May 2026)
**Produced:** `PROJECT_DESCRIPTION.md`, `SEED_DOC.md`, `CONTEXT.md` — product framing, problem/opportunity thesis, discovery gaps and risk register. Git repo initialized and pushed to GitHub. `PLAN.md` drafted.
**Divergences from plan:** N/A — initial setup.
**Commit:** `3680b24`, `280a6c5`

### Iteration 1 — Authorship Artifact v0.1 Schema (May 2026)
**Produced:** `ARTIFACT_SPEC.md` — four-layer QAR schema (session envelope, interaction log, human decision record, authorship summary), JSON-LD + PDF format decision, required/recommended/excluded field breakdown, 10 open questions for IP attorney validation.
**Divergences from plan:** One significant finding outside original scope: EU AI Act Article 50 almost certainly does not apply directly to AI-assisted code development. The compliance framing in `SEED_DOC.md` requires qualification. Flagged in spec; attorney confirmation needed before updating product positioning. C2PA added as a forward-compatibility consideration not in the original scope — now a named input for Iteration 2.
**Commit:** `03b9eb2`
