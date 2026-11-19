# Upgrading prompter

*Moved out of the README on 2026-08-28 so the landing page stays a 4-minute read. Nothing was
cut — this is the full upgrade guide.*

## v1.3.0 to v1.4.0

Nothing was removed, and nothing you rely on stops working. The changes:

- **The command is `/prompter`.** Earlier docs said `/skill prompter`, which isn't a Claude Code command. A plugin install uses `/prompter:prompter`.
- **The mode menu appears less often.** It now shows only when you type `/prompter` by itself. A direct request ("make this prompt clearer: …") gets the rewrite straight away, and so does a session started in a project whose CLAUDE.md has the Prompt Rewriting rule.
- **"Remove from CLAUDE.md" is on the menu** whenever the rule exists, and the menu says which file it's in.
- **"Add to CLAUDE.md" asks where.** Choose this project (`CLAUDE.md` at the project root) or all projects (`~/.claude/CLAUDE.md`).
- **Declining a rewrite runs your original prompt** as you wrote it.
- **Slash commands and pasted material are left alone.** Logs, stack traces, code and diffs are no longer candidates for rewriting.
- **Rewrites follow named patterns** (diagnose before fixing, pin the dimensions, name the deliverable, and others), so they look more like the worked examples.
- **The CLAUDE.md template's last line changed.** It now tells Claude to load the `prompter` skill and skip its menu. Existing blocks keep working. If you re-run "Add to CLAUDE.md" where an older block exists, the skill notices the difference and asks whether to keep, replace or merge; it keeps yours by default.

To upgrade, follow "How to upgrade" below. For a plugin install, run `claude plugin update prompter@prompter` in a terminal instead.

## v1.2.x to v1.3.0

v1.3.0 is a backward-incompatible rewrite. If you installed prompter before May 31, 2026, the version on disk is older and won't match this README. **You will not get errors** — Claude Code will keep using the old SKILL.md happily — but the behavior will not match what's documented here until you upgrade.

### What changed

- **"Next N prompts" mode is gone.** The countdown-tracking relied on the LLM emitting and preserving a markdown status line across turns, which is unreliable across context summarization. Use "This session" (covers the same use case for one conversation) or "Add to CLAUDE.md" (covers the cross-session case). If your habit was `/prompter` → "Next N prompts" → "10", change it to `/prompter` → "This session" — same end result, no fragile bookkeeping.
- **"Flag exemplary rewrites" feature is gone.** The trigger axes rarely fired in practice and the per-rewrite flag was busywork. Star your own favorites in `examples/Prompter-Examples.md` if you want a curated list.
- **CLAUDE.md template changed.** v1.2.x installed a 14-line block that duplicated the skip list from SKILL.md. v1.3.0 installs a 6-line block that points at SKILL.md instead. This prevents the skip list from silently drifting across the two files. If you have a v1.2.x block in a project's CLAUDE.md and you re-run "Add to CLAUDE.md," the skill will detect the difference and ask whether to keep, replace, or merge — your existing block is preserved by default.
- **Re-invocation behavior changed.** v1.2.x always re-showed the mode picker on every `/prompter` invocation. v1.3.0 skips the picker if your message has prompt content alongside `/prompter` (signaling "evaluate THIS prompt now") and only shows it when you type `/prompter` alone (signaling "I want to manage settings"). This removes the friction of re-answering the picker on every rewrite.

### How to upgrade

```bash
# Pull the latest from GitHub.
cd /path/to/your/local/prompter && git pull origin main

# OR re-clone fresh.
git clone https://github.com/Terryc21/prompter.git /tmp/prompter-latest
```

Then copy over the install location. The skill now lives in `skills/prompter/` inside the repo (earlier versions kept `SKILL.md` at the repo root), so copy from there. The new SKILL.md will replace the old one in place:

```bash
# Global install upgrade:
cp /tmp/prompter-latest/skills/prompter/SKILL.md ~/.claude/skills/prompter/SKILL.md

# OR replace the whole skill folder (preserves nothing custom):
rm -rf ~/.claude/skills/prompter && cp -r /tmp/prompter-latest/skills/prompter ~/.claude/skills/prompter
```

If you customized SKILL.md, diff before overwriting:

```bash
diff ~/.claude/skills/prompter/SKILL.md /tmp/prompter-latest/skills/prompter/SKILL.md
```

**Restart Claude Code after replacing SKILL.md.** The skill file is read at session start; existing sessions will keep using the version they loaded.

### CLAUDE.md sections in your projects

If you've used "Add to CLAUDE.md" in multiple projects under v1.2.x, those projects all have the v1.2.x-style 14-line block with the skip list inline. They will keep working — the behavior described by the v1.2.x block is consistent with v1.3.0. You don't need to migrate them unless you want the cleaner v1.3.0 template (which avoids the drift risk).

To upgrade a project's CLAUDE.md to the v1.3.0 template:
1. Run `/prompter` → "Add to CLAUDE.md" in that project.
2. The skill detects the existing block, sees it differs from the v1.3.0 template, and asks whether to (a) keep yours, (b) replace with the template, or (c) merge specific lines.
3. Pick (b) to upgrade or (a) to keep your customizations.

If you have customizations in the v1.2.x block (added project-specific rules), pick (c) and merge by hand.

---
