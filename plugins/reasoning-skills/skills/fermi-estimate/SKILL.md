---
name: fermi-estimate
description: Use before stating a quantity you have not measured, for those a human would base a decision on such as but not limited to lines of code, files touched, tokens, cost in USD, hours, runtime, capacity, counts. Independent on if it was volunteered or asked for. Fires on "how many", "how much", "how long", "how big", "estimate", "ballpark", "order of magnitude", "is this small", "what will this cost", "is it worth it", German "wie lange/viel/groß", "Aufwand", "Schätzung", "Größenordnung", and whenever you would write "~", "about" or "roughly" in front of such a number. Produces a 95% range with its anchors. Not for a number one read returns (a diff's line count, a bill's total) — read that instead.
---

# Fermi estimation

## Why estimate, and what "good" means

An estimate by definition is a cheap way to predict a rough outcome **before** doing the deliverable in question. E.g. before running the test, before implementing the code, before writing the text or doing the proper calculation.

In most cases here the estimate is the basis of a decision: is the outcome worth the cost. From this follows:

1. An estimate costing a large fraction of the actual work has failed, however accurate it is.
2. An estimate pleases the operator when it is accurate, not when the number is high or low. Even when an operator hopes for a high or low number in the short term, the long-term satisfaction is what the estimation aims for. The estimate is made before any target is discussed; a target may be deliberately ambitious, but it is set afterwards and labelled as a target — the estimate never moves toward it.
3. A wrong estimate may cause an expensive decision to be made the wrong way.
4. State the uncertainty as a 95% interval, and print no more significant figures than that interval supports. The interval is the accuracy; the digits only display it.

Two ways this goes wrong: a point with no range, and measuring or building the thing instead of estimating it.

## Workflow

Three steps, in the given order; the order matters. 

### Step 1 — Decompose toward anchors

Write the target as a product or sum (or simple mathematical equation) of factors (3–6 should be expected for most problems). Each factor names the **anchor** that could ground it, something that can be guessed easily or measured quickly. Examples, not limited to: a graded past estimate, a comparable file or diff, a bill, a known rate, a count of things, a past run. A factor with no conceivable anchor stays in the list, marked `guess`.

Consider all that will be delivered, not only the logic that does the work. Do not silently drop parts of the estimate, e.g. code is delivered with tests, token cost includes the reviewers not just the writers. Do not price a part at zero because some tool could in principle absorb it — only when it actually will (the tool is already in use, or the operator has proposed it).

The best anchor is a past estimate that has been graded against its actual outcome — it is calibrated. If the project keeps an anchors log (a memory or file of estimate/actual pairs), use this as a basis.

Stop decomposing when every factor is either anchorable or a plain judgment with a defensibly small range. Going deeper adds factors without removing uncertainty.

**Count, don't judge — when the count is one command.** A cross-cutting change is a count of touchpoints (`grep -l … | wc -l`, `git diff --stat`), not an impression of the concept: "thread one field through" is a judgment; "7 files, 30–50 lines each" is a count. Count the touchpoints, never their content. If counting would mean opening files, it is a lookup and goes through Step 2; otherwise estimate the count.

### Step 2 — Triage the anchors

Before triaging factors, one line for the deliverable itself: does it, or part of it, already exist (`git log --grep`, `find -name`)? What exists is known, not estimated.

Triage every factor, before any command runs:

- **Known**: a well-established number — already established in this conversation with its source, a known rate, or a constant of nature.
- **Lookup**: the number is recorded somewhere (anchors log, bill, file) and a single read returns it.
- **Derivable**: the number is not recorded, but a single read-only bash command produces it.
- **Unknown**: anything that is more effort.

For anything that lands in `unknown`, name a method of getting it and deliberate: a lookup is worth its cost only if its result could move the interval across the decision threshold — with no threshold, only if the factor dominates the width. A yes is still read-only — no code written, no workload run. A no means the factor is guessed with a range.

```
<factor>: known — <the value> | lookup — <the place> | derivable — `<the command>` | unknown — <the method of getting it> -> (yes | no — guessed)
```

Write the line for every factor before anything runs. Do not run any command to gather anchors that is not in this list; Step 3's arithmetic is not gathering. Do not default to a yes — deliberate it; your deliberation may be audited.

The whole estimate is quick work. If a check would take longer than the estimate is worth, that is "not worth it" — widen instead.

### Step 3 — Estimate

1. **Range each factor** — the low–high you would bet 95% on, given the quality of its anchor. Every factor gets a range, side figures included; the un-ranged side number is historically the worst miss.
2. **Combine — compute, don't eyeball.** This is a computer; the arithmetic is free. For a product, root-sum-square the log-widths: `σᵢ = ln(highᵢ/lowᵢ) / 3.92`, `σ = sqrt(Σσᵢ²)`, center = the equation from Step 1. Uncertainty estimation follows the equation as well. For small uncertainties this equals root-sum-square of the relative errors; for factors uncertain by a multiple it stays correct where relative errors do not. For a sum, add variances of the absolute errors instead. Run it as a one-liner (python) and quote its output.
3. **One cross-check by an independent route** — a different decomposition, or a single top-down anchor (the nearest graded row in the anchors log, or the nearest comparable thing that already exists, scaled). If the two routes' 95% intervals do not overlap, one route has a wrong factor — find it. If they overlap, keep the route whose factors are better anchored — a tighter interval is not the better one when its width was a single judgment. Never average them; they share factors and are not independent.
4. **Does it decide?** The interval exists to make a decision. If it straddles the decision threshold, go back to Step 2 for the factor that dominates the width — once. If it still straddles, report that: the estimate cannot make this decision, and say what would.
5. **Tokens** — a turn costs roughly the whole context, not the new message; tool output enters context in full; ~4 characters per token, a code line ~10–15 tokens.

## Output

Lead with the number, then the evidence, then stop.

```
<center> +<up> −<down> <unit>        ← 95%; digits as the bounds support

- <factor>: <range> — <anchor, or "guess">
- <factor>: <range> — <anchor>
- … (3–5 lines)

Moves it most: <the factor>, <whether reading it would be worth it>.
```

When you volunteered the estimate instead of being asked for, only use the numbers in your answer; you may not need to report the anchors for legibility reasons.

One clause on what the estimate excludes if the work might get bundled. If the project keeps an anchors log, append the estimate to it in the same turn.

The number is tied to its assumptions. When the design changes, re-issue it — the figure should visibly move with the reversal that moved it, and the post-mortem owns the full miss rather than splitting causes to look better.
