# session-detail

A Claude skill that **records a work session verbatim instead of summarizing it.** It produces a full markdown archive of everything that happened, plus a compact context-handoff block you paste into your next session to resume work without losing state.

If you have ever ended a long Claude session and then started a new one only to spend twenty minutes re-explaining what you already decided, this skill is the fix.

---

## Table of Contents

- [Why This Exists](#why-this-exists)
- [What It Produces](#what-it-produces)
- [Skill vs Command](#skill-vs-command)
- [Trigger Phrases](#trigger-phrases)
- [Installation](#installation)
- [Full Example Output](#full-example-output)
- [Field Reference](#field-reference)
- [How It Behaves](#how-it-behaves)
- [Customization](#customization)
- [FAQ](#faq)
- [Author](#author)
- [License](#license)

---

## Why This Exists

Most "summary" tools compress. They throw away the exact wording of decisions, the prompts you drafted but did not run, and the small technical notes that turn out to matter three days later. That compression is the problem this skill avoids.

**The distinction in one line:** a summary tells you *what the session was about*; this archive tells you *what actually happened, in enough detail to act on it later.*

Three concrete situations it is built for:

1. **Resuming work.** You stop mid-project. Next session, you paste the Context Updates block and Claude knows your locked decisions, open questions, and known gotchas immediately.
2. **Record-keeping.** You want a dated, durable log of a build, a debugging session, or a research thread that you can drop into a repo, a wiki, or a project tracker.
3. **Handoff.** Someone else (or future you) needs to pick up the work and needs the real detail, not a paragraph.

---

## What It Produces

The skill always outputs **exactly two sections** and nothing else.

### 1. `<session_full>` (the archive, roughly 15 to 20KB)

The complete record. Includes what was accomplished, every significant decision with its rationale and alternatives, issues with root causes and resolutions, prompts that were executed, **prompts drafted for the future captured verbatim**, code changes, technical notes, memory state, and next-session priorities.

### 2. `<context_updates>` (the handoff, compact)

The forward-facing block. New persistent facts, changed or corrected facts, locked decisions, open questions, and carry-forward warnings. This is the part you paste into your next conversation.

---

## Skill vs Command

This repo ships the same archive in two forms. Pick based on where you want the output to land.

| | Skill (`SKILL.md`) | Command (`claude-code/session-detail.md`) |
|---|---|---|
| Installs to | `.claude/skills/session-detail/` | `~/.claude/commands/session-detail.md` |
| Invoked by | Natural-language phrases ("session detail", "session archive") | The explicit `/session-detail` slash command |
| Output goes to | Downloadable file on Claude.ai/Cowork (chat is the fallback); chat on Claude Code | Written to a file in `_sessions/` |
| Chat output | Both full sections | One-line confirmation plus a `Display output in chat? (y/N):` prompt |
| Works on | Claude.ai and Claude Code | Claude Code only |
| Same-session detection | Scans conversation context | Matches `$CLAUDE_SESSION_ID`, falls back to date |
| "Prompts Generated for Next Session" section | Included | Omitted by design |

Short version: use the **skill** when you want the archive as a portable file (or in chat) outside any repo. Use the **command** when you want a durable dated file committed to the repo without cluttering the chat.

### Incremental checkpoints and file naming

Each checkpoint carries a hidden marker on its first line that records the tool, the session key, a sequence number, and the time it was generated:

```
<!-- session-detail | tool=code | session=<key> | seq=2 | generated=2026-06-20-11-30 -->
```

That marker is how a later run knows a prior checkpoint exists and which session it belongs to. The incremental boundary itself is positional: a new incremental covers only conversation content that comes after the previous checkpoint's position (or, for files read from disk, only work the previous checkpoint's content does not already cover). The `generated` field is metadata for humans and tooling, not the boundary mechanism.

Files are named with a tool prefix so their origin is obvious, and a session-ID fragment so files from the same session associate without any counting:

- `code-YYYY-MM-DD-HH-MM-<sid8>.md` (full, written by the Claude Code command)
- `code-YYYY-MM-DD-HH-MM-<sid8>-inc-NN.md` (incremental, `NN` equals the marker `seq` and starts at 02)
- `claude-...` and `cowork-...` are the prefixes used for the file the skill produces on Claude.ai and Cowork (or for the suggested filename when output goes to chat). Only the Claude Code command writes into a repo's `_sessions/`.

`sid8` is the first 8 alphanumeric characters of the session key: the first 8 characters of `$CLAUDE_SESSION_ID` on Claude Code, or date digits derived from the first message timestamp elsewhere. Files sharing a `sid8` belong to the same session, so `code-...-a3f9c12b.md` and `code-...-a3f9c12b-inc-02.md` pair by inspection. This replaces the old session-counting scheme (`session-##`), which required directory scans and broke across midnight. The marker's full `session=` key remains the ground truth for matching; `sid8` is the human-readable hint. Old `session-##` files are still detected correctly because matching reads markers, not filenames.

---



Say any of these to invoke the skill. No setup or slash command required.

- `session detail`
- `detailed session summary`
- `session archive`

By default the skill auto-detects whether a checkpoint already exists for the current session. If one does, it produces an **incremental** that covers only the work since that checkpoint. If none does, it produces a **full** archive. You can force either mode:

**Force a full archive** (captures the whole session even if a checkpoint already exists):
- `full session detail`
- `full session-detail`
- `session-detail full`
- `session detail full`

**Force an incremental** (captures only new work since the last checkpoint):
- `session detail incremental`
- `session-detail inc`

On Claude Code the command takes the same modes as an argument: `/session-detail`, `/session-detail full`, or `/session-detail inc`.

If a prior checkpoint exists but the skill cannot confirm it belongs to the current session, it asks one question and defaults to full.

---

## Installation

### Option A: Claude Code (clone)

The repo name matches the skill name, so cloning produces the correct folder automatically.

**Project-level** (available in this project only):
```bash
cd .claude/skills/
git clone https://github.com/thebpandey/session-detail.git
```

**User-level** (available across all your projects):
```bash
cd ~/.claude/skills/
git clone https://github.com/thebpandey/session-detail.git
```

Either way the result is:
```
.claude/skills/session-detail/SKILL.md
```

Recent versions of Claude Code pick up new skills in `.claude/skills` without a restart. If the skill does not trigger, restart the session.

**Optional: install the slash command too.** This repo also ships a Claude Code command that writes the archive to a file instead of printing it to chat. Copy it into your commands directory:

User-level (all projects):
```bash
cp claude-code/session-detail.md ~/.claude/commands/session-detail.md
```

Then run `/session-detail` in any Claude Code session. It writes the archive to `_sessions/code-<YYYY-MM-DD-HH-MM>-<sid8>.md` in the current repo, prints a one-line confirmation, and asks `Display output in chat? (y/N):` (defaulting to N). See [Skill vs Command](#skill-vs-command) for when to use which.

### Option B: Claude.ai (zip upload)

1. Download this repo as a ZIP (the green **Code** button, then **Download ZIP**), or zip the folder so that `SKILL.md` sits inside a folder named `session-detail`.
2. In Claude.ai, open **Settings** and find the custom Skills upload area, then upload the ZIP.
3. Custom skill upload requires a Pro, Max, Team, or Enterprise plan with code execution enabled.

Note: the exact Settings menu label for skill upload has shifted between releases. If you do not see it, check both the Capabilities and Features areas of Settings.

---

## Full Example Output

Below is a fabricated example showing what the skill returns. The imaginary session was a two-hour block building a tenant-screening dashboard. This is illustrative only; your real output is populated from your actual conversation.

> Everything between the two horizontal rules is the skill's output. It begins immediately with the first section and ends with the single closing line. No preamble.

---

<session_full>

# Session claude-2026-05-28-09-15-20260528
Date: 2026-05-28
Duration: 2 hours 10 minutes

## Accomplished
- Built the initial React component for the tenant-screening results table, including sortable columns for applicant name, credit band, and submission date.
- Wired the table to a mock data source so the UI can be developed before the screening API is ready.
- Added a status pill component (Pending, Approved, Flagged) with color coding driven by a single status-to-color map for maintainability.
- Fixed a column-width bug where the credit-band column collapsed when the dataset was empty.

## Decisions Made
- Decision: Use a mock JSON data source for now instead of integrating the live screening API.
  - Rationale: The screening API contract is not finalized, and blocking UI work on it would stall the whole sprint.
  - Alternatives considered: Stubbing the API at the network layer with a service worker, which was judged heavier than needed for this stage.
  - Impact: UI work can proceed independently. A swap-in task is now required once the API contract lands.
- Decision: Drive all status colors from a single map object rather than inline conditionals.
  - Rationale: Status colors will be reused across the table, the detail view, and notification badges.
  - Alternatives considered: Inline ternaries per component, rejected as duplicative and error prone.
  - Impact: One source of truth for status styling. Adding a new status is a one-line change.

## Issues Encountered
- Issue: The credit-band column collapsed to zero width when the data array was empty.
  - Root cause: The column width was being inferred from content, and there was no content to infer from.
  - Resolution: Set an explicit min-width on the column and added an empty-state row.
  - Time spent: Approximately 25 minutes.
  - Prevention: Define explicit min-widths on data-driven columns by default.

## Prompts Executed
1. **Prompt:** Generate the results table component with three sortable columns.
   - **Outcome:** Working table with click-to-sort on all three columns.
   - **Key Files:** src/components/ScreeningTable.jsx
   - **Commit:** feat: add sortable screening results table
2. **Prompt:** Add a reusable status pill driven by a color map.
   - **Outcome:** StatusPill component plus a shared statusColors map.
   - **Key Files:** src/components/StatusPill.jsx, src/lib/statusColors.js
   - **Commit:** feat: add status pill with centralized color map

## Prompts Generated for Next Session

### Swap mock data source for live screening API
**Intent:** Replace the temporary mock JSON with the real screening API once its contract is finalized.
**Suggested filename:** docs/prompts/swap-mock-for-live-api.md
**Target session/phase:** After the API contract is signed off.
**Full prompt body:**
```
Replace the mock data source in ScreeningTable.jsx with a live fetch against
the screening API. Requirements:
- Keep the existing column structure and sorting untouched.
- Add a loading state that shows skeleton rows while the request is in flight.
- Add an error state with a retry button.
- Map the API status values to the existing StatusPill statuses; do not invent
  new statuses without confirming the mapping first.
- Preserve the empty-state row behavior.
Output the full updated component and a short note on any status values that
did not map cleanly.
```

## Code Changes
- New files:
  - src/components/ScreeningTable.jsx (the results table)
  - src/components/StatusPill.jsx (status indicator)
  - src/lib/statusColors.js (status-to-color map)
- Modified files:
  - src/App.jsx (mounted the table and passed in mock data)
- Deleted files: None.

## Technical Notes
- The status color map lives in src/lib/statusColors.js. Any new status must be added there first or the pill will render with a default gray.
- Sorting is client-side only. When the live API arrives, decide whether sorting moves server-side for large datasets.
- The empty-state row is keyed separately so it does not interfere with sort logic.

## Next Session
- [ ] Run the swap-mock-for-live-api prompt once the API contract is signed.
- [ ] Decide client-side versus server-side sorting based on expected row counts.
- [ ] Add the detail view that reuses the StatusPill component.
</session_full>

---

<context_updates>

## Context Updates for Next Session

### New Persistent Facts
- The tenant-screening dashboard uses a mock JSON data source at this stage. The live screening API is not yet integrated.
- Status colors are centralized in src/lib/statusColors.js and reused across components.
- Table sorting is currently client-side only.

### Changed or Corrected Facts
- OLD: Assumed the screening API would be ready for this sprint. NEW: The API contract is not final, so UI was built against mock data instead.

### Decisions Locked
- Status styling is driven by a single color map, not inline conditionals, because the styling is reused in at least three places.
- UI development proceeds against mock data so it is not blocked by the unfinished API.

### Open Questions
- Should sorting move server-side once real dataset sizes are known?
- What is the exact set of status values the live API will return, and do they map cleanly to Pending, Approved, and Flagged?

### Carry-Forward Warnings
- Do not add a new status without updating statusColors.js first, or the pill silently renders gray.
- The mock-to-live swap is staged in docs/prompts and must run before any real screening data flows through the table.
</context_updates>

---

> Output ready. Session 2026-05-28-09-15-SESSION-01 archived.

---

## Field Reference

| Section | Field | What goes in it |
|---|---|---|
| session_full | Accomplished | Concrete things built, fixed, or configured |
| session_full | Decisions Made | Each decision with rationale, alternatives, and impact |
| session_full | Issues Encountered | Problem, root cause, resolution, time spent, prevention |
| session_full | Prompts Executed | Prompts that were run, with outcomes and files touched |
| session_full | Prompts Generated for Next Session | Prompts drafted but not run, captured verbatim |
| session_full | Code Changes | New, modified, and deleted files |
| session_full | Technical Notes | Patterns, configs, and details worth preserving |
| session_full | Next Session | Checklist of priority tasks |
| context_updates | New Persistent Facts | Facts established this session that carry forward |
| context_updates | Changed or Corrected Facts | Superseded assumptions, in OLD then NEW form |
| context_updates | Decisions Locked | Final decisions that should not be relitigated |
| context_updates | Open Questions | Unresolved items needing answers |
| context_updates | Carry-Forward Warnings | Known risks, blockers, and gotchas |

---

## How It Behaves

- **No questions asked.** On trigger, it reads the whole conversation and generates both sections immediately.
- **No placeholders.** If a field cannot be filled from the conversation, it is omitted or marked `N/A` rather than left as a raw bracket.
- **Prompts are verbatim.** Any prompt you drafted but did not run is captured in full, including code blocks and constraints, so it is recoverable later.
- **Exact wording preserved.** Code and specific copy produced during the session are not paraphrased.
- **Session ID format.** `YYYY-MM-DD-HH-MM-SESSION-01`, derived from the timestamp of the first message.

---

## Customization

The skill is a single `SKILL.md` file. To adjust it, edit that file:

- **Change the closing line.** Edit the Closing Line section near the bottom.
- **Add or remove archive fields.** Edit the FULL VERSION template block.
- **Change trigger phrases.** Edit the `description` frontmatter and the Behavior section, then keep them consistent.

Keep the body under 500 lines for best performance, and keep the `description` under 1024 characters.

---

## FAQ

**Does it summarize or compress anything?**
No. It is a verbatim record by design. If you want a short summary, this is the wrong tool.

**Will it leak placeholder brackets into my output?**
No. Empty fields are omitted or marked `N/A`.

**Does it work in both Claude Code and Claude.ai?**
Yes. See the two installation options above.

**Can I rename it?**
Yes, but the folder name must match the `name` field in the frontmatter, and trigger phrases must stay consistent across the frontmatter and the body.

---

## Author

**Bhaskar Pandey**
Founder, Almora Technology

- LinkedIn: [linkedin.com/in/pandeybhaskar](https://www.linkedin.com/in/pandeybhaskar)
- GitHub: [github.com/thebpandey](https://github.com/thebpandey)
- Email: [bhaskar.knp@gmail.com](mailto:bhaskar.knp@gmail.com)

---

## License

MIT. See [LICENSE](LICENSE).
