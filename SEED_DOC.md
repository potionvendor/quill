# AuthorChain — Seed Document
### Problem, Opportunity, and Strategic Foundation
*Version 1.0 — May 2026*

---

## The Problem in One Paragraph

Copyright law requires human authorship. AI cannot own or be granted copyright. But the threshold for "sufficient human authorship" is fuzzy, uncodified, and entirely undocumented in most AI-assisted development workflows. As AI-assisted coding crosses 51% of committed code, companies building software IP on top of AI tools have a latent, growing liability: they may not be able to prove they own what they built. The documentation that would establish that proof does not exist in any systematic form. No one has built the tooling to create it.

---

## What Just Changed (Why Now)

**March 2, 2026 — Supreme Court denies cert in *Thaler v. Perlmutter*.** This is the final word for the foreseeable future. AI-generated works cannot receive copyright protection. The human authorship requirement is now settled law, not pending litigation. This removes the "maybe the law will change" hedge that was slowing urgency.

**November 2025 — USPTO revised inventorship guidance.** Confirmed AI cannot be named as inventor. Human conception remains the legal standard. Same principle, patent domain.

**August 2, 2026 — EU AI Act Article 50 enforcement begins.** Machine-readable provenance disclosure required for AI-generated content. This is a compliance mandate, not a recommendation. Companies without a provenance story face regulatory exposure.

**2026 — 51% of GitHub commits are AI-generated or AI-assisted.** The scale of the problem is no longer theoretical. Every company with a meaningful software IP portfolio is exposed.

---

## The Core Insight

The legal guidance is now clear and consistent across multiple authorities: **document every human creative decision made during AI-assisted work.** IP attorneys, the Copyright Office, and the USPTO are all saying the same thing. What none of them have done is build the tool to actually do it.

This is the gap. The advice exists. The tooling does not.

---

## What "Authorship Evidence" Actually Means

To support a copyright claim on AI-assisted code, a company needs to demonstrate:

1. **What AI generated** — which model, which version, which prompt
2. **What the human decided** — which output was selected, rejected, or modified, and why
3. **What the human added** — architectural decisions, refactoring, business logic, integration choices
4. **The sequence** — a timestamped chain showing the creative process unfolded with human direction

Today, none of this is captured in any structured way. Git commits capture the final state. IDE history captures keystrokes. Neither produces a legally-oriented authorship record.

---

## The Opportunity

**Primary market:** Software companies that (a) use AI coding tools heavily and (b) have IP they need to protect. This is a massive and growing cohort.

**Acute pain triggers:**
- M&A due diligence — acquirers are starting to ask about AI code provenance
- Series B+ fundraising — IP schedules in investment docs don't account for AI-assisted code
- Enterprise sales — procurement requiring IP clean certifications
- Copyright registration — Copyright Office now requires disclosure of AI-generated material

**Market surface:** The AI coding tools market is ~$12.8B in 2026. AuthorChain is not a coding tool — it's the legal infrastructure layer that sits on top of all of them. Every company using Copilot, Cursor, Claude Code, or any AI coding tool is a potential customer.

**Pricing signal:** Legal/IP insurance products for AI are emerging. Companies are willing to spend on risk mitigation in this space. The buyer is the GC or CISO, not the developer — which means higher ACVs and budget access.

---

## Initial Product Hypothesis

**AuthorChain is a developer-transparent, legally-oriented authorship logging system.**

It captures the human creative decisions made during AI-assisted development — passively, in the background — and produces a structured authorship artifact that can be exported for copyright registration, legal due diligence, and compliance reporting.

**Key design principles:**
- **Passive by default.** Developers should not have to do extra work. The tool captures what's already happening.
- **Legally oriented output.** The artifact is designed with the Copyright Office's "significant human contribution" standard in mind, not just developer productivity.
- **Chain of custody.** Timestamped, tamper-evident, auditable. Suitable for legal proceedings.
- **Tool-agnostic.** Works regardless of which AI coding tool the developer uses.

---

## Competitive Landscape (Current State)

| Category | Players | Gap |
|---|---|---|
| AI coding tools | GitHub Copilot, Cursor, Claude Code, Amazon Q | None track authorship for legal purposes. Copilot Enterprise has IP indemnification (legal insurance), not authorship evidence. |
| Code documentation | Mintlify, Doxygen, GitBook | Document what code *does*, not who *created* it and how. |
| Open source scanning | Black Duck, FOSSA | Scan for license contamination, not authorship provenance. |
| Legal/IP tech | IP management platforms, patent drafting tools | Serve legal teams, not developers. Don't touch the dev workflow. |
| **AuthorChain** | **Nobody** | **Captures human creative decisions during development for legal authorship purposes.** |

The white space is real and currently unoccupied.

---

## Revenue Model Hypotheses (Unvalidated)

- **SaaS per-seat** for dev teams (developer-led, bottoms-up growth)
- **Enterprise contract** for legal/GC buyers (top-down, higher ACV)
- **Compliance reporting module** as a premium tier (EU AI Act, copyright registration packages)
- **M&A/diligence report** as a one-time deliverable product (services-adjacent)

---

## What This Is Not

- Not a code documentation tool (we don't generate docs about what code does)
- Not an open source scanner (we don't detect license contamination)
- Not an AI coding tool (we don't generate code)
- Not a legal practice (we don't give legal advice; we produce evidence)
- Not trying to replace lawyers (we produce the record that lawyers and legal teams work from)
