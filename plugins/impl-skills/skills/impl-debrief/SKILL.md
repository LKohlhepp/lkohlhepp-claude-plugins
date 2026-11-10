---
name: impl-debrief
description: >-
  Debrief a finished /implement run: ratify the changes the run made to the plan,
  and find the work the plan called for that never landed. Use on explicit
  invocation ("impl-debrief", "debrief the run", "debrief the implementation",
  "review what the run did against the plan"), or at the end of a standalone
  /implement run, which invokes it directly. NOT a code review — no bug hunting,
  no style checks, no entropy pass, no sweep of the diff; the run's own reviewers
  own that, and /code-review is the skill for it. This reviews the changes to the
  plan, not the code.
---

# Implementation debrief

Close a run out. Check what was done, what should have been done, and what
changed on purpose. Two jobs:

1. **Ratify the changes to the plan** — the divergences the run documented, each
   needing the user's approval.
2. **Find the undocumented ones** — above all *work not done*: in the plan,
   absent from the diff, never reported.

The session that did the work cannot do this honestly on its own: it is
sunk-cost-loaded and blind to its own omissions. The mechanisms below exist to
buy that honesty back.

**Not a code review.** No bug hunt, no style, no entropy — the run's per-commit
and wave-end reviewers own those. Reading code is allowed and expected where it
is needed to adjudicate a change; there is no sweep of the diff for its own sake.

## The five steps

1. Find the artifacts from the plan; if the plan is not finished, say so and stop.
2. Dispatch one subagent with the fixed brief; it writes the findings document.
3. Present one message: the completeness verdict, the changes to approve, and
   what is missing.
4. Walk it — answer correctness questions by reading the code, record each
   disposition into the document as it is taken.
5. Write the outcomes: dispositions recorded, everything missing appended to the
   plan as work, design changes to plan.md, one doc-only commit.

One subagent, one message, one commit. Nothing beyond this shape.

## 1. Scope and entry

**The input set:** the cumulative branch diff and git history, `impl-plan.md`
with its ticks and provisional annotations, `impl-log.md` (which is where the
border reports persist), `plan.md` in plan mode, and `impl-debrief.md` itself on
a re-run. **The diff is ground truth** — plan, log and reports are checked
against it, never taken as given.

**Finished plans only.** Finished means every commit box is either ticked, or
unticked with a log entry explaining the drop. An unticked box with no log entry
means the run is genuinely not over: the verdict is "the plan is not finished
yet", and the skill stops there — no mid-wave conduct, no dirty-worktree
handling. A logged drop is a *change* for the walk, not missing work.

**The false-tick rule.** The presence check therefore runs on *ticked* items: a
ticked box whose work is absent from the diff is the flagship finding — claimed
but never done. Dishonest tick → finding; honest gap → back to /implement.

**Once started, a debrief finishes.** Work this skill appends to the plan never
interrupts the walk in progress. Appended lines are not exempt from the finished
test, so a later invocation correctly answers "not finished yet" until they land.

**Standalone runs are covered by their derived plan.** /implement's standalone
mode writes a derived plan (phases, each a sequential commit list) and its change
log to a temp location; traverse that derived plan as the plan. Everything stays
scratch: nothing is written into the repo and nothing is committed.

**Invocation.** In plan mode the user invokes this skill. At the end of a
standalone /implement run it is invoked immediately as part of ending the run —
see the self-disclosure line under Honesty. Open taints are fixed before the
debrief; this skill never meets one.

## 2. The analysis — always a subagent

The analysis is **always** a subagent, on both invocation paths, with no size
test. Uniformity is the point: the walk is conducted against a document someone
else wrote.

**Pass the brief below verbatim.** It is fixed text, never composed ad hoc —
that is precisely what keeps the looking-for list from being shaded by whoever
dispatches it. Add only the artifact paths and the branch/base refs.

> You are analysing a finished implementation run. Read the plan document, the
> implementation log, the concept document if one is given, any previous rounds
> of the findings document, and the cumulative branch diff. **The diff is ground
> truth**; the plan, the log and every report in it are claims to be checked
> against it.
>
> Traverse **the plan**, item by item — the plan is the spine, not the branch
> history. The unit of comparison is the cumulative diff, not individual
> commits; *when* something was done, or in which commit, does not matter, and
> commit-level attribution is needed only to cost a revert.
>
> Find two things:
>
> **The changes to the plan.** Every change the run recorded — logged
> divergences with their assumptions and rejected alternatives, provisional
> annotations, plan lines marked as added, logged drops of planned work. Each
> one is an item the user must approve or send back.
>
> **The completeness gaps.** For every plan item, establish that its work is
> present in the diff, or that a log entry explains its absence. A ticked item
> whose work is absent from the diff is a finding. A report claiming work the
> diff denies is a completeness finding, not a change. This is a presence check:
> an item absent from the diff with no log entry needs rework by definition, and
> no intent-level judgment is required to route it.
>
> Re-derive; never consume. Border reports are inputs to be checked, not
> conclusions to be relayed. What no per-commit reviewer could see is a commit
> that never happened — that whole-plan view is the point of this analysis.
>
> Out of scope: bugs, style, entropy, present-but-hollow work, and any reviewer
> finding the run dismissed or denied. Read code where it is needed to adjudicate
> a change; do not sweep the diff for its own sake.
>
> Write your findings to the findings document before reporting anything. Two
> sections — the changes, then the completeness check. Number items in one
> continuous sequence; on a re-run, continue after the last number of the
> previous round and never re-raise a finding an earlier round already had
> approved. Give every item a short human name alongside its number. Collapse
> items that share one cause under a single number.
>
> Each item has **four fields: what · where · why · cost.**
> - *What* — one line.
> - *Where* — a markdown link plus a symbol name, never a hunk.
> - *Why* — mandatory. For a change, the recorded reasoning and the alternatives
>   the run rejected. For missing work, why it is absent. **"Nothing on record"
>   is a legitimate and informative value; never replace it with a guess.**
> - *Cost* — structural only, and cheap: the SHA from the log's keep-or-revert
>   index, and the dependent plan items read off plan structure. Label it
>   *structural*, because plan structure is not code dependency and can be wrong
>   in both directions. Do not attempt a code-level dependency analysis.
>
> An item's *why* may name another item where the two are coupled. Ask no
> questions inside an item.

## 3. The findings document

`impl-debrief.md`, the third sibling of `impl-plan.md` and `impl-log.md` — in the
temp location in standalone mode. One file across rounds, round-sectioned,
appended to.

- It is **written before anything is presented**. An item may be argued during
  the walk; it may never be omitted or softened.
- It always carries every item in full and **is the handoff**. The chat is the
  walk view over it.
- It joins the input set on re-runs, so an approved disposition never resurfaces
  as a finding.

## 4. Presenting — one message

**Two sections, not a class taxonomy.** Section 1, *the changes*, each to be
approved or sent back. Section 2, *the completeness check*: every plan item done,
or listed as not done.

**The header is a completeness verdict** — whether everything in the plan is
done, and if not, that N things are not. Never a commit count or a commit
inventory.

**Everything in one message.** Understanding the run fast is a single-read
requirement; progressive disclosure turns reading into interrogation. All items,
with items sharing a cause collapsed under one number.

**One continuous numbering, stable for the whole walk.** An item keeps its number
and its human name until the walk ends, so a clarifying question costs no
re-orientation. **One numbered item per line** — never several bundled into a
paragraph.

**One anatomy, four fields: what · where · why · cost** — as specified in the
brief, carried through to the message unchanged. "Nothing on record" is stated as
such, never filled in with a guess.

**Correctness leads, cost follows.** Each item's frame is whether the change is
*right*, with the recorded why and the rejected alternatives in front. Cost is
supporting information, carried where dependents exist, and never the frame.

**No question inside an item.** The item carries its cost line; one closing ask
restates every item by its human name. Prose and evidence, never diffs — the user
reads those in the IDE.

## 5. The walk

Take dispositions in any order. **Answer correctness questions by reading the
code**, never from memory of having written it. Record each disposition into
`impl-debrief.md` as it is taken, not at the end.

**Cost is cheap by default, deep on demand.** For an item the user is actually
considering sending back, run a real code-level dependency check as a per-item
deep dive. Only then.

Close with an **echo by name** of what is settled and what is still outstanding.
The walk ends with a **proper report** — the dispositions taken, the work
appended to the plan, and anything left outstanding — never just the last echo.

**Two verdicts, one vocabulary, both sections:** an item is either **approved** or
it **needs rework**. Missing work needs rework by definition — no dropping, no
deferring; it is work that was skipped. Reverting provisional work is a success
of the process, not a failure of it.

**This skill executes nothing.** No implementer, no code edit, no revert, no
follow-up work. Reviewing and doing stay separated; execution happens after, on
its own footing. The load-bearing consequence: **the record is the handoff.** A
rework disposition must be actionable by whoever picks it up — it carries the
SHAs and what falls with them, or the plan line it became. Nothing may be left
implicit in the conversation.

## 6. Outcomes

**Work still to do → the plan.** Append it to `impl-plan.md` as ordinary unticked
commit lines in a **new final wave** (`Wave N+1`, single track; in phase mode a
new final phase). Every appended line carries a trailing
`*(added — debrief finding N)*`, pointing into the findings document.

Finding numbers run **continuously across debrief rounds** — round two starts
where round one ended — so "debrief finding N" stays unambiguous without a round
prefix.

**A change to what the feature *is* → plan.md.** `impl-plan.md` cannot hold a
design decision. In plan mode, plan.md sits beside impl-plan.md in the working
folder and receives the amendment. In standalone mode there is no concept
document: record the design-level finding in the findings document only.

**One doc-only commit** at the end of the walk, carrying the findings document
and the recorded dispositions. That commit is the record of what the user
ratified. In standalone mode nothing is committed — plan, log and findings stay
scratch siblings in the temp location.

**Done is defined by approval, not by detection.** Done means no item is left
unapproved and no item needs rework. This is the terminal state of the whole
pipeline, not something the analysis computes — which means the debrief may be
entered more than once: rework happens outside the skill, appended plan work
makes the plan unfinished, so a further run and a further debrief follow. The
feature is done when a debrief round ends with everything approved and nothing
needing rework.

## Honesty

Invoking a skill does not start a new session, so on the auto-invoked path the
session that did the work conducts the walk. Four things buy honesty instead:

- **The analysis brief is fixed text in this file**, never composed ad hoc, so
  the looking-for list cannot be shaded.
- **The findings document is written before anything is presented** and is the
  record — an item may be argued, never omitted or softened during the walk.
- **Drill-downs are answered by reading the code**, never from memory of having
  written it.
- **On the auto-invoked path, one line stating that this session is the one that
  did the work**, and that the findings document supports an independent walk
  elsewhere. That hands the user the choice instead of asserting an independence
  that is not there.
