# prompter

**A Claude Code skill that rewrites your prompts for clarity before they run.**

Built for [Stuffolio](https://stuffolio.app), an iOS/macOS inventory management app, and extracted into its own skill for general use.

<a href="https://buymeacoffee.com/stuffolio"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" width="120"></a>

If this skill saves you time, a [coffee](https://buymeacoffee.com/stuffolio) is appreciated. [Sponsoring](https://github.com/sponsors/Terryc21) supports further development.

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
- **Next N prompts:** apply prompter to the next N prompts (you specify N), then stop
- **Current session only:** the evaluate/rewrite behavior applies until the session ends
- **Persist via CLAUDE.md:** add the rewrite rule to your project's CLAUDE.md so it applies across sessions
- **Remove persistent rewriting:** strip the rule from CLAUDE.md when you no longer want it

The skill walks you through the choice on first invocation.

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

A second case where the under-specification problem is the threat model the user has in their head but didn't write down:

**Original:**

> find a security issue in the auth code

**Rewritten:**

> Audit the authentication code for vulnerabilities, scoped to the OWASP Top 10 categories most relevant to auth: broken access control, cryptographic failures, identification/authentication failures, and security misconfiguration. For each finding, provide: file and line, OWASP category, exploit scenario in one sentence, and severity (critical/high/medium/low). Skip theoretical issues that require an attacker model not present in this codebase (e.g., physical device access if this is a server). Don't propose fixes yet — just the audit.

What prompter did:

- **Replaced "a security issue" with a threat model.** The singular "a security issue" was the leak: it framed the task as bug-hunting until something is found, then stop. The rewrite reframes it as a scoped audit against a specific industry standard (OWASP Top 10), which means the audit *terminates with a complete report* instead of stopping at the first hit.
- **Cut the categories that don't apply.** OWASP has 10; the rewrite picked the 4 that touch auth and explicitly noted to skip the rest. Without this, Claude would either over-audit (covering injection in a code path with no user input) or arbitrarily under-audit.
- **Demanded a structured finding format.** File, line, OWASP category, exploit scenario, severity. A reviewer can scan this in 60 seconds and decide what to prioritize. A wall of prose can't be triaged.
- **Added the scope discipline.** "Skip theoretical issues that require an attacker model not present in this codebase" — the line that prevents Claude from listing a CSRF concern on a backend service that has no browser-facing surface.
- **Added the same diagnose-before-fix guardrail.** "Don't propose fixes yet — just the audit." Findings before remediation.

prompter also handles the smaller, more common cases — typos, dangling references like "that file," stacked questions, prompts that should *not* be rewritten because they're already clear, and the cases where it correctly holds back to preserve the user's voice. Twenty worked examples covering typos, dangling references, stacked questions, hedging preservation, no-rewrite cases, and threat-model framing are in [examples/Prompter-Examples.md](examples/Prompter-Examples.md).

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

> [!NOTE]
> Manual install only — prompter is not yet packaged as a Claude Code plugin (unlike sibling skills `bug-echo`, `radar-suite`, etc.). Plugin packaging is on the roadmap.

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

## Related Claude Code skills

Other skills built during development of Stuffolio:

- [**tutorial-creator**](https://github.com/Terryc21/tutorial-creator): personalized coding tutorials generated from your own codebase
- [**bug-echo**](https://github.com/Terryc21/bug-echo): after fixing a bug, locate and rate similar patterns elsewhere in the codebase
- [**workflow-audit**](https://github.com/Terryc21/workflow-audit): multi-layer behavioral audit of SwiftUI user workflows
- [**radar-suite**](https://github.com/Terryc21/radar-suite): behavioral audit suite for iOS/macOS Swift projects

---

## History

`prompter` was originally bundled with [tutorial-creator](https://github.com/Terryc21/tutorial-creator) (then named `code-smarter`). It was split into its own repo so the audience that wants prompt rewriting can find it without first finding a tutorial-generation tool. Commit history is preserved.

---

## Author

Created by **Terry Nyberg**, [Coffee & Code LLC](https://stuffolio.app/).

## License

Apache 2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
