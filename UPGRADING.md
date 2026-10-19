# Upgrading prompter from v1.2.x

*Moved out of the README on 2026-08-28 so the landing page stays a 4-minute read. Nothing was
cut — this is the full upgrade guide.*

v1.3.0 is a backward-incompatible rewrite. If you installed prompter before May 31, 2026, the version on disk is older and won't match this README. **You will not get errors** — Claude Code will keep using the old SKILL.md happily — but the behavior will not match what's documented here until you upgrade.

### What changed

- **"Next N prompts" mode is gone.** The countdown-tracking relied on the LLM emitting and preserving a markdown status line across turns, which is unreliable across context summarization. Use "This session" (covers the same use case for one conversation) or "Add to CLAUDE.md" (covers the cross-session case). If your habit was `/skill prompter` → "Next N prompts" → "10", change it to `/skill prompter` → "This session" — same end result, no fragile bookkeeping.
- **"Flag exemplary rewrites" feature is gone.** The trigger axes rarely fired in practice and the per-rewrite flag was busywork. Star your own favorites in `examples/Prompter-Examples.md` if you want a curated list.
- **CLAUDE.md template changed.** v1.2.x installed a 14-line block that duplicated the skip list from SKILL.md. v1.3.0 installs a 6-line block that points at SKILL.md instead. This prevents the skip list from silently drifting across the two files. If you have a v1.2.x block in a project's CLAUDE.md and you re-run "Add to CLAUDE.md," the skill will detect the difference and ask whether to keep, replace, or merge — your existing block is preserved by default.
- **Re-invocation behavior changed.** v1.2.x always re-showed the mode picker on every `/prompter` invocation. v1.3.0 skips the picker if your message has prompt content alongside `/prompter` (signaling "evaluate THIS prompt now") and only shows it when you type `/prompter` alone (signaling "I want to manage settings"). This removes the friction of re-answering the picker on every rewrite.

### How to upgrade

```bash
# Pull the latest from GitHub.
cd /path/to/your/local/prompter && git pull origin main

# OR re-clone fresh.
git clone https://github.com/Terryc21/prompter.git /tmp/prompter-v1.3.0
```

Then copy over the install location. The skill now lives in `skills/prompter/` inside the repo (earlier versions kept `SKILL.md` at the repo root), so copy from there. The new SKILL.md will replace the old one in place:

```bash
# Global install upgrade:
cp /tmp/prompter-v1.3.0/skills/prompter/SKILL.md ~/.claude/skills/prompter/SKILL.md

# OR replace the whole skill folder (preserves nothing custom):
rm -rf ~/.claude/skills/prompter && cp -r /tmp/prompter-v1.3.0/skills/prompter ~/.claude/skills/prompter
```

If you customized SKILL.md, diff before overwriting:

```bash
diff ~/.claude/skills/prompter/SKILL.md /tmp/prompter-v1.3.0/skills/prompter/SKILL.md
```

**Restart Claude Code after replacing SKILL.md.** The skill file is read at session start; existing sessions will keep using the version they loaded.

### CLAUDE.md sections in your projects

If you've used "Add to CLAUDE.md" in multiple projects under v1.2.x, those projects all have the v1.2.x-style 14-line block with the skip list inline. They will keep working — the behavior described by the v1.2.x block is consistent with v1.3.0. You don't need to migrate them unless you want the cleaner v1.3.0 template (which avoids the drift risk).

To upgrade a project's CLAUDE.md to the v1.3.0 template:
1. Run `/skill prompter` → "Add to CLAUDE.md" in that project.
2. The skill detects the existing block, sees it differs from the v1.3.0 template, and asks whether to (a) keep yours, (b) replace with the template, or (c) merge specific lines.
3. Pick (b) to upgrade or (a) to keep your customizations.

If you have customizations in the v1.2.x block (added project-specific rules), pick (c) and merge by hand.

---
