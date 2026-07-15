# Phase-1 plan template

The section list below is the proven shape. Sections marked *(domain-optional)* appear only
when the problem needs them — a data-pipeline plan needs schema strategy; a refactor plan
doesn't. Never pad a plan with empty sections to match the template.

Write for two readers at once: the user deciding whether to bet on this, and the cold-started
implementer who will execute it without you. Every section should survive the question
*"could someone act on this without asking me anything?"*

---

## §0 Verdict on the request itself

If the user asked (or implied) "is this necessary? already solved? is there a simpler way?" —
answer it FIRST, in three bullets or fewer:

- **Necessity**: what existing tools/prior art you checked and why they don't (or do!) cover it.
- **Conception**: where the user's framing holds and where it breaks. Name the load-bearing
  correction explicitly (e.g., "bit-exactness is achievable through tier X but not tier Y, so
  the goal must be restated as …").
- **The simpler path**: the discoveries that shrink the work. The best plans find the problem
  is half-built and list exactly which existing pieces carry which load.

Honesty here is the product. "Yes, there's a far simpler way" is a *successful* outcome.

## §1 Mission, non-goals, terminology

- **Mission**: one paragraph; what exists when this is done, and for whom.
- **Non-goals**: explicit, each with the reason it's excluded (scope discipline, honesty
  boundary, or ranked-later). Ranked-later items also appear in the backlog (§9).
- **Terminology**: define every term the plan uses precisely enough to detect misuse —
  especially terms with near-miss confusions (the kind that caused past bugs). These
  definitions become binding for all artifacts downstream.

## §2 Chosen architecture and why

- The decision, stated in one sentence, then the data-flow (a compact ASCII diagram earns its
  space when there are ≥3 components).
- **Rejected alternatives**: one paragraph each — what it was, and the *evidence* that killed
  it (not taste). Include "extend the existing thing" and "do nothing" when they were live
  options.
- Key technology/dependency choices with the same evidence standard.

## §3 Source-grounded integration map

The section that separates a plan from an essay. Two tables:

1. **Seams/hooks**: | hook | exact location (file:line, verified by reading) | what it does |.
   If you haven't read it, it doesn't go in this table.
2. **Unknowns**: | question | cheapest killing experiment | cost |. Every entry becomes a
   milestone-0 checklist item. An unknown you can resolve for one build+run is never allowed
   to become a design assumption.

Also record inherited constraints you're deliberately NOT fighting (quirks of upstream systems
both sides must live with), so nobody "fixes" them later by accident.

## §4 Contracts / schemas / interfaces *(domain-optional)*

When independent components (or independent lanes, later) must agree on a format: define the
normal form here — versioned, with exact field/serialization rules and a completeness boundary
(what's covered, what's explicitly excluded and why). Prefer one small verifiable normal form
over a sprawling mirror of someone's internals. If completeness can be *checked* (against a
registry/enum/upstream list), specify that check as a test, not a promise.

## §5 Verification strategy *(shape varies by domain)*

How correctness will be *demonstrated*, not argued:

- Ordered gates / kill chain when there are layered failure classes — first red stops, later
  tiers report NOT_EVALUATED (a pass must be impossible when a precondition is unknown).
- Exactness boundaries: which comparisons are bit-exact, which are tolerance-gated, who
  declares the tolerance and where it's recorded.
- **Planted-defect standard** for any tool/oracle: the tool is proven by deliberately breaking
  each thing it claims to catch and watching it catch it.
- Localization: how a failure routes to a cause (bisection, first-divergence keys) without
  drowning in artifacts.

## §6 Milestones

Each: **inputs → outputs → pass criteria → resource bounds.** Pass criteria must be objective
(counts asserted, byte-identical reruns, planted defect caught, experiment answered in
writing). Keep "tool/feature complete" distinct from "corpus/rollout green" when both exist —
they close at different times. M0 is usually "resolve the §3 unknowns".

## §7 Fixtures, CI, data hygiene *(domain-optional)*

Synthetic/committed test data vs. private/local data; what may never enter public artifacts;
how reports redact it.

## §8 Risks, anti-patterns, stop conditions

| risk | trigger/mitigation | — plus a short list of banned moves (the anti-patterns this
specific plan is vulnerable to), and explicit stop conditions: the observations that mean
"stop and rethink" rather than "push harder". Include the honesty boundary of any claims
(what this work will NOT prove even when green).

## §9 File ownership + backlog

- Which paths each part of the work owns (this becomes the Phase-2 lane map).
- The ranked-later backlog: named items with one line each on why they're later, not never.

---

## Style standard (applies to every section)

- Cite or hedge: `path/file.ext:line` for every load-bearing code claim.
- Decisive verbs; no "we could consider".
- No time/difficulty framing ("this is complex", "quick win") — state what and why.
- Numbers over adjectives: budgets, counts, dimensions, tolerances.
- If the plan reverses a prior decision or supersedes an existing tool, say so by name and
  leave a pointer at the superseded thing's home (people find docs from where they used to
  look).
