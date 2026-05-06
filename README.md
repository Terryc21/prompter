# prompter

**A Claude Code skill that rewrites your prompts for clarity before they run.**

Built for [Stuffolio](https://stuffolio.app), an iOS/macOS inventory management app, and extracted into its own skill for general use.

<a href="https://buymeacoffee.com/stuffolio"><img src="https://cdn.buymeacoffee.com/buttons/v2/default-yellow.png" alt="Buy Me A Coffee" width="120"></a>

If this skill saves you time, a [coffee](https://buymeacoffee.com/stuffolio) is appreciated. [Sponsoring](https://github.com/sponsors/Terryc21) supports further development.

---

## Why this exists

Small prompt problems create surprisingly large downstream problems: ambiguity, vague references, overloaded requests, unclear goals, missing constraints. A typo or imprecise verb can send Claude in the wrong direction for a long time before you notice.

`prompter` intercepts your prompts before execution, rewrites them for clarity, and asks for approval before continuing. Over time, this becomes a subtle feedback loop: you start internalizing what makes prompts effective simply by watching them improve.

The skill is opinionated about what *not* to rewrite: option selections, permission responses ("yes", "proceed"), follow-up answers, and prompts that are already clear. It only rewrites when there is a real improvement to make.

---

## Usage

```
/skill prompter
```

Activation modes:

- **Current session only:** the rewrite behavior applies until the session ends
- **Persist via CLAUDE.md:** add the rewrite rule to your project's CLAUDE.md so it applies across sessions
- **Remove persistent rewriting:** strip the rule from CLAUDE.md when you no longer want it

The skill walks you through the choice on first invocation.

---

## Example rewrite

A real-feeling case where Claude has just been discussing a specific Swift file in the conversation, and the user types a vague follow-up:

**Original:**

> can you check that file for the same pattern

**Rewritten:**

> Can you check `Sources/AI_Backend/OCRResultSheet.swift` for the same pattern?

prompter resolved "that file" using the prior message, where `OCRResultSheet.swift` had just been discussed. Without the rewrite, Claude might have asked "which file?" or guessed wrong. With the rewrite, the next turn goes straight to the work.

A larger collection of example rewrites is in [examples/Prompter-Examples.md](examples/Prompter-Examples.md). It covers seven categories: typos, ambiguous references, vague verbs, multi-question prompts, prompts where rewriting should be skipped, preserving intent under restraint, and the side-effect feedback loop you notice after a few weeks of running prompter every day.

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

- [**code-smarter**](https://github.com/Terryc21/code-smarter): personalized coding tutorials generated from your own codebase
- [**bug-echo**](https://github.com/Terryc21/bug-echo): after fixing a bug, locate and rate similar patterns elsewhere in the codebase
- [**workflow-audit**](https://github.com/Terryc21/workflow-audit): multi-layer behavioral audit of SwiftUI user workflows
- [**radar-suite**](https://github.com/Terryc21/radar-suite): behavioral audit suite for iOS/macOS Swift projects

---

## History

`prompter` was originally bundled with [code-smarter](https://github.com/Terryc21/code-smarter) (then named `code-smarter`). It was split into its own repo so the audience that wants prompt rewriting can find it without first finding a tutorial-generation tool. Commit history is preserved.

---

## Author

Created by **Terry Nyberg**, [Coffee & Code LLC](https://stuffolio.app/).

## License

Apache 2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
