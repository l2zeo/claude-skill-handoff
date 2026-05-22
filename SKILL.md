---
name: handoff
description: Create a reviewed semantic handoff artifact that transfers high-value reasoning, decisions, and state from the **current dying session** to a **fresh replacement session**. Handoff is for session REPLACEMENT (1:1), not for spawning parallel work. Use this skill whenever the user invokes `/handoff`, says "handoff", "hand off this session", "create a handoff", "전달 문서", "세션 인수인계", or signals that the current session has become inefficient — long context, `/compact` failed, token cost climbing, hallucination creeping in, or the user explicitly wants to "start fresh" / "새 세션으로 넘기다" / "이어서 다른 세션에서". Also trigger when the user wants to pass the current state to another model or to a teammate. If the user instead wants to split work into a parallel agent or worktree while keeping the current session alive, this is NOT handoff — ask before proceeding. The workflow is human-guided: draft → verify with user → finalize to OS temp directory. Never finalize without explicit user approval.
---

# /handoff

Create a **reviewed semantic handoff artifact** for a new independent AI session.

This workflow is **HUMAN-GUIDED**. Never finalize without explicit user approval.

The handoff is **not** documentation, **not** a transcript, **not** a session log.
It is a **semantic state-transfer artifact**: maximum information density, minimum
tokens, reasoning preserved over narration.

---

## When to use this skill

Handoff exists to solve **one specific problem**: the current session has become
inefficient (long context → token cost, slower responses, hallucination risk,
`/compact` failed) and it is time to **kill it and start a fresh one** that
inherits only the essential state.

The pattern is **1:1 session replacement**, not work multiplication.

**Use handoff when:**
- The current session is too long or expensive to continue, and the user wants
  to retire it and continue the same work in a new session.
- The user explicitly wants to start fresh ("새 세션으로 넘기다", "start over",
  "this chat is too long").
- The current state needs to be transferred to a different model, another
  teammate, or a different tool — and the source session will be abandoned.

**Do NOT use handoff when:**
- The user wants to **fork off a parallel agent / worktree** while the current
  session keeps running. That is task delegation, not handoff. The current
  session staying alive contradicts handoff's purpose (reducing context cost).
- The user wants to **save documentation** about the project. That belongs in
  `docs/`, not in a temp-directory state-transfer artifact.
- The user wants to **summarize** what happened. That is a summary, not a
  handoff. Just write the summary in chat.

**If the request looks like a misfit**, ask before drafting:

> "This sounds like [parallel agent work / documentation / summary] rather
> than a session handoff. Handoff is for retiring the current session and
> moving to a fresh one. Do you plan to end this session after?
> If not, we probably want a different approach."

If the user confirms it's actually a handoff, proceed. If not, drop the skill
and help them with what they actually need.

---

## Workflow

The skill runs as a five-step pipeline. Do not skip Step 4 (human verification).

### Step 1 — Determine Scope

Read the user's handoff instructions carefully. Identify:

- what **should** transfer
- what **should not** transfer
- the **exact mission** of the next session

Treat these as **hard boundaries**. If the user did not specify scope, ask one
focused question before proceeding — do not guess.

### Step 2 — Analyze Current Session

Walk the conversation and extract:

- relevant **decisions**
- relevant **reasoning** (why, not just what)
- relevant **files** (with paths)
- **unresolved problems**
- **failed approaches** (and why they failed)
- **constraints**
- **architectural intent**

Actively ignore:

- unrelated conversation
- obsolete exploration that was superseded
- low-value chatter, acknowledgments, small talk
- raw logs unless a specific log line is load-bearing

### Step 3 — Create Draft Handoff

Use `assets/draft_template.md` as the structure. Fill these sections:

1. **Mission**
2. **Current Status**
3. **Important Decisions**
4. **Relevant Files**
5. **Failed Approaches**
6. **Outstanding Issues**
7. **Suggested Next Actions**
8. **References**

Compress aggressively. See **Semantic Compression Rules** below.

### Step 4 — Human Verification (mandatory)

**Do NOT finalize.** Present the draft in the chat and explicitly ask:

- Is the **scope** correct?
- Is anything **important missing**?
- Is any **irrelevant context** present that should be removed?
- Any **sensitive content** to redact?

The user is the **semantic boundary validator**. Wait for explicit approval
("looks good", "approved", "ship it", "ok", "네 좋아요", etc.) before Step 5.
If the user requests changes, revise and re-present — loop until approved.

### Step 5 — Finalize

After approval:

1. Build the final document from `assets/final_template.md`.
2. Compute the filename: `handoff-YYYYMMDD-HHMM.md` (local time).
3. Save to the **OS temp directory** — never to the repo, never to `docs/`.
   - Linux/macOS: `$TMPDIR` if set, else `/tmp`
   - Windows: `%TEMP%`
   - In Python: `tempfile.gettempdir()`
4. Print the **absolute path** of the saved file.
5. Print a **one-paragraph summary** of the next session's objective so the
   user can paste it as the kickoff prompt for the new session.

---

## Templates

The `assets/` folder contains:

| File | When to use |
|---|---|
| `assets/draft_template.md` | Step 3 — structure for the draft you present to the user |
| `assets/final_template.md` | Step 5 — structure for the saved file after approval |

Templates use `{{PLACEHOLDERS}}` for substitution. Fill them in directly; do not
leave placeholder syntax in the final output.

---

## Security Rules

**Never include** in any handoff, draft or final:

- secrets
- API keys
- passwords
- tokens (auth, refresh, session)
- credentials of any kind
- connection strings with embedded credentials
- PII (real names, emails, phone numbers, addresses, ID numbers, etc.) unless
  the user explicitly confirms they are non-sensitive (e.g., a public test account)

If sensitive data appeared in the source conversation, **redact** it in the
handoff. Use `[REDACTED: api_key]`, `[REDACTED: email]`, etc. — preserve the
fact that something was there, but not its value.

When in doubt, redact and mention it in Step 4 so the user can decide.

---

## Semantic Compression Rules

The handoff **maximizes information density** and **minimizes token usage**.

**Preserve:**
- reasoning (why decisions were made)
- tradeoffs (what was given up for what was gained)
- unresolved uncertainty (do not paper over open questions)
- active state (what is in-flight, what is blocked)
- failed approaches with their failure modes

**Strip:**
- narrative connectives ("So then we tried…", "After that, the user said…")
- conversational politeness
- restating things the next session can read from the listed files
- speculative tangents that did not influence the current state

**Style guide:**
- bullets over paragraphs where possible
- imperatives in Suggested Next Actions ("Verify X.", not "We should probably verify X.")
- absolute or repo-relative file paths, never vague references ("the config file")
- one phrase of justification per file in Relevant Files — not a description

**Length budget:**

A handoff that is too long defeats its own purpose — the next session pays
the token cost on every turn. Target:

- **Sweet spot: 150–400 lines** (~1k–3k tokens).
- **Hard ceiling: ~600 lines** (~5k tokens). Going over means you are
  documenting, not transferring state.
- If you cannot fit under the ceiling, the **scope is too wide**. Push back
  in Step 4: ask the user to split into two handoffs, or to drop a section.
- Files, decisions, failed approaches: one-liners. If a single item needs a
  paragraph to explain, that item probably belongs in a file in the repo
  (referenced from the handoff), not inside the handoff itself.

A good handoff reads like a **briefing**, not a story.

---

## Failure Modes to Avoid

- **Drafting for the wrong pattern.** If this is parallel work / documentation /
  a summary, stop and ask before drafting (see "When to use this skill").
- **Finalizing without Step 4.** The human verification is non-negotiable.
- **Treating the draft as the final.** Always present it as a draft and ask.
- **Writing a transcript.** If the output reads chronologically, compress harder.
- **Bloating past the length budget.** A 1000-line handoff costs the next
  session more tokens than it saves. Push back on scope, do not pad.
- **Saving to the repo.** OS temp directory only.
- **Leaking secrets.** When uncertain, redact.
- **Losing the "why".** A list of decisions without reasoning is low-value.
- **Over-trimming uncertainty.** If something is unresolved, say so — do not
  invent false closure.
