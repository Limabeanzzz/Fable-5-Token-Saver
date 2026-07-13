# Token Saver

A Claude skill for token-efficient orchestration. When a task involves bulk work — summarizing dozens of files, large extractions, multi-part workloads — this skill teaches Claude to hand the mechanical chunks to cheaper, faster models (Haiku, Sonnet) while keeping judgment, synthesis, and quality control with the top-tier orchestrator. The result: the same quality deliverable with less token burn in your main session.

## How it works

Every candidate subtask passes through a **four-gate test** before delegation:

1. **Self-contained** — describable in a written handoff with no session context needed
2. **Size** — the work must clearly outweigh the handoff overhead
3. **Verifiable by artifact** — returns a file or table checkable without redoing the work
4. **Quality floor** — objectively within the cheaper model's competence

Any gate fails → the work stays with the orchestrator. Quality outranks savings by design.

Delegation is **hybrid**: mechanical Haiku-grade work delegates automatically; Sonnet/Opus handoffs and anything touching your files get proposed first. Every task ends with a brief delegation report, and a capped 10-line Calibration section lets the skill improve from experience without ever growing its context cost.

## Designed for Claude Fable 5

The skill is tuned for Fable 5 as the orchestrator. On other models it degrades gracefully: it will offer to switch to Fable 5, or continue with routing adjusted so it only delegates downward from the current model.

## Install

**Claude desktop app (Cowork):** download `token-saver.skill`, drop it into a Cowork chat, and click **Save skill**.

**Claude Code:** copy the `token-saver/` folder into `~/.claude/skills/`.

## What's in this repo

- `token-saver.skill` — one-click installer package
- `token-saver/SKILL.md` — the skill itself (readable; this is what Claude follows)
- `token-saver/references/model-capabilities.md` — model routing reference, loaded only when needed
- `token-saver/evals/evals.json` — the test cases used to validate the skill

## Benchmark

Tested against a no-skill baseline on three tasks (bulk summarization, small-task restraint, mixed extraction+analysis): 100% assertion pass rate with the skill vs 72% without, with correct delegation behavior in all three cases — including correctly *refusing* to delegate when overhead would exceed the job.
