# Tests

`trigger-eval-set.json` — should-trigger / should-not-trigger cases for prompter's activation description, in the format `skill-creator`'s `run_eval.py` / `run_loop.py` expect (`{"query": "...", "should_trigger": true|false}`).

There's no in-repo runner. Run it with the `skill-creator` plugin's eval tooling:

```bash
uv run --python 3.12 --with pyyaml -m scripts.run_loop \
  --eval-set /path/to/prompter/tests/trigger-eval-set.json \
  --skill-path /path/to/prompter \
  --model claude-sonnet-5 \
  --max-iterations 5 \
  --verbose
```

(`run_loop.py` lives in the `skill-creator` plugin's `scripts/` directory, not in this repo.)

**Known limitation: 0% recall is expected under `run_loop.py`, not a real defect.** Prompter's first-invocation behavior (`SKILL.md`, "Initial invocation") calls `AskUserQuestion` to show the mode picker — but `AskUserQuestion` doesn't exist in the headless `claude -p` subprocess the eval harness uses to test each query. The skill *does* trigger correctly (confirmed 2026-08-07 via a direct manual run — `TOOL_USE: Skill` fires immediately), it just can't get past the mode picker in a non-interactive shell, so it never emits the `**Prompter:**`-prefixed rewrite line the harness scores as a pass. This holds regardless of how the description is worded — six iterations across two runs (2026-08-06/07) all scored identically (50% accuracy, 0% recall on should-trigger cases), because the harness is testing something this skill's activation path can't demonstrate outside an interactive session.

Don't re-run `run_loop.py` against this eval set expecting it to converge — it can't, until prompter's first-invocation path degrades gracefully without `AskUserQuestion` (e.g., falling back to a plain-text mode prompt). Before trusting *any* other unexpected eval score from this file, spot-check the failing query manually first: `claude -p "<query>" --model claude-sonnet-5` from this repo, with `--output-format json` if you need to see which tool actually fired.
