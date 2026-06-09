# Session Detail

Review the entire current session and generate a structured archive immediately. Do not ask the user for input. Use the actual content of the session to populate every field -- do not leave placeholders unfilled.

**STRICT OUTPUT RULE: Generate exactly two sections -- `<session_full>` and `<context_updates>`. No other sections.**

Derive the following from context:
- **Session date**: use today's date
- **Session ID**: format as `YYYY-MM-DD-HH-MM-SESSION-01` using the timestamp of the first message
- **Duration**: estimate from the first and last message timestamps if available, otherwise omit
- **Project name**: infer from the session, or use "Untitled Project" if unclear

**Pull live git state** using bash tools before generating output:
- `git branch --show-current` for branch name
- `git log -1 --format="%h %s"` for commit SHA and message
- `git status --short` for working tree state

Output must be **pure markdown only**, clean and copy-paste ready. No preamble, no meta-commentary, no instructions to the user above the output. Deliver both sections and close with a single line telling the user the output is ready.

---

## Output Format

Generate **exactly two sections** using the exact XML tags below. No other sections. All content inside the tags must be properly formatted Markdown.

**Required sections:**
1. `<session_full>` -- Full Version
2. `<context_updates>` -- Context Updates

---

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

---

### CONTEXT UPDATES

<context_updates>
## Context Updates for Next Session

### New Persistent Facts
[Any new project facts, configurations, or constraints established this session that must carry forward. List each as a bullet with enough context to be self-explanatory without the session history.]

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

- Do not leave any placeholder unfilled. If the session does not contain enough information to populate a field, omit the field or write "N/A" -- never output raw bracket placeholders like `[What was decided]`.
- Preserve exact wording for any code, copy, or specific formulations produced during the session. Do not paraphrase deliverables.
- Do not include pleasantries, meta-discussion, or conversational overhead. Only substance.
- Do not add any text above or below the two output sections except the single closing line.
- **Style:** no em dashes (use commas, periods, or parentheses). No emojis.
