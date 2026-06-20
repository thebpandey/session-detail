---
description: Archive the current session verbatim to _sessions/ as a timestamped markdown file
allowed-tools: Bash(date:*), Bash(ls:*), Bash(mkdir:*), Write
---

# Session Detail (Claude Code command)

Generate a complete, verbatim archive of the current session and write it to a file in the project. Do not summarize or compress. The file is the deliverable, not the chat output.

## Steps

1. **Resolve the timestamp.** Run `date "+%Y-%m-%d-%H-%M"` to get the current timestamp. Use it for both the session ID and the filename.

2. **Ensure the sessions directory exists.** Run `mkdir -p _sessions` at the repo root.

3. **Resolve the session number.** Run `ls _sessions/` and find any files whose name begins with today's `YYYY-MM-DD` prefix. Take the highest existing `session-##` number for today and add one. If there are none for today, start at `01`. Always zero-pad to two digits.

4. **Build the filename and session ID.**
   - Filename: `_sessions/<YYYY-MM-DD-HH-MM>-session-<##>.md`
   - Session ID inside the file: `<YYYY-MM-DD-HH-MM>-SESSION-<##>`

5. **Generate the archive content** using the exact structure in the Output Format section below. Populate every field from the actual conversation. Do not leave bracket placeholders; omit a field or mark it `N/A` if the conversation does not supply it. Preserve exact wording for any code or specific copy produced this session.

6. **Write the file** to the resolved path using the Write tool.

7. **Confirm with a single line only.** After writing, output exactly one line:
   `Session <SESSION-ID> archived to _sessions/<filename>.`

8. **Prompt before displaying.** On the next line, output exactly:
   `Display output in chat? (y/N):`
   Then stop and wait. The default is N. Only print the archive body to chat if the user replies `y` or `yes`. For N, an empty reply, or anything else, do not print the content.

## CRITICAL OUTPUT RULE

Do not print the archive body to chat while generating it. The file on disk is the deliverable. Chat receives only the one-line confirmation and the `Display output in chat? (y/N):` prompt. Dumping the full content to chat defeats the purpose of writing it to disk and is the primary failure mode this command guards against.

## Output Format

Write the file with exactly two sections, in this order. No other sections.

<session_full>
# Session [YYYY-MM-DD-HH-MM-SESSION-##]
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
- The chat output is the confirmation line plus the y/N prompt only. Nothing else.
- This command does not include a "Prompts Generated for Next Session" section by design.
