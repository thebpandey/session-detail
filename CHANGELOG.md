# Changelog

All notable changes to this skill are documented here.

---

## [1.2.0] - 2026-08-20

### Security
- **Mandatory sensitive-value redaction.** Both the skill and Claude Code command now redact passwords, API keys, tokens, cookies, private keys, secret-bearing connection strings, cloud credentials, and other identified secrets before preserving session content.
- **Security rules override verbatim-preservation rules.** Generated prompts, code, and copy remain complete only after sensitive values are replaced with typed redaction placeholders.
- **Credential-file access is restricted.** The skill and command must not inspect `.env*`, credential stores, SSH private keys, cloud credential files, secret-manager exports, or similar sources solely for archival.
- **Final archive review added.** Both the skill and Claude Code command perform a second sensitive-data review of the complete generated archive before delivery or writing.
- **Session archives are excluded from Git.** The Claude Code command adds `_sessions/` to the repository's local Git exclude file and verifies the rule before writing. The skill repository also ignores `_sessions/` directly.

### Changed
- Archives that contain redactions include a security notice under `## Technical Notes` without exposing the original values.
- Documentation now distinguishes complete preservation of safe content from reproduction of sensitive values.

---

## [1.1.0] - 2026-07-06

### Changed
- **File output on Claude.ai and Cowork.** The skill now writes the archive to a downloadable .md file when file creation is available, printing to chat only as a fallback or on explicit request. A 15-20KB archive printed into chat is carried in context on every subsequent turn; a file is not. On Claude Code the skill still prints to chat; the slash command remains the repo file-writing form.
- **Filenames now use a session-ID fragment (`sid8`) instead of session counting.** `code-<timestamp>-<sid8>.md` replaces `code-<timestamp>-session-##.md`. `sid8` is the first 8 alphanumeric characters of the session key. This removes directory scans for session numbering, the "Nth distinct session today" logic, and the midnight-rollover edge case. Old `session-##` files are still detected because same-session matching reads the hidden marker, not the filename.
- **Incremental boundary is now defined positionally, not by timestamp.** An incremental covers conversation content after the prior checkpoint's position (or work not covered by a checkpoint file's content). The marker's `generated` field is demoted to metadata. Conversation messages carry no visible timestamps, so a wall-clock boundary was never executable; this change makes the instruction match what a model can actually do.
- **`## Memory State` removed from the session_full template.** Its content fully overlapped the Context Updates section (persistent facts, locked decisions, carry-forward warnings). Context Updates now solely owns the carry-forward role; the archive body keeps the detail in Technical Notes and Decisions Made.

### Fixed
- **Incremental detection gap on Claude Code (skill form).** The skill prints to chat, so a checkpoint generated earlier in the same Claude Code session left no file in `_sessions/` and was invisible to the disk-only scan. Detection now scans both `_sessions/` markers and the conversation context, using whichever prior checkpoint has the highest `seq`.
- **README placeholder email** replaced with the real contact address.
- **README restart claim softened** from an absolute statement about a specific release to conditional guidance with a restart fallback.

---

## [1.0.0] - Initial release

- Structured session archive with two sections (session_full, context_updates).
- Incremental checkpoint detection with hidden markers, force-full and force-incremental overrides.
- Tool-prefixed filenames (`code-`, `claude-`, `cowork-`).
- Claude Code slash command writing to `_sessions/` with chat-output suppression defaulting to N.
