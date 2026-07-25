@AGENTS.md

## Claude Code specifics

Everything above comes from `AGENTS.md` — the canonical, tool-agnostic orientation. Edit that
file, not this one, for anything another agent should also know. This shell holds only what is
Claude Code-specific.

- The session workflow skills in `.claude/skills/` are available as slash commands here:
  `/init-project`, `/start-session`, `/save-context`, `/end-session`.
- A `PreCompact` hook in `.claude/settings.json` reminds you to checkpoint STATUS before
  compaction. Honor it — run `/save-context` if STATUS is stale before context is condensed.

### Command discipline (keep permission prompts low)

Chained/compound shell commands (`&&`, `;`, pipes, `$(…)`) usually can't be matched against the
permission allowlist, so each forces a manual approval. Default to **one allowlist-friendly command
per Bash call**:

- Don't bundle unrelated commands — run them as separate Bash calls (independent ones in a single
  message still run in parallel).
- Drop cosmetic `echo` headers/separators; put that context in the call's `description`.
- Use the **Grep / Glob / Read** tools instead of `grep` / `find` / `cat` / `head` / `tail`.
- Limit output with flags (`git log -n 10`, `git diff --stat`), not by piping to `head`.
- Avoid `cd X && …`; use absolute paths (the working dir already persists between calls).

Safe, reusable permissions live in tracked `.claude/settings.json`; keep machine-local or secret
overrides in `.claude/settings.local.json` (gitignored). Once your common commands settle, run
`/fewer-permission-prompts` to fold them into the allowlist.
