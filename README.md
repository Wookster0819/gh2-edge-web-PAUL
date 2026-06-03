# GH2 EDGE — Public Web Front-End

Collaborator workspace for the **public** consumer pages only:

- `quick.html` — 5-input quick planner
- `ballpark.html` — full ballpark planner
- `pricing.html` — pricing / tiers page

These are self-contained (inline CSS + JS). They call the **public** API at
`/api/optimize-quick` — no engine source, no internal tools, no strategy. The
GH2 EDGE engine, research scripts, patents, and strategy live in a **separate
private repository** and are intentionally not here.

---

## Run it locally

Any static server works:

```bash
npx serve .
# then open http://localhost:3000/quick.html
```

- **UI / layout / copy** work fully offline this way.
- **To test the live calculation** (the form's "Check my plan" result), the page
  needs the API. Either test on the deployed site, or temporarily point the
  `fetch('/api/optimize-quick', …)` call at the absolute URL
  `https://edge.gh2benefits.com/api/optimize-quick` (the public API is CORS-open,
  so it works from `localhost`). Revert that before committing.

---

## Workflow — READ THIS

**Every branch you create must start with `PAUL/`.**

```bash
git clone <this-repo-url>
cd gh2-edge-web-PAUL

# 1. Make a PAUL/ branch off main — name it for the work:
git checkout -b PAUL/fix-married-input

# 2. Edit, commit, push YOUR branch:
git add -A
git commit -m "Fix: married selection on quick.html"
git push -u origin PAUL/fix-married-input

# 3. Open a Pull Request into `main` and assign it to Jae.
```

**Do NOT push to or merge `main` directly.** Jae reviews every `PAUL/` PR and
merges it. Pushing to `main` is what triggers the live deploy, so that stays
with Jae.

Branch examples: `PAUL/pricing-copy`, `PAUL/ballpark-mobile`, `PAUL/quick-a11y`.

---

*Curated slice of the GH2 EDGE site. © GH2 Benefits LLC.*
