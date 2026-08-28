# prompter

![Version](https://img.shields.io/github/v/tag/Terryc21/prompter?label=version) ![Last commit](https://img.shields.io/github/last-commit/Terryc21/prompter) ![Stars](https://img.shields.io/github/stars/Terryc21/prompter?style=flat) ![Issues](https://img.shields.io/github/issues/Terryc21/prompter) ![License](https://img.shields.io/github/license/Terryc21/prompter)

**You type a short prompt. Claude runs with it in a direction you didn't choose, and the next
turn is spent reorienting.**

Not because the prompt was wrong — it was grammatically fine and perfectly clear to you. It was
under-specified along the dimensions that turned out to matter. prompter catches that before the
run, shows you a rewrite, and waits for your approval.

*4 min read · scan the TL;DR if you only have 30 seconds · [worked examples](examples/Prompter-Examples.md) · [upgrading from v1.2.x](UPGRADING.md)*

## TL;DR

- **What:** Intercepts your prompts, evaluates whether rewriting would meaningfully improve them, and shows you the rewrite for approval before Claude acts.
- **Why:** Most prompts are short, grammatically fine, and under-specified along the dimensions that matter most. Claude runs with them in a direction you didn't choose, and the next turn is spent reorienting.
- **Install:** `git clone` into `~/.claude/skills/`; then `/skill prompter` in any session.
- **Try first:** `/skill prompter`, choose "Try it once," then submit a one-liner like *"the tests are flaky, fix them"* — see what prompter would have made of it.
- **Example output:** [20 worked examples covering typos, dangling references, threat-model framing, and no-rewrite cases](examples/Prompter-Examples.md).
- **Maturity:** v1.3.0; manual install (plugin packaging on the roadmap); built and used daily while shipping [Stuffolio](https://stuffolio.app).
- **Upgrading from v1.2.x?** See [Upgrading](#upgrading-from-v12x) — v1.3.0 removed "Next N prompts" mode and the "Flag exemplary rewrites" feature.

## Newer to Claude Code?

A **skill** is a markdown file Claude Code knows how to run. When you type `/skill prompter`, Claude follows the instructions in this skill, intercepts your next prompt, and asks whether you want to rewrite it before acting. You don't have to memorize anything — the skill tells Claude what to do, you approve or skip each rewrite.

---

## Why this exists

`prompter` intercepts your prompts before execution, **evaluates whether rewriting would meaningfully improve them**, and — when it would — shows you the rewrite and asks for approval before continuing. Over time, this becomes a subtle feedback loop: you start internalizing what makes prompts effective simply by watching them improve.

The skill is opinionated about what *not* to rewrite: option selections, permission responses ("yes", "proceed"), follow-up answers, and prompts that are already clear. It only rewrites when there is a real improvement to make.

---

## Usage

```
/skill prompter
```

Activation modes:

- **Try it once:** evaluate and (if warranted) rewrite the next prompt, then stop
- **This session:** the evaluate/rewrite behavior applies until the session ends
- **Add to CLAUDE.md:** add the rewrite rule to your project's CLAUDE.md so it persists across sessions
- **Remove from CLAUDE.md:** strip the rule when you no longer want persistent rewriting

The skill walks you through the choice on first invocation.

**Re-invocation behavior.** Once a mode is active, typing `/prompter` alongside your prompt evaluates *that* prompt without re-asking for a mode. The mode picker only appears when you type `/prompter` by itself, signaling you want to change settings.

---

## Example rewrites

<!-- If this heading is renamed, update the two cross-references in examples/Prompter-Examples.md (search for `#example-rewrites`). -->

prompter's biggest payoff is on prompts that are short, grammatically fine, and **under-specified along the dimensions that matter most.** A blunt verb plus a familiar complaint is the classic shape: "Fix the flaky tests." "Refactor this." "Audit the auth code." Claude will run with it, but in a direction you didn't choose, and the next turn is spent reorienting.

**Original:**

> the tests are flaky, fix them

**Rewritten:**

> Diagnose flakiness in the test suite before changing anything. Categorize each failing test as: (1) timing-dependent (sleeps, polling, real clocks), (2) order-dependent (shared state, leaking fixtures), (3) environment-dependent (network, filesystem, machine-specific), or (4) genuinely racy (concurrency bugs in the code under test). Produce a ranked table with the test name, category, and confidence. Don't apply fixes yet.

What prompter did:

- **Refused to skip the diagnosis.** "Fix them" became "diagnose, categorize, rank." The original would have started editing tests before anyone knew *which kind* of flake was at play — and a fix for a timing flake makes an order-dependence flake worse.
- **Pinned the analysis dimensions.** Four well-known flake categories that every test engineer recognizes, rather than "whatever Claude thinks is wrong."
- **Specified the deliverable shape.** A ranked table with name, category, and confidence — reviewable in 30 seconds, not a wall of prose.
- **Added the missing guardrail.** "Don't apply fixes yet" — the obvious thing the user would have written if they hadn't been mid-thought.

The same shape shows up wherever the user's real intent is a threat model, a scope, or a
deliverable format they never wrote down. [A second worked example — "find a security issue in
the auth code" — is in the examples file](examples/Prompter-Examples.md), with the same
per-bullet commentary.

prompter also handles the smaller, more common cases — typos, dangling references like "that file," stacked questions, prompts that should *not* be rewritten because they're already clear, and the cases where it correctly holds back to preserve the user's voice. Twenty-one worked examples covering typos, dangling references, stacked questions, hedging preservation, no-rewrite cases, threat-model framing, and pinning open-ended analytical asks into named categories are in [examples/Prompter-Examples.md](examples/Prompter-Examples.md).

---

## Scoping a run

prompter doesn't take a file or feature argument — there's nothing to scan. It scopes by **how long the rewrite behavior stays active**. On first invocation it asks:

| Mode | Behavior |
|---|---|
| **Try it once** | Evaluate the next prompt, rewrite if warranted, then stop. Good first run. |
| **This session** | Rewrite for the rest of this Claude Code conversation. No files modified. |
| **Add to CLAUDE.md** | Insert a Prompt Rewriting rule into your project's CLAUDE.md so it persists across sessions. |
| **Remove from CLAUDE.md** | Strip the rule when you no longer want persistent rewriting. |

**Fresh vs prior history.** prompter is stateless by design — every prompt is evaluated from scratch against the rules in SKILL.md. There's no per-user model that "learns your style," no prior-rewrite cache that biases the next call. Re-invocation in the same session detects an active mode: if your message contains prompt text, the skill silently evaluates it under the active mode; if your message is `/prompter` alone, the mode picker reappears so you can switch settings. The Add-to-CLAUDE.md path never duplicates silently — it diffs against any existing section and asks before overwriting. If you want a different style, edit SKILL.md or override the rule in your project's CLAUDE.md.

---

## What it doesn't do

`prompter` is intentionally narrow:

- It does not generate code
- It does not make architectural decisions
- It does not pick libraries or frameworks for you
- It does not edit your prompt history or transcript

It just rewrites the next prompt for clarity, shows you the rewrite, and waits for your approval before the rewritten version goes to Claude.

---

## Install

Two commands in Claude Code, run one at a time:

```
/plugin marketplace add Terryc21/prompter
```

```
/plugin install prompter@prompter
```

> **Why two commands?** Claude Code's slash-command dispatcher treats the second `/plugin` as text inside the first command. Run them one at a time.

**Manual install** (if you'd rather not use the plugin system):

```bash
git clone https://github.com/Terryc21/prompter.git
```

**Global install** (all projects):

```bash
mkdir -p ~/.claude/skills && cp -r prompter ~/.claude/skills/
```

**Project-specific install** (one project only):

```bash
mkdir -p /path/to/project/.claude/skills && cp -r prompter /path/to/project/.claude/skills/
```

---

## Upgrading from v1.2.x

v1.3.0 is a backward-incompatible rewrite. If you installed prompter before May 31, 2026, the
version on disk won't match this README — no errors, but the behavior won't match what's
documented here until you upgrade.

**[What changed, and how to upgrade →](UPGRADING.md)**

## History

`prompter` was originally bundled with [tutorial-creator](https://github.com/Terryc21/tutorial-creator) (then named `code-smarter`). It was split into its own repo so the audience that wants prompt rewriting can find it without first finding a tutorial-generation tool. Commit history is preserved.

---

## Sibling skills

- [**bug-echo**](https://github.com/Terryc21/bug-echo) — sibling-bug scan after a fix
- [**bug-prospector**](https://github.com/Terryc21/bug-prospector) — forward-looking bug hunt before a release
- [**workflow-audit**](https://github.com/Terryc21/workflow-audit) — 5-layer SwiftUI behavioral flow audit
- [**unforget**](https://github.com/Terryc21/unforget) — one-file deferred-work ledger
- [**radar-suite**](https://github.com/Terryc21/radar-suite) — 6-skill suite tracing user behavior paths through the app (iOS + macOS)
- [**skill-reviewer**](https://github.com/Terryc21/skill-reviewer) — candid reviews of other Claude Code skills
- [**tutorial-creator**](https://github.com/Terryc21/tutorial-creator) — annotated tutorials from your codebase

## Author

Terry Nyberg, [Coffee & Code LLC](https://stuffolio.app/). If prompter has saved you a wasted reorientation turn, [a coffee](https://buymeacoffee.com/stuffolio) is appreciated. Issue reports about what worked or didn't are more useful.

[![Buy Me A Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-FFDD00?style=flat&logo=buy-me-a-coffee&logoColor=black)](https://www.buymeacoffee.com/stuffolio)

## License

Apache 2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
