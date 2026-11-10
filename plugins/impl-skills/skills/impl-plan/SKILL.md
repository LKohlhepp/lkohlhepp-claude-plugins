---
name: impl-plan
description: >-
  Turn a ratified planning document (plan.md from /brainstorm, or any concept
  document the user points to) into a code-grounded, commit-structured
  implementation plan (impl-plan.md) ready to hand to /implement. Use on
  explicit invocation ("impl-plan", "make the implementation plan", "turn
  plan.md into an implementation plan") or when a brainstorm has just been
  ratified and the user asks for the implementation planning step. Hands-off:
  reads the concept, researches the real codebase, drafts waves/tracks of
  commits, self-checks, presents. Not for high-level design (that is
  /brainstorm) and not for executing the plan (that is /implement).
---

# Implementation plan

Own the translation between concept and code. A brainstorm ends with the
*what*, the *why*, and the *how* fixed to a certain extent; /implement begins
by handing acceptance criteria to an implementer that makes no design
decisions. This skill closes that gap: read the concept document, specify the
how further against the **actual source**, create a sensible order, and finish
with a plan the user hands to /implement.

The run is **hands-off, not headless**: the plan governs, interaction stays
minimal — but the user is present throughout. When something is genuinely
unclear, ask; the two-moment cadence below is the default rhythm, not a gag
order.

This is a planning phase: never edit product code, never commit, never modify
the concept document.

## Input and output

- **Input:** `plan.md` produced by /brainstorm, or whatever concept document
  the user points to. Read it in full before anything else.
- **Output:** `impl-plan.md`, written beside the input document.

## The plan document: one text, two readers

Everything about the format serves a double audience:

- **The human** reads it top-to-bottom in minutes and sees the shape and order
  of the work at a glance.
- **/implement** executes any single commit without making design decisions,
  and dispatches independent tracks concurrently. A wave/track plan must stay
  valid when simply executed top-to-bottom (wave barriers become ordering,
  disjoint tracks run back-to-back). Never emit a plan that needs parallel
  execution to be correct.

**Structure encodes dependency.** Commits in the same track are sequential;
tracks in the same wave are independent; a wave boundary is a barrier. No
"depends on:" fields — position is the dependency statement.

### Structure and modes

The **commit is the atom**. Above it, one grammar with two vocabularies:

- **Waves (default):** `Wave N` → `Track A, B, …` → commits. Wave shape —
  how many waves, how many tracks, how commits are sized — is this skill's
  judgment. A wave may have a single track. **Never parallelize where it is
  not genuinely possible:** all tracks execute in ONE shared working
  directory, so tracks sharing a wave must be disjoint at the file level — no
  file touched by two tracks, ever. A file conflict between parallel
  implementers goes horribly wrong; when in doubt, same track or a later
  wave. Disjoint files are not enough: a track must not need a symbol a
  sibling track creates. Commits land across tracks as soon as they are ready,
  so the consumer can land first and go red — put producer and consumer in the
  same track.
- **Phases (user override):** when the user's invocation asks for
  non-parallel ("phases", "sequential", "no parallelism" — natural phrasing,
  no flag syntax), write `Phase 1..N`, each a sequential commit list. The
  concept document itself never carries execution directives.

If no wave ends up with two tracks, write the plan in phase vocabulary instead.

The vocabulary signals the mode at a glance, but it does not carry the
ordering semantics by itself — the ordering sentence does. **Every plan opens
with it, verbatim, above the verification line.** Phase mode: "Phases run in
order; commits within a phase are strictly sequential — a commit starts only
after the previous one has landed." Wave mode: "Waves run in order; tracks
within a wave are independent and may run in parallel; commits within a track
are strictly sequential." This line is load-bearing, not polish: executor
tests showed vocabulary alone can be misread (a bare "Phase" list invited
parallelizing dependent commits; the explicit sentence eliminated it). Don't
mix vocabularies in one plan.

**Numbering:** commits number per-track (or per-phase) starting at 1. Global
references are `wave.track.commit` (`2.A.2`) or `phase.commit` (`2.3`) — plus
`wave.F.n` for wave-level fix commits (see below). This keeps edits local —
inserting a commit never cascades renumbering through the document.

### Verification

All tracks share one working directory, so only the wave-end full suite — run
on the quiet workdir once every track in the wave has committed — actually
claims green. The plan states **one verification line**, directly under the
ordering sentence: the repo's proper full suite as a real command. Add
per-track test-target hints only where the right target is non-obvious.
Per-commit checking (the tests covering a commit's files, scoped checks) is
the implementer's own derivation — /implement owns it; the plan does not spell
it out. In phase mode the phase boundary is the full-suite point.

### Per-commit content — lean

Human legibility is a hard constraint: lean prose, no fielded boilerplate.
Each commit gets a commit-subject-style heading plus a short prose paragraph —
what it does and where, key files named inline. Add a "done when" sentence
only where completion is non-obvious. Verification lives at plan level; a
commit mentions it only when it needs a check beyond the obvious. If the plan
quotes size estimates, follow the user's LOC-estimation protocol.

**The plan is tickable.** Every commit is a task-list item — number kept, box
added: `1. [ ] **Heading** — …`, ticked to `[x]` as the commit lands — so
impl-plan.md doubles as the progress tracker across /implement passes. Boxes
exist only at commit level; a wave, track, or phase is done when its commits
are. /implement may tick a commit whose plan-story broke, annotated:
`[x] **Heading** — … *(provisional — log entry N)*`. The work *is* done — only
its standing is in doubt — so that is legal plan format, not corruption; the
story lives in impl-log.md. impl-plan never writes it.

Sketch of the shape:

> Waves run in order; tracks within a wave are independent and may run in
> parallel; commits within a track are strictly sequential.
>
> **Verify:** `uv run pytest` (full suite) + `ty check src`, at each wave end
> on a quiet working directory. (Track B tests live in `tests/cli`.)
>
> **Wave 1**
> *Track A*
> 1. [x] **Add scan-scope column + migration** — new migration in
>    `migrations/`, field on `ScanState` in `loop/state.py`.
> 2. [ ] **Gate resume on scope match** — extend `resume_check()` in
>    `loop/resume.py`; done when a drifted scope pauses instead of resuming.
>
> *Track B* (touches only the CLI)
> 1. [ ] **Surface scope in `loop status` output** — `cli/status.py`.
>
> **Wave 2**
> *Track A*
> 1. [ ] **Wire scope gating and the status line into the pause banner** —
>    `loop/banner.py`, joining the wave-1 pieces.

### Wave-level fix commits

/implement's wave-end review turns up work that belongs to no single track —
cross-track seams, findings in files no track owns. Those commits are unplanned
by construction; the plan's job is only to hold a legal slot for them, so
/implement inserts rather than improvises:

- **Placement:** at wave level, after the wave's last track, under a
  `*Wave fixes*` heading. The heading is created when the first one lands and is
  absent otherwise — impl-plan never writes it up front.
- **Numbering and reference:** numbered per wave from 1, referenced
  `wave.F.n` (`2.F.1`). The `F` slot sits beside the track letters, so inserting
  a fix commit renumbers nothing: sibling refs stay `2.A.3`.
- **Tick box:** identical to any other commit — `1. [ ] **Heading** — …`,
  ticked when it lands. A wave is done when its tracks *and* its wave fixes are.
- **Marked as added:** every plan line written during the run — a wave fix, or
  a commit appended to a track — carries a trailing `*(added — log entry N)*`,
  so a reader sees instantly what the plan did not originally say. The reasoning
  stays in impl-log.md; the plan line stays one line.

In phase mode there is no track ownership and nothing to sit outside: such a
commit is appended to the current phase's list, numbered next, marked as added.

Example of an inserted line, in place under Wave 1:

> *Wave fixes*
> 1. [x] **Reconcile scope naming between state and CLI** — `loop/state.py`,
>    `cli/status.py`. *(added — log entry 7)*

### Calls & deviations

A short section in the plan listing every judgment call made below the
blocking bar — derivations that deviate from a literal reading of the concept,
small decisions the concept left open. This is how the run stays quiet without
hiding anything: the user reviews every exercised judgment at handoff.

## Run structure

1. **Read the concept document in full.**
2. **Grounding research.** Find the real touchpoints in the source. Scale by
   judgment: for a large surface, fan out Explore subagents (set
   `model: "opus"` explicitly); for a handful of files, read them directly.
   The plan must be written against actual code — the failure mode this
   defends against is a plan written against imagined code.
3. **Interruption moment 1 — end of research.** Raise ALL blockers found so
   far in one batch and halt for answers. Never raise blockers one-at-a-time
   during research; a later finding often reshapes an earlier question.
4. **Draft** `impl-plan.md` per the format above.
5. **Self-check.** Dispatch one fresh-eyes subagent (explicit `model: "opus"`)
   that answers three questions about the draft:
   1. **Sound** — does the structure hold? No commit needs an artifact from
      a later wave/phase; no two tracks in one wave touch the same file.
   2. **Executable** — could a capable implementation agent (opus-level)
      execute each commit as written, with no design decision left to
      make? Verify the few riskiest commits against the actual source;
      elsewhere spot-check only where a commit looks doubtful — the
      grounding research already verified the touchpoints.
   3. **Complete** — does every decision and constraint in the concept
      document land in some commit?
   It reads the plan and the concept in full, the source selectively.
   Report findings only — no restating what's fine. Fix real findings
   before presenting; skip cosmetic ones.
6. **Interruption moment 2 — present and halt.** Present the plan; blockers
   discovered after moment 1 (mid-draft or by the self-check) ride along as
   explicit open questions rather than triggering an extra interruption. The
   user reviews and hands the document to /implement themselves.

## When to complain

The blocking test: **would different reasonable answers produce structurally
different plans?**

- **Blockers** — raise them (at the two moments above, unbounded batch; every
  blocker is raised, none dropped): the concept contradicts code reality (the
  user picks which is authoritative); the concept's decisions conflict with
  each other; a genuine design decision the concept doesn't settle.
- **Below the bar** — never interrupt: make the derivation call and log it in
  Calls & deviations.
- **Genuinely unclear and blocking progress right now** — ask; hands-off is
  not headless.
