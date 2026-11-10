---
name: distill
description: Distill code, research, documentation, or a complex problem down to what a project-savvy developer needs to understand it or decide on it. Produces one calibrated artifact — core takeaway, the load-bearing detail, then conclusion/next steps — at the shortest length that stays faithful. Use whenever asked to "distill", "boil this down", "summarize this for me", "TL;DR", "make this digestible", "what's the takeaway", "explain this simply", or to condense something just read (code/docs/research) into something a human can act on. Do NOT trigger on bare revision phrases like "too long", "too technical", "you lost me", or "make it shorter" on their own — those are plain editing requests; only treat them as distill feedback when a distillation from this skill is already in play.
---

# Distill

## The problem this solves

Distillation fails in four predictable ways: too technical (a buzzword mess), too simple (nuance lost), too long (not worth reading), too short (useless for a decision). All four share one root cause: **distilling without separating what is load-bearing from how it is said.**

A specific failure dominates: **right after reading code or technical material, the source's vocabulary contaminates the output.** The model mirrors the jargon it just absorbed — deleting words while keeping class names and identifiers. This contamination is sticky; it reverts even after being corrected in the same context.

The structural fix is the one invariant of this skill: **the writer never sees the raw source.** Extraction happens where the source lives; expression happens in a clean subagent that only ever sees the curated claims. Contamination cannot leak into prose that was never exposed to the source.

## The reader

Distillation is always *for someone, for a purpose* — that anchors every other choice. Fix both before extracting; if the request doesn't state them, infer the most likely pair from the material and say which you assumed (don't stop to ask — that defeats a fast tool).

**Default reader**, unless the task implies otherwise: a developer who **knows this project but not its code at the line/class level** — wants concepts and consequences, not identifiers.

**Adapt the reader to the material.** The default is code-shaped; recalibrate for other inputs — e.g. research/docs → an informed generalist weighing a decision, not a specialist; a tangled problem → the person who has to make the call or walk into the meeting. Whatever the reader, prefer the simplest framing that stays true — never be technical for its own sake. If something can be said plainly, say it plainly.

## Workflow

Three passes. Main session orchestrates and may be "contaminated" — that is fine, because main does not write the final prose.

### Pass 1 — Extract (where the source lives)

Build the **extract**: an ordered list of the load-bearing claims, tradeoffs, and decision-relevant facts. Nothing else.

- If the source is already in context, extract from it directly.
- If the source is not yet read (a file/doc pointer, a topic to research), read/gather it now — or delegate to a subagent to keep main lean. Either way, the goal is the extract.

**The general principle:** keep what the reader needs *for the purpose*; strip provenance, jargon, and noise. "Load-bearing" is defined relative to the reader and purpose you fixed above — not in the abstract. What "strip" means depends on the input:

- **Code / technical material** — translate identifiers to roles: `AuthTokenRefresher` → "the component that refreshes auth tokens". Park the real identifier / source reference in a trailing parenthetical so names can be restored on demand, but keep them out of the default phrasing. Mark any claim whose likely next action requires touching that exact code as action-critical (keep its name).
- **Research / docs** — de-jargon the same way (plain phrasing, term parked in parentheses). Additionally **keep uncertainty explicit**: mark contested/fast-moving claims, give ranges not false precision, and distinguish well-established from single-source. Dropping the uncertainty is how a research distillation becomes confidently wrong — it is load-bearing.
- **A complex problem** — distilling is *clarifying, not summarizing*. Find the real question under the symptoms, the core tension, the constraints that actually bind (and name what each one rules out — that's the insight that reframes things), which "options" are real vs premature/red-herring and why, and the critical unknown the decision hinges on. Name the emotional/political/pet-preference **noise** briefly so it can be set aside. Draw the inferences the source implies even if it didn't connect the dots.

The extract is the contract. It must be complete (no load-bearing claim missing) and self-sufficient (understandable without the source).

### Pass 2 — Express (clean subagent, source-blind)

Spawn a subagent via the Agent tool. Give it **only**: the extract, the reader profile, the output skeleton, and the standing rules below. **Do not give it the raw source, the file contents, or the original identifiers** (except names explicitly marked action-critical). It returns the finished distillation.

This boundary is the whole point — do not shortcut it by writing the prose in the main session, even when it seems faster. The clean room is what stops contamination and what stops within-session reversion.

### Pass 3 — Verify (main session)

Check the returned prose against the extract:

- **Complete?** Two layers, because checking only the prose-against-extract is circular — a claim the extract itself dropped is invisible that way:
  - *Prose vs extract* — every extracted load-bearing claim made it into the prose; no nuance or uncertainty dropped in the writing.
  - *Extract vs source* — the extract didn't omit a load-bearing claim in the first place. This can ONLY be checked against the source, so when the source is reachable (in context, or re-ask the extractor / re-open the file) spot-check it for anything load-bearing the extract missed. When the source is genuinely unreachable, say so — completeness is then bounded by the extract's fidelity and you can't fully vouch for it.
- **Clean?** No surviving provenance jargon that isn't carrying weight (subject terms may stay).
- **Simple enough?** Could be plainer without losing truth → send back.
- **Faithful?** Nothing invented beyond the extract (watch for slipped-in specifics — a number, a timebox). If the source is in context, spot-check claims against it.

Fix issues by **re-invoking the writer subagent** with a targeted instruction — never by hand-editing in the polluted main context.

## Output skeleton

A light structure, not a form. If the material is genuinely simple, collapse it — do not pad to fill sections.

1. **The core** — one or two lines. The single most important thing / the bottom line.
2. **The explanation** — the load-bearing detail, at the reader's level.
3. **Conclusion / suggestions / next steps** — what to do or decide with this.

## Standing rules (give these to the writer)

- **As short as possible, as long as necessary.** Length follows the material, never a target. The model over-talks — cut anything not load-bearing.
- **Simplest faithful framing wins.** Plain over technical whenever it survives the truth.
- **Drop provenance jargon, keep subject terms.** Cut class/identifier/source names unless marked action-critical or the reader asked for them. But terms that *are* the subject (a chemistry, a drug, a named option) stay — dropping them would remove meaning, not jargon.
- **No trace by default.** Omit source references unless they're needed to act.
- **Preserve uncertainty.** Carry contested/fast-moving claims and ranges through as-is; don't flatten a range into false precision or a "maybe" into a fact.
- **Invent nothing.** Say only what the extract supports — no fabricated specifics (a number, a timebox, a name) that aren't in it. If the extract is silent, stay silent or stay general.

## Responding to feedback

Each feedback type re-runs the relevant pass through the **writer subagent** with the same extract plus a targeted instruction. Feedback never truncates existing text in place.

| Feedback | Action |
|----------|--------|
| "too long" | Writer: drop non-load-bearing claims, tighten wording. **Never** cut a load-bearing claim. |
| "too technical" | Writer: re-express, replace surviving identifiers with roles, plainer language. |
| "you lost me" / "too vague" | Restore a dropped tradeoff or add one concrete anchor. If the claim isn't in the extract, re-extract it first. |
| "keep the names" / "give me refs" | Writer: include the recorded identifiers / source references for the relevant claims. |
