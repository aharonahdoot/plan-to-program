# plan-to-program

A [Claude Code skill](https://docs.claude.com/en/docs/claude-code) that turns a build request
into (1) an architecture plan strong enough to bet on, then (2) a multi-agent execution
program sized to the token throughput you declare ("3 lanes at 15k each").

Extracted from a real workflow: this is the method behind the numbered programs that built
[RecompCore](https://github.com/aharonahdoot/RecompCore),
[GXRuntime](https://github.com/aharonahdoot/GXRuntime), and
[StrikersRecomp](https://github.com/aharonahdoot/StrikersRecomp) — see RecompCore's
[`docs/development-history/`](https://github.com/aharonahdoot/RecompCore/tree/main/docs/development-history)
for what its artifacts look like in production (a program board, per-arc handoffs, and the
knowledge base the agents maintained).

## The two phases

- **Phase 1 — the plan.** A decision-grade, source-grounded document: a verdict on whether the
  request is even right, a cited seam map (`file:line` or it doesn't go in), every unknown
  paired with its cheapest killing experiment, milestones with objective gates. Honesty is the
  product — "there's a far simpler way" is a successful outcome.
- **Phase 2 — the program.** The approved plan decomposed into **windows × lanes**: stable
  agent identities with disjoint file ownership, whole units with named sub-boundaries and
  token budgets, lock classes for exclusive resources (GUIs, single-instance tools, shared
  build dirs), paste-ready dispatch cards, and a resume contract so a cold-started agent can
  continue from the docs alone.

Every concept in the skill exists because a real failure mode demanded it; the anti-patterns
list at the bottom of `SKILL.md` is a list of things that actually burned a real program.

## Install

Copy this directory to your user-level skills:

```sh
git clone https://github.com/aharonahdoot/plan-to-program \
  ~/.claude/skills/plan-to-program
```

Claude Code picks it up automatically; it triggers on plan/architecture requests and on
"make this a program", or invoke it explicitly with `/plan-to-program`.

## Files

| File | Role |
|---|---|
| `SKILL.md` | The skill: research standards, decomposition algorithm, budget math, environment notes (Claude Code / OpenCode), anti-patterns |
| `references/plan-template.md` | Phase-1 plan skeleton with per-section guidance |
| `references/program-templates.md` | Phase-2 skeletons: program board, resume prompt, orchestration playbook, agent definition, dispatch cards, plus a compressed filled example |

## License

MIT
