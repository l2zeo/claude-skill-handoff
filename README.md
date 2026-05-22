# claude-skill-handoff

**English** · [한국어](./README.ko.md)

A Claude skill for creating **reviewed semantic handoff artifacts** when retiring
a long-running session and continuing the work in a fresh one.

> Handoff is for **1:1 session replacement** — not for parallel work,
> not for documentation, not for transcripts. It exists to solve the cost
> of long contexts: kill the heavy session, start a clean one, and let
> the new session inherit only what matters.

---

## Why this skill exists

Sessions accumulate context. Past a certain point this hurts:

- token cost climbs on every turn
- responses slow down
- hallucination risk rises
- `/compact` sometimes fails or strips out load-bearing reasoning

The instinct is to "just keep going". Often the better move is to **end the
session and start a new one** — but only if the new session can pick up where
the old one left off. That's what this skill produces: a compressed, human-
verified state-transfer artifact that the next session reads as its first
input.

The handoff preserves what's expensive to re-derive (decisions, reasoning,
failed approaches, open questions) and drops what isn't (chronology, dead
exploration, chatter).

---

## What the skill does

When invoked, the skill walks a five-step pipeline:

1. **Scope** — confirms what should and shouldn't transfer.
2. **Analyze** — extracts decisions, reasoning, files, failures, open issues.
3. **Draft** — fills the draft template and shows it in chat.
4. **Verify** — asks the user to approve, correct, or redact. Mandatory.
5. **Finalize** — saves `handoff-YYYYMMDD-HHMM.md` to the OS temp directory,
   prints the absolute path, and prints a one-paragraph kickoff prompt for
   the next session.

The skill enforces a length budget (target 150–400 lines, hard ceiling ~600),
refuses to save to the repo, and redacts secrets / API keys / PII.

---

## Installation

### Claude Code

Skills live in `~/.claude/skills/` (per-user) or `.claude/skills/` (per-repo).

```bash
# clone the repo somewhere convenient
git clone https://github.com/<you>/claude-skill-handoff.git
cd claude-skill-handoff

# install for your user account
mkdir -p ~/.claude/skills
cp -r handoff ~/.claude/skills/

# verify
ls ~/.claude/skills/handoff
# SKILL.md  assets/
```

Or install per-project:

```bash
mkdir -p .claude/skills
cp -r /path/to/claude-skill-handoff/handoff .claude/skills/
```

Restart your Claude Code session (or start a new one) and the skill becomes
available. You don't need to register or enable it explicitly — Claude reads
`SKILL.md` files in those folders automatically.

### Claude.ai (web/desktop)

Upload the `handoff/` folder as a skill via Settings → Capabilities → Skills.
Or zip it and upload as a `.skill` file.

---

## Usage

### Invoking

Any of these work:

```
/handoff
```

```
이 세션 너무 길어졌어. handoff 만들어줘.
```

```
This chat is getting expensive. Let's hand it off to a new session.
```

The skill will then ask for scope (what should transfer, what shouldn't,
what the next session's mission is), draft the handoff, and present it for
your review.

### The full flow

```
You: /handoff WDS Text 마이그레이션만 다음 세션에서 마무리할 예정.
              Tamagui 쪽 실험 흔적은 빼줘.

Claude: [confirms scope]
        [analyzes the session silently]
        [shows draft handoff in chat with 4 verification questions]

You:    [approve, request edits, or cancel]
        ↓
Claude: [if approved] saves to /tmp/handoff-20260522-1430.md
        prints absolute path + kickoff prompt for the next session
```

### Continuing in a new session

```bash
# end the current session
/exit

# start fresh
$ claude

# point it at the handoff
You: /tmp/handoff-20260522-1430.md 읽고 이어서 작업해줘.
```

The new session reads the handoff and starts at Suggested Next Action #1.

---

## When NOT to use this

Handoff is the wrong tool for:

- **Parallel work / worktree spawning.** If the current session stays alive,
  you're not handing off — you're delegating. The skill will ask you to
  confirm before drafting.
- **Project documentation.** That belongs in `docs/`, not in a temp file.
- **Session summaries.** Just ask for a summary in chat — no skill needed.

The skill is designed to refuse misfit cases by asking first.

---

## Repository layout

```
claude-skill-handoff/
├── README.md                    ← you are here
├── LICENSE                      ← MIT
├── CHANGELOG.md
├── .gitignore
└── handoff/                     ← the skill itself; copy this folder to install
    ├── SKILL.md
    └── assets/
        ├── draft_template.md    ← used in Step 3
        └── final_template.md    ← used in Step 5
```

Everything inside `handoff/` is the skill. Everything outside is repo
metadata for humans.

---

## Design principles

A few choices worth knowing about, in case you fork:

- **Human-in-the-loop is non-negotiable.** Step 4 (verification) cannot be
  skipped. The user is the semantic boundary validator — only they know
  what's really relevant to the next session.
- **Save to OS temp directory, not the repo.** Handoffs are ephemeral by
  design. If a handoff lives long enough to need version control, it
  should have become documentation instead.
- **Length budget is enforced.** A bloated handoff defeats its own
  purpose — the next session pays the token cost on every turn. Target
  150–400 lines.
- **Redact secrets by default.** Never include API keys, tokens,
  passwords, or PII. When in doubt, redact and flag for the user.

See `handoff/SKILL.md` for the full spec.

---

## Contributing

Issues and PRs welcome. A few ground rules:

- Keep `SKILL.md` under 500 lines (Anthropic's skill-creator guideline).
- If you add a template, put it in `handoff/assets/` and reference it from
  `SKILL.md` with explicit guidance on when to use it.
- Don't add a `scripts/` folder unless it solves a real problem — file
  saving is one `tempfile.gettempdir()` call away and doesn't need
  abstraction.
- Test changes by actually invoking the skill in a real session before
  opening a PR. Subjective feel matters here.

---

## License

MIT — see [LICENSE](./LICENSE).

---

## Acknowledgments

The workflow design was inspired by the pain of `/compact` failures, the
cost of carrying a 100k-token session through a third hour, and the
realization that "summarize this conversation" is fundamentally a
different problem than "transfer the active state of this conversation
to a fresh agent".
