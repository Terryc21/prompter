# prompter

![Version](https://img.shields.io/github/v/tag/Terryc21/prompter?label=version) ![Last commit](https://img.shields.io/github/last-commit/Terryc21/prompter) ![Stars](https://img.shields.io/github/stars/Terryc21/prompter?style=flat) ![Issues](https://img.shields.io/github/issues/Terryc21/prompter) ![License](https://img.shields.io/github/license/Terryc21/prompter)

**Your prompt is clear to you. Is it clear to Claude?**

prompter reads each prompt before Claude acts on it and decides whether a rewrite would help.
Most of the time it wouldn't, and your prompt runs as written. When a prompt is vague, misspelled,
or missing something obvious, prompter shows you a sharper version and waits. You approve it,
correct it, or run your original.

*5 min read · [21 worked examples](examples/Prompter-Examples.md) · [what changed in v1.4.0](UPGRADING.md)*

---

## The problem

Short prompts feel clear to the person who wrote them. "Fix the flaky tests." "Audit the auth
code." "Make this match the design." You know what you meant. Claude has to guess.

So it guesses. It starts editing tests before anyone knows why they fail, or audits for the
problems it thinks matter instead of the ones you were worried about. The work isn't wrong,
exactly. It's aimed somewhere you didn't choose, and the next turn goes to steering it back.

The prompt wasn't badly written. It left out the things that turned out to matter: what to check,
what to hand back, and what not to touch yet.

---

## What prompter does

Before Claude acts, prompter asks one question: would rewriting this prompt make a real
difference? If not, it says so in one line and gets out of the way. No rewrite is a good result.

When a rewrite would help, it does only what the prompt needs. The most common moves:

- **Diagnose before fixing.** "The build is slow, fix it" becomes: find the cause, report it, then wait.
- **Name what to check.** "Audit this file" gets the categories to audit against.
- **Name what comes back.** A ranked table, a grouped list, or a one-paragraph answer.
- **List before changing everything.** "Rename this everywhere" becomes: show me every place first.
- **Add the guardrail you forgot.** "Don't edit yet." "Ask before pushing."
- **Resolve "that file".** When the conversation makes it clear which file, the rewrite names it. When it doesn't, prompter asks.
- **Fix typos, keep your voice.** Spelling gets fixed; casual phrasing stays.

The rewrite appears under the label **Rewritten prompt:**, and nothing runs until you answer.
Say yes and the rewrite runs. Correct it and you get a new version. Say no and your original runs
exactly as you typed it.

---

## What it leaves alone

prompter stays out of the way of anything that isn't a new request:

- answers like "yes", "go ahead", or "Option 2"
- replies to a question Claude asked you
- slash commands such as `/commit`
- pasted logs, stack traces, code, and diffs
- prompts that are already clear

It also never changes what you're asking for. If you asked for a fix, the rewrite won't add a
refactor. It may add a step that comes first, like "find the cause before changing code", and you
see that before anything runs.

---

## What a rewrite looks like

<!-- examples/Prompter-Examples.md links to this heading (#what-a-rewrite-looks-like). Update that link if you rename it. -->

**Original:**

> the tests are flaky, fix them

**Rewritten:**

> Diagnose flakiness in the test suite before changing anything. Categorize each failing test as: (1) timing-dependent (sleeps, polling, real clocks), (2) order-dependent (shared state, leaking fixtures), (3) environment-dependent (network, filesystem, machine-specific), or (4) genuinely racy (concurrency bugs in the code under test). Produce a ranked table with the test name, category, and confidence. Don't apply fixes yet.

What prompter did:

- **Refused to skip the diagnosis.** "Fix them" became "diagnose, categorize, rank." The original would have started editing tests before anyone knew *which kind* of flake was at play, and a fix for a timing flake can make an order-dependence flake worse.
- **Pinned the analysis dimensions.** Four well-known flake categories that every test engineer recognizes, rather than "whatever Claude thinks is wrong."
- **Specified the deliverable shape.** A ranked table with name, category, and confidence: reviewable in 30 seconds, not a wall of prose.
- **Added the missing guardrail.** "Don't apply fixes yet," the obvious thing the user would have written if they hadn't been mid-thought.

[All 21 worked examples](examples/Prompter-Examples.md) cover typo repair, unclear references,
stacked questions, threat-model framing for "find a security issue in the auth code", and the
cases where prompter correctly leaves a prompt alone.

---

## Turning it on

Type `/prompter` in any session (`/prompter:prompter` if you installed it as a plugin) and pick
how long it should stay on:

| Mode | What happens |
|---|---|
| **Try it once** | Evaluates your next prompt, rewrites it if that helps, then stops. A good first run. |
| **This session** | Evaluates every prompt for the rest of this conversation. No files change. |
| **Add to CLAUDE.md** | Writes a short Prompt Rewriting rule into this project's `CLAUDE.md` or your global `~/.claude/CLAUDE.md`, your choice, so it stays on in future sessions. If a rule is already there, prompter shows you the difference and keeps yours unless you say otherwise. |
| **Remove from CLAUDE.md** | Takes the rule out again. Offered only when the rule exists, and names the file it's in. |

**Try it first.** Choose "Try it once," then send a one-liner like *"the tests are flaky, fix
them"* and see what prompter makes of it.

**When the menu appears.** Only when you type `/prompter` by itself. Type `/prompter` with a
prompt in the same message and prompter evaluates that prompt. Ask directly ("make this prompt
clearer: …") and you get the rewrite. With the rule in CLAUDE.md, new sessions start rewriting
without asking again.

It doesn't learn your style: every prompt is judged fresh against the rules in
[SKILL.md](skills/prompter/SKILL.md). To change how it rewrites, edit that file, or add your own
lines to the Prompt Rewriting section in your CLAUDE.md.

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

The skill lives in `skills/prompter/` inside the repo, so copy that folder, not the repo root: a skills folder that holds the whole repo has no `SKILL.md` at the top and the skill will not load.

**Global install** (all projects):

```bash
mkdir -p ~/.claude/skills && cp -r prompter/skills/prompter ~/.claude/skills/
```

**Project-specific install** (one project only):

```bash
mkdir -p /path/to/project/.claude/skills && cp -r prompter/skills/prompter /path/to/project/.claude/skills/
```

**Updating.** For a plugin install, run `claude plugin update prompter@prompter` in a terminal.
For a manual install, copy the new `skills/prompter/` folder over the old one. Either way, start a
new session afterward: a running session keeps the version it loaded. Coming from an earlier
version? [UPGRADING.md](UPGRADING.md) lists what changed.

---

## Where it stands

**v1.4.0.** Built and used daily while shipping [Stuffolio](https://stuffolio.app). It started
inside [tutorial-creator](https://github.com/Terryc21/tutorial-creator) (then called
`code-smarter`) and moved to its own repository, commit history included, so people looking for
prompt rewriting could find it.

**Real limits, worth knowing:**

- **It only knows what the conversation shows.** It can name "that file" only if the file came up.
  Otherwise it asks, which costs you a reply.
- **Every rewrite costs one reply.** Nothing runs until you approve, correct, or decline. Clear
  prompts cost nothing, because prompter skips them.
- **A long session can lose track of the mode.** When Claude Code compacts a long conversation,
  "Try it once" and "This session" can be forgotten. Run `/prompter` again to restart. The
  CLAUDE.md rule doesn't have this problem.
- **The trigger tests haven't been re-run since v1.4.0.** Earlier runs scored 0% on prompts that
  should have triggered the skill, because the mode menu blocked the test harness. v1.4.0 removes
  that cause; [tests/README.md](tests/README.md) has the details.

**What would help most:** [open an issue](https://github.com/Terryc21/prompter/issues) with a
prompt prompter rewrote badly, or one it should have rewritten and didn't.

---

## More Claude Code skills

Other skills from the same author, each in its own repository.

**Working with Claude Code**
- [unforget](https://github.com/Terryc21/unforget): one ledger per project for deferred work, so
  nothing you put off gets lost between releases.
- [inspect-before-install](https://github.com/Terryc21/inspect-before-install): audits a third-party
  repository before you run its install command.
- [skill-reviewer](https://github.com/Terryc21/skill-reviewer): candid reviews of Claude Code skills,
  with file and line citations and ranked actions.

**Finding bugs in Swift and SwiftUI apps**
- [radar-suite](https://github.com/Terryc21/radar-suite): five audit skills and a capstone grade,
  run as one pipeline, that find bugs before your users do.
- [workflow-audit](https://github.com/Terryc21/workflow-audit): traces SwiftUI user flows for dead
  ends, broken promises and miswired data.
- [bug-prospector](https://github.com/Terryc21/bug-prospector): looks for the bugs that
  pattern-based auditors miss, such as broken assumptions, state-machine gaps and boundary conditions.
- [bug-echo](https://github.com/Terryc21/bug-echo): after you fix a bug, finds and rates other
  instances of the same defect, and fixes them only with your approval.
- [one-star-risk](https://github.com/Terryc21/one-star-risk): re-scores audit findings by how likely
  each one is to earn a one-star App Store review.
- [sitrep](https://github.com/Terryc21/sitrep): A to F grades for an iOS or Swift codebase across
  nine categories, reported once for engineers and once for the people who fund them.

**Learning**
- [tutorial-creator](https://github.com/Terryc21/tutorial-creator): annotated code-reading lessons
  built from your own codebase, with a quiz and a vocabulary you keep.
- [lexicon](https://github.com/Terryc21/lexicon): collects unfamiliar terms from what you read and
  work on into subject glossaries.

---

## Author

Terry Nyberg, [Coffee & Code LLC](https://stuffolio.app/). If prompter has saved you a wasted reorientation turn, [a coffee](https://buymeacoffee.com/stuffolio) is appreciated. Issue reports about what worked or didn't are more useful.

[![Buy Me A Coffee](https://img.shields.io/badge/Buy%20Me%20A%20Coffee-FFDD00?style=flat&logo=buy-me-a-coffee&logoColor=black)](https://www.buymeacoffee.com/stuffolio)

## Privacy

prompter collects no data of its own. It has no server, no accounts, no telemetry and no analytics, and it sends nothing to its author. It reads your prompt and the conversation around it, and, when you open its menu, your CLAUDE.md files to see whether its rule is already there. It writes nothing unless you choose "Add to CLAUDE.md" or "Remove from CLAUDE.md", which change one short rule block in the CLAUDE.md you pick: this project's, or your global `~/.claude/CLAUDE.md`. It makes no network requests and runs no scripts. Anything it saves stays on your own machine. What you ask it to read is processed by Claude in your own session, under your own Anthropic terms and settings. Anthropic's plugin directory may show the author aggregate install counts; those come from the directory, not from the plugin.

## License

Apache 2.0. See [LICENSE](LICENSE) and [NOTICE](NOTICE).
