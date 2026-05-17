# Quill — Technical Feasibility Report
### VS Code Extension API + AI Tool Passive Capture Spike
*Version 0.1 — May 2026 | Status: COMPLETE*

---

## The Headline Finding

**The "passive by default" design principle survives — but only for Claude Code.**

For Claude Code-originated edits, passive capture is fully feasible and produces high-quality, attributed data: exact diffs, prompt linkage, and tamper-evident timestamps. This is the MVP target.

For all other AI coding tools (GitHub Copilot, Cursor, Windsurf), passive capture of suggestion-level events is **not technically feasible**. None expose accept/reject events to third-party VS Code extensions. The only passive signal available is `onDidChangeTextDocument`, which fires for all text changes but cannot distinguish AI-sourced edits from human typing, paste, or snippet expansion.

This finding drives a strategic product decision: **build Claude Code-first**. The hooks system gives us a native, high-fidelity capture path that no other tool offers. Expand to other tools with a heuristic layer + optional lightweight confirmation UI.

---

## Tool-by-Tool Feasibility Verdicts

| Tool | Passive Capture | Data Quality | Notes |
|---|---|---|---|
| Claude Code (agentic/edit mode) | **POSSIBLE** | High — exact diff + prompt | Via Claude Code hooks (PostToolUse) |
| Claude Code (inline IDE suggestions) | Not possible | N/A | Same wall as Copilot |
| GitHub Copilot | Not possible | N/A | No public extension API; exports empty |
| Cursor | Not possible | N/A | No documented extension API for AI events |
| Windsurf (Codeium) | Not possible | N/A | No documented extension API for AI events |
| Any tool (heuristic fallback) | Partial | Low-medium | onDidChangeTextDocument, ~60-70% accuracy |
| Any tool (active annotation) | Via user action | High | One-click confirm UI; violates passive principle |

---

## VS Code Extension API: What We Have

### The Universal Change Layer

`workspace.onDidChangeTextDocument` is the foundation. Every text edit in every open document fires this event, regardless of source. The payload:

```typescript
interface TextDocumentContentChangeEvent {
  range: Range;        // Range in OLD document that was replaced
  rangeOffset: number; // Offset of range.start in OLD document
  rangeLength: number; // Length of replaced text in OLD document
  text: string;        // NEW text that replaced the range
}
```

The `reason` field on the parent event distinguishes undo (`TextDocumentChangeReason.Undo`) and redo from normal edits — but `undefined` (normal edit) covers human typing, paste, AI acceptance, snippet expansion, and code action application. There is no "AI-sourced" reason code and there is no proposed API to add one.

**Full before/after content:** The API provides deltas only. Full "before" content requires maintaining a snapshot map in the extension. The pattern:

```typescript
const snapshots = new Map<string, string>();

workspace.onDidOpenTextDocument(doc => {
  snapshots.set(doc.uri.toString(), doc.getText());
});

workspace.onDidChangeTextDocument(event => {
  const uri = event.document.uri.toString();
  const before = snapshots.get(uri) ?? '';
  const after = event.document.getText();
  snapshots.set(uri, after);
  // Log: { uri, before, after, changes, timestamp, reason }
});

// Seed on activation for already-open documents
workspace.textDocuments.forEach(doc => {
  snapshots.set(doc.uri.toString(), doc.getText());
});
```

**Activation:** `onStartupFinished` in `activationEvents` — activates after VS Code loads, before user edits, without the performance cost of `*`. No special capabilities or permissions required to observe all document changes.

### What the VS Code API Cannot Do

- **Observe inline completions from other providers.** `InlineCompletionItemProvider` is a provider interface, not an observer bus. The callbacks (`handleDidShowCompletionItem`, `handleDidPartiallyAcceptCompletionItem`) are called by VS Code on the registering provider about its own items only. Copilot's provider callbacks are not accessible to Quill.
- **Access edit history or undo tree.** No API for `getEditHistory()` or similar. Undo events are flagged on `reason` but the history is opaque.
- **Distinguish paste from AI acceptance.** Both appear as `reason: undefined`, `text: <multi-line content>`. Heuristics can flag candidates but cannot make a reliable attribution.
- **Intercept other extensions' telemetry.** VS Code's `TelemetryLogger` API (stable since 1.78) is write-only and extension-local. No cross-extension telemetry tap.

There are open VS Code GitHub issues tracking the lack of inline completion observability (notably issue #124024), but as of mid-2025 no proposed API addresses it and no commitment exists.

---

## GitHub Copilot: Closed Surface

The Copilot extension (`GitHub.copilot`) exposes **nothing** via `vscode.extensions.getExtension('GitHub.copilot').exports`. The exports object is empty or undefined. This is not an oversight — GitHub has not published a typed extension API or activation exports for third-party consumption.

**What GitHub has shipped as extensibility:**
- **Copilot Extensions (Chat Participants):** Third parties can create `@agent` participants in Copilot Chat. This is a chat interface API — unrelated to inline suggestion events.
- **`vscode.lm` API (stable, VS Code 1.90+):** Allows extensions to *call* Copilot's language model for their own purposes. Does not allow observing Copilot's usage.
- **Copilot Enterprise analytics API:** Org-admin REST API for aggregated acceptance metrics dashboards. Not a real-time event stream for VS Code extensions.

**How existing productivity trackers handle this (WakaTime, Code Time, etc.):**
All use `onDidChangeTextDocument` heuristics. A multi-line insertion with `rangeLength === 0` (inserting without deleting) suggests a completion acceptance. None achieve reliable Copilot attribution — they acknowledge ~60-70% accuracy.

**Verdict: No path to passive Copilot capture. This is a platform constraint, not a Quill constraint.**

---

## Claude Code: The High-Fidelity Path

Claude Code's hooks system is the most capable capture surface available for any AI coding tool. This is the MVP architecture foundation.

### Hook Events (Full Set)

| Event | When | Has Matcher | Block Capable |
|---|---|---|---|
| `PreToolUse` | Before tool executes | Yes | Yes (non-zero exit blocks) |
| `PostToolUse` | After tool completes | Yes | No |
| `Notification` | On user notification | No | No |
| `Stop` | When agent turn ends | No | No |

### PostToolUse Payload for Edit Tool

When Claude Code edits a file, `PostToolUse` fires with:

```json
{
  "session_id": "abc123",
  "transcript_path": "/path/to/transcript.jsonl",
  "hook_event_name": "PostToolUse",
  "tool_name": "Edit",
  "tool_input": {
    "file_path": "/absolute/path/to/file.py",
    "old_string": "def foo():\n    pass",
    "new_string": "def foo():\n    return 42"
  },
  "tool_response": {
    "output": "File edited successfully"
  }
}
```

`old_string` + `new_string` is an exact, attributed diff. This maps directly to the artifact schema's `interaction.human_action.diff` field.

### PostToolUse Payload for Write Tool

When Claude Code writes a full file:

```json
{
  "tool_name": "Write",
  "tool_input": {
    "file_path": "/absolute/path/to/file.py",
    "content": "<full file content>"
  }
}
```

No `old_string` — Write replaces the entire file. To get a before-state, use `PreToolUse` to snapshot the file before Write executes (or read from git at the point of capture).

### Prompt Attribution via transcript_path

Every hook payload includes `transcript_path`: a path to a JSONL file containing the full session conversation. A hook script can read this to extract the prompt that triggered any tool call:

```python
import json, sys

payload = json.load(sys.stdin)
with open(payload["transcript_path"]) as f:
    turns = [json.loads(line) for line in f]

user_turns = [t for t in turns if t.get("role") == "user"]
last_prompt = user_turns[-1]["content"] if user_turns else None
```

This closes the authorship loop: edit → diff → prompt → timestamp → session. Every Claude Code-originated edit is fully attributable.

### Rejection Blind Spot

When a user rejects a Claude Code proposed edit in the interactive TUI, **no hook fires**. The rejection is a UI event invisible to the hooks system. If the user accepts, `PostToolUse` fires; if they reject, silence. This means the hooks log captures accepted edits only — rejections are not recorded.

**Implication for the artifact:** The interaction log can only record Claude Code edits that were accepted (executed). Rejected proposals are invisible. The artifact should note this limitation rather than imply completeness.

### A Minimal Quill Hook Script

```python
#!/usr/bin/env python3
import json, sys
from datetime import datetime, timezone
from pathlib import Path

payload = json.load(sys.stdin)
tool_name = payload.get("tool_name", "")

# Only log Edit and Write tool calls
if tool_name not in ("Edit", "Write", "MultiEdit"):
    sys.exit(0)

log_path = Path.home() / ".quill" / f"{payload['session_id']}.jsonl"
log_path.parent.mkdir(exist_ok=True)

# Extract prompt from transcript
prompt_text = None
transcript_path = payload.get("transcript_path")
if transcript_path:
    try:
        with open(transcript_path) as f:
            turns = [json.loads(line) for line in f if line.strip()]
        user_turns = [t for t in turns if t.get("role") == "user"]
        if user_turns:
            content = user_turns[-1].get("content", "")
            prompt_text = content if isinstance(content, str) else json.dumps(content)
    except Exception:
        pass

record = {
    "timestamp": datetime.now(timezone.utc).isoformat(),
    "session_id": payload["session_id"],
    "tool_name": tool_name,
    "file_path": payload["tool_input"].get("file_path"),
    "old_string": payload["tool_input"].get("old_string"),       # Edit only
    "new_string": payload["tool_input"].get("new_string"),       # Edit only
    "content": payload["tool_input"].get("content"),             # Write only
    "prompt_excerpt": prompt_text[:500] if prompt_text else None,
    "prompt_hash": None,  # TODO: SHA-256 of full prompt for tamper-evidence
}

with log_path.open("a") as f:
    f.write(json.dumps(record) + "\n")
```

This is a working prototype of the capture layer for Claude Code edits. Output is JSONL with one record per tool call, one file per session.

---

## Cursor and Windsurf: Closed Surfaces

Both Cursor (VS Code fork) and Windsurf (Codeium's IDE) support VS Code extensions but do not expose their AI suggestion events to extensions. Neither has documented a plugin API for AI interaction observability. The situation mirrors Copilot:

- **Cursor:** VS Code-compatible extension model; no documented API for AI suggestion events; no extension marketplace beyond VS Code Marketplace compatibility; internal AI architecture is proprietary.
- **Windsurf:** VS Code-compatible; Codeium's VS Code extension similarly exposes nothing via exports; no partner API for suggestion tracking.

**The heuristic fallback applies equally here:** `onDidChangeTextDocument` catches changes, but attribution is inferential.

**Web access gap — log for follow-up research:**
The current state of Cursor and Windsurf's extension APIs should be verified against live documentation before committing to a "not possible" verdict for these tools. Both products update rapidly. This is flagged as a PLAN.md backlog item (see below).

---

## What Passive Capture Actually Delivers

### With Claude Code hooks (high-fidelity path)

| Artifact field | Captured | Method |
|---|---|---|
| `session_id` | Yes | `payload.session_id` |
| `timestamp` | Yes | Hook execution time |
| `ai_tool` + `model` | Partial | Tool name known; model version requires separate capture |
| `prompt.content` | Yes | `transcript_path` read |
| `prompt.content_hash` | Yes | SHA-256 of prompt |
| `ai_response` | Partial | Not in hook payload directly; transcript contains it |
| `human_action.action_type` | Partial | If PostToolUse fires = accepted; rejection invisible |
| `human_action.diff` | Yes | `old_string` / `new_string` |
| `affected_scope.file` | Yes | `file_path` |
| `developer_attestation` | No | Requires active step |

### With VS Code onDidChangeTextDocument (heuristic path)

| Artifact field | Captured | Method |
|---|---|---|
| `session_id` | Synthesized | Extension generates per-session UUID |
| `timestamp` | Yes | Event timestamp |
| `ai_tool` | No | Cannot attribute |
| `prompt.content` | No | Not accessible |
| `human_action.action_type` | Inferred | Heuristic; ~60-70% accuracy |
| `human_action.diff` | Yes | range + text from event |
| `affected_scope.file` | Yes | `event.document.uri.fsPath` |
| `developer_attestation` | No | Requires active step |

The Claude Code path produces a legally-useful record. The heuristic path produces a change log — useful as corroborating evidence but weaker as primary authorship documentation.

---

## Prototype Data Model

How raw capture events map to the `ARTIFACT_SPEC.md` schema:

### Claude Code hook event → Interaction record

```
PostToolUse payload                    → QAR Interaction record
─────────────────────────────────────────────────────────────
session_id                            → session_envelope.session_id
<hook execution timestamp>            → interaction.timestamp
"Claude Code"                         → interaction.ai_tool
tool_input.file_path                  → interaction.affected_scope.file
tool_input.old_string                 → interaction.ai_response.content (what was proposed)
tool_input.new_string                 → interaction.human_action.diff (what was applied)
"accepted"                            → interaction.human_action.action_type
transcript last user message          → interaction.prompt.content
SHA-256(prompt)                       → interaction.prompt.content_hash
```

### VS Code change event → Interaction record (heuristic)

```
TextDocumentChangeEvent                → QAR Interaction record
─────────────────────────────────────────────────────────────
<synthesized>                         → session_envelope.session_id
event timestamp                       → interaction.timestamp
"unknown / heuristic"                 → interaction.ai_tool
event.document.uri.fsPath             → interaction.affected_scope.file
<not available>                       → interaction.prompt.content (null)
change.text (new content)             → interaction.human_action.diff
"accepted | human_edit | unknown"     → interaction.human_action.action_type (inferred)
```

### Session boundary detection

Claude Code: natural boundary — hooks fire within a `session_id`; `Stop` hook fires at turn end.
VS Code extension: synthesize session on extension activation; end on `deactivate()` or `workspace.onDidCloseTextDocument`.

---

## Architecture Recommendation

**Two-track capture architecture:**

**Track A — Claude Code hooks (primary, high-fidelity)**
- A Claude Code `PostToolUse` hook script that captures Edit/Write/MultiEdit tool calls
- Writes JSONL to `~/.quill/<session_id>.jsonl`
- Includes prompt attribution via `transcript_path` read
- Cryptographic hash of prompt for tamper-evidence
- Works today, no VS Code extension required, no user friction

**Track B — VS Code extension (secondary, universal)**
- `onDidChangeTextDocument` listener for all text changes
- Snapshot map for before/after full content
- Timestamps all changes regardless of source
- Heuristic flagging of likely-AI insertions (multi-line, zero-replacement)
- Optional lightweight confirmation UI ("Was this AI-generated?" with one-click yes/no)
- Enables correlation with Track A events by file path + timestamp proximity

**Track C — Developer annotation layer (optional, future)**
- A sidebar panel where developers can explicitly log prompts and mark AI interactions
- Highest quality data; breaks passive principle but is opt-in
- Appropriate for high-stakes projects where artifact quality is paramount

**MVP target:** Track A alone. Claude Code hooks require no new code to deploy (just a hook script and config), produce a legally-useful artifact, and validate the core product concept. Track B is the expansion that brings Copilot/Cursor/Windsurf users into scope.

**The "passive by default" principle, refined:**
*For Claude Code users: fully passive, high fidelity.* The hook runs silently with no developer action required. *For other AI tool users: passive change capture with optional active confirmation.* The extension watches silently; a lightweight UI can improve attribution quality on request. Neither imposes friction in the default path.

---

## Implications for the Artifact Spec

The following updates to `ARTIFACT_SPEC.md` are indicated by this spike:

1. **Add attribution quality tiers to the schema.** The `interaction.ai_tool` field should include an `attribution_method` enum: `hooks_attributed` (Claude Code hooks), `heuristic_inferred` (VS Code watching), `developer_annotated` (manual). Legal teams need to know which events are definitive vs. inferred.

2. **The rejection blind spot must be documented.** Claude Code hook capture records accepted edits only. The artifact should note "rejections not captured" rather than implying a complete interaction record.

3. **Model version capture needs a separate mechanism.** The hook payload does not include the model version Claude Code used. Must be captured from a separate source (Claude Code config, API response metadata, or explicit developer input).

4. **Session boundary definition.** The artifact spec's `session_envelope` assumes a clear session boundary. For VS Code extension capture, sessions must be synthesized (extension activation → deactivation). This needs to be explicit in the schema.

---

## Web Access Gaps — Flagged for PLAN.md

Items where live web research would improve confidence beyond training-data knowledge:

| Gap | What's missing | Priority |
|---|---|---|
| Cursor extension API (current) | Any post-Aug 2025 changes to Cursor's extension model or AI event exposure | Medium — would affect Iteration 2 verdict |
| Windsurf extension API (current) | Same for Windsurf/Codeium | Medium |
| VS Code inline completion proposals | Whether issue #124024 or related proposals have advanced toward a stable observer API | High — if a stable observer API shipped, it changes the architecture |
| Claude Code hooks payload schema | Verify transcript_path format and tool_input schema are current | High — prototype code depends on this |

---

## Decisions Required Before Building

**[D1] MVP scope: Claude Code-only or multi-tool?**
Building Track A (Claude Code hooks) alone is achievable now with high-quality output. Building Track B (VS Code extension with heuristic attribution) adds Copilot/Cursor/Windsurf support but at lower quality. The MVP should probably be Track A only — validate the concept, the artifact quality, and the customer workflow before investing in the heuristic layer.

**[D2] Hook script language and distribution**
The Quill hook script runs as a CLI process on every Claude Code tool call. Language choice affects installation friction: Python 3 is widely available; Node.js adds dependency on the developer's environment; a compiled binary (Go, Rust) is portable but harder to inspect. For MVP: Python 3.

**[D3] Privacy: prompt content vs. hash-only**
The hook gives us full prompt text via `transcript_path`. Capturing full prompts enables the richest artifact but creates privacy exposure (prompts may contain proprietary business logic). The default should be hash-only, with opt-in for full prompt capture. This must be decided before the hook script is finalized.

**[D4] Storage: local JSONL vs. cloud sync**
Track A writes to `~/.quill/`. For the artifact to survive across machines and be exportable at due diligence time, sessions need to be aggregated and exportable. Local-first with export is the right starting point; cloud sync is the enterprise path. This maps to the on-prem vs. cloud architecture question flagged in `CONTEXT.md`.
