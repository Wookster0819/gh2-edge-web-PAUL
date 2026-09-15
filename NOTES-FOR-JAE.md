# Notes for Jae — from the dashboard/heat-map polish pass (Sept 2026)

Written by Paul + Claude while iterating on `PAUL/ballpark-polish`. Three kinds
of notes: a **priority question about engine thresholds**, a
**marketing/product observation** Paul flagged as important, and the
**integration flags** we accumulated while building.

Everything below is based on the *engine outputs* in `data/plan-fixtures.json`
(the three test households). We don't have the engine itself and aren't asking
to touch it — these are observations for you to weigh.

---

## 0. PRIORITY QUESTION — the feasibility tiers on Quick Check (Paul)

While rebuilding Quick Check's result as the three-tier verdict card
(AT RISK / NEEDS ATTENTION / ON TRACK), we ran live profiles through
`/api/optimize-quick` and watched `feasibilityRating` flip. Same household
(age 55, retire at 67, $100K salary, Comfortable spending), savings only
changing:

| Savings | Required return | `feasibilityRating` | Card shows |
|---|---|---|---|
| $700K | 6.2% | well_funded / conservative | ON TRACK |
| $660K | 6.6% | moderate | NEEDS ATTENTION |
| $600K | 7.4% | aggressive | AT RISK |
| $450K | 9.8% | aggressive | AT RISK |

Two things to weigh, Paul's ask is that you decide before this goes live:

1. **A 6.2% required return reads as fully ON TRACK**, while the same card
   says *"Safe zone is under 5.0%."* Those two lines sit an inch apart and
   contradict each other. Either the on-track threshold should sit at or
   below the safe-zone number, or the safe-zone line should say what it
   actually means (e.g. the return a conservative mix delivers).
2. **The NEEDS ATTENTION band is very narrow** — roughly 6.3% to 7.2% on
   this profile. Most households will jump straight from green to red, so
   the amber tier almost never appears. If that's intended, fine; if the
   middle tier is meant to catch "fixable with modest changes" cases, the
   band probably needs to be wider.

**Also priority — the Net Outcome Trend series must start equal.** Both
lines are the same household on signup day, so `comparison.series[0].edge`
must equal `series[0].conventional`. Today they don't (Henderson $375K vs
$403K, Mahoney $1.10M vs $1.06M). The dashboard draws the series exactly as
sent, so the fix has to be at the source. Paul's stronger ask: the conventional line should never rise above EDGE.
In today's kept-dollars basis it does for the first few years (Roth taxes
paid up front), and the rows say the same (−$27K next 12 months). Two
honest options: (a) define the comparison on a basis where EDGE leads from
day one (lifetime value rather than dollars-kept-to-date), or (b) keep the
basis and accept the early dip, with the rows matching. Front-end will
follow whichever you pick; please don't leave it to the chart to fake.

**IMPORTANT (Paul, Sept 14) — the Net Outcome Trend must reflect the tier.**
The projection EDGE draws for a member has to be computed from what that
member's tier actually does, not one curve for everyone. Quick Check runs on
numbers the member types in and re-runs only when they do, so its projection
should carry the lower efficacy of a plan that is checked occasionally.
Autopilot re-runs on every market move, rule change, and life event with
linked accounts, so its curve should sit higher. Concierge adds the four
proprietary strategies on top, so higher again. Same household, three
different "EDGE" lines, and the dashboard shows the one for the member's tier.
This is also the honest basis for the upgrade prompts: the gap between the
tiers is the pitch. Front-end needs nothing new for it — `comparison.series`
per tier is enough — but the engine has to produce it.
Standing rule from Paul: whenever EDGE shows behind (a negative window row
or the conventional line above EDGE), the card shows an amber asterisk note
explaining why in one sentence. The current wording assumes the reason is
Roth taxes paid up front — tell us if the engine can name a different cause.

The front-end maps `well_funded`/`conservative` → ON TRACK, `moderate` →
NEEDS ATTENTION, `aggressive`/`extremely_aggressive` → AT RISK, identically
on `quick.html` and `ballpark.html`. Nothing to change on our side once the
thresholds are where you want them.

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

## 1a. Information vs. advice — copy that may need your (or counsel's) call

Paul asked for a read of the tier copy against the advice line. Not a legal
opinion; flagged for you to decide. GH2's footer says it is not a registered
investment adviser, and the tiers are paid, so sentences that pair "for you"
with a specific action are the exposure. Flagged, worst first:

1. Autopilot bullet: "The move EDGE ranked first for you, sized to the
   dollar, with every reason behind it." A specific action, personalised,
   for a fee.
2. "All 80 retirement strategies ranked for your household." (pricing card,
   dashboard rankings card, results page.) "Ranked for you" reads as a
   recommendation.
3. Quick Check: "One EDGE recommendation a year." The word itself.
4. (Resolved Sept 14: the "a person answers" promise was removed from
   Concierge; the only human offer is Concierge+, which links to your site.)
5. Results-page sample reasoning ("Claiming at 67 beats 70 for you", "Moving
   to Florida at 70 would keep about $61,000 more", "Converting more would
   spill into 22%") is directive and about account/withdrawal choices. It's
   placeholder now but it sets the pattern for engine copy.
6. "Projected savings this year ~$10,000" has no hypothetical-results caveat.
7. Comparison table: "Tells you the return your portfolio actually needs",
   "Updates the moment the answer changes" — mild, but "tells you" / "the
   answer" are advice words.

Fine as written: the Quick Check verdict lines, heat-map statuses, dashboard
numbers, "Link your accounts", "Re-run on every market move", price lock,
founders pricing, and the footer disclaimer.

If you want a house style, it is: EDGE models, scores, shows, projects; it
never recommends, tells you, or says you should. Copy swaps are ready
whenever you say (e.g. "The strategy EDGE's model scores highest for your
inputs, with the math behind it"; "modeled and scored for your inputs"; "One
EDGE analysis a year"; a "Projections are hypothetical, not guaranteed, and
not advice" line under the ranked list and the savings panels). Paul chose to
leave the copy as-is until you weigh in.

## 1b. Not functional yet on the member pages — and what each needs (Paul, Sept 14)

Every link below is marked **NOT WIRED YET** in amber on localhost/preview
hosts (nothing shows on edge.gh2benefits.com). Grouped by what it takes:

**Needs an endpoint only (front-end is built, one flag flips it)**
- Sign-in / auth gate → `GET /api/me`; set `AUTH_ENABLED = true` on dashboard + heat map.
- Dashboard numbers → `GET /api/dashboard`; set `LIVE_DATA = true`.
- Heat-map scoring → `GET /api/heatmap` (status per regime); set `LIVE_SCORING = true`.
- Ask EDGE chat → `POST /api/edge-chat`; set `CHAT_CONFIG.connected = true` (dashboard, heat map, and the Quick Check tier's lighter chat).
- Household resolution → the session decides; delete the `?hh=` / `?tier=` review hooks.

**Needs an endpoint AND a page or view**
- "What changed since your last visit" → `GET /api/changes` (events + realized $ kept). Card is built; only data.
- "Your strategies, ranked" + "See all 80 strategies" → `GET /api/strategies` (ranked list, status, $/yr, proprietary flag). Card is built; the full-list page is not.
- Regime cards on the heat map (all seven) + "View details" on Next Key Action → a regime/strategy detail view (trigger, impact, recommended actions). Data is `/api/heatmap` + `/api/strategies`; the page doesn't exist.
- "View calendar" on Next Check-In → a check-in schedule from the engine (`nextCheckin[]` with dates + reasons) and either an in-app calendar view or a downloadable `.ics` per check-in. Simplest v1: the `.ics`.
- "Why did this move?" on the hero → month-over-month attribution from the engine (what changed and by how much). No page yet; could be a modal.
- "View all drivers" → the full "Where the Money Goes" breakdown (all flows, by year). Data + page.
- Jae's Corner (Quick Check tier) → `GET /api/newsletter` + an archive page.

**Needs a product decision before build**
- "My plan (PDF)" (now a "Download PDF" row inside the profile card, Autopilot and Concierge only) → server-side PDF of the projection and scheduled actions. Needs a template and a generator.
- "Update my info" → a member profile form (retired, moved, spouse died, new numbers) that re-runs the plan. Needs the input contract for the full profile.
- "Alerts & settings" → account page: email-on-change preference, billing (Shopify), sign out.
- Concierge "Ask a question" → human-routed question queue (chat first, anomalies to you). Needs the routing rule.
- Concierge "Quarterly review brief" → document delivery; template + schedule.

- "Alerts & settings" is now two icon buttons in the member top bar (bell = Alerts, gear = Settings, label in the tooltip). Both open the profile page's Alerts & settings card for now; what actually lives behind each is yours to decide.

Front-end will keep degrading gracefully on demo data for all of these.

## 1c. Where do dependents go? (Paul, Sept 14)

Nowhere yet. Neither Quick Check, Ballpark, nor the profile form asks about
kids or other dependents, and nothing in the payload carries them. For tax
planning that matters: child tax credit and its phase-outs, head-of-household
filing, dependent-care credit, 529 vs. Roth for the kids, the "kiddie tax" on
UTMA accounts, and later, who is on the household's ACA plan. Paul wants a
place for it. Suggested shape: a "Dependents" row in the profile form (count
plus birth years, optional "still claimed on your return" toggle), with Quick
Check and Ballpark left as they are so the 30-second promise holds. Needs the
engine to accept it first — say what the input contract looks like and the
form is a small addition.

## 1e. Quick Check is the manual tier (Paul, Sept 14, after the PR went up)

Quick Check = the full plan on numbers the member types in themselves: the
dashboard, the profile form with "Save & re-run", Ask EDGE, Jae's Corner, the
PDF. What it does NOT get is anything that needs connectors or automatic work:
linked accounts, re-runs on market moves / rule changes / life events, the
email alert when the answer changes, the change feed, and the heat map's
detail. The ranked strategies are open on every tier (the four Concierge
rows stay blurred below Concierge, as before). Those are Autopilot and up. On Quick Check the heat
map is fully open: every tile shows its colour and its status (AT RISK / NEEDS
ATTENTION / ON TRACK), and the numbers and labels on the tile are blurred, with
one line under the header pointing at Autopilot. The dashboard's attention card
works the same way (dots and colours visible, words blurred). The change feed
keeps its dates and chips and blurs the specifics; nothing is veiled. Tier bullets on pricing.html and results.html say the
same thing. Engine side: Quick Check members will need the profile endpoint
(`POST /api/profile` + a re-run) but never a connector.

## 1d. Small things fixed in the final check (Sept 14 evening)

- Quick Check → Ballpark handoff now carries the 8% contribution assumption,
  so the ballpark form no longer stops on the one field Quick Check never asks.
- Preview sign-in: the fields no longer carry `required`, so a bare click on
  Sign in works the way the note under the button says. Flip `AUTH_ENABLED` and
  the real validation returns.
- The preview tier (Quick Check / Autopilot / Concierge switcher) now rides
  along on the Dashboard, Heat Map, and Profile links via sessionStorage
  `gh2_preview_tier`. Preview only; the session should decide the tier.
- Console shows 401s on the pricing page: that is the Shopify `products.json`
  call on your store, not the site. `favicon.svg` is referenced but not in the
  repo; the live host serves it.

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
- **"Where the Money Goes" by stage (Paul, Sept 14).** Today the four tiles
  are lifetime *retirement* flows for every household, so a 50-year-old sees
  withdrawals and Medicare premiums fifteen years out and nothing about the
  saving they're doing now. Please send `DATA.stage` (`'working'` |
  `'retired'`); the card already retitles and recaptions on it. Proposed
  flow set for `working` (most important first): **Contributions in** (yours
  + employer match, per year and to retirement), **Investment growth to
  retirement** (in), **Taxes in working years** (out), **Retirement income
  at your retirement age** (in: SS + withdrawals, per year). For `retired`
  the current four stand, plus **Pension** (in) when there is one. The
  renderer draws whatever `drivers[]` contains; add `direction` per item.
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
- **`profile.html` is new** (Sept 14): the member's profile, the numbers EDGE
  runs on (editable, mirrors the ballpark inputs), and alerts & settings.
  Reached from the Profile button top-right of the dashboard and heat map,
  and from "Update my info" / "Alerts & settings" in the utility strip.
  Preview writes back to the same sessionStorage record ballpark uses.
  Endpoints: `GET/POST /api/profile` (household inputs + identity, triggers a
  re-run), `POST /api/settings` (email-on-change, newsletter, quarterly
  brief), and a billing URL (Shopify) for the Manage link.
- **`home.html` is gone; the dashboard is the post-login home** (Sept 14).
  Sign-in lands on `dashboard.html`; the results-page tier buttons go to
  `login.html?next=dashboard.html%3Ftier%3DN`. What moved onto the dashboard:
  a utility strip under the top bar (plan PDF, update my info, alerts &
  settings, Concierge extras or the upgrade link), a "What changed since
  your last visit" card with the realized-savings ledger, and "Your
  strategies, ranked". Quick Check (tier 1) members see their annual
  recommendation + Jae's Corner up top and the rest of the page as blurred,
  locked previews with a See-plans hook. Tier comes from `?tier=` for review
  only (the PREVIEW bar); in production the session sets it and that bar
  should be deleted. Endpoints these need: `GET /api/strategies` (ranked
  list), `GET /api/changes` (feed + realized savings), `GET /api/newsletter`
  (tier 1 archive). All three are sample data until then.
- **Three consumer tiers, not four (Paul, Sept 14).** Quick Check, Autopilot,
  Concierge are the tiles on the results page and the pricing page.
  Enterprise is no longer a fourth tile anywhere consumer-facing; it stays as
  the separate "For advisors, plan sponsors, and enterprises" block at the
  bottom of pricing with the contact link. Paul is hoping you're on board.
- **Concierge no longer promises a human** (Paul, Sept 14: "the human part
  will never scale"). "Ask a question any time — a person answers" is gone
  from pricing, results, and the dashboard. Concierge is now: everything in
  Autopilot, the 4 proprietary strategies run on the member's plan, and the
  quarterly review brief. The human lives only in Concierge+ (the dedicated
  advisor add-on, priced separately). The "why they reach further" line
  says the four work across regimes at once (bracket, IRMAA line, RMDs in
  one move) — please correct that if it misdescribes them; the Concierge
  plan-run delta (above) is what would let the page show the actual lift.
- **Tier 2 is now called "Autopilot"** (Paul, Sept 14; was Full Plan, briefly
  Autopilot) — the plan re-runs itself on every market move, rule change, and
  life event. Icon: two arrows cycling round a centre. Shopify product
  titles will need to follow.
- **The personal-advisor upgrade is promoted inside the Concierge card**
  on pricing and results (gold inset with its own link to your site), so
  someone who wants that can go straight there. Paul's calls (Sept 14): no
  price shown yet, and no name — most visitors won't know who Jae is, so it
  sells the role, not the person. Working label is "Concierge+" (Paul, Sept 14), shown as a
  faint gold panel just above the Get Concierge button on both results.html and
  pricing.html: "Concierge+ — Add a dedicated, full-time advisor to your plan.
  See details →". Rename if you have a better one. `ADVISOR_URL` in
  pricing.html is the link to set, and the price goes back in when you set it.
- **Pricing page theme switcher removed.** The Trust / Warm / Fresh / Calm
  light themes and the `?design=1` bar are gone; the dark EDGE look is the
  page's only theme and its tokens now live in `:root`. If you want the light
  variants back they're in git history (`pre-port-2026-09-14` tag).
- **Concierge should show better numbers than Autopilot (Paul, Sept 14).**
  Today the dashboard draws one plan run per household regardless of tier,
  so Concierge and Autopilot show identical projections; only what's
  unlocked differs. If the four proprietary strategies are applied to a
  Concierge member's plan, the engine should return a second run (or a
  delta) — e.g. `comparison.concierge` with its own `outcomes`/`endGap` —
  so the Concierge view shows the higher numbers. On Autopilot the same
  delta becomes the upsell: a locked, blurred line "with GH2 proprietary
  strategies: +$X lifetime" next to the Lifetime row. Front-end will render
  it as soon as the field exists; nothing is faked meanwhile.
- **`results.html` is the new Ballpark results page** (Sept 14). Ballpark
  stores the engine response in `sessionStorage.gh2_ballpark_last_result`
  and redirects there after the calculating card. Everything the engine
  returns is used: `feasibilityRating` → tier, `verdict.feasible/showReturn`,
  `requiredIRR`, `targetEndAge`, `depletionAge`, `projectedRetirementBalance`,
  and the whole `precision` block (pct, `halfWidthPp`, `sharpen[]` → the
  "data points that sharpen this" list). Marked `// placeholder` in the page
  script and illustrative until you have endpoints for them: the top-ranked
  move and its dollar amount (the blurred figure), the five reasons, the
  "also considered" line, the regime cards, and the receipt counts (80 / 7 /
  437). The old in-page results section in `ballpark.html` is dormant, not
  deleted, until this settles.
- **Two AT RISK outputs on Quick Check and Ballpark.** The verdict card shows
  a different body line when `verdict.feasible === false` ("No realistic
  market return covers this…") versus a normal at-risk result with a number
  ("Your savings need to earn more than markets reliably deliver…"). We
  verified the engine sets that flag correctly on live calls (retire-today
  → false with the solver-floor IRR; 55→67 with $450K → true at 9.8%). Paul
  wants you aware that the copy depends on `feasible` and `showReturn`
  staying authoritative; nothing to build, just don't repurpose those flags.
- **Household switcher decision.** Paul's call: hide it from members (done).
  Delete the dev hook entirely, or keep `?dev=1` for your testing — your
  preference.

*Questions → Paul.*
