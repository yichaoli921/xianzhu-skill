# xianzhu-skill

Version: 1.0.0

`xianzhu-skill` is a Codex skill for systematic and auditable specification search when a user wants to improve the stability or statistical significance of an economics regression result under a fixed research question.

## What it does

- distinguishes between identifying the problem first and direct specification search
- prioritizes transformations of the current indicator before substitute indicators
- enforces traceable experiment logs
- keeps failed specifications instead of hiding them
- favors interpretability over raw significance

## Structure

- `SKILL.md`: main skill definition
- `references/`: method menu, transform playbook, Stata conventions, and case examples
- `agents/`: agent metadata
- `evals/`: evaluation scaffold

## Intended use

Use this skill when the research question is already defined and the task is to search admissible specifications around an existing x-to-y relationship.

## Version

Current release: `1.0.0`

## License

MIT
