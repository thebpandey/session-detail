---
name: session-detail
description: "Generates a structured, detailed session archive in markdown format for long-term logging, record-keeping, or session handoff. Triggers on: \"session detail\", \"detailed session summary\", \"session archive\". Always trigger this skill on these exact phrases. Do not answer them directly without consulting this skill. Output is copy-paste ready markdown with exactly two sections: Full Version (session_full) and Context Updates (context_updates)."
---

# Session Detail Skill

## Behavior on Invocation

When this skill triggers, **do not ask the user for input.** Review the entire current conversation history and generate both the Full Version and Context Updates immediately. Use the actual content of the conversation to populate every field -- do not leave placeholders unfilled.

**STRICT OUTPUT RULE: Generate exactly two sections -- `<session_full>` and `<context_updates>`. No other sections.**

Derive the following from context:
- **Session date**: use today's date
- **Session ID**: format as `YYYY-MM-DD-HH-MM-SESSION-01` using the timestamp of the first message
- **Duration**: estimate from the first and last message timestamps if available, otherwise omit
- **Project name**: infer from the conversation, or use "Untitled Project" if unclear

Output must be **pure markdown only**, clean and copy-paste ready. No preamble, no meta-commentary, no instructions to the user above the output. Deliver the Full Version and close with a single line telling the user the output is ready.

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

## Memory State
[Key project facts, decisions, and context that should persist across conversations.]

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
- **Generated prompts must be captured verbatim.** If a prompt was drafted during the session but not executed, paste the entire prompt body into the "Prompts Generated for Next Session" section. Never capture only the name or a summary -- the prompt is unrecoverable without its full body.
- Preserve exact wording for any code, copy, or specific formulations produced during the session. Do not paraphrase deliverables.
- Do not include pleasantries, meta-discussion, or conversational overhead. Only substance.
- Do not add any text above or below the two output sections except the single closing line.
