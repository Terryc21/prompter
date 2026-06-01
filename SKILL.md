---
name: prompter
description: Evaluate whether rewriting the user's prompt for clarity, typo correction, or actionability would meaningfully improve it; rewrite only when it would. Use when the user says "rewrite my prompt," "fix the typos in this prompt," "make this prompt clearer," "make this more actionable," or when a prompt is short and the intent is ambiguous. Display the rewrite and wait for approval.
version: 1.3.0
author: Terry Nyberg, Coffee & Code LLC
license: Apache-2.0
---

# Prompter

## What this skill does

Evaluate whether rewriting the user's prompt would meaningfully improve it for clarity, typo correction, or actionability. When it would, display the rewritten version and wait for approval before taking any action. When the original is already clear and actionable, proceed without rewriting.

## Core rule: evaluate first, rewrite only when it helps

**The default behavior is evaluation, not rewriting.** Most prompts don't need rewriting. Before producing any rewrite, ask: would rewriting this prompt for clarity, typo correction, or actionability **meaningfully improve it?** Only rewrite when there is a real improvement to make:

- Typos to fix
- Ambiguous references to resolve (e.g., "that file" when a specific file name is knowable from context)
- Vague intent to specify (e.g., "make it better" → name the dimensions of "better")
- Missing scope guardrails the user clearly forgot (e.g., "don't edit yet," "ask before pushing")

If the original is already clear and actionable, proceed without rewriting. Emit the canonical "no rewrite needed" notice from the Skip notice format section. **No rewrite is a successful outcome, not a failed one.**

## Skip rewriting for

These categories are NEVER rewritten — they're operational responses, not new prompts:

- Permission responses (yes, no, proceed, approve, looks good, etc.)
- Selecting from options you presented (e.g., "Option 2", "Proceed (Recommended)")
- Follow-up answers to your clarifying questions
- Acceptance of a previously-displayed rewrite (user said "approved", "yes", or similar in response to your "**Rewritten prompt:**" output)

## Activation modes

Three modes are supported.

- **Try it once** — Rewrite just the next prompt, then stop. Good for seeing what prompter does before committing.
- **This session** — Rewrite prompts for the rest of this conversation. No files are modified.
- **Add to CLAUDE.md** — Add a Prompt Rewriting rule to the project's CLAUDE.md so it persists across all future sessions. Pair with "Remove from CLAUDE.md" to uninstall later.

### Initial invocation

When `/skill prompter` is run for the first time in a session, ask the user which mode to activate using AskUserQuestion. The options are the three above.

### Re-invocation in the same session

When the user re-invokes prompter while a mode is already active, **distinguish their intent from the message shape**:

- **Re-invocation with prompt content alongside `/prompter`** (e.g., the user typed a question or task in the same message and appended `/prompter`): the user is signaling "evaluate THIS prompt now." Do NOT re-show the mode picker. Treat the current message's prompt content as the prompt to evaluate under the already-active mode. The mode picker is settings management; it should not interrupt every rewrite.

- **Re-invocation with `/prompter` alone, no other prompt content**: the user is signaling "I want to manage prompter settings." Show the mode picker per the rules below.

When the mode picker IS shown (the standalone case above):

- **Already in "Try it once":** ask whether to restart the counter (treat next prompt as the new "once") or switch modes.
- **Already in "This session":** acknowledge it's already on and offer to switch modes.
- **Already added to CLAUDE.md:** detect the existing section (per "Adding to CLAUDE.md" below) and offer to update or remove it rather than insert a duplicate.

**Why this rule exists:** Conflating "set the mode" with "evaluate THIS prompt" forces the user to re-answer the mode picker on every invocation, even when their mode choice hasn't changed. Origin: 2026-05-31 session where the user invoked `/prompter` five times after writing prompts and got the picker each time despite having chosen "This session" once at the start.

### After "Try it once" expiration

When the single rewrite has been evaluated (whether it produced a rewrite or a "no rewrite needed" notice), show:

> **Prompter:** Done. Like it? Run `/skill prompter` to keep it going — choose "This session" or "Add to CLAUDE.md" to make it permanent.

## How rewriting works (when you do rewrite)

1. **Evaluate first.** Apply the Core rule above. If the prompt is already clear and actionable, emit the "no rewrite needed" notice and stop.
2. **Rewrite** when evaluation says yes: fix typos, clarify intent, make it specific and actionable for Claude Code.
3. **Display** the rewritten prompt with the prefix "**Rewritten prompt:**".
4. **Wait** for the user to approve before proceeding.
5. **If the user approves** (yes, looks good, approved, etc.), execute the rewritten prompt.
6. **If the user corrects the rewrite**, incorporate their feedback and show the updated version.

### Skip notice format

When skipping a rewrite, emit a single-line notice with the prefix `**Prompter:**` followed by the reason. Use one of:

- `**Prompter:** skipped (permission response) — executing immediately.`
- `**Prompter:** skipped (option selection) — executing immediately.`
- `**Prompter:** skipped (follow-up answer) — executing immediately.`
- `**Prompter:** skipped (rewrite approved) — executing rewritten prompt.`
- `**Prompter:** no rewrite needed — prompt is already clear and actionable.`

**Note for downstream parsers.** The `**Prompter:**` prefix is also used by the expiration message (`**Prompter:** Done.`). To disambiguate, match the suffix:
- `^\*\*Prompter:\*\* skipped \(` → skip notice
- `^\*\*Prompter:\*\* no rewrite needed` → no-rewrite-needed notice
- `^\*\*Prompter:\*\* Done\.` → "Try it once" expiration message

## Rewriting style guidelines

When you do rewrite, follow these:

- Fix spelling and grammar.
- Resolve ambiguous references. ("that file" → specific file name if knowable from context.)
- Add specificity where the intent is clear but the wording is vague.
- Keep the user's intent. Don't add scope or change what they're asking for.
- Keep it concise. One to three sentences is ideal.
- Don't add scope creep. If the user asked for a fix, don't rewrite into "fix it and also refactor."

## CLAUDE.md plumbing (Add/Remove modes)

### Adding to CLAUDE.md

When the user chooses "Add to CLAUDE.md", insert this minimal section (it points back at this skill rather than duplicating the skip list — that way the skip list never drifts):

```markdown
## Prompt Rewriting

Before acting on any user prompt, evaluate whether rewriting it for clarity, typo correction, or actionability would meaningfully improve it. **Only rewrite when there is a real improvement to make** — typos to fix, ambiguous references to resolve, or vague intent to specify. If the original prompt is already clear and actionable, proceed without rewriting and without showing a rewrite. When you do rewrite, display the rewritten version with the prefix "**Rewritten prompt:**" and wait for approval before proceeding.

**Full rules** (skip categories, mode behavior, rewriting style): see `/skill prompter` SKILL.md. Do NOT duplicate the skip list here; the skill file is the source of truth.
```

**Detecting an existing section.** A Prompt Rewriting section is considered to exist if the file contains a line matching `^## Prompt Rewriting\b`. The block boundary is from that line to the next `^## ` heading or end of file. If the existing block differs from the template above, do not overwrite — instead, show the user a diff and ask whether to (a) keep their version, (b) replace with the template, or (c) merge specific lines. Default to (a) and proceed without changes.

Regex examples (verify intent if editing):
- ✅ Matches: `## Prompt Rewriting`
- ✅ Matches: `## Prompt Rewriting (project-specific)` — `\b` accepts following space + parens
- ❌ Does not match: `### Prompt Rewriting` — wrong heading level
- ❌ Does not match: `## prompt rewriting` — case-sensitive
- ❌ Does not match: `## Some Prompt Rewriting Notes` — `^` anchors to start of line

### Removing from CLAUDE.md

When the user chooses "Remove from CLAUDE.md":

1. Locate the section using the detection rule above.
2. If the block matches the template verbatim, remove it cleanly — the heading line through the last line before the next `^## ` or end of file.
3. If the block has been modified, show the user the modified content and confirm before removing.
4. If no Prompt Rewriting section exists, say so and stop.
5. Remove a single trailing blank line if the deletion left two consecutive blank lines.

## What was removed in v1.3.0

For maintainers reading this file's history: the following features existed in v1.2.x and were removed in v1.3.0 (2026-05-31). If a user expects them, they're gone:

- **"Next N prompts" mode.** Removed because the countdown-tracking design relied on the LLM emitting and preserving a markdown status line across turns, which is unreliable across context summarization. "This session" + "Add to CLAUDE.md" cover the same use cases without the bookkeeping failure mode.
- **"Flag exemplary rewrites" feature.** Removed because the trigger axes (pinning analytical asks, resolving ambiguous references, surfacing forgotten guardrails) rarely fired in practice, the flag was busywork on every rewrite, and the user can star their own favorites in `examples/Prompter-Examples.md` without needing a per-rewrite marker.
- **Skip-list duplication in the CLAUDE.md template.** Removed because the manual-sync burden (skill + CLAUDE.md template needing to match) was a future bug. The CLAUDE.md template now points at the skill file instead of duplicating its rules.
