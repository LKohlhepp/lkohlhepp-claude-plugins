---
name: implement
description: >-
  Plan-native implement → review → commit workflow for turning a plan into committed code.
  Use after a planning phase, when handed an impl-plan.md or any planning document, or when
  asked to implement something straight from chat ("implement this plan", "build this", "now
  do the implementation", "carry out the plan"). Runs waves/tracks of commits with one
  implementer plus a reviewer pair per track, reviews every commit (correctness + entropy),
  commits from the main session, and halts at every wave border for the user.
  Self-contained: needs no external instruction files.
---

# Implement

Turn an approved plan into reviewed, committed code. The main session orchestrates; subagents do
the heavy lifting. The run is **hands-off, not headless**: real divergences raise to the human,
and the papertrail is what makes the autonomy safe. This skill is self-contained — the implementer
doctrine and both reviewers are defined below, so do not look for `implementor.md`,
`architect.md`, or similar files.

## Roles (who does what)

- **Main session (you)** — orchestrate, decide on review findings, **do every git commit**, keep
  the plan doc and the log, present at borders. Verification runs in subagents; your role there
  is trust-but-verify — spot-check surprising claims, don't re-run the whole suite yourself.
  Never delegate a commit.
- **Implementer subagent, one per track** — edits files only, inside its allowlist (§ 3). Never
  commits.
- **Reviewer pair, one per track** — Reviewer A (correctness & safety) and Reviewer B (entropy),
  spawned in parallel. They report; you decide.

Heavy work stays in subagents and must remain so — spawn more when you need them. A high
orchestrator context water mark is accepted; there is no per-wave driver agent.

---

## Modes

### Plan mode
An impl-plan.md (or an equivalent commit-structured plan) governs the run. Waves, tracks,
phases, commits and tick boxes come from the document.

### Standalone mode
A chat-invoked run with no plan document is **a degenerate phase-mode plan**, not a different
workflow. At the entry gate you write the derived plan out — phases, each a sequential list of
commits — and run the same machinery against it, with gates at phase borders.

- Standalone is **ALWAYS attended**, so the termination block never applies here.
- There is no working folder: the derived plan and the change log go to a **temp location** (the
  session scratchpad if the harness provides one, otherwise a temp dir), named in the gate report.
- Neither temp file is committed, so exactly two rules lapse: ticks are progress tracking rather
  than staged into their commits, and resume ground truth is git history plus the temp log.
  Everything else runs unchanged.
- The implementation debrief is **invoked immediately** at run end, not left to the user (§ 7).

### Attended vs unattended
- **Attended** (default, and the only standalone mode): every wave/phase border halts.
  **Unattended**: borders become non-halting checkpoints, full border report written to the log.
- The declaration rule — what counts as one, its scope, how it ends — is the border-rule
  blockquote in § 6; follow it literally. A declaration made with the invocation still waits for
  the user's go at the unconditional entry gate, and governs only the borders after it.

---

## 1. Entry gate — ALWAYS halts

Before any implementation work, run the entry gate and **end your turn**. The halt is
unconditional — it happens even when everything looks clean, because the user wants to see the
baseline pass and give the go from there. Gate report content:

1. **Mode declaration** — plan mode or standalone; attended or (if declared) unattended and its
   scope; the derived phase plan if standalone.
2. **Stale-plan check** — which plan document you are using and why you believe it belongs to
   *this* work. An auto-detected impl-plan.md may be left over from a previous feature; check it
   against the branch, the linked issue if there is one, and the conversation. Say what you
   checked.
3. **Plan-shape restatement** — waves/tracks or phases, commit counts, the ordering semantics you
   read out of the document, the per-track path allowlists you derived, and the plan's full-suite
   command.
4. **Branch check** — the current branch, and whether it is the one this work belongs on.
5. **Baseline verification result** — actually run the plan's verification command once, now,
   before the first commit, while the user is present. A red at 3am must be attributable to a
   pre-existing failure rather than to the run. Report the real output, not a summary of intent.
6. **Hazards** — anything that will bite later: a dirty worktree, a pre-commit framework that
   stashes unstaged changes (see Mechanics), a plan whose ordering sentence is missing or
   contradictory (see below).

Detection ambiguity is resolved here, by the user, not by a heuristic. Then stop.

---

## 2. Reading the plan (format contract)

The plan's structure encodes dependency: commits in a track are sequential, tracks in a wave are
independent, a wave boundary is a barrier; phases run in order with strictly sequential commits.
Commits number **per track** (or per phase) from 1; global references are `wave.track.commit`
(`2.A.2`), `phase.commit` (`2.3`), or `wave.F.n` for wave-level fix commits. Every commit is a
tick box: `1. [ ] **Heading** — …`.

**The ordering sentence at the top of the plan is authoritative; the wave/track vocabulary is
only a hint.** If the ordering sentence is absent or contradicts the structure, **execute
strictly sequentially and say so** — in the entry-gate report, explicitly, as a deviation from
the plan's apparent shape.

**The plan names the full-suite command** — the wave-end gate — directly under the ordering
sentence, with per-track test-target hints where non-obvious. Per-commit checks are the
implementer's own derivation (§ 4). An older plan spelling out both levels stays valid: its
narrow set becomes the hint.

**Wave-level fix commits have a slot in the format** — use it, do not improvise: at wave level
after the wave's last track, under a `*Wave fixes*` heading created when the first one lands;
numbered per wave from 1, referenced `wave.F.n` (`2.F.1`); an ordinary tick box; a trailing
`*(added — log entry N)*`. A wave is done when its tracks **and** its wave fixes are. In phase
mode there is nothing to sit outside — append to the current phase, numbered next, marked the
same way.

---

## 3. Running a wave

### Track dispatch
Spawn one implementer and one reviewer pair per track in the wave. All tracks share ONE working
directory; the plan guarantees they are file-level disjoint, but you defend against violations
anyway.

Each implementer gets a **path allowlist at folder granularity**, deny-by-default: the folders
its plan touchpoints live in, and nothing else. An implementer that needs a path outside its
allowlist **stops and reports without editing**.

### The commit cycle (per commit, serialized inside a track)
Within a track, commits go in plan order, one at a time. Across tracks, commits land **as ready** —
no cross-track lockstep. The ticks record the actual landing order.

Before dispatching a commit: if its acceptance criteria are ambiguous or a prerequisite is
missing, **do not dispatch against a guess** — attended, raise it and wait; unattended, treat it
as a detected taint and apply the taint rules.

For each commit:

1. **Implement.** Dispatch the track's implementer with the commit's acceptance criteria, the
   allowlist, and the Implementer doctrine below (paste or summarize it — do not point it at
   this file). It reports **the list of files it edited**.
2. **Narrow verification.** The implementer runs its derived checks (§ 4) and reports command
   and result line. Red **on this track's own files** → straight back to the same implementer,
   until green. A "cannot make this green" report: judge it first — if the task still looks
   solvable, send the implementer back with direction; genuinely structural → run the taint
   test. Red elsewhere is not this track's red (see § 4).
3. **Review — one round, both reviewers in parallel.** A and B review the whole prospective
   commit, once. Not one dispatch per finding, no pipelining ahead into commit N+1.
4. **Apply findings.** Decide per Synthesis rules; route code changes back through the same
   implementer so one coherent author keeps context — trivial edits you may make directly. Apply
   everything you intend to apply **before** the commit, never "fix it after".
5. **Cross-check the file list** (see below).
6. **Tick, then commit** (see Commit mechanics).
7. **Log entry**, in its own doc-only commit.
8. **Narrate** one line.

Only then does commit N+1 begin.

### File-list cross-check
Compare the implementer's reported file list against the plan's touchpoints and against
`git status`. The two lists double-check each other, but `git status` cannot tell an unreported
new file from a sibling track's — so the **binding** check is the post-commit assertion below,
plus the end-of-run untracked sweep. On a mismatch:

- A file outside the track's plan touchpoints → **do not commit**; treat it as a divergence and
  run the taint test. Local and explainable → document it and proceed.
- A write into **another track's** files → **hard stop A** (see Taint).

### Commit mechanics
- **Write the tick as the action immediately before staging, and stage it into the commit it
  marks** — tick and commit are atomic, so a crash can never separate them. **Never begin another
  track's commit while a tick is pending**: `git add` is file-granular, so track A's pending tick
  would ride into track B's commit, and the post-commit assertion cannot catch it
  (`impl-plan.md` is in the intended set either way). Serializing tick-and-commit is the only
  defence.
- **Stage file-scoped**: `git add -- <exact paths>`. Never `git add -A` or `git add .` — a
  sibling track's live work is in the same worktree.
- **Message**: a concise subject, a terse body only if it earns its place, the repo's existing
  conventions (a trailer, if it uses one), and the log entry referenced **by stable index**, not
  by SHA — the entry is written after the commit and carries the SHA.
- **Post-commit assertion**: `git show --name-only HEAD` must equal the intended set. On any
  mismatch, or if the commit fails at all (bad pathspec, nothing staged, `index.lock`, a hook),
  **un-tick immediately** and handle the failure first.

### Narration
In plan mode, narrate **one line per landed commit or track event** — commit ref, subject, and
anything notable. With gates presenting only at borders, narration is the steering channel
between them.

---

## 4. Verification split

Verification is not track-isolated in one shared workdir — whole-package tools see the union of
all tracks. So:

- **Per commit — narrow, implementer-derived.** The implementer runs what proves this commit
  works: the tests covering its files, plus scoped lint/type checks as useful. Never the full
  suite mid-wave — it sees sibling tracks' half-finished work and produces false reds.
- **Wave end — the proper full suite**, on the quiet post-barrier workdir.
- **Attribute diagnostics by file.** Where a check cannot scope below a package both tracks share,
  it reports on a sibling's half-finished files too: only diagnostics on the track's own files
  block its commit. Note the rest and proceed — without this, two tracks deadlock each other.
- **Green is claimed only post-barrier.** Never call mid-wave work green. Border SHAs are recorded
  only after a clean post-barrier run.

**Strict wave barrier**: wave N+1 starts only when every track in wave N is committed and the
wave-end review has run. A track that fails mid-wave lets its siblings finish and surfaces at the
wave gate — or immediately, if its failure blocks a sibling.

**An incomplete wave never runs the wave-end suite.** If a track is parked, sibling tracks finish
their own work and the run **holds at the barrier** — no full-suite run, because there is no
informational red to interpret. Record the border SHA with the honest annotation
`not-green (track X parked)`. On a halt the worktree stays dirty for the user's investigation.

---

## 5. Review tiers

Three tiers. All of them use the Reviewer prompts below.

- **Per commit** (essential, never skipped, in either mode): the track's reviewer pair, one round,
  as described in the commit cycle — findings compound. The per-commit **entropy** reviewer may
  run on a cheaper model than correctness; correctness stays at the default tier.
- **Wave end**: cross-track seams, plus a full entropy pass at wave level, plus adjudication of
  out-of-track findings — root entropy is visible only at wave scale.
- **End of plan**: completeness — what was implemented versus what the plan says, where "the plan"
  includes changes appended mid-run.

The wave-end and end-of-plan reviews run on a **fresh reviewer pair with no track allegiance**, at
the default tier — cross-track seams are exactly what a track's own reviewers were scoped away
from.

A wave-end review may emit **wave-level fix commits**. They belong to no track, so no existing
implementer may touch them: spawn a wave-level implementer allowlisted to exactly that fix's
touchpoints and run it through the same commit cycle. Each gets its plan line under `*Wave fixes*`
and its own log entry.

---

## 6. Wave border

### The border rule — follow this literally

> Border rule (attended): at every wave border, halt — end your turn with the
> border report. Never cross a border on inference. A go-ahead in the user's
> CURRENT message continues exactly one wave; past approvals, standing habits,
> or phrases from earlier turns ('commit and continue', 'looks good' some waves
> ago) authorize nothing now.
> Unattended exception: only an explicit, named mode declaration by the user
> ('run unattended', 'run overnight', optionally scoped: 'unattended until wave
> N') converts later borders into non-halting checkpoints (full border report
> written to the log, run continues). The declaration must name the mode —
> enthusiasm, urgency, or repeated approvals never imply it. It ends at its
> declared scope, at run end, or the moment the user sends any new message;
> after that, borders halt again.

The user's explicit override remains fully available — the prohibition targets inference, not
obedience.

### Border report
Checkable evidence, not assurances, and it follows the user's completion-message rules.
**Only what the per-commit entries do not already carry** — the report is the wave's new
information, never a digest of its own entries:

- the **wave-end review's findings**, short form: one line each plus the verdict — cross-track
  seams, the wave-level entropy pass, and any wave-level fix commits it emitted. This is the
  wave-scope review no per-commit entry could hold, and it is why the report exists;
- the wave-end verification **output**;
- provisional/tainted work, and any open decision the user must rule on to proceed;
- the border SHA (or the not-green annotation if a track is parked).

Everything else is a pointer, never a restatement: the wave's commit range instead of a tick
table, a count of findings applied and denied instead of their reasons, one line on whether the
committed files matched the plan's touchpoints — naming only the exceptions. **If a bullet would
repeat an entry, cut it to a count or a ref.**

Prose and evidence, **never diffs** — the user reads those in the IDE.

**The border report goes into impl-log.md in every mode**, and in attended mode is presented as
well.

---

## 7. End of run

Do the end-of-run untracked-files sweep first (see Mechanics). Then run the end-of-plan review
and present the final report: what landed, what diverged, what is provisional, what was deferred,
and every open decision.

The run is not finished when the code is. What closes it is the walk-through — user and model
going through what actually happened, divergence by divergence, until the user says whether they
agree. That is the implementation debrief's job (`impl-debrief`): in **plan mode** it is
user-invoked, so point at it in the final report with the inputs it needs (ticked plan,
impl-log.md, border reports, branch diff); in **standalone mode** invoke it immediately, as part
of ending the run. If that skill does not exist yet, say so plainly and hand over the same inputs
in the final report instead.

---

## Divergence: taint

**Taint** is the state: work whose plan-story broke. Downstream work is tainted / provisional.

**Taint detected** is the event. The test:

> *Does the plan's stated intent for this commit and its neighbors still hold?*

Apply the test **to the divergence, never to the fix you picked**. Testing your own chosen fix
lets structural problems pass by picking the local workaround.

Practical reading: a divergence whose fix changes other commits, other tracks, or the plan's
structure → raise. Strictly local to the current commit → patch and document.

**Raise taint**, per mode:
- **Attended**: halt and put it to the user **before tainted work exists**.
- **Unattended**: log the assumption you made *and the alternatives you rejected*, mark downstream
  work provisional, and apply the rules below. The default is proceed-with-taint — one branch,
  pre-MR, border SHAs, full papertrail, and a debrief obligated to cost reverts honestly; the
  threshold and the two hard stops are its only exceptions.

**Halt threshold (unattended)**: halt at the barrier if the divergence's consequences reach
**beyond patch-scale** — when honoring it would change what other commits or tracks are supposed
to do to a degree that makes continuing likely rework. If **two or fewer commits remain, finish
regardless** — the tail is cheaper to finish and debrief than to stop.

**Hard stop A — cross-track write.** An implementer wrote into another track's files: freeze that
track immediately, leave its work uncommitted, let sibling tracks finish their own tracks, hold
at the barrier.

**Hard stop B — can't go green.** Fires only when all three hold: the wave-end suite is red, AND
the red is attributable to this wave's own diff, AND one repair round by the responsible
implementer has failed. Then hold at the barrier with state saved.

**Environment failure is not a taint.** A red attributable to infrastructure rather than to the
wave's diff stops the run as *"environment failure, state saved"*: a different report, no taint
recorded, no false blame on the work. Make the attribution explicitly — was this path passing at
the entry-gate baseline, does the failure name infrastructure rather than the diff, does it
reproduce on untouched code?

**Representation.** No third checkbox state: tainted work IS done, so `[x]` is correct.
Provisional-ness is an annotation written atomically with the tick —
`[x] … *(provisional — log entry N)*` — with the divergence that tainted it recorded in the log.

---

## Termination block — UNATTENDED MODE ONLY

**Active in unattended mode only.** In attended mode none of this applies: you present and wait,
exactly as the attended path always has.

Every termination decision resolves to one verdict:

| Verdict | Meaning |
| --- | --- |
| **done** | The unit is finished and may be built upon. |
| **resume** | Another round is warranted and allowed by the round limits. |
| **escalate** | Park the track / hold at the barrier, `escalate_reason` logged. |

**Done-gate preconditions** — all three, or the verdict is not *done*:
1. the implementer's derived checks pass,
2. correctness findings resolved,
3. the entropy round spent.

**Round limits** (revise these if stops turn out to happen too frequently):
- **Entropy findings**: ONE round, then proceed. Leftovers are deferred to the log.
- **Correctness findings**: one full round, then a SECOND round that is only a single Reviewer-A
  verification pass. Still failing → **escalate**.

---

## Papertrail: two files

Progress lives in **impl-plan.md** (ticks and provisional annotations). What departed from the
plan lives in the sibling **impl-log.md**. Silent doc↔code divergence is the failure mode this
defends against. You are the **sole writer of both**; both are on the implementer no-touch list.

### impl-log.md
**Why it exists**: to catch two things and nothing else — planned work that got dropped without
anyone deciding to drop it, and the plan changing without authorization. Every rule below follows
from that. An entry is evidence about conformance to the plan, not a record of the work.

- **One entry per event, each committed in its own doc-only commit, immediately. Never batched to
  the border** — batched entries die in rollbacks and block reverts.
- **Entry format**: anchored to `wave.track.commit` + the code SHA, wave-sectioned, with a
  keep-or-revert index at the top. Written after the code commit, referencing its SHA. **The
  index is load-bearing — `impl-debrief` reads its SHAs, and selective revert needs it — so it
  stays complete however short the rest of the log gets.**
- **Length follows divergence.** A commit that landed as its plan line described is **one line**:
  anchor, SHA, "as planned". You are not summarizing the work — the diff is the work. Only a
  divergence earns prose, and only the divergence itself: what the plan said, what happened
  instead, the assumption you made, the alternatives you rejected, and whether it was authorized.
  Three to six lines for most. Needing more than ~15 means you are recording a second divergence,
  or recording something that is not one — split it or cut it. **An entry that is long because the
  work was interesting is long for the wrong reason.**
- **What goes in**: divergences, as above; work the plan called for that did not land, with the
  reason; changes appended to the plan; escalations with their `escalate_reason`; denied findings,
  one line each; every out-of-track finding awaiting wave-end adjudication; every border report.
  All of it in every mode.
- **What stays out**: anything that is not evidence about the plan. Findings that were applied
  (the diff is the record — a count is enough), work that turned out fine, superseded drafts,
  reviewer disagreements that resolved, corrections to your own briefs, recovered tooling errors,
  per-commit file lists (`git show --name-only`), test output.
- The log survives the merge to main — it rides in the MR diff, and reviewers may re-litigate a
  logged denial, which is arguably a feature.

### Appended changes
Every change to the work that the plan did not call for is documented **as a change**, appended
on purpose and kept **separate from the original plan**: a log entry always, plus — when it adds
a commit — a plan line carrying the format's `*(added — log entry N)*` marker. An *alteration* to
an existing commit's intent has no marker in the format: it lives in the log, referenced from the
border report. Implementers report their divergences; you record them.

### Resume (doc-based)
On resume, **the committed plan is ground truth**: diff `git show HEAD:impl-plan.md` against the
worktree copy. The judgment-style "last ticked vs next unticked" comparison is the fallback, not
the primary check. If the plan is associated with an issue, check the branch too.

Two reachable divergence states:
- **Tick without commit** — the tick sits in the worktree with no commit behind it. That names
  the failed commit exactly.
- **Uncommitted work with no tick** — an interrupted implementer.

Real divergence raises to the human. Hands-off, not headless.

---

## Reuse and context budget

Reuse a track's implementer across its review→fix cycles so it keeps its working context, and
reuse its reviewers across successive commits — the accumulated repo-layout context makes
follow-up reviews faster and more consistent. Reviewers **MUST re-read the actual source each
round** (`git diff` + the touched files), never the implementer's narration: reuse carries repo
layout only, never a substitute for reading the real diff.

Spawn a **fresh** replacement for any subagent that shows confusion, repetition, or hallucination,
or that exceeds its size ceiling: **~250k tokens** for an implementer, **~120k for a reviewer**.
Reviewers get the tighter ceiling because they re-read the diff every round anyway — their
accumulated context is spent history that every later call pays for again. The budget is tracked
**per agent**; reviewers reach their ceiling well before implementers do, and letting them run past
it silently produces degraded reports. Warn the user when you swap for incoherence
(contradicting prior decisions, misremembering the public API of files it touched earlier, asking
"what was commit 2.A.1 again?", structurally degraded reports), not for size.

---

## Implementer doctrine

Implement exactly what the acceptance criteria require — nothing speculative.

- **Reuse first, consolidate on contact.** Search before writing: existing logic is used, not
  rewritten; existing copies are consolidated and built upon, never joined by another — that is
  in-scope, not scope creep. Copies outside your allowlist: build yours, report the duplication.
  Entropy you don't directly touch: report, don't fix. Never introduce sharing across parts the
  repo keeps standalone (service, app, package — whatever its unit is).
- **No premature structure.** No interface, wrapper, or base class unless two or more consumers
  exist *right now*. No parameters, fields, or code paths no criterion requires.
- **No dead defense.** No nil checks, try/catch, or validation for states the type system or the
  caller contract already makes impossible.
- **Test what matters.** Concentrate tests where a break would actually hurt — core logic, tricky
  invariants, boundary conditions, and the error paths real callers hit — and pin those well.
  Judgment about *what* to cover beats volume: a few tests on load-bearing behavior beat broad
  shallow coverage. Skip the trivial — no test-per-line, no enumerating permutations, no
  re-asserting what the type system already guarantees. Never pad to hit a number: test LOC
  approaching the code under test (~1×) is already a high upper bound, and landing well under it
  with the risky paths covered is the goal, not a shortfall.
- **Comments state and reasons, never history.** A comment must be self-contained: never
  reference the plan, the commit sequence, ticket/capability IDs, "later commit", "as decided
  above", etc. Rephrase any such note as a plain design statement about the code.
- **No backwards-compat work** unless a criterion explicitly demands it. Don't preserve old
  interfaces or add migration paths on spec.
- **Stay inside the allowlist.** You are given a folder-granularity path allowlist,
  deny-by-default. Needing a path outside it — including "just one line" in a shared registry,
  barrel file, or config — means **stop and report without editing**. Another track's files are
  live in this same working directory.
- **Never claim green without a passing run.** Report the exact command and its result line —
  pass/fail counts, failing test names on red. No paraphrased results, no full-output dumps; the
  orchestrator asks if it needs more.
- **A stubborn red means keep working.** If the fix is small and inside your allowlist — even
  when the cause isn't your own change — make it and note it. Stop and report only when the fix
  needs a path outside the allowlist, contradicts the acceptance criteria, or outgrows a small
  explainable change: that is a divergence report, not a failure.
- **Report every file you edited**, per commit, as a list. It is cross-checked against the plan.
- **No-touch list**: `impl-plan.md`, `impl-log.md`, and any plan/spec artifact. Do not commit,
  do not stage, do not stash.
- **Report your own divergences** from the plan's stated intent, rather than smoothing them over.
- Run git read commands with `--no-optional-locks`.

---

## Reviewer prompts

Give both reviewers the diff, the acceptance criteria for the commit (or wave), the track's file
ownership, and freedom to explore the wider repo. **Neither reviewer flags naming, comment style,
or other cosmetic issues** — out of scope, needless churn. Each reviewer reports "none found" per
category rather than skipping silently.

Three standing rules for both, in every mode:

- **Exploration scales with the change.** Read the diff and every file it touches in full — that is
  the review. Past that, let the size of the change set the size of the search: a two-file commit
  does not earn a repo-wide sweep, one that moves a contract does. Follow the diff outward — callers
  of what changed, siblings on the path it touches, the existing home of logic it may have
  re-invented — not casting around for what might turn up. Prefer one wide search to a string of
  narrow probes, and read a file once rather than in slices. Thoroughness is depth on what this
  change touches, not breadth across what it does not.
- **When in doubt, report it.** The orchestrator decides what to dismiss; a reviewer withholding a
  borderline finding removes the decision from where it belongs.
- **Findings outside your track's files are reported, not fixed.** They go to the log and are
  adjudicated by the wave-end review, which may emit wave-level fix commits. (In phase/standalone
  mode this is vacuous — there is no track ownership.)

### Reviewer A — Correctness & safety
At a wave end, runs the full suite independently and reports the output. Per commit, re-runs the
implementer's reported test command by sampled judgment — always on a track's last commit, always
when the diff and the claimed result don't square; otherwise trust the reported result. Then
checks every category. The bar is: *"would a senior engineer send this back, or just merge it?"*
1. **Acceptance criteria** — does the code satisfy every stated requirement?
2. **Plan conformance** — does this commit do what the plan says this commit does? Silent drift
   inside a track's own files is caught by nothing else.
3. **Test quality** — are assertions meaningful, not just "it runs without crashing"?
4. **Contract correctness** — do API shapes, types, and return values match what callers depend on?
5. **Bugs** — off-by-one, missing error handling at system boundaries, wrong status codes.
6. **Repo coherence** — integration points, shared utilities, or conventions missed or
   contradicted. Look beyond the diff.

### Reviewer B — Entropy reduction
Find code that should not ship. Report everything — the orchestrator decides what to dismiss, not
the reviewer. Structure the report in two sections:
- **Introduced / worsened by this change** — these block shipping.
- **Pre-existing (informational)** — list only severe items (dead modules, duplicated business
  logic); skip pre-existing cosmetic entropy.

Categories: dead code; redundant wrappers (no logic over what they delegate to); duplication of
logic that already exists **in this service/boundary**; unnecessary abstraction (one consumer);
overly defensive code unreachable given real call sites; configuration that duplicates defaults.
Flag concretely: file, line range, and what the simpler version looks like.

**Then, additionally, scan the tests — a separate pass, not a substitute for the source review
above.** Go through every test file the change touches and judge whether each test earns its
place. Flag: redundant permutations of the same code path (collapse to one parametrized case);
tests that exercise the framework or type system rather than this code's behavior; assertions
that merely restate the implementation; a suite disproportionate to the code it covers. For each,
name the file, line range, and exactly what to cut or merge — same concreteness bar as above.

At a **wave end**, run the same review across the wave's whole diff, and additionally look for
what only becomes visible at that scale: an abstraction that was licensed by one consumer at
commit 1 and is dead or duplicated by commit 5; parallel structures grown in sibling tracks; seams
between tracks that now overlap.

---

## Synthesis rules

Read both reports fully before acting.

- **Correctness findings are presumed actionable.** Fix them. Downgrade to informational only if:
  factually wrong (cite file/line), cross-boundary (different service with its own tests), or
  pre-existing and untouched by this change. State the reason inline.
- **Entropy findings** clear the same bar — *"would a senior engineer send this back, or merge
  it?"* Fix what clears it; let the rest go. Dismiss a finding only with evidence: factually
  wrong; breaks an acceptance criterion (name it); the fix is genuinely more complex (give the
  reachable edge case it fails — hypothetical future callers don't count); cross-boundary;
  pre-existing and untouched; or below the bar (a senior would merge it). Do not use time or
  context pressure as a reason to skip something a senior would block on.
- **Conflicts.** When A says code is load-bearing and B says remove it, correctness wins unless a
  test or contract already covers A's concern (cite it). When A and B flag the same area, treat it
  as a design problem and raise it explicitly.
- **Every dismissed finding is logged as deferred** — finding, reason, commit — in impl-log.md, in
  every mode. A denial that leaves no trace is indistinguishable from a finding that was never
  read.

---

## Mechanics

- **Subagents run git with `--no-optional-locks`**; the orchestrator retries **once** on an
  `index.lock` collision before treating it as a commit failure (and un-ticking).
- **End-of-run untracked sweep**: `git status --porcelain --untracked-files=all` before the final
  report, to catch new files nobody reported. Anything unexpected is a divergence, not a cleanup.
- **Pre-commit hook caution**: a pre-commit framework that stashes unstaged changes will yank
  sibling tracks' live work out from under them mid-run. Check for one at the entry gate, and if
  the repo has one, name it in the gate report along with how your commit path accounts for it —
  the user is standing at that halt anyway.
- **Never plain `git stash`** — it sweeps the dirty plan doc and log along with the code. If you
  ever must stash, stash by pathspec.
- **Revert, never reset**, when undoing a landed commit: a reset to a border SHA destroys the log
  entries that explain what went wrong.
