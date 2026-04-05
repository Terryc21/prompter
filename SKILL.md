---
name: prompter
description: Rewrite user prompts for clarity, fix typos, and make them more actionable before execution. Display the rewrite for approval.
author: Terry Nyberg
---

# Prompter

Rewrite the user's prompt so it is clear, typo-free, and actionable. Display the rewritten version and wait for approval before taking any action.

## On invocation

When `/skill prompter` is run, ask the user how long prompt rewriting should stay active:

- **Try it once** -- Rewrite just the next prompt, then stop. Good for seeing what prompter does before committing.
- **Next N prompts** -- Ask how many prompts to rewrite (e.g., 5, 10), then count down. Show remaining count after each rewrite.
- **This session** -- Rewrite prompts for the rest of this conversation. No files are modified.
- **Add to CLAUDE.md** -- Add a Prompt Rewriting rule to the project's CLAUDE.md so it persists across all future sessions.
- **Remove from CLAUDE.md** -- Remove the Prompt Rewriting rule from CLAUDE.md to stop persistent rewriting.

Then follow the rules below for the chosen duration.

### After expiration

When "Try it once" or "Next N prompts" expires, show:

> **Prompter:** Done. Like it? Run `/skill prompter` to keep it going -- choose "This session" or "Add to CLAUDE.md" to make it permanent.

### Counting behavior (N prompts)

When the user chooses "Next N prompts":
1. Ask how many prompts to rewrite
2. After each rewrite, show the remaining count (e.g., "**Prompter:** 4 rewrites remaining")
3. When the count reaches 0, show the expiration message above.

### Adding to CLAUDE.md

When the user chooses "Add to CLAUDE.md", insert this section (or update it if it already exists):

```markdown
## Prompt Rewriting

Before acting on any user prompt, rewrite it for clarity, correct typos, and make it more actionable. Display the rewritten version and wait for approval before proceeding. See `/skill prompter` for full rules.

**Skip rewriting for:**
- Permission responses (yes, no, proceed, etc.)
- Selecting from options you presented (e.g., "Option 2", "Proceed (Recommended)")
- Follow-up answers to your clarifying questions
```

## Rules

1. **Rewrite** the user's prompt: fix typos, clarify intent, make it specific and actionable for Claude Code
2. **Display** the rewritten prompt with the prefix "**Rewritten prompt:**"
3. **Wait** for the user to approve before proceeding
4. If the user approves (yes, looks good, etc.), execute the rewritten prompt
5. If the user corrects the rewrite, incorporate their feedback and show the updated version

## Skip rewriting for

- Permission responses (yes, no, proceed, etc.)
- Selecting from options presented (e.g., "Option 2", "Proceed (Recommended)")
- Follow-up answers to clarifying questions

## Rewriting guidelines

- Fix spelling and grammar
- Resolve ambiguous references ("that file" -> specific file name if known from context)
- Add specificity where the intent is clear but the wording is vague
- Keep the user's intent -- don't add scope or change what they're asking for
- Keep it concise -- one to three sentences is ideal
- If the prompt is already clear and actionable, say so and proceed without rewriting
