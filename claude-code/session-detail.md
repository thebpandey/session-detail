---
description: Archive the current session to _sessions/ as a full or incremental timestamped markdown file
argument-hint: "[full|inc]  (optional: force a full or incremental archive)"
allowed-tools: Bash(date:*), Bash(ls:*), Bash(mkdir:*), Bash(echo:*), Bash(cat:*), Bash(grep:*), Read, Write
---

# Session Detail (Claude Code command)

Generate an archive of the current session and write it to a file in the project. Do not summarize or compress. The file is the deliverable, not the chat output. By default this auto-detects whether a checkpoint already exists for this session and writes an incremental covering only new work; otherwise it writes a full archive.

`$ARGUMENTS` may contain `full` or `inc` to force a mode. Anything else is ignored.

## Steps

1. **Read the session key.** Run `echo "$CLAUDE_SESSION_ID"`. If it returns a non-empty value, this is the ground-truth key for matching prior checkpoints from this same session. If it is empty, fall back to date-based matching and mark the marker `session=unverified-<timestamp>`.

2. **Resolve the timestamp.** Run `date "+%Y-%m-%d-%H-%M"`. Use it for the session ID and filename.

3. **Ensure the sessions directory exists.** Run `mkdir -p _sessions` at the repo root.

4. **Find any prior checkpoint for THIS session.** Run `ls _sessions/` then, for candidate files, read their first marker line (for example `grep -h "session-detail |" _sessions/*.md`). A prior checkpoint belongs to this session when its marker `session=` equals the current `$CLAUDE_SESSION_ID`. Record the highest `seq` among matches and the `generated` time of that latest match.

5. **Resolve the mode.**
   - If `$ARGUMENTS` contains `full` -> FULL.
   - If `$ARGUMENTS` contains `inc` -> INCREMENTAL. If no prior checkpoint matches this session, print one line saying there is nothing to increment from, then produce a FULL instead.
   - No argument and **no matching prior checkpoint** -> FULL, `seq=1`.
   - No argument and **a matching prior checkpoint with a confirmed `$CLAUDE_SESSION_ID`** -> INCREMENTAL, `seq = highest match seq + 1`.
   - No argument, `$CLAUDE_SESSION_ID` was empty, and **a same-date file exists** but cannot be confirmed as this session -> ASK exactly one line and wait:
     `A prior session-detail checkpoint exists for today but I cannot confirm it is from this session. Generate (i)ncremental from it, or (f)ull? [f]:`
     Default to FULL on an empty or unclear reply.

6. **Resolve the session number `##`.** This is the Nth distinct session today. Among today's `code-<date>-session-##` base files in `_sessions/`, for a FULL take the highest `##` and add one (start at `01` if none). For an INCREMENTAL, reuse the `##` of the matched session.

7. **Build the filename and identifiers** (prefix is always `code` for this command).
   - FULL filename: `_sessions/code-<YYYY-MM-DD-HH-MM>-session-<##>.md`
   - INCREMENTAL filename: `_sessions/code-<YYYY-MM-DD-HH-MM>-session-<##>-inc-<NN>.md` where `NN` starts at `02` and increments per checkpoint.
   - Session ID label inside the file: the same stem uppercased, for example `code-2026-06-20-09-15-SESSION-01-INC-02`.

8. **Generate the archive content** using the Output Format below. Populate every field from the actual conversation. Do not leave bracket placeholders; omit a field or mark it `N/A`. Preserve exact wording for any code or specific copy produced this session.
   - For an INCREMENTAL, capture only work done AFTER the matched checkpoint's `generated` time. Do not repeat already-captured content. Begin the body with one line: `Incremental checkpoint, seq <n>, continues from <prior label> generated <time>. Covers only work since that point.`

9. **Stamp the marker** as the very first line inside the file, before the `# Session` heading:
   `<!-- session-detail | tool=code | session=<key> | seq=<n> | generated=<YYYY-MM-DD-HH-MM> -->`

10. **Write the file** to the resolved path using the Write tool.

11. **Confirm with a single line only.** After writing, output exactly one line:
    `Session <SESSION-ID> archived to _sessions/<filename>.`

12. **Prompt before displaying.** On the next line, output exactly:
    `Display output in chat? (y/N):`
    Then stop and wait. The default is N. Only print the archive body to chat if the user replies `y` or `yes`. For N, an empty reply, or anything else, do not print the content.

## CRITICAL OUTPUT RULE

Do not print the archive body to chat while generating it. The file on disk is the deliverable. Chat receives only the one-line confirmation and the `Display output in chat? (y/N):` prompt. Dumping the full content to chat defeats the purpose of writing it to disk and is the primary failure mode this command guards against.

## Output Format

Write the file with exactly two sections, in this order. No other sections.

<session_full>
<!-- session-detail | tool=code | session=[key] | seq=[n] | generated=[YYYY-MM-DD-HH-MM] -->
# Session [code-YYYY-MM-DD-HH-MM-SESSION-## or -INC-NN for an incremental]
Date: [YYYY-MM-DD]
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

## Code Changes
- New files: [List with brief purpose]
- Modified files: [List with what changed]
- Deleted files: [List]

## Technical Notes
[Implementation details, patterns, configurations, or technical context worth preserving.]

## Memory State
[Key project facts, decisions, and context that should persist across conversations.]

## Next Session
- [ ] [Priority task 1]
- [ ] [Priority task 2]
- [ ] [Priority task 3]
</session_full>

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

## Guardrails

- Do not leave any bracket placeholder unfilled. Omit the field or write `N/A`.
- Preserve exact wording for code and specific copy produced this session. Do not paraphrase deliverables.
- The chat output is the confirmation line plus the y/N prompt only. Nothing else. (On ambiguity, the single detection question is the one allowed exception.)
- **Same-session matching uses `$CLAUDE_SESSION_ID`, never the date alone.** A same-date file is not proof of the same session. Use date only as a fallback when the env var is empty, and ask before treating it as this session's checkpoint.
- **Incrementals capture only post-checkpoint work.** Never repeat content already written in the matched checkpoint. The marker `seq` and `generated` fields define the boundary.
- This command does not include a "Prompts Generated for Next Session" section by design.
