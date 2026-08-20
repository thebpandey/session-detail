---
description: Archive the current session to _sessions/ as a full or incremental timestamped markdown file
argument-hint: "[full|inc]  (optional: force a full or incremental archive)"
allowed-tools: Bash(date:*), Bash(ls:*), Bash(mkdir:*), Bash(echo:*), Bash(cat:*), Bash(grep:*), Bash(printf:*), Bash(git:*), Read, Write
---

# Session Detail (Claude Code command)

Generate an archive of the current session and write it to a file in the project. Do not summarize or compress safe content. The file is the deliverable, not the chat output. By default this auto-detects whether a checkpoint already exists for this session and writes an incremental covering only new work; otherwise it writes a full archive.

`$ARGUMENTS` may contain `full` or `inc` to force a mode. Anything else is ignored.

## Sensitive Data Protection (Mandatory)

Before writing the archive, review every candidate excerpt for credentials, authentication material, and other secret values. **Never reproduce a secret value. This requirement overrides every instruction to preserve content verbatim, in full, exactly, without summarization, or without abbreviation.**

Redact all of the following when present:
- Passwords, passphrases, PINs, recovery codes, and backup codes.
- API keys, access tokens, refresh tokens, bearer tokens, session tokens, cookies, and authorization headers.
- OAuth client secrets, webhook secrets, signing secrets, encryption secrets, private keys, and certificate blocks containing private-key material.
- Database credentials and connection strings with embedded usernames, passwords, or tokens.
- Cloud-provider credentials, package-registry tokens, SSH credentials, and secrets copied from `.env` or credential files.
- Any additional value that the user identifies as secret, private, or sensitive.

Preserve useful surrounding structure, labels, variable names, and safe code, but replace each secret value with a typed placeholder such as `OPENAI_API_KEY=[REDACTED: API KEY]` or `Authorization: Bearer [REDACTED: TOKEN]`. Replace an entire private-key or credential block with one placeholder such as `[REDACTED: PRIVATE KEY]`. Never retain a partial value, prefix, suffix, fingerprint, or URL parameter that could help reconstruct or use the secret. If uncertain whether a value is sensitive, redact it.

Never inspect `.env*` files, credential stores, authentication files, SSH private keys, cloud credential files, secret-manager exports, or similar sources solely to include their contents in an archive. Reading a prior session archive is allowed only for checkpoint detection and incremental-boundary comparison; sensitive values found there must still be redacted from the new output.

If any redaction occurs, add this bullet under `## Technical Notes`:
`- Security: Sensitive values were redacted from this archive.`
Do not identify the original value or include enough information to reconstruct it.

## Steps

1. **Read the session key.** Run `echo "$CLAUDE_SESSION_ID"`. If it returns a non-empty value, this is the ground-truth key for matching prior checkpoints from this same session. If it is empty, fall back to date-based matching and mark the marker `session=unverified-<timestamp>`.

2. **Resolve the timestamp.** Run `date "+%Y-%m-%d-%H-%M"`. Use it for the session ID and filename.

3. **Ensure the sessions directory exists and exclude it from Git locally.** Run `mkdir -p _sessions` at the repo root.
   - Run `git rev-parse --is-inside-work-tree`.
   - If the command succeeds and returns `true`, run `git rev-parse --git-path info/exclude` to resolve the repository's local exclude file.
   - Check that the exclude file contains an exact `_sessions/` line. If it does not, append `_sessions/` with `printf`. Do not edit the project's tracked `.gitignore`.
   - Verify the rule with `git check-ignore -q --no-index _sessions/example.md`. If verification fails, stop before writing an archive and report that `_sessions/` could not be excluded safely.
   - If the current directory is not a Git worktree, continue without the exclusion step.

4. **Find any prior checkpoint for THIS session.** Run `ls _sessions/` then, for candidate files, read their first marker line (for example `grep -h "session-detail |" _sessions/*.md`). A prior checkpoint belongs to this session when its marker `session=` equals the current `$CLAUDE_SESSION_ID`. Record the highest `seq` among matches and the `generated` time of that latest match.

5. **Resolve the mode.**
   - If `$ARGUMENTS` contains `full` -> FULL.
   - If `$ARGUMENTS` contains `inc` -> INCREMENTAL. If no prior checkpoint matches this session, print one line saying there is nothing to increment from, then produce a FULL instead.
   - No argument and **no matching prior checkpoint** -> FULL, `seq=1`.
   - No argument and **a matching prior checkpoint with a confirmed `$CLAUDE_SESSION_ID`** -> INCREMENTAL, `seq = highest match seq + 1`.
   - No argument, `$CLAUDE_SESSION_ID` was empty, and **a same-date file exists** but cannot be confirmed as this session -> ASK exactly one line and wait:
     `A prior session-detail checkpoint exists for today but I cannot confirm it is from this session. Generate (i)ncremental from it, or (f)ull? [f]:`
     Default to FULL on an empty or unclear reply.

6. **Derive `sid8`.** Take the session key (from step 1), strip all non-alphanumeric characters, and use the first 8 characters. For a `$CLAUDE_SESSION_ID` UUID this is the first 8 hex characters. For the `unverified-<timestamp>` fallback key this degenerates to `unverifi`; that is acceptable because same-session matching uses the full marker key, never the filename. There is no session counting: no Nth-session-today logic, no directory scans to number sessions, no day-rollover edge.

7. **Build the filename and identifiers** (prefix is always `code` for this command).
   - FULL filename: `_sessions/code-<YYYY-MM-DD-HH-MM>-<sid8>.md`
   - INCREMENTAL filename: `_sessions/code-<YYYY-MM-DD-HH-MM>-<sid8>-inc-<NN>.md` where `NN` equals the marker `seq` and starts at `02`.
   - Session ID label inside the file: the same stem uppercased, for example `code-2026-06-20-09-15-A3F9C12B-INC-02`.

8. **Generate the archive content** using the Output Format below. Populate every field from the actual conversation. Do not leave bracket placeholders; omit a field or mark it `N/A`. Preserve exact wording for non-sensitive code and copy after mandatory sensitive-value redaction.
   - Before finalizing the archive, perform a second sensitive-data review of the complete generated content. Redact any secret value that was missed during extraction.
   - For an INCREMENTAL, the boundary is the matched checkpoint's content, not a wall-clock time: read the matched checkpoint file and capture only work it does not already cover. Conversation messages carry no visible timestamps, so never attempt to apply a time boundary; the marker's `generated` field is metadata, not the boundary mechanism. Do not repeat already-captured content. Begin the body with one line: `Incremental checkpoint, seq <n>, continues from <prior label> generated <time>. Covers only work since that point.`

9. **Stamp the marker** as the very first line inside the file, before the `# Session` heading:
   `<!-- session-detail | tool=code | session=<key> | seq=<n> | generated=<YYYY-MM-DD-HH-MM> -->`

10. **Write the file** to the resolved path using the Write tool only after the Git-exclusion verification and final sensitive-data review succeed.

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
# Session [code-YYYY-MM-DD-HH-MM-SID8 or -INC-NN for an incremental]
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
[Implementation details, patterns, configurations, or technical context worth preserving. If any redaction occurred, include the required security bullet.]

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

- **Sensitive Data Protection is mandatory and takes precedence over every completeness, exact-copy, full-body, and verbatim-preservation instruction in this command.**
- Never place credentials, authentication material, private keys, secret-bearing connection strings, or other secret values in the archive. Replace them with typed redaction placeholders.
- Never read a credential-bearing file solely to archive it. Only read files required for checkpoint detection or incremental comparison, and redact sensitive values from any new output.
- Perform a second sensitive-data review of the complete generated archive before writing it.
- If any redaction occurs, include `- Security: Sensitive values were redacted from this archive.` under `## Technical Notes`.
- Do not leave any bracket placeholder unfilled. Omit the field or write `N/A`.
- Preserve exact wording for non-sensitive code and copy produced this session. Redact sensitive values before applying this requirement.
- Do not write an archive inside a Git worktree until `_sessions/` is verified as locally ignored through the repository's Git exclude file.
- The chat output is the confirmation line plus the y/N prompt only. Nothing else. (On ambiguity, the single detection question is the one allowed exception.)
- **Same-session matching uses `$CLAUDE_SESSION_ID`, never the date alone.** A same-date file is not proof of the same session. Use date only as a fallback when the env var is empty, and ask before treating it as this session's checkpoint.
- **Incrementals capture only post-checkpoint work, bounded by content.** Never repeat content already written in the matched checkpoint. The matched checkpoint's actual content defines the boundary; the marker `seq` and `generated` fields are bookkeeping, not the boundary mechanism.
- This command does not include a "Prompts Generated for Next Session" section by design.
