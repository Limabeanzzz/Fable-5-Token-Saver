---
name: token-saver
description: Token-efficient orchestration by delegating self-contained subtasks to cheaper models (Haiku, Sonnet) via the Agent tool. Use this skill whenever a task involves bulk file processing (many files to read, summarize, classify, or convert), large extraction or drafting workloads, multi-part tasks with independent chunks, or long sessions where the main context should stay lean — even if the user never mentions delegation, subagents, or model names. Also use it when the user asks about saving tokens, reducing cost, running things in parallel, or "handing off" work. Do NOT use for single-step tasks, quick questions, or work that depends on live screen/UI state.
---

# Model Delegation

Orchestrate like a general contractor: keep judgment, sequencing, and quality control with the orchestrator (the model running the session), and subcontract mechanical, well-specified work to cheaper models. The goal is a final product of undiminished quality delivered with fewer total tokens in the orchestrator's context.

The reason this works: subagent token burn happens in the *subagent's* context, not yours. Sixty files read by a Haiku subagent cost you only the handoff prompt and the returned artifact. The reason it can fail: subagents start cold — zero session memory — so a handoff that needs your accumulated context produces garbage. The four gates below exist to separate those two cases.

## Orchestrator Check (run once, when the skill first triggers)

This skill was designed for Claude Fable 5 as the orchestrator. Identify which model you are. If you are Fable (or Mythos-class), proceed. If not, tell the user before orchestrating — once, briefly: "This skill was designed for Claude Fable 5. Want to switch to Fable 5 for this task, or continue with me? I'll adjust routing so I only delegate to models cheaper than myself." Use AskUserQuestion if available. Respect their choice and don't ask again in the session. If continuing, shift the routing table downward relative to yourself — delegating sideways or upward defeats the purpose (see references/model-capabilities.md).

## The Four-Gate Test

Run every candidate subtask through all four gates. Any single failure → keep the work inline. When in doubt, don't delegate; quality outranks savings.

1. **Self-contained.** The subtask must be fully describable in a written handoff (paths, formats, definition of done). If doing it well requires the current screen state, earlier conversation decisions, or feel for the user's intent, it fails.
2. **Size.** The subtask's working tokens must clearly dwarf the handoff overhead (writing the prompt + the sub re-reading inputs + you verifying output). Reading 40 files passes; editing one column fails — the paperwork costs more than the job.
3. **Verifiable by artifact.** The sub must return something checkable cheaply — a file, table, or diff you can spot-check or validate with a script, without redoing the work. If verification costs as much as doing it inline, it fails.
4. **Quality floor.** The task type must be objectively within the cheaper model's competence: mechanical, well-specified, with a crisp definition of done. Anything needing judgment, ambiguity resolution, or final synthesis stays with the orchestrator regardless of gates 1–3.

## Routing Table

| Subtask type | Route to |
|---|---|
| Bulk extraction, classification, per-file summaries, format conversion, boilerplate | haiku |
| Report drafting from finished data, standard coding, clearly-scoped research | sonnet |
| Architecture decisions, ambiguous analysis, final synthesis, final verification | keep in orchestrator |
| Anything needing live UI, screen, or session state | never delegate |

When a routing call is unclear (a subtask sits between tiers, or you're orchestrating from a non-Fable model), read references/model-capabilities.md — it stays out of context until needed.

## Trigger Policy (hybrid)

- **Auto-delegate** Haiku-grade mechanical work that passes all four gates. Note it in the task list; no need to ask.
- **Propose first** before any Sonnet/Opus handoff, and before any delegation that writes to or modifies the user's own files (not the sandbox/outputs scratch space). One or two lines: subtask, model, why it passes the gates. Wait for OK.
- If the Agent tool is unavailable in the current environment, note this in the delegation report and do the work inline — never skip work because delegation wasn't possible.

## Handoff Prompt Template

Subagents start cold, so the prompt must be self-sufficient. Include every field:

```
Goal: <one sentence>
Inputs: <absolute file paths; never "the files we discussed">
Output: <exact format and absolute destination path>
Constraints: <e.g., read-only on inputs; no network; style rules>
Definition of done: <objective completion test>
Context you need: <the 2-3 facts from the session the task depends on — nothing else>
```

## Orchestrator Rules

- Spawn independent handoffs in parallel in the same turn; sequential spawning wastes wall-clock time for nothing.
- Reuse a running agent with SendMessage instead of respawning — a respawn pays the cold-start cost twice.
- Verification of merged results always stays with the orchestrator. Prefer a validation script (row counts, schema checks, spot-diffs) over re-reading the sub's inputs.
- Never delegate the final quality pass on the deliverable.

## End-of-Task Delegation Report

Close every task where this skill triggered with a brief report (3–6 lines, prose or compact list):

- What was delegated, to which model, and what came back.
- What was considered but kept inline, and which gate blocked it.
- Rough token effect (e.g., "≈60 file-reads kept out of main context").
- **Proposed calibration edits**, if this task revealed a misjudgment — phrased as a replace/merge against the Calibration section below. If nothing was learned, say "no calibration changes."

The report is the feedback loop: the user reviews proposed edits, and approved edits get applied to the Calibration section. In environments where skill files are read-only, present the exact edited line(s) so the user can apply them via skill settings or a reinstall.

## Calibration

Hard cap: 10 bullet lines. Edits must replace or merge existing lines — never append beyond the cap. This keeps the skill's recurring context cost constant no matter how much it learns.

- Structured/greppable text at any volume → process with a script inline; delegate to Haiku only when per-item judgment (summarizing, classifying free text) is required. (User-approved, from log-analysis eval, 2026-07-13.)
