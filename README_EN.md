# deepcode-audit

> AI-written code, audited by AI — not one AI, but a guild of AIs cross-verifying each other.

## What is this

deepcode-audit is a [Claude Code](https://claude.ai/code) skill that audits your codebase file by file, line by line, using **8 core perspectives + N domain perspectives** with cross-validation.

Inspired by MMO quest boards — each file × perspective is a quest. Agents claim quests, read files, perform 7-layer reviews, and submit evidence. The Guild Master verifies every submission.

## Highlights

- **8-Perspective Cross-Validation**: Engineering / Product / Quality / AI Content / Self-Review / Cross-Role / Security / Performance. Same file reviewed independently by multiple agents.
- **MMO Quest Board Scheduling**: Guild Master + Adventurers + quest claim/submit/verify/reject loop with real-time progress.
- **Anti-Cheat**: Agents must quote the exact middle line of each file as "content witness" — can't fake it without actually reading the file.
- **7-Layer Review Framework**: Code line → Function → File → Feature → Module → Architecture → System, bottom-up, no skipping.
- **Auto-Fix + Regression**: Safe-to-auto-fix items get fixed, committed, and re-audited. Needs-human items go through interactive walkthrough.
- **Auto-Discovery**: Domain perspectives (data engineering, mobile) auto-activate when keywords are detected in source code.

## Quick Start

1. Load this skill in Claude Code (place in `.claude/skills/` or load via skill management)
2. Trigger: `deep audit` / `code audit` / `architecture audit` / `security audit`
3. The skill scans the codebase, generates a quest board, and dispatches reviews file by file

## Configuration

Optional: edit `audit-config.yaml` to adjust scope, enable/disable perspectives, or tune scheduling. Uses sensible defaults otherwise.

## License

MIT
