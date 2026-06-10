# Session Detail

Review the entire current session and generate a structured archive immediately. Do not ask the user for input. Use the actual content of the session to populate every field -- do not leave placeholders unfilled.

Derive the following from context:
- **Session date**: use today's date
- **Session ID**: format as `YYYY-MM-DD-HH-MM-SESSION-01` using the timestamp of the first message
- **Duration**: estimate from the first and last message timestamps if available, otherwise omit
- **Project name**: infer from the session, or use "Untitled Project" if unclear

**Pull live git state** using bash tools before generating output:
- `git branch --show-current` for branch name
- `git log -1 --format="%h %s"` for commit SHA and message
- `git status --short` for working tree state

---

## Output Behavior

**CRITICAL: Do not print the archive content to chat under any circumstances until the user explicitly requests it in step 6.**

1. Generate the full archive content (both sections) in memory only. Do not print it to chat.
2. Determine the output filename using the format `YYYY-MM-DD-HH-MM-session-detail.md`, derived from the timestamp of the first message in the session. If the timestamp is not determinable, use today's date and `0000` for the time component.
3. Create the `_sessions/` directory in the current working directory if it does not already exist, using `mkdir -p _sessions`.
4. Write the full archive content to `_sessions/YYYY-MM-DD-HH-MM-session-detail.md` using bash file-write tools.
5. Print exactly one line to chat -- nothing else:
   `Written to: _sessions/YYYY-MM-DD-HH-MM-session-detail.md`
   If the write fails, print exactly one line:
   `Error: [description of what failed]`
   Do not print anything else regardless of success or failure.
6. Then ask in chat: `Display output in chat? (y/N):` and wait for the user's reply.
   - If the user replies `y` or `Y`, print the full archive content to chat, then close with: `Output ready. Session [SESSION-ID] archived.`
   - If the user replies `n`, `N`, or presses enter with no input, do nothing further. Do not print anything.

---

## Output Format

Generate exactly two sections using the exact XML tags below. All content inside the tags must be properly formatted Markdown.

### FULL VERSION (~15-20KB)

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

## Tasks Executed
[For each significant task or operation in the session:]
1. **Task:** [Brief description]
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
[Key project facts, decisions, and context that should persist across sessions.]

## Next Session
- [ ] [Priority task 1]
- [ ] [Priority task 2]
- [ ] [Priority task 3]
</session_full>

### CONTEXT UPDATES

<context_updates>
## Context Updates for Next Session

### New Persistent Facts
[Any new project facts, configurations, or constraints established this session that must carry forward.]

### Changed or Corrected Facts
[Format: "OLD: [what was believed] -> NEW: [what is now true]".]

### Decisions Locked
[Decisions made this session that are final and should not be relitigated. Include rationale in one sentence.]

### Open Questions
[Unresolved questions or ambiguities that need answers before or during the next session.]

### Carry-Forward Warnings
[Known risks, blockers, or gotchas the next session should be aware of before starting.]
</context_updates>

---

## Guardrails

- **Do not print archive content to chat** unless the user explicitly replies `y` in step 6. This is the highest-priority rule in this file.
- Do not leave any placeholder unfilled. If the session does not contain enough information to populate a field, omit the field or write "N/A".
- Preserve exact wording for any code, copy, or specific formulations produced during the session. Do not paraphrase deliverables.
- Do not include pleasantries, meta-discussion, or conversational overhead. Only substance.
- **Style:** no em dashes (use commas, periods, or parentheses). No emojis.
