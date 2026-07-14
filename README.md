# xianzhu-skill

> A Codex skill for systematic, auditable, and interpretation-first specification search in empirical economics.

`xianzhu-skill` is designed for the common situation where the research question is already fixed, but the researcher still needs a disciplined way to test admissible transformations, alternative measurements, and compatible estimation strategies around an existing `x -> y` relationship.

Instead of encouraging blind p-hacking, the skill turns “try until it becomes more convincing” into a transparent workflow with explicit boundaries, experiment logs, and model-selection discipline.

## Version

**Current release:** `1.0.0`

## What This Skill Is For

This skill is useful when the user says things like:

- “试到显著”
- “换个口径试试” 
- “这个变量能不能做 `log` / `asinh` / `ln(x+1)`？”
- “方向是对的，但不显著，怎么办？”
- “别换研究问题，围绕当前方程继续试”

Typical use cases include:

- improving the stability of an existing empirical result
- exploring admissible transformations of the current treatment or outcome
- deciding whether to keep the current proxy or move to a substitute measure
- keeping a complete and auditable record of failed and retained specifications
- producing manuscript-ready justification for why one specification is chosen over others

## Core Design Principles

`xianzhu-skill` is built around five principles:

1. **Fixed question first**  
   The research question must already be known. The skill is not for inventing topics from scratch.

2. **Current indicator before substitute indicator**  
   It first searches transformations of the current variable, and only after that considers same-concept substitutes.

3. **Interpretation before significance**  
   A more significant result is not automatically a better result if its economic meaning becomes weaker.

4. **Failure must be recorded**  
   The workflow never hides failed specifications. Search logs are part of the output.

5. **Stata-friendly empirical workflow**  
   The skill is designed to fit real economics projects where do-files, logs, and output tables must be reproducible and manuscript-ready.

## Workflow Overview

The main workflow in `SKILL.md` follows this logic:

1. Lock the boundary of the research question.
2. Decide whether the search target is mainly `x`, `y`, or both.
3. Choose a compatible method set.
4. Search the current indicator first.
5. Log every attempt.
6. Only after current-indicator failure, consider substitute indicators.
7. Return both a recommended specification and a full experiment log.

In practice, the skill distinguishes between two modes:

- **Mode A: identify then search**  
  Use when the user has not fully fixed the exact variable or equation.

- **Mode B: direct experiment**  
  Use when the user already has an equation, result table, or do-file and just wants to keep pushing the same empirical question.

## Repository Structure

```text
xianzhu-skill/
├── SKILL.md
├── README.md
├── LICENSE
├── agents/
│   └── openai.yaml
├── evals/
│   └── evals.json
└── references/
    ├── case-examples.md
    ├── method-menu.md
    ├── mode-selection.md
    ├── spec-log-template.md
    ├── stata-output-conventions.md
    └── transform-playbook.md
```

## Reference Files

The repository includes several supporting references:

- `references/method-menu.md`  
  A compact menu of admissible estimation approaches.

- `references/transform-playbook.md`  
  Practical guidance on `log`, `ln(x+1)`, `asinh`, winsorization, discretization, and re-expression.

- `references/spec-log-template.md`  
  Minimum traceability requirements for specification-search logs.

- `references/stata-output-conventions.md`  
  Naming, do-file separation, and output/log conventions for Stata-based empirical work.

- `references/case-examples.md`  
  Reusable examples showing how the workflow should behave in real economics tasks.

## What This Skill Does Not Do

This skill is intentionally narrow. It does **not**:

- invent a research question from nothing
- replace literature grounding
- fabricate substitute variables without support
- hide unsuccessful results
- turn every “more significant” specification into the recommended one
- treat aggressive p-hacking as valid empirical work

## Recommended Output Style

A successful run should usually produce:

- mode identification (`Mode A` or `Mode B`)
- the locked-in `x`, `y`, and method choice
- the side currently being searched
- a list of tried specifications
- the preferred specification
- rejected specifications and reasons
- a complete experiment log for reproducibility

## Intended Audience

This repository is especially useful for:

- empirical economics researchers
- paper-revision workflows
- Stata-first regression projects
- Codex users who need a specification-search skill with explicit methodological boundaries

## License

MIT
