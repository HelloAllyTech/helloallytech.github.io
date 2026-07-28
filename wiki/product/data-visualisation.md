---
title: Data Visualisation
tags: [product, best-practices, data-viz, charts, dashboards, analytics, metrics]
summary: Rules for showing numbers honestly at Ally — pick the chart from the question, never hide sample size or uncertainty, and design the dashboard around a decision.
---

# Data Visualisation

*Part of [Product Management Best Practices](best-practices.md).* **Maturity: Draft.**

> [!IMPORTANT]
> **Building an actual chart or dashboard? Open [Chart & Dashboard Design Principles](chart-dashboard-principles.md).**
> That page is the craft reference — how people read a chart, required anatomy, colour and
> emphasis rules, the persuasion-vs-deception boundary, chart-type lookup tables, dashboard
> composition, a per-chart acceptance checklist and an anti-pattern list. **This** page is the
> shorter Ally-specific rule set: what we are allowed to show, to whom, with what caveats.
> Read this one first, then build against that one.

## Why this matters for Ally

Ally shows numbers to people who make consequential judgements with them: a trainer deciding
whether a learner is ready, an admin deciding where a cohort is weak, a counsellor looking at
their own performance. Several of these numbers are **LLM-derived scores over small samples** —
the easiest kind of number to over-trust. A chart that implies more precision than the data
carries is a product bug, not a styling choice.

## Principles

1. **Start with the question, then the chart.** Write the sentence the viewer should be able
   to say out loud after looking ("my Kannada sessions score lower than my English ones").
   If you can't write it, the chart has no job.
2. **Form follows question**, roughly: change over time → line; comparison across categories →
   bar; part-to-whole → stacked bar (rarely a pie, never more than ~4 slices);
   relationship → scatter; distribution → histogram/box, not an average; single tracked
   number → stat tile with a trend and a comparison basis.
3. **An average without a distribution is a half-truth.** Where variance matters — and in
   scoring it always does — show spread, or at least min/max/n.
4. **Always show n.** Small-sample scores must be labelled as such, and below a documented
   threshold should show "not enough data" instead of a confident number.
5. **Never truncate a bar-chart axis.** Bars encode magnitude by length; a non-zero baseline
   lies. Line charts may use a zoomed axis when the change, not the level, is the point —
   label it clearly.
6. **Comparison basis is part of the number.** "+12%" is meaningless without "vs. previous
   30 days" next to it.
7. **Colour carries meaning consistently, and never alone.** One palette across the product;
   the same series is the same colour on every chart; encode categories with position/label
   too, so the chart survives colour-blindness and greyscale printing/PDF export.
8. **Design the empty and thin states first.** A new tenant's dashboard is the state most
   users see first — it should teach, not show five broken axes.
9. **Every dashboard needs a decision.** If no one can name the action a panel changes, cut
   the panel. Dashboards accrete; prune them deliberately.
10. **Aggregate, don't leak.** Cross-user and cross-cohort views must respect tenant isolation
    and never expose session content or PHI through a drill-down. Small cohorts can
    de-anonymise an individual — apply a minimum-group-size rule before showing a breakdown.
11. **Label the provenance of derived scores.** If a number came from an LLM judge, say so,
    and pin the rubric/model version it came from — comparisons are only valid within one
    version.
12. **Exports must carry their context.** A CSV or PDF that leaves the app without the date
    range, filters and n attached will be misread in a meeting.

## Checklist

Ally-specific gates. For chart craft, also run the fuller per-chart checklist in
[Chart & Dashboard Design Principles](chart-dashboard-principles.md).

- [ ] The one-sentence takeaway is written down and the chart delivers it.
- [ ] Chart type matches the question type; axis starts at zero for bars.
- [ ] `n` shown; below-threshold cases show "not enough data".
- [ ] Comparison basis and time window labelled on the surface, not just in a tooltip.
- [ ] Consistent palette; meaning not carried by colour alone; readable in greyscale.
- [ ] Empty / thin-data / loading / error states designed.
- [ ] Units, rounding and precision are honest (no 2-decimal scores from a 5-point rubric).
- [ ] Tenant isolation + minimum group size respected in every breakdown and drill-down.
- [ ] Derived-score panels state their source and version.
- [ ] Export includes filters, range and n.

## Anti-patterns

- **Truncated y-axis** turning a 2% difference into a cliff.
- **Score with no sample size** — "82%" from three sessions.
- **Dual y-axes** implying a correlation the data doesn't support.
- **Pie charts with nine slices**, or any pie where the comparison is the point.
- **Rainbow palettes** where colour ordering implies a ranking that doesn't exist.
- **Dashboard as data dump** — twelve panels, no decision.
- **Tooltip-only truth** — the caveat that only appears on hover never reaches the screenshot
  that ends up in the board deck.
- **Comparing across rubric or model versions** as if they were the same scale.

## Ally-specific notes

- Analytics surfaces span [ally-be](../repos/ally-be.md) (`analytics/`, incl. Metabase-backed
  dashboards), the web dashboards in [ally-web](../repos/ally-web.md), and charting in
  [ally-mobile](../repos/ally-mobile.md) (ECharts / chart-kit). Keep the same metric named and
  computed the same way across all three, or say explicitly why it differs.
- Product analytics (PostHog) answers *how the product is used*; platform analytics answers
  *how counsellors are performing*. Do not mix them on one dashboard — different audiences,
  different decisions.
- LLM-judge-derived quality scores follow the pinning discipline described in
  [Language-Quality Evaluation & RCA](../platform/language-quality-eval.md) — read that
  before charting any language or quality metric.
- Anything that turns a number into a reward is also a [Gamification](gamification.md)
  decision.
- Dashboard *composition* — salience across the whole screen, shared visual language,
  consistent encodings across tiles, density tiers by role — is covered in
  [Chart & Dashboard Design Principles](chart-dashboard-principles.md) §15.

## Open questions

- What is our documented minimum `n` before a score is shown at all?
- Is there a single approved palette and does it survive dark mode + PDF export?
- Do the web and mobile definitions of the headline learner metrics currently match?
