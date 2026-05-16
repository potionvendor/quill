# Quill — Authorship Artifact Specification v0.1
### Research-Grounded Draft for IP Attorney Validation
*Version 0.1 — May 2026 | Status: DRAFT — Not validated by legal counsel*

---

## Purpose of This Document

This is the design-first document for Quill. It specifies what the authorship artifact contains, in what format, at what level of granularity — before any capture layer is built. The pipeline exists to serve this output, not the other way around.

This v0.1 is a research-grounded draft. It is **not validated by IP attorneys** and must not be relied upon for actual legal filings. Section 8 (Open Questions for IP Attorneys) lists the specific questions that must be answered before this spec can be considered v1.0.

---

## What This Artifact Is

A **Quill Authorship Record (QAR)** is a timestamped, tamper-evident, machine-readable document that captures the human creative decisions made during an AI-assisted software development session. It is designed to support:

1. **Copyright registration** — providing the disclosure and evidence required by the US Copyright Office for AI-assisted works
2. **M&A IP due diligence** — serving as structured, auditable documentation of human authorship and AI tool governance
3. **Internal governance** — giving organizations a systematic record of AI tool usage and human creative control over AI output

What it is not: a legal opinion, a copyright guarantee, or a compliance certification. It is the evidentiary record that attorneys and legal teams work from.

---

## Important Reframe: EU AI Act Article 50

**The SEED_DOC.md presents EU AI Act Article 50 as a core compliance hook for Quill. This requires significant qualification.**

After researching the full text of Regulation (EU) 2024/1689 and its recitals:

- **Article 50 likely does not apply to AI-assisted code development.** Article 50's transparency obligations target public-facing deception scenarios: deep fakes (50(3)), AI-generated text for informing the public on matters of public interest (50(4)), and GPAI provider watermarking obligations (50(5)). Code committed to a repository or used in internal software development is almost certainly not within scope.

- **GPAI provider obligations (Article 50(5)) were enforceable from August 2, 2025.** This means the AI model providers (Anthropic, OpenAI, etc.) already had watermarking obligations. Quill's customers are *deployers*, and deployer obligations under Article 50 don't apply to code development workflows.

- **No implementing format has been specified.** The Commission was authorized under Article 50(7) to issue implementing acts specifying machine-readable disclosure formats — but as of the time of this writing, no implementing act had been published. C2PA is the leading industry candidate.

**Revised framing for Quill's EU compliance angle:**
Quill is not an Article 50 compliance tool for software development. The more accurate framing is:
- Quill's provenance practices are *consistent with the regulatory direction of travel* established by the EU AI Act
- Quill is directly relevant where customers use AI to generate public-facing *content* (not just code) — in those cases, deployer Article 50 obligations do apply
- The EU Cyber Resilience Act (CRA) and sector-specific regulations (AI in medical devices, critical infrastructure) are more likely compliance hooks for software-development provenance

This does not weaken the core value proposition — the US copyright and M&A diligence use cases are real and strong. But the product should not overclaim EU regulatory mandate.

**This finding updates Assumption #3 in CONTEXT.md** (EU AI Act as a compliance forcing function) — it is weaker for the software development use case than previously stated.

---

## Research Foundations

### US Copyright Office

The operative standard for AI-assisted works, established in the Copyright Office's 2023–2024 guidance:

- AI-generated elements **cannot be registered**. Only the human-authored elements receive copyright protection.
- Human authorship survives when a human exercises **creative selection, arrangement, coordination, or modification** of AI-generated material — even if the raw material was AI-generated.
- The Copyright Office **requires disclosure** of AI-generated content at registration time. Failure to disclose is grounds for cancellation of registration.
- "Sufficient human authorship" means the human made **expressive creative choices** about the AI output — selecting which suggestions to accept, how to modify them, how to arrange them, what to reject. Mechanical or random selection does not qualify.
- For copyright registration of AI-assisted software, the registration covers the **human-authored expression only** — the selection, structure, architecture decisions, modifications, and arrangements that reflect human creative choices.

**What this means for the artifact:** The artifact must document the human creative decisions, not just log that AI was used. Logging that Copilot made a suggestion and the developer accepted it is insufficient. Logging that the developer reviewed the suggestion, modified the function signature, refactored the error handling, and integrated it with a human-authored module — that is the record that matters.

### M&A IP Due Diligence

As of 2025–2026, the questions appearing in IP schedules and technical diligence questionnaires include:

- Does the company have a written AI tool use policy? Produce it with version history.
- Which AI coding tools are authorized and in use? What are their commercial terms?
- What percentage of the codebase is AI-assisted? Is this tracked?
- Have developer and contractor agreements been updated to include AI-output IP assignment language?
- Has any AI-generated output been incorporated verbatim without human modification?
- Does the company's AI tool usage comply with each tool's terms of service?
- Has the company received any IP claims related to AI-generated output?

What acquirers consider "sufficient documentation" is converging on:
- A written, dated AI governance policy
- Developer attestations that they reviewed, modified, and took authorship responsibility for AI output
- Evidence of human editorial decisions (PR review records, commit messages, code review comments)
- Tool-level records where available (vendor telemetry, logs)
- The "sufficient human creative control" standard — evidence that humans selected, arranged, modified, or made creative choices about AI output

**What this means for the artifact:** The artifact is not just a technical log — it is an evidentiary package. It needs to be legible to a non-technical IP attorney, exportable in a format that can be attached to a rep & warranty schedule, and signed/attested by identifiable humans.

---

## The Quill Authorship Record: Schema

A QAR consists of four layers:

### Layer 1: Session Envelope
Top-level metadata for a capture session (typically one development session or work period).

```json
{
  "qar_version": "0.1",
  "session_id": "<uuid>",
  "project_id": "<uuid>",
  "repo_identifier": "<git remote URL or local path hash>",
  "session_start": "<ISO 8601 timestamp>",
  "session_end": "<ISO 8601 timestamp>",
  "developer_id": "<pseudonymous or actual identifier>",
  "capture_tool": {
    "name": "Quill",
    "version": "<semver>"
  },
  "ai_tools_active": [
    {
      "name": "<tool name, e.g. GitHub Copilot>",
      "model": "<model identifier if available>",
      "version": "<tool version>"
    }
  ],
  "integrity": {
    "hash_algorithm": "SHA-256",
    "session_hash": "<hash of the full session record>",
    "timestamp_authority": "<RFC 3161 timestamp token if applicable>"
  }
}
```

### Layer 2: Interaction Log
A chronological record of each AI interaction and the human's response to it.

```json
{
  "interactions": [
    {
      "interaction_id": "<uuid>",
      "timestamp": "<ISO 8601>",
      "ai_tool": "<tool name>",
      "model": "<model identifier>",
      "prompt": {
        "content": "<prompt text>",
        "content_hash": "<SHA-256 hash>",
        "privacy_redacted": false
      },
      "ai_response": {
        "content_hash": "<SHA-256 hash of raw response>",
        "privacy_redacted": false
      },
      "human_action": {
        "action_type": "accepted | rejected | modified | partially_accepted | ignored",
        "modification_applied": true,
        "modification_description": "<free text — what the human changed and why>",
        "diff": "<unified diff of human changes to AI output, if applicable>"
      },
      "affected_scope": {
        "file": "<relative file path>",
        "lines_affected": "<line range>",
        "function_or_module": "<name if applicable>"
      }
    }
  ]
}
```

**Privacy design note:** The `content` field for prompts and responses may contain proprietary business logic. For privacy-sensitive environments, `content_hash` captures the fact of the interaction while `privacy_redacted: true` omits the raw content. The hash alone is sufficient to prove a specific prompt was used, without exposing its contents. This is a first-class design choice, not a workaround.

### Layer 3: Human Decision Record
A structured log of human creative decisions that go beyond individual AI interactions — architectural choices, refactoring decisions, integration work, and design decisions that represent human creative contribution independent of any specific AI suggestion.

```json
{
  "human_decisions": [
    {
      "decision_id": "<uuid>",
      "timestamp": "<ISO 8601>",
      "decision_type": "architectural | refactoring | integration | rejection | selection | design | business_logic | other",
      "description": "<free text — what was decided and why>",
      "affected_scope": {
        "files": ["<relative file path>"],
        "functions_or_modules": ["<names>"]
      },
      "ai_contribution": "none | informed | modified | rejected",
      "human_creative_element": "<brief description of the specific human creative choice>"
    }
  ]
}
```

### Layer 4: Authorship Summary
A file-level and session-level summary suitable for export to non-technical audiences.

```json
{
  "authorship_summary": {
    "session_overview": {
      "total_interactions": 0,
      "accepted_without_modification": 0,
      "accepted_with_modification": 0,
      "rejected": 0,
      "human_decisions_logged": 0
    },
    "file_summaries": [
      {
        "file": "<relative path>",
        "human_decisions_count": 0,
        "ai_interactions_affecting_file": 0,
        "human_modifications_to_ai_output": 0,
        "verbatim_ai_incorporation": false,
        "authorship_characterization": "human_primary | human_directed_ai | ai_assisted | ai_primary_human_reviewed"
      }
    ],
    "developer_attestation": {
      "developer_id": "<identifier>",
      "attestation_text": "I certify that I reviewed the AI-generated outputs reflected in this record, made the creative decisions described herein, and take authorship responsibility for the human-authored elements of the work product.",
      "attestation_timestamp": "<ISO 8601>",
      "attestation_signature": "<cryptographic signature if applicable>"
    }
  }
}
```

---

## The Human-Readable Report Layer

The machine-readable JSON schema is the ground truth. But legal and diligence teams need a document they can read, attach to a filing, and present in a proceeding. The human-readable report is generated from the schema and contains:

**Cover page:**
- Project name, developer, date range, Quill version
- Total sessions covered
- High-level authorship summary (interaction counts, modification rates)
- Integrity hash and timestamp authority reference

**Section 1 — AI Tools Used**
Table listing each AI tool, model version, and usage period.

**Section 2 — Authorship Narrative**
A plain-language description of the development session, generated from the human decision records. Example:
> "During this session, the developer used GitHub Copilot to generate initial implementations of the authentication middleware. In 12 of 17 interactions, the developer modified the AI-suggested output, including restructuring the error handling logic, replacing the session management approach with a human-designed token model, and integrating the module with an existing human-authored permission system. 5 AI suggestions were rejected without incorporation. The developer logged 4 explicit architectural decisions, including the choice of JWT over session cookies and the design of the token refresh flow."

**Section 3 — File-Level Authorship Summary**
Table: file, human decision count, AI interactions, modification rate, authorship characterization.

**Section 4 — Developer Attestation**
The signed attestation text with timestamp.

**Section 5 — Integrity and Chain of Custody**
The session hash, timestamp authority token, and instructions for independent verification.

**Appendix — Questions for Copyright Registration**
Pre-filled responses to the Copyright Office's standard AI disclosure questions, generated from the record:
- Did the work involve AI-generated content? Yes/No
- Which portions are AI-generated? (Generated from file summaries)
- What is the nature of the human authorship? (Generated from decision records)

---

## Field Decisions: Required / Recommended / Excluded

### Required (artifact is not credible without these)
- Session timestamps (start and end, to ISO 8601 precision)
- AI tool identities (name and model version for each tool used)
- Human action type for each AI interaction (accepted / rejected / modified)
- Developer attestation with timestamp
- Session integrity hash
- Verbatim AI incorporation flag per file (none / flagged)

### Recommended (significantly strengthens the record)
- Prompt content or hash (content preferred; hash acceptable for privacy-sensitive environments)
- Diff of human modifications to AI output
- Human decision descriptions (free text)
- Architectural and design decisions logged separately from individual interactions
- RFC 3161 third-party timestamp authority token (makes timestamps tamper-evident and independently verifiable)
- File-level authorship characterization

### Excluded (risk of harm — overclaim or underclaim)
- **Exact human/AI percentage claims** (e.g., "67% human-authored"). These suggest false precision and may create liability by implying that the AI-generated percentage is unprotectable without attorney guidance. Use characterization buckets instead.
- **Legal conclusions** ("this work is copyright-protected", "this file qualifies for registration"). Quill produces evidence; attorneys make conclusions.
- **Blanket coverage claims** ("all human decisions are captured"). The record captures what was logged; it cannot certify completeness.
- **Inferred intent** without developer confirmation. Never auto-generate a description of why a human made a decision — only log what the developer explicitly attested to.

---

## Format Decision

**Machine-readable layer: JSON-LD**

JSON-LD (JSON for Linked Data) is the right choice because:
- It is a W3C standard with broad tooling support
- It supports semantic typing, which allows the schema to be extended without breaking existing consumers
- It is compatible with C2PA manifest structures (the leading EU AI Act compliance candidate) and with W3C Verifiable Credentials (for developer identity attestation)
- It is natively parseable by every major programming language
- It can be embedded in or linked from an SPDX 3.0 AI profile record, which is the emerging standard for software-component AI provenance in supply chain / CRA contexts

**Human-readable layer: PDF**

Generated from the JSON-LD record. PDF is the format legal teams attach to filings, rep & warranty schedules, and diligence packages. The PDF is a view of the JSON-LD record, not a separate data source — the JSON-LD is authoritative.

**Registration export: Copyright Office disclosure fields**

A structured export pre-mapped to the Copyright Office's current AI disclosure requirements (form fields and deposit copy notes). This is not a separate format — it is a projection of the JSON-LD record onto Copyright Office expectations.

**On C2PA compatibility:**
C2PA (Coalition for Content Provenance and Authenticity) is the most likely candidate for EU AI Act implementing act technical standards. While it is primarily designed for media files (images, video, audio), its cryptographic manifest structure and provenance chain model are directly applicable to code. Quill's JSON-LD schema should be designed so that a C2PA-compatible manifest can be generated from it without redesign — even if C2PA is not the primary output format today. This is a forward-compatibility decision, not a current requirement.

---

## Open Questions for IP Attorneys
*These must be resolved before this spec advances to v1.0. They constitute the agenda for attorney validation sessions.*

**Q1 — Granularity of AI disclosure at copyright registration**
Does the Copyright Office require identification of specific files, functions, or lines as AI-generated? Or is session-level or project-level disclosure sufficient? The spec currently produces file-level summaries — is that the right granularity for registration purposes?

**Q2 — Attestation weight**
How much legal weight does a developer's signed attestation ("I reviewed and modified this AI output") carry without corroborating technical evidence? Is attestation alone sufficient for a copyright claim, or is the technical capture log (diffs, interaction records) necessary to make the attestation credible?

**Q3 — The modification threshold**
Is there a minimum level of human modification that constitutes "sufficient" creative authorship? One character change? Structural refactoring? Architectural decisions made after reviewing AI output? The spec needs to flag when human modification was minimal — but we need to understand what "minimal" means legally before we characterize it in the record.

**Q4 — Self-documentation liability**
Can the artifact create liability by documenting verbatim AI incorporation? If the record shows that a developer accepted a Copilot suggestion verbatim with no modification, does that documentation create a problem that would not otherwise exist — e.g., strengthening a copyright infringement claim against the company? Should the artifact flag this as a risk, or avoid documenting it?

**Q5 — Developer identity at registration**
Copyright registration requires identification of the human author(s). The spec currently supports pseudonymous developer identifiers for privacy. Is pseudonymous identity acceptable for registration purposes, or must actual developer identities be tied to the record?

**Q6 — Record retention requirements**
How long must authorship records be retained to be useful in litigation, registration disputes, or M&A diligence? The answer affects data architecture decisions (storage duration, archival format, retrieval obligations).

**Q7 — Third-party timestamp authority**
Does RFC 3161 timestamping by a recognized timestamp authority (e.g., DigiCert, Sectigo) satisfy evidentiary standards for tamper-evident timestamps in a copyright dispute or litigation context? Are there alternative or preferred timestamping approaches in IP proceedings?

**Q8 — The EU AI Act framing**
The Quill seed documents describe Article 50 of the EU AI Act as a compliance forcing function. Research suggests Article 50 likely does not apply directly to AI-assisted software development. Can you confirm this reading? And if so, what is the most accurate EU regulatory frame for the provenance use case in software development?

**Q9 — M&A rep & warranty language**
What language in a purchase agreement rep & warranty schedule would Quill's artifact be designed to satisfy? Is there emerging standard language that defines what "documentation of human authorship of AI-assisted code" means in an acquisition context?

**Q10 — Completeness vs. selective logging**
Can a partial record — one that captures some sessions but not all — be used in a copyright filing or diligence context without creating a negative inference about the uncaptured sessions? Or does selective logging create more problems than it solves?

---

## Assumptions Updated

This research updates the following assumptions from `CONTEXT.md`:

**Assumption #3 (GCs/IP counsel aware of this problem) — No change.** The M&A diligence findings confirm the problem is real and felt in deal contexts. The awareness exists; the tooling does not.

**Assumption #4 (authorship artifact will be accepted as credible) — Refined.** The diligence research shows what credibility requires: developer attestation + technical evidence + chain of custody + human-readable narrative. A log alone is insufficient. An attestation alone is insufficient. Both together, with integrity verification, is the target.

**EU AI Act as compliance forcing function — Weakened.** Article 50 does not directly mandate AI-assisted code provenance. The compliance hooks that are real: US copyright registration requirements, M&A rep & warranty language, internal AI governance policies, EU Cyber Resilience Act SBOM requirements, and sector-specific regulations for high-risk AI applications. The EU AI Act is relevant as regulatory direction of travel, not as direct mandate for this use case.

**C2PA as a format consideration — Added.** Not in the original docs. C2PA is the leading technical candidate for machine-readable AI provenance standards under EU AI Act implementing acts. Quill's schema should be designed for C2PA forward-compatibility. This is an architectural consideration for the technical spike (Iteration 2).

---

*This document is a discovery artifact, not a legal document. All legal framing must be validated by qualified IP counsel before use in any filing, due diligence package, or product representation.*
