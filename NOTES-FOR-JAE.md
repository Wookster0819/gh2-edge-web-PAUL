# Notes for Jae — from the dashboard/heat-map polish pass (Sept 2026)

Written by Paul + Claude while iterating on `PAUL/ballpark-polish`. Two kinds
of notes: a **marketing/product observation** Paul flagged as important, and
the **integration flags** we accumulated while building.

Everything below is based on the *engine outputs* in `data/plan-fixtures.json`
(the three test households). We don't have the engine itself and aren't asking
to touch it — these are observations for you to weigh.

---

## 1. IMPORTANT — the retiree segment (Paul)

**Observation.** The households that are *already in retirement* show the
most dramatic EDGE impact in absolute dollars:

| Household | Stage (from the series shape) | Starting kept wealth | Lifetime gain vs a good conventional plan | Gain as % of starting wealth |
|---|---|---|---|---|
| Mahoney | retired, spending ~$96K/yr (drawdown from day one) | ~$1.06M | **+$578K** | ~55% |
| Henderson | pre-retirement (still accumulating) | ~$403K | +$249K | ~62% |
| Reyes | retired, depleting | ~$278K | $0 | 0% |

Mahoney has **zero Roth conversions** in the EDGE plan yet still gains $578K —
so for a retiree the value is coming from *sequencing*: withdrawal ordering,
Social Security timing, IRMAA/Medicare-premium management, and tax-bracket
management in drawdown. Their driver mix (Portfolio withdrawals 85%, Medicare
premiums 10%, Taxes 8% of impact) is very different from Henderson's, where
the plan schedules a $123K Roth conversion in year one.

**Paul's ask:**
1. **Marketing:** if EDGE moves the needle this much for people *at or in*
   retirement, that crowd deserves its own message. Today's public copy
   ("will your savings last?") speaks to the pre-retiree. A retiree hears
   "I'm already drawing down — can EDGE still help?" The Mahoney answer is
   a compelling yes, and it's a segment with the most assets and the most
   urgency.
2. **Product:** consider whether retirees need **retiree-specific features
   or emphasis** — e.g. withdrawal-sequencing view, an IRMAA cliff tracker,
   RMD runway, SS-claiming decision (if not yet claimed), LTC planning (your
   "next tier adds" hook already points here).
3. **Data-set question for you:** do the inputs shift in *importance* for a
   retiree vs. an accumulator? Our guess from the outputs: account mix
   (pre-tax/Roth/taxable) and spending level dominate for retirees;
   contribution rate and salary growth matter little. If the engine's
   sensitivities confirm that, the Ballpark "sharpen your number" list
   could be **reordered per stage** (we already sort by impact — it just
   needs stage-aware weights from you).

**Caveat:** three fixture households isn't a study. But the pattern is
strong enough that Paul wants it on your radar before positioning is set.

---

## 2. Integration flags from this pass

- **Household resolution.** Members must see *their* household. The `?hh=`
  parameter and the dropdown (now behind `?dev=1`) exist only to exercise
  fixtures. The engine/API has to resolve the household from the session;
  the front-end must never choose.
- **Rows vs. curves on Net Outcome Trend.** For Mahoney the rows say
  **+$51K after year 1**, but `comparison.series` shows the EDGE–conventional
  gap at ~$45K on day one narrowing to ~$18K by 2032. The rows and the series
  aren't measuring the same thing. Please define each so the card can't
  contradict itself. Also: fixture 3-year (+$47K) < 1-year (+$51K) — if the
  windows are cumulative-from-today as labeled, that's inconsistent.
- **Rolling windows.** Rows are labeled *"Next 12 months (from today)"*,
  *"Next 3 years (from today)"*, and *"Lifetime (since you signed up in
  YYYY)"*. The first two should roll with the calendar; lifetime accrues from
  signup. Needs engine support: `cmp.realizedAdvantage` (gain to date) and
  ideally a yearly `cmp.advantageSeries`.
- **Chart shape for retirees.** Both plans' kept dollars fall early for a
  drawdown household (that's spending, not loss). Paul is fine with that for
  retirees but a first-timer may read it as "losing money." Optional: a
  stage-aware caption you're comfortable with, e.g. *"You're in retirement,
  so both plans spend down; the green gap is what EDGE keeps for you."*
- **Tier label.** Topbar shows "Your plan · GH2 EDGE™". Set `DATA.tier` in
  the payload to show the member's real tier.
- **Premium blur (Ballpark).** Blurred +$/yr and extra-% are CSS-only —
  omit those values from the public payload server-side.
- **Ballpark placeholders** marked `// Jae:` — precision %, range width,
  per-input impact weights — are display stand-ins until the engine owns
  them.
- **Household switcher decision.** Paul's call: hide it from members (done).
  Delete the dev hook entirely, or keep `?dev=1` for your testing — your
  preference.

*Questions → Paul.*
