# Prompter — Example Rewrites

A representative sample of what Prompter does to user input before Claude Code acts on it. Each example below shows a real-feeling raw prompt, then the rewrite Prompter would produce.

> [!NOTE]
> This file is a **sample of Prompter's output**, illustrating the kinds of changes the skill makes. Real Prompter sessions show the rewrite inline with the prefix `**Rewritten prompt:**` and wait for your approval before executing.

---

**Raw:**
> can you scna documents folde in root for stale documeents

**Rewritten:**
> Can you scan the `docs/` folder at the root for stale documents?

---

**Raw:**
> wht is hapeneing in this fucntion

**Rewritten:**
> What is happening in this function?

---

**Raw:**
> chek if hte test for cateogry coutn matches th nu mber of cateogries in hte enum

**Rewritten:**
> Check if the test for category count matches the number of categories in the enum.

---

**Raw:**
> can you check that file for the same pattern

**Rewritten:**
> Can you check `src/auth/session.ts` for the same pattern?

(Prompter resolved "that file" using the prior message in the conversation, where `session.ts` had just been discussed.)

---

**Raw:**
> do the thing we just did but for the other endpoint

**Rewritten:**
> Apply the same input-validation refactor to the `/api/users/:id` endpoint that we just applied to `/api/users`.

---

**Raw:**
> what should I do next

**Rewritten:**
> *(Already clear and actionable; no rewrite.)* What should I do next?

(Prompter doesn't fabricate specificity when the user is genuinely asking an open question.)

---

**Raw:**
> look at the auth tests

**Rewritten:**
> Read `tests/auth/login.test.ts` and report what you find.

---

**Raw:**
> deal with the failing migration test

**Rewritten:**
> Diagnose or fix the failing test `migrations/2024_add_user_role.test.ts` (currently throwing "column 'role' already exists").

---

**Raw:**
> set up the linter

**Rewritten:**
> Install ESLint with the project's existing TypeScript config preset, add a `lint` script to `package.json`, and run it once to surface the current violation count.

---

**Raw:**
> is the snapshot test still recording and if so does it need to flip back before commit and what about the visual reg suite as a whole

**Rewritten:**
> Three questions:
> 1. Is the snapshot recorder flag still set to `true` in the test config?
> 2. If yes, does it need to flip back to `false` before committing?
> 3. What is the overall state of the visual regression suite — passing, failing, or skipped?

---

**Raw:**
> commit the api changes also push them

**Rewritten:**
> Commit the changes in the `api/` directory, then push to origin.

---

**Raw:**
> yes

**Rewritten:**
> *(Skipped — permission response. Prompter executes immediately.)*

---

**Raw:**
> Option 2

**Rewritten:**
> *(Skipped — selecting from options Claude already presented. Prompter executes immediately.)*

---

**Raw:**
> Run npm test

**Rewritten:**
> *(Already clear and actionable; no rewrite. Prompter says so explicitly.)* Run `npm test`.

(Prompter still surfaces a one-line "no rewrite needed" notice so the user knows the skill saw the prompt and made a deliberate decision, rather than silently skipping.)

---

**Raw:**
> fix the typo on line 47

**Rewritten:**
> Fix the typo on line 47.

(A worse rewriter might say "fix the typo on line 47 and also any nearby typos" — that's adding scope. Prompter resists.)

---

**Raw:**
> why does this work

**Rewritten:**
> Why does this work?

(Sometimes the user wants the open-ended "why does this work?" question precisely because they don't know what they don't know yet. Prompter doesn't pre-empt the answer by guessing what they meant.)

---

**Raw:**
> can you maybe possibly take a look and see if anything jumps out

**Rewritten:**
> Can you take a look and see if anything jumps out?

(Light hedging is part of how some people work. Prompter trims redundant hedging ("maybe possibly") but keeps the conversational register.)

---

**Raw:**
> the build is slow, fix it

**Rewritten:**
> Profile the build, identify the top 3 bottlenecks (slow-compiling files, expensive build-step scripts, or unnecessary work in pre/post hooks), and propose fixes ranked by expected wall-clock savings. Don't apply changes yet.

---

**Raw:**
> the tests are flaky, fix them

**Rewritten:**
> Diagnose flakiness in the test suite before changing anything. Categorize each failing test as: (1) timing-dependent (sleeps, polling, real clocks), (2) order-dependent (shared state, leaking fixtures), (3) environment-dependent (network, filesystem, machine-specific), or (4) genuinely racy (concurrency bugs in the code under test). Produce a ranked table with the test name, category, and confidence. Don't apply fixes yet.

---

**Raw:**
> find a security issue in the auth code

**Rewritten:**
> Audit the authentication code for vulnerabilities, scoped to the OWASP Top 10 categories most relevant to auth: broken access control, cryptographic failures, identification/authentication failures, and security misconfiguration. For each finding, provide: file and line, OWASP category, exploit scenario in one sentence, and severity (critical/high/medium/low). Skip theoretical issues that require an attacker model not present in this codebase. Don't propose fixes yet — just the audit.

---

**Raw:**
> CLAUDE.md has gotten large. make recommendations for cleaning it up.

**Rewritten:**
> CLAUDE.md has grown large. Read the file end-to-end and recommend a cleanup, identifying duplicated content, sections that are stale, sections that should be split into separate files (and where), and anything that's better living in memory or a separate doc. Produce a prioritized recommendation list with rough line-count savings, but do not edit CLAUDE.md yet.

---

**Raw:**
> audit user.service.ts

**Rewritten:**
> Audit `src/services/user.service.ts` for: (1) error-handling gaps (swallowed errors, missing user-facing surfacing), (2) async correctness (unhandled promise rejections, missing awaits), (3) dead code, and (4) violations of the project's coding standards documented in CONTRIBUTING.md. Output as a prioritized list with line numbers; don't edit the file.

---

*Real Prompter sessions look the same but happen inline in your Claude Code conversation.*
