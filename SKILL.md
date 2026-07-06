---
name: session-detail
description: "Generates a structured, detailed session archive in markdown format for long-term logging, record-keeping, or session handoff. Triggers on: \"session detail\", \"detailed session summary\", \"session archive\". Force a full archive with \"full session detail\", \"full session-detail\", \"session-detail full\", \"session detail full\". Force an incremental with \"session detail incremental\", \"session-detail inc\". Always trigger this skill on these exact phrases. Do not answer them directly without consulting this skill. By default the skill auto-detects whether an earlier checkpoint exists in the current session and produces an incremental from that point, otherwise a full. Output is markdown with exactly two sections: Full Version (session_full) and Context Updates (context_updates), delivered as a downloadable file where the environment supports it, otherwise in chat."
---

# Session Detail Skill

## Behavior on Invocation

Do not ask the user to paste the conversation. You already have it in context. The only question you may ask is the single ambiguity prompt described in Mode Resolution below.

**STRICT OUTPUT RULE: Generate exactly two sections -- `<session_full>` and `<context_updates>`. No other sections.**

Output destination depends on the environment:
- **Claude.ai and Cowork**: if file creation is available, write the archive to a downloadable .md file using the suggested filename and present it to the user; print to chat only if file creation is unavailable or the user explicitly asks for chat output. A 15-20KB archive printed into chat is carried in context on every subsequent turn; a file is not.
- **Claude Code**: print to chat. The separate Claude Code slash command is the file-writing form for repo `_sessions/` archives; this skill does not write into the repo.
In every case, include a `Suggested filename:` line inside the archive so the user can save or rename it.

### Step 1: Detect the tool and build the prefix

Determine which tool you are running in and set the filename prefix:
- **Claude Code**: filesystem and `$CLAUDE_SESSION_ID` are available. Prefix is `code`.
- **Claude Cowork**: a desktop workspace is present but this skill still does not write files. Prefix is `cowork`.
- **Claude.ai**: no persistent filesystem. Prefix is `claude`.

If you cannot tell Cowork from Claude.ai, default the prefix to `claude`.

### Step 2: Resolve the mode (full, incremental, or ask)

1. **Explicit override in the trigger phrase wins.**
   - Force full: "full session detail", "full session-detail", "session-detail full", "session detail full". Produce a FULL archive of the entire session regardless of any prior checkpoint.
   - Force incremental: "session detail incremental", "session-detail inc". Produce an INCREMENTAL. If no prior checkpoint can be found, say so in one line and produce a FULL instead (there is nothing to increment from).

2. **No override: auto-detect.** Look for the most recent prior checkpoint belonging to THIS session.
   - **Claude Code**: scan both `_sessions/` on disk AND the current conversation context for `<session_full>` blocks. A prior checkpoint qualifies if its marker `session=` equals the current `$CLAUDE_SESSION_ID`. Check both sources because this skill prints to chat (not disk), so a checkpoint generated earlier in this same Claude Code session exists only in the conversation context, not in `_sessions/`. If candidates are found in both locations, use the one with the highest `seq`. (This skill reads to detect; it still does not write.)
   - **Claude.ai and Cowork**: scan the current conversation context for the most recent `<session_full>` block that you generated earlier in this same conversation, including the contents of any archive file you created earlier this session (the file's content passes through your tool calls and remains visible in context).
   - **None found** -> FULL, `seq=1`.
   - **Found and same-session is certain** -> INCREMENTAL, `seq = prior seq + 1`.
   - **Found but same-session cannot be confirmed** (for example `$CLAUDE_SESSION_ID` is unavailable and only a same-date file exists, or a checkpoint appears in context that may have been pasted in from a different conversation) -> ASK exactly one line and wait:
     `A prior session-detail checkpoint exists but I cannot confirm it is from this same session. Generate (i)ncremental from it, or (f)ull? [f]:`
     Default to FULL on an empty or unclear reply.

### Step 3: Honor the incremental boundary

For an INCREMENTAL, the boundary is positional, not temporal: capture only the conversation content that comes AFTER the prior checkpoint's position in this conversation (or, when the prior checkpoint was read from disk, only work not already covered by its content). Conversation messages carry no visible timestamps, so never attempt to apply a wall-clock boundary; the prior block's position and content define what is already captured. The marker's `generated` field is metadata for humans and file tooling, not the boundary mechanism. Do not repeat anything already captured in the prior checkpoint. Begin the archive body with one line:
`Incremental checkpoint, seq <n>, continues from <prior label> generated <time>. Covers only work since that point.`

If the prior checkpoint is missing from context (for example it was compacted out on a long conversation), say so in one line, then produce a FULL rather than guessing at a boundary.

### Step 4: Derive identifiers

- **Timestamp**: `YYYY-MM-DD-HH-MM` for now. On Claude Code use `date "+%Y-%m-%d-%H-%M"`; elsewhere use the current time.
- **Session key**: `$CLAUDE_SESSION_ID` on Claude Code (or `unverified-<timestamp>` if unavailable); the first user message timestamp on Claude.ai and Cowork. The key goes in the marker and is the ground truth for same-session matching.
- **sid8**: the first 8 alphanumeric characters of the session key (strip dashes and other punctuation first). On Claude Code this is the first 8 characters of the session UUID; on Claude.ai and Cowork it degenerates to the date digits, which is acceptable because there is no disk there to collide on. sid8 is a human-readable hint in the filename; the marker remains the ground truth.
- **Session ID label**: `<prefix>-<YYYY-MM-DD-HH-MM>-<sid8>` for a full, or `<prefix>-<YYYY-MM-DD-HH-MM>-<sid8>-INC-<NN>` for an incremental. NN equals the marker `seq` and starts at 02. There is no session counting: no directory scans to number sessions, no Nth-session-today logic, no day-rollover edge.
- **Suggested filename**: the lowercase form with `.md`, for example `code-2026-06-20-09-15-a3f9c12b-inc-02.md`.
- **Project name**: infer from the conversation, or use "Untitled Project" if unclear.
- **Duration**: estimate from first and last message timestamps if available, otherwise omit.

### Step 5: Stamp the marker

Place this marker as the very first line inside `<session_full>`, before the `# Session` heading:
`<!-- session-detail | tool=<prefix> | session=<key> | seq=<n> | generated=<YYYY-MM-DD-HH-MM> -->`
where `<key>` is `$CLAUDE_SESSION_ID` on Claude Code (or `unverified-<timestamp>` if unavailable), and the first user message timestamp on Claude.ai or Cowork.

Output must be **pure markdown only**, clean and copy-paste ready. No preamble, no meta-commentary above the blocks. Deliver the archive and close with the single line specified in Closing Line.

---

## Output Format

Generate **exactly two sections** using the exact XML tags below. No other sections. All content inside the tags must be properly formatted Markdown.

**Required sections:**
1. `<session_full>` -- Full Version
2. `<context_updates>` -- Context Updates

---

### FULL VERSION (~15-20KB)

<session_full>
<!-- session-detail | tool=[code|claude|cowork] | session=[key] | seq=[n] | generated=[YYYY-MM-DD-HH-MM] -->
# Session [PREFIX-YYYY-MM-DD-HH-MM-SID8 or -INC-NN for an incremental]
Date: [YYYY-MM-DD]
Suggested filename: [prefix-yyyy-mm-dd-hh-mm-sid8.md, or -inc-NN.md for an incremental]
Duration: [HH hours MM minutes, or omit if not determinable]

## Accomplished
[Each item with specific details: what was built, fixed, or configured. Be thorough.]

## Decisions Made
[For each significant decision:]
- Decision: [What was decided]
  - Rationale: [Why]
  - Alternatives considered: [What else was evaluated]
  - Impact: [How this affects the project]

## Issues Encountered
[For each issue:]
- Issue: [Description]
  - Root cause: [What caused it]
  - Resolution: [How it was fixed, or current status if unresolved]
  - Time spent: [Approximate]
  - Prevention: [How to avoid in future, if applicable]

## Prompts Executed
[For each significant prompt or task in the session:]
1. **Prompt:** [Brief description]
   - **Outcome:** [What was accomplished]
   - **Key Files:** [Files created or changed, if any]
   - **Commit:** [Git commit message or SHA if applicable]

## Prompts Generated for Next Session
[For each prompt generated during this session but NOT executed -- these are prompts created for future work, including Claude Code prompts, fix prompts, audit prompts, or any prompt drafted but deferred. CAPTURE THE FULL BODY VERBATIM, not just a name. Without the full body, the prompt is unrecoverable next session.]

### [Prompt Name or Purpose]
**Intent:** [What problem this prompt is meant to solve]
**Suggested filename:** [e.g., docs/prompts/fix-checkbox-table-width.md]
**Target session/phase:** [When this should run]
**Full prompt body:**
```
[Paste the entire prompt body verbatim, including code blocks, constraints, and output requirements. Do not summarize. Do not abbreviate.]
```

[Repeat block above for each generated prompt. If no prompts were generated for next session, write "None this session."]

## Code Changes
- New files: [List with brief purpose]
- Modified files: [List with what changed]
- Deleted files: [List]

## Technical Notes
[Implementation details, patterns, configurations, or technical context worth preserving.]

## Next Session
- [ ] [Priority task 1]
- [ ] [Priority task 2]
- [ ] [Priority task 3]
</session_full>

---

### CONTEXT UPDATES

<context_updates>
## Context Updates for Next Session

### New Persistent Facts
[Any new project facts, configurations, or constraints established this session that must carry forward. List each as a bullet with enough context to be self-explanatory without the conversation.]

### Changed or Corrected Facts
[Any previously held assumptions or facts that were corrected or superseded this session. Format: "OLD: [what was believed] -> NEW: [what is now true]".]

### Decisions Locked
[Decisions made this session that are final and should not be relitigated. Include the rationale in one sentence so future sessions understand the why.]

### Open Questions
[Unresolved questions or ambiguities that surfaced this session and need answers before or during the next session.]

### Carry-Forward Warnings
[Any known risks, blockers, or gotchas that the next session should be aware of before starting.]
</context_updates>

---

## Closing Line

After both sections, output exactly this line (substituting the session ID):

> Output ready. Session [SESSION-ID] archived.

---

## Guardrails

- Do not leave any placeholder unfilled. If the conversation does not contain enough information to populate a field, omit the field or write "N/A" -- never output raw bracket placeholders like `[What was decided]`.
- **This skill never writes into the repo.** On Claude.ai and Cowork it produces a downloadable file (chat is the fallback); on Claude Code it prints to chat. Only the separate Claude Code slash command writes files to `_sessions/`.
- **Incrementals capture only post-checkpoint work, bounded positionally.** The boundary is the prior checkpoint's position and content in the conversation, never a wall-clock time. Never repeat content already captured in the prior checkpoint. If the prior checkpoint is not visible in context, produce a full and say so in one line rather than guessing the boundary.
- **Ask at most one question, and only on genuine ambiguity** (a prior checkpoint exists but cannot be confirmed as this session). Default to full on an unclear reply. Never ask the user to paste the conversation.
- **Generated prompts must be captured verbatim.** If a prompt was drafted during the session but not executed, paste the entire prompt body into the "Prompts Generated for Next Session" section. Never capture only the name or a summary -- the prompt is unrecoverable without its full body.
- Preserve exact wording for any code, copy, or specific formulations produced during the session. Do not paraphrase deliverables.
- Do not include pleasantries, meta-discussion, or conversational overhead. Only substance.
- Do not add any text above or below the two output sections except the single closing line.
