---
name: pairwise-weights
description: Derive criterion weights from user pairwise comparisons using a 1/5/10 scale with reciprocal fill, row sums, and normalization to 100%. Use when weighting scoring criteria, building a rubric, or the user asks for pairwise comparison / AHP-style weights.
---

# Pairwise comparison weights

## Scale

| Answer | Value | Reciprocal filled at M[j][i] |
|---|---|---|
| About the same | 1 | 1 |
| More | 5 | 0.2 |
| Much more | 10 | 0.1 |

Only these three answers exist. Never offer other magnitudes.

## Asking

- Group criteria into clusters first. **Around 5 per matrix is the sweet spot** (10 pairs).
  Pairs grow quadratically, and answers get noticeably less considered past roughly ten
  questions. This is guidance, not a hard limit — if the user wants 7 or 8, run it, but
  say once that consistency usually suffers and offer to cluster instead.
- Define every cluster in plain language **before** the first pair. Name, what it covers,
  what high and low look like. One short block each.
- Ask **one pair per question**. The scale needs 5 answers but the question UI caps at 4
  options, so use exactly this layout for every pair:
  - `About the same` (1)
  - `<A> more` (5)
  - `<A> much more` (10)
  - `<B> more` (5)
  - `<B> much more` (10) — user selects Other and types it; state this once at the start,
    not on every question.

  Never vary this layout mid-run: option order is itself a bias, and changing it makes
  early and late pairs non-comparable.
- Batch 2-4 pairs per call. Track and show progress: `pair 3/10`.
- Do not re-explain the scale on every question.

## Computing

1. Build the n×n matrix. Diagonal = 1. Fill the reciprocal automatically — never ask the
   same pair twice.
2. Sum each row, including the diagonal.
3. Normalize the row sums to 100%. These are the weights.

Do not offer alternative aggregation methods unless asked. Textbook AHP uses the principal
eigenvector and will give somewhat different numbers; that is a known, accepted difference,
not a defect to correct mid-run.

## Consistency

Report intransitive judgments explicitly. If A > B and B > C but C > A, name the cycle and
ask which of the three to revisit. Do not silently average it away.

## Output

A table of criterion and weight, sorted descending. Then the matrix itself, so the user
can audit it.

Then one short diagnostic line: if two criteria carry most of the weight, or one is near
zero, say so and **name the specific pairs that caused it**. Users who find a weight
surprising need to know which comparison to revisit, not to redo the whole run.

Weights apply only within their own matrix. Never merge two matrices or average across
them without the user asking.

## Scoring options against the weights

Weights alone don't choose anything. When the user has options to rank, run a second
level and synthesize.

**Who answers which matrix:**

- **Criteria matrix — always the user.** What matters is a values question. Never fill it
  in on their behalf, and never infer it from the project.
- **Option matrices — you, once you have real context.** How well each option delivers a
  criterion is a factual question. Comparing two frameworks on load time or two sites on
  photography is research, not preference. Do not make the user answer 10 pairs per
  criterion.

**Procedure:**

1. For each criterion, pairwise-compare all options on that criterion alone. Same 1/5/10
   scale, same reciprocal fill. Produces a local weight per option, summing to 100%
   within that criterion.
2. Synthesis: `option score = Σ (criterion weight × option's local weight)`. Scores across
   all options sum to 100%.
3. Report the full ranking, not just the winner.

**Ground every option judgment in evidence you actually have** — a measurement, a page you
read, a doc you checked. If you're comparing options you haven't investigated, say so and
go investigate first. A confidently fabricated option matrix is worse than no ranking,
because the arithmetic makes it look rigorous.

Call out any option that dominates one criterion and still ranks low. That is usually the
most informative line in the output, and it points at a criterion weight the user may want
to revisit.

## Rule

Every criterion that receives a weight must be scorable on every item in the corpus. If a
criterion can only be assessed on a subset, either make it universally scorable or drop it
before weighting — a weighted criterion with nulls corrupts the ranking.
