---
name: debate
description: Highest-level reasoning primitive for finding the truth in a hard, contestable problem. Runs a structured, source-verified, adversarial debate between two subagents (Advocate FOR / Critic AGAINST a single proposition), moderated to convergence, then synthesizes a settled verdict. Use when a decision is genuinely uncertain and getting it right matters — locating the root cause of a bug, choosing the best way to solve an issue, settling a contested design or architecture call, or pressure-testing a plan or claim before committing to it. Triggers include "debate X", "let's debate whether...", "argue both sides of...", "stress-test this plan/claim", "find the real root cause of...", "which approach is actually best". Not for simple lookups, well-defined tasks, or questions with an obvious answer — debate is for problems where the strongest counterargument must be surfaced before you can trust the conclusion.
---

# Debate

A truth-finding engine. Two subagents argue opposing sides of one proposition, verify every claim against real source, and are driven to convergence by a Moderator (you, the main session) who then synthesizes a settled verdict. The value comes from forcing the *strongest counterargument* into the open and making each side concede only to verified evidence — not from winning.

## When to use

A debate is worth its cost only when the problem is **contestable and consequential**: the answer is genuinely uncertain, more than one position is defensible, and a wrong conclusion is expensive. Root-causing a stubborn bug, picking between real solution alternatives, settling a design call, pressure-testing a plan. If the answer is a lookup or obviously correct, do not debate — just answer.

## Core shape

- **Binary only.** One proposition, two sides: **Advocate** (FOR) and **Critic** (AGAINST). If a problem is genuinely multi-option (≥3 mutually exclusive choices), reframe it as a binary proposition (e.g. debate the leading candidate vs. "something else is better"), or run separate debates — do not bolt on extra debaters.
- **You are the Moderator.** The main session runs the loop: frame the proposition, spawn the two debater subagents each round, referee, decide convergence, synthesize. This keeps the human able to steer between rounds (see Steering).
- **Debaters return concise structured positions, not transcripts.** This keeps the main session's context lean across rounds.

## The proposition

Before spawning anyone, write a single, sharp, falsifiable proposition. A good proposition is one sentence that one side affirms and the other denies. Bad: "discuss the auth bug." Good: "The auth-service 500s are caused by the missing `APP_VERSION` env var in the QA terraform block." If the launch was vague, draft the proposition and confirm it before debating — a mis-scoped proposition makes both sides converge confidently on the wrong question.

## Round structure

A **round = one full exchange where both debaters speak once.** The light scaffold below structures the *epistemics*, not debate theatrics — skip rhetoric and ceremony.

1. **Round 1 — opening.** Spawn Advocate and Critic **in parallel, blind to each other.** Each independently builds the strongest source-verified case for its side. This is position commitment, not rebuttal.
2. **Rounds 2…N — rebuttal.** Each debater sees the other's prior round and issues a *triaged* rebuttal (see Debater rules). The contested set must shrink each round.
3. **Convergence check (every round).** See Convergence.

## Round cap (runaway protection)

**`MAX_REBUTTAL_ROUNDS` — default `3`.** This is purely a safety bound to stop a non-converging debate from running forever; it is **not** a target. The total exchange budget is **1 opening + `MAX_REBUTTAL_ROUNDS` rebuttals** (default = opening + 3 rebuttals = 4 exchanges max). A debate that converges earlier **must stop earlier** — never spend rounds you don't need.

To change it on a given run, the caller can say e.g. "debate … with 2 rebuttal rounds" or "max 5 rounds"; honor that value in place of the default.

## Debater rules (bake these into every debater prompt, verbatim in spirit)

These three blocks are the heart of the skill. Reproduce their substance in each Advocate/Critic prompt; the wording is deliberate, not decorative.

**1. Strong but convincible.**
> Argue your assigned side as forcefully as the evidence supports. Concede a point **only** when the other side has verified a claim against real source that you cannot refute. Never concede to confidence, rhetoric, repetition, or politeness — only to evidence. Equally, never keep defending a position the evidence has already defeated. Intellectual honesty outranks advocacy: your job is to find the truth, not to win. Stating "you are right, and here is the point that changes my position" is a success, not a loss.

**2. Verify before it counts.**
> Every factual claim you make or rely on must be checked against real source (read the actual file, run the actual command, cite the actual line) before it carries weight. An unverified claim is discarded, not debated. Cite what you verified (`path:line`, command output) so the Moderator and your opponent can check it.

**3. Triaged rebuttal (rebuttal rounds only).**
> First, list every material claim your opponent made this round. Label each `CONCEDE`, `CONTEST`, or `IRRELEVANT`:
> - `CONCEDE` / `IRRELEVANT`: one sentence only — state why it is true-but-non-decisive, or off-proposition. It is now closed; do not reopen it.
> - `CONTEST`: reserve your depth here. Identify the 1–2 **load-bearing** points (the ones that, if resolved against you, decide the proposition) and rebut each with a verified source.
> Do not re-argue points already conceded by either side in a prior round. Carry forward only the still-contested set. If you cannot find a load-bearing point to contest, say so and move toward concession.

This makes the debate exploit the model's breadth (nothing survives by neglect) while spending real reasoning only where the truth is decided.

## Moderator duties (you, each round)

- **Referee verification.** Discount any claim a debater did not actually verify against source. Do not let a confident-but-unverified assertion move the verdict.
- **Track the contested set.** It must be strictly smaller than the previous round. If a material point was raised and neither labeled nor rebutted, flag it as **dangling** and require it be addressed before you allow convergence — a dangling objection that silently "wins" produces a false verdict.
- **Decide convergence** (below).
- **Stay neutral until synthesis.** Steer for scope and missing evidence, not for a preferred answer.

## Convergence and termination

- **Converged** — the debaters agree, or you are convinced by the verified record and the contested set is empty (no dangling points). → **Synthesize immediately. No closing statements.** Stop now even if rounds remain. Do not run longer than required.
- **Hit the round cap still unresolved** — the contested set is non-empty after the last allowed rebuttal. → Call for **closing statements**: each side delivers a final **per-sub-point ledger** (which points it concedes, which it still contests, and why). Then synthesize a decision from that ledger.

Closing statements are a *non-convergence* mechanism only. On convergence they are skipped.

## The verdict (output)

Synthesize a concise verdict containing:
- **The decision** — the answer to the proposition, with the load-bearing reason.
- **What moved it** — the key verified findings and concessions that settled it.
- **Residual risk** — anything still uncertain.

**If the debate did NOT converge** (ended at the cap), open the verdict with a strong, unmissable non-convergence disclaimer — a bold callout at the **top** (e.g. `⚠️ NOT FULLY RESOLVED — decided under the round cap with points still contested`). Re-flag each still-contested decision inline where it appears, so nothing reads as settled when it isn't.

### Output mode (infer from how the debate was launched)
- **Answer-only** (default for questions like "where's this bug from?", "which approach is best?") — report the verdict concisely inline to the user. Do not create files.
- **Document** (when the launch asks to produce a written artifact, e.g. "debate and write a plan/decision doc for X") — persist the verdict as a markdown doc (default location `.claude/feature-requests/` unless the caller specifies otherwise), then summarize inline.

The debate engine is identical for both; only the rendering differs. Do not pull plan-specific machinery into the debate itself.

## Steering

Default: **you moderate round-by-round inline**, so between rounds the human can inject course corrections — the decisive interventions are early (a mis-scoped proposition, or a fact both sides missed). Surface a one-line "round N done; contested: …" between rounds so steering is possible.

If the launch says to run autonomously ("run it to a verdict", "don't ask me", `--auto`), skip soliciting steering between rounds and run straight to convergence or the cap.

## Spawning debaters

Spawn each debater as a subagent (`general-purpose`). Give each: the proposition, its assigned side, the three debater rule blocks above, the opponent's prior round (for rebuttal rounds), and an instruction to return a **concise structured position** (its claims with `path:line` citations, its triage labels, and whether it concedes the proposition). Run the two sides of a round concurrently when they are blind to each other (round 1); in rebuttal rounds each needs the other's prior output, so spawn them with that input.
