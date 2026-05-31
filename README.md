# session-detail

A Claude skill that generates a structured, detailed session archive in markdown format. Designed for long-term logging, record-keeping, and cross-session handoff.

## What It Does

When triggered, the skill reviews the entire current conversation and produces two output sections:

- **`<session_full>`** -- A full (~15-20KB) archive covering what was accomplished, decisions made with rationale, issues and root causes, verbatim prompts generated for future sessions, code changes, technical notes, memory state, and next session priorities.
- **`<context_updates>`** -- A compact forward-facing block with new facts, changed facts, locked decisions, open questions, and carry-forward warnings. Built to be pasted into the next session to restore context without re-reading the archive.

## Trigger Phrases

Say any of the following to invoke the skill:

- `session detail`
- `detailed session summary`
- `session archive`

## Installation

### Option A: Claude Code (clone)

Clone directly into your skills directory. The repo name matches the skill name, so the resulting folder is correct automatically.

Project-level (this project only):
```
cd .claude/skills/
git clone https://github.com/thebpandey/session-detail.git
```

User-level (all projects):
```
cd ~/.claude/skills/
git clone https://github.com/thebpandey/session-detail.git
```

Result: `.claude/skills/session-detail/SKILL.md`

### Option B: Claude.ai (zip upload)

1. Download this repo as a ZIP (Code > Download ZIP), or zip the folder so that `SKILL.md` sits inside a folder named `session-detail`.
2. In Claude.ai, go to **Settings > Capabilities** (Skills), and upload the ZIP.
3. Requires a Pro, Max, Team, or Enterprise plan with code execution enabled.

## Output Behavior

- Generates exactly two sections: `<session_full>` and `<context_updates>`.
- No preamble, no meta-commentary, no placeholders left unfilled.
- Prompts drafted during the session but not yet executed are captured verbatim in the archive.
- Closes with a single confirmation line: `Output ready. Session [SESSION-ID] archived.`

## Notes

- The skill never abbreviates or summarizes. It is a verbatim record.
- If a field cannot be populated from conversation context, it is omitted or marked `N/A`.
- Session ID format: `YYYY-MM-DD-HH-MM-SESSION-01`

## License

MIT. See [LICENSE](LICENSE).
