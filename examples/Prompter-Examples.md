# Prompter — Example Rewrites

A representative sample of what Prompter does to user input before Claude Code acts on it. Each example below shows a real-feeling raw prompt, then the rewrite Prompter would produce. Examples are grouped by the category of fix.

> [!NOTE]
> This file is a **sample of Prompter's output**, illustrating the kinds of changes the skill makes. Real Prompter sessions show the rewrite inline with the prefix `**Rewritten prompt:**` and wait for your approval before executing.

---

## 1. Typos and spelling

The most common fix. Prompter corrects clear typos without changing meaning, and lets the original word stand when the typo is creative enough to be a coined term.

### Example 1.1 — Heads-down typing

**Raw:**
> can you scna documents folde in root for stale documeents

**Rewritten prompt:**
> Can you scan the `Documentation/` folder at the root for stale documents?

### Example 1.2 — Phonetic substitution

**Raw:**
> wht is hapeneing in this fucntion

**Rewritten prompt:**
> What is happening in this function?

### Example 1.3 — Cascading typo paragraph

**Raw:**
> chek if hte test for cateogry coutn matches th nu mber of cateogries in hte enum

**Rewritten prompt:**
> Check if the test for category count matches the number of categories in the enum.

### Example 1.4 — Don't over-correct

**Raw:**
> add a defenstive comment above the VStack about the type-tree threshold

**Rewritten prompt:**
> Add a defensive comment above the VStack about the type-tree threshold.

(`defenstive` was clearly meant as `defensive`, not the rare verb "to throw out a window." Prompter trusts context to disambiguate.)

---

## 2. Resolving ambiguous references

Prompter pins down "that file," "the thing we just did," and "it" to specific concrete targets when the surrounding context makes the intent unambiguous.

### Example 2.1 — "That file"

**Raw:**
> can you check that file for the same pattern

**Rewritten prompt:**
> Can you check `Sources/AI_Backend/OCRResultSheet.swift` for the same pattern?

(Prompter resolved "that file" using the prior message in the conversation, where `OCRResultSheet.swift` had just been discussed.)

### Example 2.2 — "The thing we just did"

**Raw:**
> do the thing we just did but for the other view

**Rewritten prompt:**
> Apply the same `@ViewBuilder` split refactor to `EnhancedItemDetailView+Sections.swift atAGlanceSection`.

### Example 2.3 — Don't over-resolve

**Raw:**
> what should I do next

**Rewritten prompt:**
> *(Already clear and actionable; no rewrite.)* What should I do next?

(Prompter doesn't fabricate specificity when the user is genuinely asking an open question.)

---

## 3. Adding specificity where intent is clear but wording is vague

Prompter sharpens vague verbs ("look at," "deal with," "handle") into specific actionable verbs ("read," "fix," "delete," "audit") when the context tells it which is meant.

### Example 3.1 — "Look at"

**Raw:**
> look at the privacy tests

**Rewritten prompt:**
> Read `StuffolioUITests/PrivacyOptInUITests.swift` and report what you find.

### Example 3.2 — "Deal with"

**Raw:**
> deal with the failing migration test

**Rewritten prompt:**
> Disable or fix the failing test `V2ToV3MigrationTests.v2StoreSurvivesMigrationAndAllRowsHaveStableUUIDId` (currently throwing "Duplicate version checksums detected").

### Example 3.3 — "Set up"

**Raw:**
> set up bug-echo

**Rewritten prompt:**
> Install the `bug-echo` skill via `/plugin marketplace add Terryc21/bug-echo` followed by `/plugin install bug-echo@bug-echo`.

---

## 4. Restructuring multi-question prompts

When a prompt asks two or three things at once, Prompter keeps them all but presents them as a numbered list so each gets answered.

### Example 4.1 — Stacked questions

**Raw:**
> is the snapshot test still recording and if so does it need to flip back before commit and what about the visual reg suite as a whole

**Rewritten prompt:**
> Three questions:
> 1. Is `SnapshotConfig.isRecording` still set to `true` in `SnapshotTestCase.swift`?
> 2. If yes, does it need to flip back to `false` before committing?
> 3. What is the overall state of the visual regression suite — passing, failing, or skipped?

### Example 4.2 — Implicit "also"

**Raw:**
> commit the radar-suite changes also push them

**Rewritten prompt:**
> Commit the changes in the `radar-suite` repo, then push to origin.

---

## 5. Skip rewriting (Prompter's discipline)

Prompter is supposed to **not rewrite** in three cases. The rewrite either degrades the message or wastes the user's turn.

### Example 5.1 — Permission response

**Raw:**
> yes

**Rewritten prompt:**
> *(Skipped — permission response. Prompter executes immediately.)*

### Example 5.2 — Option selection

**Raw:**
> Option 2

**Rewritten prompt:**
> *(Skipped — selecting from options Claude already presented. Prompter executes immediately.)*

### Example 5.3 — Already clear and actionable

**Raw:**
> Run /unforget list --target=THIS

**Rewritten prompt:**
> *(Already clear and actionable; no rewrite. Prompter says so explicitly.)* Run `/unforget list --target=THIS`.

(Prompter still surfaces a one-line "no rewrite needed" notice so the user knows the skill saw the prompt and made a deliberate decision, rather than silently skipping.)

---

## 6. Preserving intent — what Prompter will *not* do

The skill's biggest discipline is restraint. Examples where a less-careful rewriter would have changed the meaning, but Prompter holds back.

### Example 6.1 — Don't add scope

**Raw:**
> fix the typo on line 47

**Rewritten prompt:**
> Fix the typo on line 47.

(A worse rewriter might say "fix the typo on line 47 and also any nearby typos" — that's adding scope. Prompter resists.)

### Example 6.2 — Don't rephrase a deliberate question

**Raw:**
> why does this work

**Rewritten prompt:**
> Why does this work?

(Sometimes the user wants the open-ended "why does this work?" question precisely because they don't know what they don't know yet. Prompter doesn't pre-empt the answer by guessing what they meant.)

### Example 6.3 — Don't change the user's voice

**Raw:**
> can you maybe possibly take a look and see if anything jumps out

**Rewritten prompt:**
> Can you take a look and see if anything jumps out?

(Light hedging is part of how some people work. Prompter trims redundant hedging ("maybe possibly") but keeps the conversational register.)

---

## 7. Specifying deliverable shape for analytical asks

Some prompts are short and grammatically clean but under-specified along the dimensions that matter most for a useful answer. "Audit X." "Make recommendations." "Fix the slow build." Without rewriting, Claude will pick a direction, usually a reasonable one, and produce something. But that something often misses the implicit deliverable shape the user had in mind, and the next turn is spent reorienting. Prompter's biggest payoff lives in this category: it surfaces the dimensions of the analysis, names the deliverable shape, and adds the obvious guardrails ("don't edit yet") that the user would have remembered to write if they hadn't been mid-thought.

### Example 7.1 — "Recommend a cleanup"

**Raw:**
> CLAUDE.md has gotten large. make recommendations for cleaning it up.

**Rewritten prompt:**
> CLAUDE.md has grown large. Read the file end-to-end and recommend a cleanup, identifying duplicated content, sections that are stale, sections that should be split into separate files (and where), and anything that's better living in memory or `DEFERRED.md` instead. Produce a prioritized recommendation list with rough line-count savings, but do not edit CLAUDE.md yet.

(Prompter pinned down four audit dimensions, duplication, staleness, splittability, offload candidates, specified the output shape as a prioritized list with savings estimates, and added a guardrail against premature edits. The raw prompt would have produced a wandering response; the rewrite produces a deliverable.)

### Example 7.2 — "Fix the slow build"

**Raw:**
> the build is slow, fix it

**Rewritten prompt:**
> Profile the iOS Simulator build of the Stuffolio scheme, identify the top 3 bottlenecks (compile-time hot files, slow build phase scripts, or unnecessary work in `Run Script` phases), and propose fixes ranked by expected wall-clock savings. Don't apply changes yet.

(A blunt verb, "fix," becomes a measurement plan with a deliverable shape. Prompter resists jumping to a fix because the diagnosis hasn't happened. Without this, Claude often starts editing files before establishing what's actually slow.)

### Example 7.3 — "Audit this file"

**Raw:**
> audit ScoutSession.swift

**Rewritten prompt:**
> Audit `Sources/AI_Backend/ScoutSession.swift` for: (1) concurrency correctness (Sendable, @MainActor, actor isolation), (2) error-handling gaps (`try?` swallowing, missing user-facing surfacing), (3) dead code, and (4) alignment with the project's "Completed Migrations" rules in CLAUDE.md. Output as a prioritized list with line numbers; don't edit the file.

("Audit" without dimensions becomes a vague file walkthrough. Prompter chooses dimensions based on the project's stated standards, visible in CLAUDE.md, and pins the output to a reviewable list with line numbers.)

---

## 8. Surprising side effect: prompter as a feedback loop

Over time, watching Prompter sharpen your prompts becomes a quiet practice in writing better ones. You start noticing the patterns it cleans up, vague verbs, dangling pronouns, stacked questions, and writing prompts that already have those qualities. Some users report that after a month of running Prompter, their typo rate drops without effort, because the act of seeing the typo in the rewrite makes the original more conscious. The skill is supposed to make Claude better at understanding you. It also makes you a slightly better prompt writer, almost as a side effect.

---

*Examples generated 2026-05-03 to demonstrate Prompter's behavior; Section 7 (deliverable shape) added 2026-05-07. Real Prompter sessions look the same but happen inline in your Claude Code conversation.*
