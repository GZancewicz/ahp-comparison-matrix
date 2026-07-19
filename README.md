# pairwise-weights

A Claude Code skill for turning "which of these matters more?" into defensible numeric weights.

When you need to score things against multiple criteria — vendors, candidates, designs,
websites — the hard part isn't the scoring, it's deciding how much each criterion counts.
Guessing at percentages produces weights nobody trusts. This skill derives them from a
series of simple two-way comparisons instead.

It's a variant of the **Analytic Hierarchy Process** (AHP, Saaty 1977), simplified to a
three-point scale.

## How it works

Criteria are grouped into clusters (max 5 per matrix). Claude asks you about each pair:

> Which matters more: **Photography & visual identity** or **Mobile & performance**?
>
> - About the same
> - Photography more
> - Photography much more
> - Mobile more
> - Mobile much more

Answers map to a 1/5/10 scale. The reciprocal is filled in automatically, so you're never
asked the same pair twice. Row sums are normalized to 100%.

Claude reports two sets of weights — raw-sum and column-normalized — because the choice
materially changes the answer, and you should see it rather than inherit it silently.
Intransitive answers (A > B > C > A) are named rather than quietly averaged away.

## Worked example

Four criteria for choosing an apartment: **Price**, **Commute**, **Space**, **Natural light**.

Six pairs, six answers:

| Pair | Answer | Value |
|---|---|---|
| Price vs Commute | Price more | 5 |
| Price vs Space | Price much more | 10 |
| Price vs Natural light | Price much more | 10 |
| Commute vs Space | Commute more | 5 |
| Commute vs Natural light | Commute much more | 10 |
| Space vs Natural light | About the same | 1 |

### The matrix

Diagonal is 1. Everything below it is the reciprocal of its mirror above — filled in
automatically, never asked.

|  | Price | Commute | Space | Light | **Row sum** |
|---|---|---|---|---|---|
| **Price** | 1 | 5 | 10 | 10 | **26.0** |
| **Commute** | 0.2 | 1 | 5 | 10 | **16.2** |
| **Space** | 0.1 | 0.2 | 1 | 1 | **2.3** |
| **Light** | 0.1 | 0.1 | 1 | 1 | **2.2** |
| | | | | **Total** | **46.7** |

### Normalizing to 100%

Divide each row sum by the total:

| Criterion | Row sum | Weight |
|---|---|---|
| Price | 26.0 / 46.7 | **55.7%** |
| Commute | 16.2 / 46.7 | **34.7%** |
| Space | 2.3 / 46.7 | **4.9%** |
| Natural light | 2.2 / 46.7 | **4.7%** |
| | | 100.0% |

### The second method

Normalize each column to sum to 1, then average across each row. Same matrix, different
arithmetic:

| Criterion | Raw sum | Column-normalized |
|---|---|---|
| Price | 55.7% | 63.8% |
| Commute | 34.7% | 26.3% |
| Space | 4.9% | 5.2% |
| Natural light | 4.7% | 4.8% |

Note the top two: an 8-point swing on Price and an 8-point swing on Commute, from
identical judgments. Neither method is wrong — column-normalized is closer to textbook
AHP, raw-sum is simpler to explain to people who have to accept the result. The skill
reports both so the choice is visible rather than buried.

The bottom two barely move. That's typical: the methods diverge most where it matters
most.

## Install

```
/plugin marketplace add GZancewicz/pairwise-weights
/plugin install pairwise-weights@GZancewicz
```

## Use

Just describe the problem:

> Help me weight these criteria for evaluating job candidates.

Or invoke it directly with `/pairwise-weights`.

## Design notes

A few opinions are baked in, learned the hard way:

- **Five clusters maximum.** Ten pairs is about where careful judgment stops. Twenty
  criteria means 190 comparisons and the back half is noise.
- **Define the criteria before the first pair.** Most inconsistent results come from the
  user's understanding of a criterion shifting halfway through, not from genuine
  ambiguity about importance.
- **Every weighted criterion must be scorable on every item.** A criterion you can only
  assess on some of the corpus corrupts the ranking, no matter how well-weighted.

## License

MIT
