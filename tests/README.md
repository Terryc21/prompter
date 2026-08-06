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

**Known false negatives.** If every case in a should-trigger row fails identically regardless of how `SKILL.md`'s description is reworded, suspect the eval harness's `claude -p` subprocess getting intercepted by something outside prompter's control (a global `CLAUDE.md` rule, an unauthenticated CLI session, etc.) before the skill ever gets a chance to fire — not a real 0% recall. Spot-check the failing query manually with `claude -p "<query>" --model claude-sonnet-5` from this repo before trusting the eval score.
