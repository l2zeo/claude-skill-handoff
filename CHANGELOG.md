# Changelog

All notable changes to this skill are documented here.
Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
versioning per [SemVer](https://semver.org/).

## [0.1.0] — 2026-05-22

### Added
- Initial release of the `handoff` skill.
- Five-step workflow: scope → analyze → draft → verify → finalize.
- `assets/draft_template.md` — structure for the draft shown during verification.
- `assets/final_template.md` — structure for the saved artifact.
- Mandatory human-in-the-loop verification (Step 4).
- Length budget: target 150–400 lines, hard ceiling ~600.
- Save location: OS temp directory only (never the repo).
- Security rules: redact secrets, API keys, PII by default.
- "When to use this skill" section with explicit misfit detection
  (parallel work / docs / summaries) and user re-prompt.
