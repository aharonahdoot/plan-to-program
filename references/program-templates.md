# Phase-2 program templates

Skeletons for the numbered doc set, the agent definition, dispatch cards, and the lane report
format — plus a compact filled example at the end. Adapt tone and headers to the target
project's existing convention when one exists; these are the defaults for a project without one.

Throughout, `<P>` = program number, `<slug>` = short program name, `<GOAL>` = the project's
goal/tracking id if it keeps one.

---

## 00_PROGRAM.md — the board

```markdown
# PROGRAM <P> — <TITLE> (<slug>)

> **Design authority:** <path to the approved Phase-1 plan>. **Shared contract:**
> `03_<NAME>_SPEC.md` (if any). **Lane contract:** `01_RESUME_PROMPT.md`.
> **Flow:** `02_ORCHESTRATION.md`.

## §1 Mission
One paragraph from the plan, restated operationally (what green looks like).

## §2 Decision rules (DR-#)
The plan's binding decisions as numbered rules a lane can cite, e.g.:
- DR-1 — <the architecture decision that must not drift>
- DR-2 — <exactness/tolerance boundary>
- DR-3 — <exclusive-resource rule: which lane holds which lock, always>
- DR-4 — Whole units: demand ranks ORDER, never scope; ranked-later items live in §9 backlog.
- DR-5 — A gate with unknown preconditions reports INVALID/blocked, never green.

## §3 State (orchestrator updates at every window close)
- <date>: Program created. Nothing executed. Next: Window 1.

## §4 Window × lane matrix
| | impl-a (<territory>; lock `<class>`) | impl-b (<territory>) | impl-c (<territory>) |
|---|---|---|---|
| **W1** | U-A1 <headline> (size) | U-B1 <headline> (size) | U-C1 <headline> (size) |
| **W2** | … | … | … |
| **W<N> (contingency)** | spillover + backlog only | — | — |

Dependency rule: every unit consumes only prior-window artifacts + the spec. No intra-window
waits — the spec doc is the interface.

## §5 Unit briefs (binding; one per unit)
### U-A1 — <name> (W1, impl-a, <size>, lock `<class>` | none)
1. <numbered concrete steps, with file paths and the plan/spec §-refs they implement>
2. …
**Gate:** <objectively checkable: counts, byte-identical reruns, planted defect caught>.
**Sub-boundary:** <named point where an early wind-down is clean> ⇒ "RESUME U-A1 at <point>".
**Owns:** <paths>. **Never touches:** <peer paths>. **Budget:** ~<N>k.

## §6 Lane ownership map (standing)
| Lane | Owns | Locks |
|---|---|---|

Private build/artifact dirs per lane; landing order rules for any shared hotspot file.

## §7 Program gates
- **G-TOOL / G-FEATURE:** <all units landed + every failure route demonstrated>.
- **G-ROLLOUT / G-CORPUS:** <the product-level closure, distinct from tool-complete>.

## §8 Cost priors + re-projection
Initial unit-size classes in tokens (small/mid/heavy) scaled to the lane budget; re-measure
after W1 and record actuals here (the program corrects itself with real numbers).

## §9 Risk register + backlog
| risk | trigger/mitigation |
Backlog: | item | why later, not never | earliest window |

## §10 File index
Pointers to every program file + the plan + state files.
```

## 01_RESUME_PROMPT.md — the per-session lane contract

````markdown
# PROGRAM <P> (<slug>) — per-session resume prompt

> **SESSION CONTRACT: ONE WHOLE UNIT. Complete it (or wind down at its NAMED sub-boundary),
> pass its gates, wind down once, stop.** Multi-agent windows: `02_ORCHESTRATION.md`.
> Solo sessions run this file directly.

## Get oriented (cold-start reads — nothing else)
1. `00_PROGRAM.md`: §2 rules, §3 state, YOUR unit's §5 brief in full.
2. `03_<NAME>_SPEC.md` — only if your unit implements/consumes the shared interface.
3. <project state file> latest entry + newest dated handoff in this dir.
4. On-demand project knowledge: <the 2-3 routed files worth naming>.
Do NOT re-read the full plan, other programs' boards, or historic handoffs.

## Hard rules (program-specific)
- <exclusive-resource lock rule + protocol location>
- <read-only paths; editable paths>
- <data hygiene: what never enters committed artifacts>
- <resource caps: disk/time/quota gates that abort rather than trim silently>

## Failure modes (inherited from prior programs — check BEFORE hypothesizing)
1. Stale binary/fixtures — rebuild after ANY edit; assert result COUNTS, not pass-lines.
2. Private build/artifact dirs per lane; gates run in the product configuration.
3. Pre-parent baseline before blaming your change or the environment: check out the parent
   state, rerun the SAME failing check there. "Environmental" is a banned first hypothesis
   under parallelism.
4. One hypothesis per new observation, ending in the cheapest killing experiment; if that
   experiment costs ~one rebuild/rerun, run it instead of writing more analysis.
5. <domain-specific traps worth their lines>

## Build / run blocks (idempotent; absolute paths)
```sh
<the exact commands for this project — build, test, run, convert>
```

## Wind-down ceremony (ONCE, at gate-pass or hard blocker)
1. <state file> evidence: result + gate EVIDENCE (numbers), ending "NEXT: <queue item>" or
   "RESUME <unit> at <sub-boundary>".
2. Dated handoff here: `<date>_<unit>_<lane>.md` — landed, evidence, surprises, next.
3. Durable learnings → the project's knowledge base (never into the board).
4. Commit every touched repo. Update THIS file's failure modes with what the unit taught.
````

## 02_ORCHESTRATION.md — the orchestrator playbook

```markdown
# PROGRAM <P> — ORCHESTRATION (v1)

> Environment: <Claude Code | OpenCode | both>. Orchestrator budget ≤ ~<10-15% of window>;
> lanes ≤ <per-lane budget>; ONE unit per lane per window, then STOP.

## 1. Roles + budgets
| Role | Env mechanism | Budget | Job |
|---|---|---|---|
| Orchestrator | this session | ≤ <N>k | read state, dispatch, serialize landings, own state files, relay to user |
| impl-a … | <Agent name= / Task> | ≤ <N>k · one unit | <territory> |

Rules: spawn each lane once per window, largest unit that fits; continue (SendMessage /
task_id) rather than respawn; dispatch by POINTER to §5 briefs — paste only budget, owned
files, private dirs, lock classes, landing order.

## 2. Lock classes
| Class | Protects | Holder |
|---|---|---|
| `<class>` | <the exclusive resource> | impl-a, every window |
Protocol: flock file under <locks dir>; holder line = lane pid utc class; break only dead-pid
locks and log it. Private-dir builds/tests take NO lock and never wait on one.

## 3. Window dispatch cards (paste-ready)
### Window 1
- **impl-a:** "Execute **U-A1** per 00_PROGRAM.md §5. Owns <paths>. Private dir <path>.
  Locks: <class>. Budget ~<N>k. Do NOT touch <peer paths>. Sub-boundary allowed: <name>."
- **impl-b:** …
- **impl-c:** …
### Window 2 … (one card block per window)

## 4. Orchestrator window checklist
1. Orient: repo status; state-file latest; newest handoff; §3 state.
2. Spawn per cards; relay one-line status per unit report.
3. Apply lanes' evidence to the state file yourself (lanes never write it — concurrent
   writes clobber); apply spec deltas yourself; sequence landings (one repo at a time).
4. Close: update §3 + §8 actual costs; dated orchestrator handoff; commit program docs;
   append retro lines below.

## 5. Retros (append-only)
- (none yet)
```

## Agent definition (Claude Code) — `.claude/agents/p<P>-implementer.md`

```markdown
---
name: p<P>-implementer
description: Executes Program <P> (<slug>) units whole, under an orchestrator. Spawn as a
  named lane (impl-a/b/c) and continue via SendMessage across windows; do not respawn per unit.
model: <choose for the work>
---
You are a Program <P> implementer lane. The orchestrator assigns ONE unit from
<program dir>/00_PROGRAM.md §5 plus a token budget. Your reports are its only eyes.

# Embedded contract (do NOT re-read the plan / resume prompt / full board)
<inline here: the §Hard rules + §Failure modes from 01_RESUME_PROMPT.md, the build block,
lane-ownership rules (touch only assigned files; state files are orchestrator's), the lock
protocol, and the report format below. The point: cold start ≤ ~2-5k input.>

# Report format (per unit)
UNIT <id> — LANDED | BLOCKED(<why + exact next command>) | STOPPED-AT-SUB-BOUNDARY(<name>)
gates: <numbers: counts, determinism y/n, planted-defect y/n>
commits: <repo shortsha pairs>
state-evidence: <2-3 paste-ready lines ending "NEXT: <queue item>">
spec-deltas: <proposals or "none">
cost: ~<N>k output tokens
retro: <one line for 01_RESUME_PROMPT, or "none">
```
For OpenCode: the same contract block lives in 02_ORCHESTRATION.md and is pasted once per
Task spawn (there is no agent registry); continue lanes via the prior `task_id`.

---

## Filled example (compressed, from a real program)

Context: build a two-sided parity oracle. User throughput: 3 lanes × ~110/95/80k, one
exclusive resource (a single-instance emulator — only one may ever run).

| | impl-a (emulator fork; lock `oracle`) | impl-b (runtime C++) | impl-c (python tools) |
|---|---|---|---|
| W1 | U-O1 fork bring-up + state dump | U-R1 runtime emitter + input identity | U-T1 trace builder + comparator core |
| W2 | U-A2 state-gate integration + oracle pixel dumps | U-R2 runtime pixel capture | U-T2 canonicalizer + pixel gates + bisect driver |
| W3 | U-I2 pixel gate + real-data first-red | U-P1 runner/CI/docs | U-V1 next-tier schema, runtime side |
| W4 (contingency) | spillover + backlog only | — | — |

Why it works: a shared spec doc (checkpoint keys, hash rules, file tags) was authored WITH the
program docs, so W1's three lanes had zero intra-window dependencies; only impl-a ever executes
the emulator (lock held every window), so b/c are pure headless/python and can't contend;
integration units (A2, I2) sit on the lock-holding lane in the window AFTER components land.

One filled brief:

> ### U-R1 — runtime emitter + input identity (W1, impl-b, heavy)
> 1. New `frontend/emit.{hpp,cpp}` emitting spec §§4–6 groups from the decode-event observer.
> 2. Thread frame-relative offsets into draw/copy events (reuse the existing offset machinery).
> 3. Converter: per-frame sha256 manifest; REJECT traces with the known un-restored region
>    (exit 2 + reason) rather than silently scoring them.
> 4. `--stop-after-draw N` prefix replay (suppress draws, keep state+copies).
> **Gate (Release, private build dir):** emitter tests green with COUNT asserted; two runs
> byte-identical.
> **Owns:** `frontend/` C++ + converter + replay tool. **Never touches:** core renderer,
> python tools. **Budget:** ~95k.
