---
name: plan-to-program
description: Two-phase delivery workflow. Phase 1 - produce a decision-grade, source-grounded architecture plan (verdict on whether the request is even right, cited seam maps, unknowns with cheapest killing experiments, milestones with objective gates). Phase 2 - decompose the approved plan into a multi-agent execution program - windows x lanes with explicit token budgets, disjoint file ownership, lock classes for contended resources, dispatch cards, and resume contracts. Use this skill whenever the user asks for a plan, architecture, design doc, or implementation strategy (especially when the session starts in plan mode via /plan); whenever they ask to "make this a program", "break this into windows/lanes/units", or mention orchestrators, parallel agents, or agent throughput like "4 agents at 15k tokens each"; and even when they only ask for the plan - offer the program decomposition after approval.
---

# Plan → Program

Turn a build/architecture request into (1) a plan strong enough to bet on, then (2) a
multi-agent execution program sized to the throughput the user declares. Phase 1 typically runs
inside plan mode (`/plan`); Phase 2 starts when the user approves and states capacity
("I can support 4 agents at 15k tokens each"). Enter directly at Phase 2 when a plan already
exists and the user says "make this a program" / "decompose this into windows".

The two phases share one purpose: **everything downstream must be executable by a cold-started
agent who has read nothing but the artifact in front of it.** That standard — not completeness
for its own sake — decides what goes in every document.

---

## Phase 1 — the plan

### Research before writing

The plan's authority comes from evidence, not eloquence:

- **Do the cold-start reads yourself.** Read the sources the user names, then follow what you
  find. Don't delegate reads whose details you'll need to cite — a plan built on a subagent's
  summary inherits its blind spots.
- **Cite everything load-bearing** (`file.cpp:123`, doc §, commit sha). A claim you can't cite
  is a hypothesis — move it to the unknowns list instead of asserting it.
- **Every unknown becomes a named experiment with a cost.** "Does X support Y on this platform?
  → one build + one run." If resolving an unknown is cheaper than designing around it, make it
  a milestone-0 checklist item rather than a design assumption.
- **Hunt for what already exists.** The strongest plans discover the problem is half-built, or
  that the request is already solved, or wrongly conceived — and say so. Check the project's
  own tools/docs/prior art before inventing anything.

### The document

Follow `references/plan-template.md` for the skeleton and per-section guidance. The invariant
sections: verdict on the request itself → mission / non-goals / terminology → chosen
architecture + rejected alternatives → source-grounded integration map with unknowns +
experiments → milestones with objective gates and resource bounds → risks / anti-patterns /
stop conditions → file ownership. Domain-specific sections (schemas, comparison strategies,
gate chains) appear only when the domain needs them.

Standards that make the plan trustworthy:

- **Verdict first.** If the user asked "is this necessary / already solved / is there a simpler
  way?", answer that before designing — including "yes, there's a simpler way" when true.
  Deliver the correction, not the requested artifact, when they conflict.
- **Be decisive.** One recommended architecture. Each rejected alternative gets a short
  paragraph naming the evidence that killed it — not a neutral survey.
- **Honest boundaries.** State exactly where guarantees end (what's provable vs.
  tolerance-gated vs. unverifiable). If a required capability might not exist, plan to stop at
  the first proof of infeasibility and name the fallback.
- **Objective gates.** Milestone pass criteria a script could check: counts, byte-identical
  reruns, a deliberately planted defect getting caught. "Looks right" is not a gate. Add
  resource bounds (disk/time/quota) wherever they bind.
- **Ranked later ≠ dropped.** Out-of-scope-for-now work is named and queued explicitly, so
  descoping is a visible decision, not silent erosion.

### Plan-mode mechanics

Write the plan into the session's plan file and call ExitPlanMode. Two common outcomes:

- **Approved with throughput stated** → go to Phase 2.
- **Rejected with "now make it a program"** → still in plan mode: extend the *same plan file*
  with the Phase-2 design (window × lane matrix + unit brief headlines + the artifact write
  list), ExitPlanMode again, and write the program artifacts after approval.

---

## Phase 2 — the program

### Throughput parameters (ask when not given)

Four inputs shape the whole decomposition. Infer what you can from the conversation; use
AskUserQuestion for the rest — guessing wrong invalidates every budget in the program.

1. **Lane count** — how many agents can run concurrently.
2. **Per-lane output-token budget** (e.g., 15k, 125k). This decides unit size.
3. **Environment** — Claude Code (Agent tool: named agents, SendMessage to continue),
   OpenCode (Task tool + `task_id` to continue), other, or mixed.
4. **Exclusive resources** — GUI apps, single-instance emulators/tools, hardware, deploy
   targets, shared build dirs. Usually inferable from the plan; confirm anything surprising.

Optional: a per-window total ceiling, and how many windows/sittings the user can run.

### Core concepts (each exists because a real failure mode demands it)

| Concept | What it is | Why it exists |
|---|---|---|
| **Program** | Numbered, self-contained doc set: board + lane contract + orchestration + specs | Any cold agent must be able to resume from the docs alone, sessions later |
| **Window** | One dispatch cycle: all lanes run concurrently, one whole unit each, then stop | Fixed sync points make landings, budget accounting, and re-planning tractable |
| **Lane** | Stable agent identity (impl-a/b/c…) with standing file ownership and lock grants | Continuation beats respawn (context is an asset); stable ownership prevents cross-lane edits |
| **Unit** | One lane-window of *whole* work: numbered steps, an objective gate, a named sub-boundary, owns/never-touches, budget | "One whole unit then stop" keeps every stop point clean; sub-boundaries make early wind-downs resumable instead of half-done |
| **Spec doc** | Interface contract authored *before* the lanes that implement it | Lets lanes build against a document instead of each other — this is what makes them independent within a window |
| **Lock class** | Named exclusive resource; at most one lane may hold it per window | Two agents sharing a GUI/single-instance tool/build dir invent phantom failures that burn real debugging time |
| **Dispatch by pointer** | Dispatch names the unit id + budget + ownership + locks; the brief lives on the board | Pasting context into dispatches re-pays the cold-start cost on every spawn |
| **Cost priors** | Rough unit-size classes (small/mid/heavy) in tokens, re-measured after window 1 | Budgets are estimates; the program must correct itself with real numbers |
| **Contingency window** | A final window holding only spillover + backlog | Overruns land somewhere planned instead of distorting the matrix |

### Decomposition algorithm

1. **Inventory** work items from the plan's milestones; note hard dependencies between them.
2. **Partition by footprint.** Group items into territories of disjoint files/dirs — ownership
   is by path, not by topic. Identify every exclusive resource and which items need it.
3. **Fix lane count** = min(user's lanes, territories with at least a meaningful unit of work
   per window). Litmus for each lane: *"If this lane's work disappeared, would the schedule
   slip?"* No → fold it into another lane rather than inventing parallel busywork.
4. **Assign exclusive resources**: each lock class belongs to exactly one lane for the whole
   program when possible (e.g., only impl-a ever runs the contended tool). Design the other
   lanes to never need it — that's what makes full parallelism safe.
5. **Size units to the budget.** A natural unit that exceeds the lane budget is split at
   *named sub-boundaries* into fully-complete sub-units across windows — never scope-cut to
   fit ("skip the rare cases" is a schedule decision disguised as a design one).
6. **Arrange windows** so every unit consumes only prior-window artifacts plus the spec doc.
   Window 1 = all items whose inputs are the plan + spec alone. Integration units (running
   both sides together) go to the lane holding the relevant lock, in the window after the
   components land.
7. **Author the spec doc now** (with the program docs, before any lane starts) if two or more
   lanes must agree on any format, schema, or interface. Mark it orchestrator-owned during
   windows: lanes propose deltas in reports; one writer applies them.
8. **Check the budget:** window burn ≈ orchestrator (~10–15% of the window) + Σ lane budgets.
   Over the ceiling → trim the least plan-critical lane first. Then add the contingency window
   and an explicit backlog table for everything ranked later.

### Budget math — degenerate cases to handle deliberately

- **Tiny budgets** (≲20k/lane): units shrink to single-coherent-change scale; cold start must
  be ≤ ~2–3k (embed the contract in the agent def / dispatch, forbid re-reading large docs);
  expect many windows. If a unit's real cost < ~2× the cold-start overhead, parallelism is
  losing to wind-up — tell the user and propose fewer, bigger lanes or a solo queue.
- **Many lanes**: ownership partitioning binds before budget does. If the work only splits
  into 3 disjoint territories, 8 lanes can't help — say so rather than manufacturing lanes.
- **One lane / solo**: skip the orchestration doc; the program is a serial queue with the same
  board + resume contract. The window concept still helps as a natural checkpoint cadence.

### Artifacts

**Inherit the project's convention first**: if the repo already has a control-plane directory
with prior programs (numbered handoff dirs, goals/ledger files, an orchestration playbook),
match its structure, numbering, and tone — write an overlay referencing the existing playbook
instead of duplicating it. Otherwise create the default numbered set under
`<project>/programs/<NN>_<slug>/` (NN = next free integer):

| File | Role |
|---|---|
| `00_PROGRAM.md` | The board: mission, decision rules, live state section, window × lane matrix, full unit briefs, program gates, ownership map, cost priors, risk register, file index |
| `01_RESUME_PROMPT.md` | Per-session lane contract: minimal cold-start reads, hard rules, known failure modes, build/run blocks, wind-down ceremony |
| `02_ORCHESTRATION.md` | Orchestrator playbook: roles + budgets, paste-ready dispatch cards per window, lock protocol, window checklist, append-only retros |
| `03_<NAME>_SPEC.md` | Only when ≥2 lanes share an interface (see step 7) |
| Agent definition | Claude Code: `.claude/agents/<program>-implementer.md` embedding the lane contract. OpenCode: the same contract block lives in `02_ORCHESTRATION.md` to paste once per spawn |

Skeletons + a filled example for all of these: `references/program-templates.md`. Also update
whatever project state files exist (goals/ledger/handoff equivalents), and commit the docs —
the program isn't real until a cold session can find it.

### Environment differences

| Concern | Claude Code | OpenCode |
|---|---|---|
| Spawn lane | Agent tool with `name:` (impl-a…), agent def embeds contract | Task tool; paste the contract block once per spawn |
| Continue lane | SendMessage to the living agent | Task with the prior `task_id` |
| Respawn | Only when context is bloated or the domain changes | Same |
| Contract delivery | Agent definition file | Contract block in the dispatch prompt |

Doctrine is environment-agnostic: one whole unit per lane per window, private
build/artifact paths per lane, dispatch by pointer, orchestrator owns shared state files.

## Anti-patterns (each one has burned a real program)

- **Two lanes editing one file** — ownership is by path; a shared hotspot file gets one owner
  and the other lane sends diffs through its report.
- **Underfilled lanes** — spawning a lane for a scrap of work doubles wind-up for nothing;
  fold scraps into a heavier lane or do them yourself.
- **Scope-cutting to fit a budget** — split at sub-boundaries instead; completeness debt must
  stay visible in the backlog.
- **Dispatch-by-paste** — copying board prose into prompts re-pays cold start every time.
- **Vague gates** ("works", "looks right") — every gate must be checkable by someone who
  didn't do the work: counts, identical reruns, planted defects caught.
- **Blaming the environment first under parallelism** — a lane that hits a failure while a
  peer runs must baseline against the pre-change state before attributing (encode this in the
  lane contract).
- **Silent skip-passes** — a gate with missing preconditions reports INVALID/blocked, never
  green.
