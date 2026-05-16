---
name: prompter
description: Evaluate whether rewriting the user's prompt for clarity, typo correction, or actionability would meaningfully improve it; rewrite only when it would. Use when the user says "rewrite my prompt," "fix the typos in this prompt," "make this prompt clearer," "make this more actionable," or when a prompt is short and the intent is ambiguous. Display the rewrite and wait for approval.
version: 1.2.1
author: Terry Nyberg, Coffee & Code LLC
license: Apache-2.0
---

# Prompter

Evaluate whether rewriting the user's prompt would meaningfully improve it for clarity, typo correction, or actionability. When it would, display the rewritten version and wait for approval before taking any action. When the original is already clear and actionable, proceed without rewriting.

## On invocation

When `/skill prompter` is run, ask the user how long prompt rewriting should stay active:

- **Try it once** -- Rewrite just the next prompt, then stop. Good for seeing what prompter does before committing.
- **Next N prompts** -- Ask how many prompts to rewrite (e.g., 5, 10), then count down. Show remaining count after each rewrite.
- **This session** -- Rewrite prompts for the rest of this conversation. No files are modified.
- **Add to CLAUDE.md** -- Add a Prompt Rewriting rule to the project's CLAUDE.md so it persists across all future sessions.
- **Remove from CLAUDE.md** -- Remove the Prompt Rewriting rule from CLAUDE.md to stop persistent rewriting.

Then follow the rules below for the chosen duration.

### Re-invocation in the same session

If the user runs `/skill prompter` again while a mode is already active:

- **Already in "Try it once" or "Next N prompts":** ask whether to restart the counter or switch modes. Don't reset silently.
- **Already in "This session":** acknowledge it's already on and re-show the mode menu — the user may want a different mode.
- **Already added to CLAUDE.md:** detect the existing section (per the rules in "Adding to CLAUDE.md" below) and offer to update or remove it rather than insert a duplicate.

### After expiration

When "Try it once" or "Next N prompts" expires, show:

> **Prompter:** Done. Like it? Run `/skill prompter` to keep it going -- choose "This session" or "Add to CLAUDE.md" to make it permanent.

### Counting behavior (N prompts)

When the user chooses "Next N prompts":
1. Ask how many prompts to rewrite
2. After each rewrite, show the remaining count (e.g., "**Prompter:** 4 rewrites remaining")
3. When the count reaches 0, show the expiration message above.

### Where N is tracked

The countdown lives in the conversation context as a single status line the LLM repeats and updates each turn. The `**Prompter:** N rewrites remaining` line is the source of truth — emit it after every rewrite so the count survives context summarization (summarizers preserve recent factual lines more reliably than mental state).

If the user notices the count appears wrong after a compaction or `/clear`, treat the most recent status line in context as truth. If no status line exists (e.g., a fresh session), the countdown is considered expired and the skill stops rewriting until re-invoked.

The countdown does NOT persist across separate Claude Code sessions. "Next N prompts" is single-session by design. Use "Add to CLAUDE.md" for cross-session persistence.

**Operational caveat.** Treating a markdown line as durable state is a real bet, not a guarantee. Summarizers preserve recent factual lines well in practice, but a sufficiently aggressive compaction or `/clear` can drop the status line entirely — at which point the countdown silently resets and the skill stops rewriting until re-invoked. For runs longer than ~10-15 exchanges, prefer "Add to CLAUDE.md" instead of "Next N prompts." If the count must survive a known-long conversation, periodically restate it in the conversation (e.g., ask "what's the current count?") so the summarizer has a fresh anchor to preserve.

### Adding to CLAUDE.md

The template below is a starting point. Users frequently evolve their Prompt Rewriting rule after using prompter for a while — adding "evaluate whether rewriting would meaningfully improve it," tightening the skip list, adding project-specific conventions. This is normal and expected; the detection rules below are designed to preserve those edits rather than overwrite them.

**Detecting an existing section.** A Prompt Rewriting section is considered to exist if the file contains a line matching `^## Prompt Rewriting\b`. The block boundary is from that line to the next `^## ` heading or end of file. If the existing block differs from the template below, do not overwrite — instead, show the user a diff and ask whether to (a) keep their version, (b) replace with the template, or (c) merge specific lines. Default to (a) and proceed without changes.

Regex examples (verify intent if editing):
- ✅ Matches: `## Prompt Rewriting`
- ✅ Matches: `## Prompt Rewriting (project-specific)` — `\b` accepts following space + parens
- ❌ Does not match: `### Prompt Rewriting` — wrong heading level
- ❌ Does not match: `## prompt rewriting` — case-sensitive
- ❌ Does not match: `## Some Prompt Rewriting Notes` — `^` anchors to start of line

When the user chooses "Add to CLAUDE.md", insert this section if no Prompt Rewriting section exists, or follow the detection rule above if one does:

```markdown
## Prompt Rewriting

Before acting on any user prompt, evaluate whether rewriting it for clarity, typo correction, or actionability would meaningfully improve it. **Only rewrite when there is a real improvement to make** — typos to fix, ambiguous references to resolve, or vague intent to specify. If the original prompt is already clear and actionable, proceed without rewriting and without showing a rewrite. When you do rewrite, display the rewritten version with the prefix "**Rewritten prompt:**" and wait for approval before proceeding. See `/skill prompter` for full rules.

<!-- Source of truth for the skip list: prompter SKILL.md § Skip rewriting for. If updating one, update both. -->

**Skip rewriting for:**
- Permission responses (yes, no, proceed, etc.)
- Selecting from options you presented (e.g., "Option 2", "Proceed (Recommended)")
- Follow-up answers to your clarifying questions
```

### Removing from CLAUDE.md

When the user chooses "Remove from CLAUDE.md":

1. Locate the section using the detection rule from "Adding to CLAUDE.md" above
2. If the block matches the template verbatim, remove it cleanly — the heading line through the last line before the next `^## ` or end of file
3. If the block has been modified, show the user the modified content and confirm before removing
4. If no Prompt Rewriting section exists, say so and stop
5. Remove a single trailing blank line if the deletion left two consecutive blank lines

## Rules

Rules below govern the evaluate/rewrite/display/skip behavior. Activation modes (Try once, Next N, This session, Add/Remove from CLAUDE.md) are in "On invocation" above; rewriting style is in "Rewriting guidelines" below.

1. **Evaluate first, then rewrite.** Before producing any rewrite, ask: would rewriting this prompt for clarity, typo correction, or actionability **meaningfully improve it?** Only rewrite when there is a real improvement to make — typos to fix, ambiguous references to resolve, vague intent to specify, or missing scope guardrails. If the original is already clear and actionable, proceed without rewriting (emit the canonical "no rewrite needed" notice from Rule 7). The default behavior is *evaluation*, not rewriting.
2. **Rewrite** when evaluation says yes: fix typos, clarify intent, make it specific and actionable for Claude Code
3. **Display** the rewritten prompt with the prefix "**Rewritten prompt:**"
4. **Wait** for the user to approve before proceeding
5. If the user approves (yes, looks good, etc.), execute the rewritten prompt
6. If the user corrects the rewrite, incorporate their feedback and show the updated version
7. **Skip notice format.** When skipping a rewrite, emit a single-line notice with the prefix `**Prompter:**` followed by the reason. Use one of:
   - `**Prompter:** skipped (permission response) — executing immediately.`
   - `**Prompter:** skipped (option selection) — executing immediately.`
   - `**Prompter:** skipped (follow-up answer) — executing immediately.`
   - `**Prompter:** no rewrite needed — prompt is already clear and actionable.`

**Note for downstream parsers.** The `**Prompter:**` prefix is shared between skip notices (Rule 7) and the N-prompts countdown status line (`**Prompter:** N rewrites remaining`). To disambiguate, match the suffix:
- `^\*\*Prompter:\*\* skipped \(` → skip notice (permission, option, or follow-up)
- `^\*\*Prompter:\*\* no rewrite needed` → no-rewrite-needed notice
- `^\*\*Prompter:\*\* \d+ rewrites remaining` → N-prompts countdown
- `^\*\*Prompter:\*\* Done\.` → expiration message

## Skip rewriting for

This list is the canonical source of truth. The same list also appears inside the CLAUDE.md template above so it ships intact when "Add to CLAUDE.md" runs — that's an accepted duplication, not a bug. The HTML comment in the template (`Source of truth for the skip list: prompter SKILL.md § Skip rewriting for`) reminds future editors to update both. Keep the *categories* identical between the two lists; wording variations (second-person vs neutral) are acceptable as long as no category is added, removed, or semantically narrowed in one place but not the other.

- Permission responses (yes, no, proceed, etc.)
- Selecting from options you presented (e.g., "Option 2", "Proceed (Recommended)")
- Follow-up answers to your clarifying questions

## Rewriting guidelines

- **Only rewrite when there is a real improvement to make.** Typos to fix, ambiguous references to resolve, vague intent to specify, missing scope guardrails. Evaluation comes first; rewriting comes only if evaluation says yes.
- Fix spelling and grammar
- Resolve ambiguous references ("that file" -> specific file name if known from context)
- Add specificity where the intent is clear but the wording is vague
- Keep the user's intent -- don't add scope or change what they're asking for
- Keep it concise -- one to three sentences is ideal
- If the prompt is already clear and actionable, say so and proceed without rewriting
- **Flag rewrites worth sharing back.** If a rewrite materially improves the prompt by (a) pinning an under-specified analytical ask into a deliverable shape, (b) resolving an ambiguous reference using non-obvious context, or (c) surfacing a guardrail the user clearly forgot ("don't edit yet," "don't push," "ask before X"), the user may want to consider it as a candidate example for prompter's own example collection. This is a flag, not a quality verdict; don't apply it to routine typo corrections or minor sharpening.
