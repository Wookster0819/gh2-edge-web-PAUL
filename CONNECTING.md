# Connecting the Subscriber Pages — Integration Guide (for Jae)

`dashboard.html` and `heatmap.html` are self-contained subscriber pages that
render **demo data** today. Every number, status, and chat reply is meant to
come from the EDGE engine — no scoring or calculation logic lives in this
public repo. Each integration point below is behind a single flag; flip it
when the endpoint is live and the page switches over (with demo fallback on
fetch failure, so pages never render empty).

All endpoints are expected same-origin (like `/api/optimize-quick`), cookie-
authenticated, JSON. Search each file for the flag name to find the exact
spot.

---

## 1. Subscriber gate — `/api/me`

| | |
|---|---|
| Flag | `AUTH_ENABLED` (in both `dashboard.html` and `heatmap.html`) |
| Endpoint | `GET /api/me` |
| Behavior | `401`/`403` → redirect to `/login.html?next=<page>`; anything else passes |

Both pages are `noindex,nofollow`.

## 2. Dashboard data — `/api/dashboard`

| | |
|---|---|
| Flag | `LIVE_DATA` in `dashboard.html` |
| Endpoint | `GET /api/dashboard` |
| Response | Exactly the shape of `DEMO_DATA` in `dashboard.html` |

Shape summary:

```jsonc
{
  "projectedLiquidNetWorth": 1247000,
  "throughAge": 87,
  "deltaSinceLastMonth": 34000,        // signed
  "deltaSinceSignup": -12000,          // signed
  "trajectory": [ { "year": 2026, "value": 890000 }, ... ],
  "hot": [ { "level": "hot|warn|ok", "title": "...", "sub": "...", "href": "..." } ],
  "nextAction":  { "title": "...", "sub": "...", "href": "..." },
  "nextCheckin": { "title": "...", "sub": "...", "href": "..." },
  "outcomes": [ { "label": "1 Year", "value": 34000 }, ... ],
  "drivers":  [ { "name": "Taxes", "amount": 440000, "share": 0.56 }, ... ]
}
```

## 3. Heat map scoring — `/api/heatmap`

| | |
|---|---|
| Flag | `LIVE_SCORING` in `heatmap.html` |
| Endpoint | `GET /api/heatmap` |
| Response | `{ "regimes": [ ... ] }` — engine-scored, one entry per regime |

```jsonc
{
  "regimes": [
    {
      "n": 1,                          // card number
      "name": "Roth Conversion Window",
      "icon": "calendar",              // calendar|shield|runner|heart|bank|people|lock
      "status": "risk",                // risk|attention|ontrack|na  ← ENGINE SCORES THIS
      "value": "47",                   // big display value (string)
      "unit": "Days\nremaining",       // \n = line break
      "unitBelow": false,              // true → unit renders under the value
      "note": "Action by Dec 31",
      "row": "top",                    // top (3 large) | bottom (4 small)
      "href": "#"                      // regime detail link (future)
    }
  ]
}
```

The pill label (RISK / ATTENTION / ON TRACK / N/A) and all colors are derived
from `status` on the page — the API sends status only, never display text.

## 4. Ask Edge chat — `/api/edge-chat`

| | |
|---|---|
| Flag | `CHAT_CONFIG.connected` (in both pages) |
| Endpoint | `POST /api/edge-chat` |

Request the page sends:

```jsonc
{
  "messages": [                        // full running transcript, oldest first
    { "role": "user", "content": "What should I do before Dec 31?" },
    { "role": "assistant", "content": "..." }
  ],
  "context": {
    "page": "dashboard",               // dashboard | heatmap | ballpark
    "access": "subscriber"             // subscriber | public — see below
  }
}
```

**Capability tiers:** the chat's capability varies by subscription level,
decided entirely server-side. `context.access` tells you which surface the
request came from: `ballpark.html` (the only non-subscriber surface — chat
added on the `PAUL/web-edits` branch / PR #1) sends `access: "public"`;
`dashboard.html` and `heatmap.html` send `access: "subscriber"`. Combine
with the authenticated user's actual tier on your side to set the reply
capability — the front-end needs no changes as tiers evolve.

Expected response: `{ "reply": "..." }` (plain text; the page renders it as
one assistant bubble). Non-200 or network failure shows a friendly retry
message — no special error shape needed.

Backend sketch (provider-agnostic on the page, so implement however you
like). If using the Claude API: default to the latest model (e.g.
`claude-sonnet-5`), pass `messages` through as-is, and put plan grounding in
the system prompt, e.g. system = subscriber's current engine outputs
(trajectory, regime statuses, next action) + guardrails (educational, not
individualized investment advice; point to a human/advisor where
appropriate). The `context.page` field tells you what the user is looking at.

While `connected: false`, the widget answers with a canned preview reply so
the UX is demoable end-to-end (typing state, mascot pose swap, history).

## 5. Mascot assets

- `assets/edge-wave.png` — idle/waving pose (transparent). Shown in the Ask
  Edge button + chat header.
- `assets/edge-abacus.png` — "thinking" pose (transparent). The chat header
  swaps to this while a reply is pending, then back to the wave. This is
  wired to the real request lifecycle — nothing to do on connect.

Both pages auto-upgrade from an inline SVG stand-in to these images when the
files are present, so the images are required in deploys (they're committed
in `assets/`).

---

*Questions → Paul.*
