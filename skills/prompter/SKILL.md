---
name: prompter
description: Evaluate whether rewriting the user's prompt for clarity, typo correction, or actionability would meaningfully improve it; rewrite only when it would. Use when the user says "rewrite my prompt," "fix the typos in this prompt," "make this prompt clearer," "make this more actionable," or when a prompt is short and the intent is ambiguous. Display the rewrite and wait for approval.
version: 1.4.0
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
- Slash commands, and messages that only invoke another skill or command (e.g., `/commit`, `/review`)
- Pasted material: logs, stack traces, error output, code, diffs, or file contents. When a message pairs pasted material with a short instruction, evaluate only the instruction and leave the pasted material untouched.

## Activation modes

Four modes are supported.

- **Try it once:** rewrite just the next prompt, then stop. Good for seeing what prompter does before committing.
- **This session:** rewrite prompts for the rest of this conversation. No files are modified.
- **Add to CLAUDE.md:** add a Prompt Rewriting rule to a CLAUDE.md file so it persists across future sessions, for this project or for all projects (see "CLAUDE.md plumbing" below).
- **Remove from CLAUDE.md:** remove that rule.

The skill is invoked as `/prompter` when installed by copying it into a skills folder, and as `/prompter:prompter` when installed as a plugin. This file says `/prompter` for both.

**"Try it once" state.** Whether the "once" has already fired is tracked the same way as everything else in this skill: from the most recent "**Prompter:**"-prefixed message in context (a rewrite, a skip notice, or the expiration message below). If context has been summarized or compacted since activation and no such message is visible, treat the "once" as expired rather than assuming it's still pending; the user can run `/prompter` again to restart it. This is the same reason the old "Next N prompts" mode was removed in v1.3.0: don't trust an implicit turn count that summarization can silently reset.

### How the skill was activated decides whether to show the mode menu

The mode menu is settings management. Show it only when the user is asking to manage settings. Work out which case applies before doing anything else:

1. **`/prompter` alone, with no other prompt content.** The user wants to manage settings. Show the mode menu (see "Showing the mode menu" below).
2. **`/prompter` with prompt content in the same message.** The user is saying "evaluate THIS prompt now." Do not show the menu. Evaluate the prompt content under the active mode, or, if no mode is active, as a one-off ("Try it once" behavior, including the expiration message).
3. **A direct request to rewrite a specific prompt** (e.g., "make this prompt clearer: 'handle the edge case'", "fix the typos in this prompt: …"). Do not show the menu. Evaluate the quoted prompt as a one-off and show the result. Do not continue rewriting later prompts unless a mode is active.
4. **You activated the skill yourself** because a prompt looked short and ambiguous. Do not show the menu. Evaluate that prompt as a one-off.
5. **A CLAUDE.md file contains a `## Prompt Rewriting` section** and you loaded this skill to get its full rules. Do not show the menu. The user already chose the persistent mode when they added that section. Evaluate the current prompt, and keep evaluating prompts for the rest of the session.

**Why this rule exists:** Conflating "set the mode" with "evaluate THIS prompt" forces the user to answer the menu when they only wanted a rewrite. Origin: 2026-05-31 session where the user invoked `/prompter` five times after writing prompts and got the menu each time despite having chosen "This session" once at the start.

### Showing the mode menu

Before showing the menu, check for an existing Prompt Rewriting section in every CLAUDE.md location (see "Where the rule goes" below). Then ask with AskUserQuestion:

- Always offer **Try it once** and **This session**.
- Offer **Add to CLAUDE.md** unless the section already exists in both the project and the global file.
- Offer **Remove from CLAUDE.md** only when a section exists somewhere. Say which file it is in.

If a mode is already active in this session:

- **Already in "Try it once":** offer to restart it (treat the next prompt as the new "once") or switch modes.
- **Already in "This session":** say it's already on and offer to switch modes.

**If AskUserQuestion isn't available** (for example, a non-interactive `claude -p` run), don't stall. List the modes as plain text and ask the user to reply with one. If nobody can reply and the message has prompt content, evaluate that prompt as a one-off.

### After "Try it once" expiration

When the single rewrite has been evaluated (whether it produced a rewrite or a "no rewrite needed" notice), show:

> **Prompter:** Done. Like it? Run `/prompter` again and choose "This session" or "Add to CLAUDE.md" to keep it going.

## How rewriting works (when you do rewrite)

1. **Evaluate first.** Apply the Core rule above. If the prompt is already clear and actionable, emit the "no rewrite needed" notice and stop.
2. **Rewrite** when evaluation says yes: fix typos, clarify intent, make it specific and actionable for Claude Code.
3. **Display** the rewritten prompt with the prefix "**Rewritten prompt:**".
4. **Wait** for the user to approve before proceeding.
5. **If the user approves** (yes, looks good, approved, etc.), execute the rewritten prompt.
6. **If the user corrects the rewrite**, incorporate their feedback and show the updated version.
7. **If the user declines the rewrite** ("no", "use mine", "run the original"), run their original prompt exactly as written, with no further rewrite. If the reply makes clear they want nothing run at all ("never mind", "stop"), do nothing.

### Skip notice format

When skipping a rewrite, emit a single-line notice with the prefix `**Prompter:**` followed by the reason. Use one of:

- `**Prompter:** skipped (permission response), executing immediately.`
- `**Prompter:** skipped (option selection), executing immediately.`
- `**Prompter:** skipped (follow-up answer), executing immediately.`
- `**Prompter:** skipped (rewrite approved), executing rewritten prompt.`
- `**Prompter:** skipped (rewrite declined), running your original prompt.`
- `**Prompter:** skipped (command or pasted material), executing immediately.`
- `**Prompter:** no rewrite needed, prompt is already clear and actionable.`

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
- Keep it as short as the fix allows. A typo fix stays one sentence; a prompt whose dimensions need pinning can run to a short paragraph or a numbered list.
- Don't add scope creep. If the user asked for a fix, don't rewrite into "fix it and also refactor." Adding a check-first step or a guardrail is not scope creep: it narrows what happens first, and the user approves it before anything runs.

### Rewrite patterns

These are the moves that make the biggest difference. Use only the ones a prompt needs; most rewrites use one or two.

- **Diagnose before fixing.** A symptom plus "fix it" ("the tests are flaky, fix them", "the build is slow, fix it") becomes: find the cause, report it, and wait before changing code. A fix aimed at the wrong cause can make things worse.
- **Pin the dimensions.** Open-ended verbs like "match", "audit", "improve", "clean up", or "review" get named categories to check against, so the result is complete and reviewable. Offer defaults when the user is likely to accept them.
- **Name the deliverable.** Say what comes back: a ranked table, a list grouped by type, a one-paragraph answer. A wall of prose is hard to review.
- **List before changing everything.** Wide edits ("rename X everywhere", "update all the callers") become: list every place that would change, grouped by kind, then wait for the user to decide.
- **Add the guardrail the user forgot.** "Don't edit yet", "ask before pushing", "don't touch the tests", when the request's shape makes one obvious.
- **Resolve references from context.** "That file" or "the other endpoint" becomes the specific name, when the conversation makes it knowable. When it doesn't, ask instead of guessing.
- **Split stacked questions.** Several questions run together become a numbered list, so none gets dropped.
- **Preserve the user's voice.** A clear question asked casually ("why does this work") stays as it is. Don't rewrite tone, hedging, or register just to sound formal; fix only what blocks understanding.

## CLAUDE.md plumbing (Add/Remove modes)

### Where the rule goes

Claude Code reads CLAUDE.md from more than one place. Prompter uses these:

- **This project:** `CLAUDE.md` at the project root. Also check `.claude/CLAUDE.md` when looking for an existing section, since some projects keep their instructions there.
- **All projects:** `~/.claude/CLAUDE.md`, which applies to every project for this user.

### Adding to CLAUDE.md

When the user chooses "Add to CLAUDE.md", ask whether the rule should apply to **this project** or **all projects**, then write to the matching file from "Where the rule goes" above. Create the file if it doesn't exist. Afterward, name the exact file you changed.

Insert this minimal section. It points back at this skill instead of duplicating the skip list, so the skip list never drifts:

```markdown
## Prompt Rewriting

Before acting on any user prompt, evaluate whether rewriting it for clarity, typo correction, or actionability would meaningfully improve it. **Only rewrite when there is a real improvement to make** — typos to fix, ambiguous references to resolve, or vague intent to specify. If the original prompt is already clear and actionable, proceed without rewriting and without showing a rewrite. When you do rewrite, display the rewritten version with the prefix "**Rewritten prompt:**" and wait for approval before proceeding.

**Full rules** (skip categories, mode behavior, rewriting style): load the `prompter` skill. This section is a persistent activation, so skip the skill's mode menu. Do NOT duplicate the skip list here; the skill file is the source of truth.
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

1. Look for the section in every location listed in "Where the rule goes", using the detection rule above.
2. If it exists in more than one file, list the files and ask which to remove it from.
3. If the block matches the template verbatim, remove it cleanly: the heading line through the last line before the next `^## ` or end of file.
4. If the block has been modified, show the user the modified content and confirm before removing.
5. If no Prompt Rewriting section exists anywhere, say so and stop.
6. Remove a single trailing blank line if the deletion left two consecutive blank lines.
7. Name the exact file you changed.

Features removed in earlier versions are listed in `UPGRADING.md` in the prompter repository.
