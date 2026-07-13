# Model Capabilities Reference

Consult this file only when a routing call is unclear — it is not loaded by default, so it costs nothing until needed. Sourced from Anthropic's documented model characteristics; the Calibration section in SKILL.md overrides anything here when they conflict, because calibration is empirical and this file is general.

## Cost/capability hierarchy (most capable and expensive first)

Fable 5 (Mythos-class) > Opus > Sonnet > Haiku

Routing is always *relative to the orchestrator*: delegate only downward in this hierarchy. Delegating sideways (Sonnet orchestrator → Sonnet sub) buys context isolation but no cost savings — do it only when the goal is keeping the main context lean, and say so in the delegation report.

## Haiku (e.g., claude-haiku-4-5)

- Strengths: fastest and cheapest; high-volume mechanical work — classification, extraction, per-file summarization, format conversion, boilerplate generation, simple well-specified coding.
- Weaknesses: multi-step reasoning under ambiguity, long-horizon planning, nuanced synthesis, tasks where the definition of done is fuzzy.
- Delegation notes: ideal for fan-out work (N files × same operation). Give it rigid output formats; it follows templates well but improvises poorly.

## Sonnet (e.g., claude-sonnet-5)

- Strengths: the workhorse — strong coding, agentic tool use, report drafting from finished data, clearly-scoped research, medium-complexity synthesis.
- Weaknesses: deep architectural judgment, resolving genuinely ambiguous requirements, final-quality synthesis on high-stakes deliverables.
- Delegation notes: first choice for drafting and standard coding subtasks. If the handoff prompt needs more than a screen of context to be safe, the task probably belongs in the orchestrator instead.

## Opus (e.g., claude-opus-4-8)

- Strengths: frontier reasoning, complex analysis, hard debugging, high-quality long-form writing.
- Weaknesses: none relevant except cost and speed — which is the point of this skill.
- Delegation notes: rarely the right delegation target. Use only when a subtask needs real judgment AND must be offloaded for context-isolation reasons (e.g., analyzing a huge corpus the orchestrator must not load). Otherwise keep judgment work in the orchestrator.

## Fable 5

- The intended orchestrator for this skill. Do not delegate *to* Fable — if a subtask needs Fable-grade judgment, it fails Gate 4 and stays inline.
