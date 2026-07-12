# Prompter — Example Rewrites

A representative sample of what Prompter does to user input before Claude Code acts on it. Each example shows a real-feeling raw prompt, then the rewrite Prompter would produce.

> [!NOTE]
> This file is a **sample of Prompter's output**, illustrating the kinds of changes the skill makes. Real Prompter sessions show the rewrite inline with the prefix `**Rewritten prompt:**` and wait for your approval before executing.

## Contents

Twenty-one examples, in file order. Pattern themes are noted in parentheses.
---
**Raw:**
> the api is returning 500 fix it

**Rewritten:**
> The API is returning 500. Before changing anything, find which endpoint is failing (check recent logs in `logs/` or run the failing test if there is one), identify the exception type and originating line, and report what you find. Do not start fixing until we've agreed on the root cause.

(The dangerous shape is "fix it" attached to a symptom. The rewrite refuses to start fixing until the diagnosis is in.)

---

**Raw:**
> make this match the design

**Rewritten:**
> Update this component's styling to match the design at `designs/checkout-v2.fig` (or whichever design source you'd like me to use — confirm if different). Be specific about what "match" means: matching layout (spacing, alignment, breakpoints), visual style (colors, typography, borders), interaction states (hover, focus, disabled, loading), or all three. Default to all three unless otherwise specified.

(*"Match the design"* is the kind of prompt where four developers will produce four different things. The rewrite forces the dimensions of "match" to be explicit before any code is written.)

---

**Raw:**
> can you check that file for the same pattern

**Rewritten:**
> Can you check `src/auth/session.ts` for the same pattern?

(Resolved "that file" using the prior message in the conversation, where `session.ts` had just been discussed.)

---

**Raw:**
> add caching here

**Rewritten:**
> Add caching to this function. Before implementing, decide together: (1) cache key (the function's argument hash, a derived key, or something custom), (2) cache invalidation strategy (TTL, manual invalidation, write-through), (3) cache layer (in-memory, Redis, the existing cache abstraction in `src/cache/`), (4) cache scope (per-request, per-user, global). Defaults: argument hash key, 5-minute TTL, in-memory, global. Confirm or adjust before I write code.

(Cache bugs are the second-hardest class of bug to debug. The rewrite catches every decision that *will* matter later, before any code is written.)

---

**Raw:**
> rename UserService to AccountService everywhere

**Rewritten:**
> Rename the class `UserService` to `AccountService` across the codebase. Before applying, list every file that contains `UserService` (imports, instantiations, type annotations, comments, test names, fixture data, configuration files, documentation), grouped by reference type. We'll review the list and decide which references to update before any edit happens.

(A blunt "rename everywhere" sounds simple but blows up across imports, configs, docs, and string literals in tests. The rewrite turns it into a list-then-decide flow so nothing gets accidentally renamed.)

---

**Raw:**
> yes

**Rewritten:**
> **Prompter:** skipped (permission response) — executing immediately.

---

**Raw:**
> the tests are flaky, fix them

**Rewritten:**
> Diagnose flakiness in the test suite before changing anything. Categorize each failing test as: (1) timing-dependent (sleeps, polling, real clocks), (2) order-dependent (shared state, leaking fixtures), (3) environment-dependent (network, filesystem, machine-specific), or (4) genuinely racy (concurrency bugs in the code under test). Produce a ranked table with the test name, category, and confidence. Don't apply fixes yet.

(For the four-bullet commentary explaining what this rewrite did, see the [worked example in the README](../README.md#example-rewrites).)

---

**Raw:**
> is the snapshot test still recording and if so does it need to flip back before commit and what about the visual reg suite as a whole

**Rewritten:**
> Three questions:
> 1. Is the snapshot recorder flag still set to `true` in the test config?
> 2. If yes, does it need to flip back to `false` before committing?
> 3. What is the overall state of the visual regression suite — passing, failing, or skipped?

(Three semantically distinct questions were buried in a run-on sentence. Splitting them turns one ambiguous ask into three reviewable ones.)

---

**Raw:**
> find a security issue in the auth code

**Rewritten:**
> Audit the authentication code for vulnerabilities, scoped to the OWASP Top 10 categories most relevant to auth: broken access control, cryptographic failures, identification/authentication failures, and security misconfiguration. For each finding, provide: file and line, OWASP category, exploit scenario in one sentence, and severity (critical/high/medium/low). Skip theoretical issues that require an attacker model not present in this codebase. Don't propose fixes yet — just the audit.

(Bug-hunt framing terminates at the first hit; scoped-audit framing terminates with a complete report. Picking four of OWASP's ten categories scopes the audit; the structured finding format makes it triageable in 60 seconds. See the README for the full version with per-bullet commentary.)

---

**Raw:**
> the build is slow, fix it

**Rewritten:**
> Profile the build, identify the top 3 bottlenecks (slow-compiling files, expensive build-step scripts, or unnecessary work in pre/post hooks), and propose fixes ranked by expected wall-clock savings. Don't apply changes yet.

(Same shape as "fix the flaky tests" — refuses to start fixing before profiling, and demands ranked findings.)

---

**Raw:**
> why does this work

**Rewritten:**
> Why does this work?

(Sometimes the user wants the open-ended "why does this work?" question precisely because they don't know what they don't know yet. Prompter doesn't pre-empt the answer by guessing what they meant.)

---

**Raw:**
> deal with the failing migration test

**Rewritten:**
> Diagnose or fix the failing test `migrations/2024_add_user_role.test.ts` (currently throwing "column 'role' already exists").

(Names the test file and quotes the error message from context. "Deal with" was the dangerous verb — it could mean fix, skip, or delete.)

---

**Raw:**
> do the thing we just did but for the other endpoint

**Rewritten:**
> Apply the same input-validation refactor to the `/api/users/:id` endpoint that we just applied to `/api/users`.

(Resolved both "the thing" and "the other endpoint" from prior conversation. If either reference were ambiguous, no rewrite — the skill would ask.)

---

**Raw:**
> audit user.service.ts

**Rewritten:**
> Audit `src/services/user.service.ts` for: (1) error-handling gaps (swallowed errors, missing user-facing surfacing), (2) async correctness (unhandled promise rejections, missing awaits), (3) dead code, and (4) violations of the project's coding standards documented in CONTRIBUTING.md. Output as a prioritized list with line numbers; don't edit the file.

(Default audit dimensions plus deliverable shape. Resists the temptation to start editing — find first, decide second.)

---

**Raw:**
> can you maybe possibly take a look and see if anything jumps out

**Rewritten:**
> Can you take a look and see if anything jumps out?

(Light hedging is part of how some people work. Prompter trims redundant hedging ("maybe possibly") but keeps the conversational register.)

---

**Raw:**
> set up the linter

**Rewritten:**
> Install ESLint with the project's existing TypeScript config preset, add a `lint` script to `package.json`, and run it once to surface the current violation count.

(Concrete steps replace the vague verb "set up." Ends with a visible state change — the violation count — so the user knows what to expect next.)

---

**Raw:**
> fix the typo on line 47

**Rewritten:**
> Fix the typo on line 47.

(A worse rewriter might say "fix the typo on line 47 and also any nearby typos" — that's adding scope. Prompter resists.)

---

**Raw:**
> commit the api changes also push them

**Rewritten:**
> Commit the changes in the `api/` directory, then push to origin.

(Two-step instruction was already clear; the rewrite only sharpens the target.)

---

**Raw:**
> chek if hte test for cateogry coutn matches th nu mber of cateogries in hte enum

**Rewritten:**
> Check if the test for category count matches the number of categories in the enum.

(Pure spelling repair. The intent was crystal-clear; the rewrite makes it readable without changing scope.)

---

**Raw:**
> Run npm test

**Rewritten:**
> **Prompter:** no rewrite needed — prompt is already clear and actionable. Run `npm test`.

(Prompter still surfaces a one-line "no rewrite needed" notice so the user knows the skill saw the prompt and made a deliberate decision, rather than silently skipping.)

---

**Raw:**
> Identify items in Claude.md that look like they were added to fix an issue that no lonerg exists or that has become stale, or have become counterproductive, or hs restricted you from making better decisions.

**Rewritten:**
> Audit `CLAUDE.md` for rules that were added to fix a past problem but are now liabilities. For each, classify which failure mode it hits: (a) **stale** — the issue it guarded against no longer exists; (b) **counterproductive** — it now causes more friction than the problem it prevented; or (c) **over-restrictive** — it blocks you from making a better decision in cases the rule's author didn't anticipate. Cite the specific line/section, explain the original intent (why it was likely added), why it's now a problem, and a proposed change. Don't edit yet — present findings in a table for approval first.

("Look for stale/counterproductive rules" is a vague analytical ask that would return an unstructured list. The rewrite pins it into a deliverable shape: three *named* failure-mode categories to classify against, a fixed per-finding structure (cite → intent → problem → proposed change), and a "don't edit yet, table first" guardrail the user clearly wanted but hadn't stated. It also silently repairs the typos. This is the highest-value class of rewrite — turning an open-ended request into something the model can execute consistently.)

---

*Real Prompter sessions look the same but happen inline in your Claude Code conversation.*
