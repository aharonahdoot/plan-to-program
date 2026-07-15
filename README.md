# plan-to-program

A [Claude Code skill](https://docs.claude.com/en/docs/claude-code) for getting big things
built by AI agents without the project collapsing into slop. It forces the work through two
phases: first a plan you could actually bet on, then that plan broken down into work that
multiple agents can run in parallel without stepping on each other.

It's the method behind the programs that built
[RecompCore](https://github.com/aharonahdoot/RecompCore),
[GXRuntime](https://github.com/aharonahdoot/GXRuntime), and
[StrikersRecomp](https://github.com/aharonahdoot/StrikersRecomp). If you want to see what its
output looks like on a real project, RecompCore publishes the actual artifacts —
[`docs/development-history/`](https://github.com/aharonahdoot/RecompCore/tree/main/docs/development-history)
has a real program board, the arc-by-arc handoffs, and the knowledge base the agents kept.

## How it works

**Phase 1: the plan.** Before anything gets built you get a plan, and the first thing it has
to answer is "should this even be built?" — if there's a simpler way, or the thing already
exists, the plan is required to say so. Every claim about the codebase has to cite a real
file the agent actually read, not a vibe. And anything unknown doesn't get assumed away — it
becomes a small cheap experiment ("does X work on this platform? one build, one run, then we
know").

**Phase 2: the program.** When you approve the plan and say what you can afford ("I can run 3
agents at 15k tokens each"), the skill turns it into a grid of work windows and agent lanes.
Each agent owns its own files and is banned from touching anyone else's. Work is cut into
whole units that fit inside a token budget — and if a unit is too big it gets split at a
named stopping point instead of quietly shrunk, because "skip the rare cases" is how projects
rot. Anything only one agent can safely use at a time (a GUI, an emulator, a shared build
folder) gets a named lock so two agents never fight over it and invent phantom bugs. And
everything gets written down well enough that a brand new session with zero memory can pick
up exactly where the last one died. That last part is the whole trick — the model is rarely
the bottleneck, memory and planning are.

## Why it looks the way it looks

Every rule in here exists because an agent burned real tokens teaching it. Two agents editing
the same file. An agent "finishing" a unit by quietly cutting its scope. A gate that said
"looks right" instead of a number someone could check. The anti-patterns list at the bottom
of `SKILL.md` is basically a list of scars.

## Install

```sh
git clone https://github.com/aharonahdoot/plan-to-program \
  ~/.claude/skills/plan-to-program
```

Claude Code picks it up on its own — it triggers when you ask for a plan or say "make this a
program", or you can call it directly with `/plan-to-program`.

## What's in here

- `SKILL.md` — the method itself
- `references/plan-template.md` — the shape of a Phase-1 plan, section by section
- `references/program-templates.md` — the Phase-2 templates (the program board, the resume
  prompt for cold-started agents, the orchestrator playbook) plus a compressed real example

## License

MIT. Take it, change it, I'm not precious about it.
