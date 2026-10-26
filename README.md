<p align="center">
  <img src="assets/cover.png" alt="Know Your Flow — Find. Learn. Act." width="100%">
</p>

# Know Your Flow

**Find. Learn. Act.** A privacy-first menstrual equity app for teens.

Built for the **Congressional App Challenge** by **Tia Gupta** and **Sera Patel**.

---

## The problem

Roughly **1 in 4 teens** has struggled to afford period products. Whether you can get
them free at school depends entirely on your zip code: **20 states and D.C.** require
schools to stock them, but only **10 states** actually fund the mandate they passed.
**18 states** still charge sales tax on period products.

Most people affected by these laws have no idea they exist.

## What the app does

| Pillar | What it delivers |
|---|---|
| **Find** | Interactive Google Maps of free product locations — schools, shelters, food banks, community centers, and H.E.R. Essentials partner sites. Search, filter, distance-sort, get directions, and submit new locations. |
| **Learn** | *CycleIQ* — 11 lessons across menstrual health, reproductive health, products, policy literacy, and myth-busting, with 6 quizzes and progress tracking. |
| **Act** | *Policy Pulse* — look up any state's actual record on school mandates, funding, and the tampon tax. Read federal bills, then contact your representatives with an editable message. |
| **Track Cycle** | Private cycle logging with symptom tracking and prediction — stored only on your device. |
| **Community** | Local advocacy wins, partner organizations, and a private notepad for drafting testimony. |

---

## Running it

Plain HTML/CSS/JavaScript. **No build step, no npm install, no dependencies.**

**Serve it locally** (recommended — geolocation requires http://, not file://):

```bash
cd know-your-flow
python3 -m http.server 8000
```

Open <http://localhost:8000>.

**Or just open `index.html`** — everything works except "sort by distance," which
browsers block on `file://`.

**Deploy it:** drag the folder onto [Netlify Drop](https://app.netlify.com/drop), or
push to GitHub and turn on Pages. It's fully static — no build configuration.

---

## Architecture

```
index.html                 App shell: header, tab bar, FAB, mount point
css/
  theme.css                Design tokens (color, type, spacing, radius, shadow) + dark mode
  base.css                 Reset, typography, icon sizing
  layout.css               App frame, header, tab bar, FAB, sheets
  components.css           Buttons, cards, chips, badges, fields, states
  screens.css              Per-screen styles
js/
  core/
    icons.js               Inline SVG icon set (no icon font, no CDN)
    dom.js                 Element builder, distance math, date helpers, toasts
    cache.js               TTL cache with stale-fallback (read-through)
    store.js               localStorage state + privacy controls
    cycle-engine.js        Cycle prediction (pure functions over local history)
    api.js                 API abstraction layer — the seam for a real backend
    ui.js                  Loading / empty / error states, sheets, renderAsync
    router.js              Hash router with :param patterns
    shell.js               Header, 5-tab nav, FAB, theme, privacy sheet
  data/                    Seed datasets (locations, learn, quizzes, policies, community)
  screens/                 One module per screen
  app.js                   Route table + bootstrap
```

### Key decisions

**API abstraction layer.** Every screen reads data through `js/core/api.js` and nothing
else. Today each method resolves bundled seed data behind a real async boundary; the
adapter has a `mode` flag and a working `fetch` transport. Pointing the app at a live
backend means editing that one file — no screen changes.

**Read-through caching.** All API reads pass through `Cache.through()`, which serves
fresh data from `localStorage`, falls back to a **stale** copy if a loader fails, and
only then surfaces a retryable error. The app stays usable offline.

**Every async read has three states.** `UI.renderAsync()` renders a skeleton, then
either content or a typed error with a retry button. There is no path that leaves a
blank screen — a screen that throws synchronously is caught by the router too.

**Google Maps embeds, not a map SDK.** The Find tab embeds `google.com/maps?…&output=embed`
in an iframe. Users pan and zoom a real Google map *inside* the app, then hand off to
turn-by-turn navigation on tap. No API key, no SDK, no billing account.

**Privacy is architectural, not a policy page.** There is no account, no server, and no
analytics. Cycle logs, saved locations, quiz scores, and notes live only in this
browser's `localStorage`. `Store.eraseAll()` is reachable in two taps from any screen
via the shield icon, and data is exportable as JSON.

**Community notes are local by design.** A shared public feed for teens would need
moderation infrastructure this app doesn't have. Rather than ship an unmoderated feed,
notes are explicitly a private on-device notepad, and the UI says so.

---

## Accessibility

Verified programmatically across 11 routes in both themes:

- **Contrast:** 0 WCAG AA failures. The brand lavender `#8B7CF6` only reaches 3.3:1
  against white, so `--primary` is a darkened `#6D51E0` for anything carrying white
  text; the bright brand hue is reserved for icons, tints, and borders.
- **Names:** every interactive control has an accessible name; every input has a label.
- **Targets:** 44×44 minimum, except calendar cells (41px — seven columns can't reach
  44px on a 375px screen; still clears WCAG 2.2 AA's 24px minimum).
- **Structure:** every screen has a top-level heading; iframes are titled.
- Visible focus rings, `prefers-reduced-motion` honored, `color-scheme` follows the
  in-app theme so native controls match.

---

## Data accuracy

**Policy data** (`js/data/policies.js`) covers all 50 states + D.C. with school mandate,
funding, and tax status. Every policy screen links to the official legislature, because
statutes change.

> **Known gap:** widely-cited advocacy figures put the number of jurisdictions with a
> school menstrual-product mandate at ~27 states + D.C. The per-state records in this
> dataset currently mark **20 states + D.C.** All counts shown in the app —
> headline, stat tiles, category breakdown — are computed from those records, so the
> app is internally consistent. To close the gap, set `requiresInSchools: true` on the
> additional states and every number updates automatically.

**Location data** is realistic sample data for the DC/Arlington/Alexandria area,
standing in for a live feed. Swap `js/data/locations.js` (or flip the API adapter to
`remote`) to go live.

**Learn content** is general health education, not medical advice — stated in-app on
every lesson.

---

## Verification

```bash
# Syntax-check every file (requires Node; not needed to run the app)
for f in js/core/*.js js/data/*.js js/screens/*.js js/app.js; do node --check "$f"; done
```

Manually verified in-browser: all 19 routes render, 404 handled, quiz scoring and
persistence, form validation (empty + malformed + valid), cycle prediction math,
API error paths, cache TTL expiry, light/dark themes, and contrast in both.

---

## Credits

Created by **Tia Gupta** and **Sera Patel**.

Organizations referenced: [H.E.R. Essentials](https://heressentials.org),
[PERIOD.](https://period.org),
[National Diaper Bank Network](https://nationaldiaperbanknetwork.org),
[Alliance for Period Supplies](https://allianceforperiodsupplies.org).
