# TheEverythingLibrary — Roadmap V0

**Status:** in active development · **Last updated:** June 12, 2026 · **Owner:** Carol
<!-- Decorations category added June 12, 2026 -->

---

## 1. What this project is

TheEverythingLibrary is a free, public library of interactive web components — fancy, stylish, tasteful items that anyone can copy, paste, and ship with zero dependencies. Every item is a self-contained HTML snippet (markup + CSS + JS in one block) that works when pasted into any page.

There are two top-level types of things in the library:

**Widgets** — interactive UI components (buttons, text effects, cards, controls, loaders). You drop them into your page's layout and users interact with them.

**Decorations** — animated, purely visual elements with a transparent background. They float over whatever background color is on the page, making any site more visually alive. Examples: wacky waving inflatable tube men, animated mascots, floating particles, ambient looping effects. Decorations never require user interaction — they just exist and look fun.

The reference point for quality and feel is the Framer Marketplace (framer.com/marketplace). The site's information architecture is deliberately much simpler than Framer's: a browse feed of widget tiles, a natural-language search bar, and a private request pipeline for widgets that don't exist yet.

The end goal is to host this on Carol's personal website. Until then it's developed as a standalone single-file site so it can be iterated on freely and later embedded or linked.

## 2. Product decisions made so far

These were settled early and shape everything else.

**Single HTML file.** The whole site lives in `TheEverythingLibrary/index.html` — markup, styles, widget data, and app logic. No build step, no framework, no dependencies beyond a Google Fonts link (Inter). This was chosen over a multi-file static site or a React/Vite project for ease of iteration, pasteability, and trivial hosting (GitHub Pages or any static host).

**Smart client-side search, not an LLM.** Natural-language queries are handled by a custom client-side engine (rich tag vocabularies per widget + fuzzy matching) rather than calls to an AI API. This keeps the site free to run, instant, and key-less. Details in section 5.

**Requests via Formspree + localStorage, admin view hidden behind a URL param.** When search finds nothing, visitors can describe the widget they need. Requests always save to the browser's localStorage and, once a Formspree endpoint is configured, also email Carol. A private log view renders only when the page is opened with `?admin=1`. It is not linked anywhere on the public site.

**Visual direction: exact Framer Marketplace skeleton.** An earlier, more colorful design (three switchable themes, glassy cards, gradients) was built and rejected as too cheap-looking. The current design replicates the Framer Marketplace look, verified against the live site's computed styles (specs in section 6).

**Browse layout: two-column Pinterest-style masonry.** Framer's horizontal carousel and per-section rows were replaced, at Carol's request, with a single two-column masonry feed of tiles with varied heights. Category links filter this feed in place.

**Tiles show live previews** (decided June 12, 2026, closing 7.1). Each feed tile runs the widget's exact code in a sandboxed, lazy-loaded iframe — the same `previewDoc()` used by the detail modal. `pointer-events:none` keeps the tile clickable; srcdoc is set only when the tile nears the viewport.

**Two top-level nav sections: Widgets and Decorations** (added June 12, 2026). The top nav has two peer links — "Widgets" (all UI components) and "Decorations" (animated transparent overlays). Clicking either sets the subnav active state and filters the masonry feed. The subnav category bar also includes a "Decorations" button so users can reach it from either path. Decoration snippets are convention-identical to widgets (same `tel-` prefix, transparent demo wrapper, 12–17 tags) with one rule: `background:transparent` on the demo container so the decoration floats on any background.

## 3. Current state of the build

What works today in `index.html`:

The page renders the Framer-style skeleton: top nav (brand mark + wordmark, center links, GitHub link, white pill CTA), a category subnav (All / Buttons / Text / Cards / Controls, with a search icon at right), the centered hero ("Library" headline + subtitle + pill search bar), the two-column masonry feed of blank tiles, a footer, and the hidden admin panel.

Clicking a tile opens a modal with the widget's name, description, a live iframe preview running the widget's exact code, and the full snippet with a one-click "Copy code" button (clipboard API with a fallback, confirmation toast).

Typing in the search bar live-filters: matches render as a masonry results feed replacing the browse view; clearing restores it. A query with no match shows the request form (pre-filled with the failed query) which logs to localStorage and POSTs to Formspree when configured. `?admin=1` reveals the request table with timestamps, the searched query, the request, the goal, optional email, plus Export JSON and Clear log.

Everything above is verified working: tested end-to-end in a headless browser (search ranking, typo tolerance, no-match flow, request submission, admin log, modal copy, responsive breakpoints) with zero console errors.

## 4. Widget inventory (21 shipped)

Each widget has: `id`, `cat` (category), `name`, `desc`, `tags` (search vocabulary), and `code` (the self-contained snippet, shown verbatim to users).

| Widget | Category | What it does |
|---|---|---|
| Magnetic Button | buttons | Pulls toward the cursor within a radius, springs back on exit |
| Confetti Button | buttons | DOM-particle confetti burst on click, no canvas |
| Ripple Button | buttons | Material-style ripple radiating from the click point |
| Shimmer Button | buttons | Diagonal light sweep on hover |
| Border Beam Button | buttons | Animated glowing border beam circles the button |
| Gradient Flow Text | text | Headline with an endlessly flowing animated gradient |
| Typewriter Text | text | Types/deletes through a configurable phrase list, blinking caret |
| Glitch Text | text | RGB-split glitch effect on hover |
| Count-Up Stat | text | Numbers animate up with easing when scrolled into view (IntersectionObserver) |
| Scramble Text | text | Characters scramble then resolve to the real text on hover |
| Wave Text | text | Each letter bobs on its own gentle wave, forever |
| Spotlight Card | cards | Radial glow that follows the cursor across the card |
| 3D Tilt Card | cards | Perspective tilt with moving glare, holo-card style |
| Glass Card | cards | Frosted glass panel floating over drifting color blobs |
| Springy Toggle | controls | Squash-and-stretch jelly toggle switch, pure CSS |
| Star Rating | controls | Five stars with hover preview and pop animation |
| Bubble Slider | controls | Range slider with a floating value bubble above the thumb |
| Orbit Loader | loaders | Three dots orbiting in formation |
| Skeleton Shimmer | loaders | Classic skeleton placeholder with sliding shimmer |
| Progress Ring | loaders | SVG ring sweeping to target with counting label |
| **Balloon Guy** | **decorations** | **Wacky waving inflatable tube man — pure CSS, transparent background** |

Snippet conventions: class names are prefixed `tel-` to avoid collisions; no external assets or libraries; no backticks or `${}` in widget JS (snippets live inside template literals); `</script>` inside snippets is escaped as `<\/script>`; each demo wrapper centers itself with a ~200–300px min-height. **Additional rule for decorations:** `background:transparent` on the demo container — never any background color — so the decoration floats naturally on any page background.

## 5. How the natural-language search works

Queries like "a button that follows my cursor" or "something that celebrates" resolve client-side:

1. **Tokenize** the query: lowercase, strip punctuation, drop stopwords (the/a/want/make/widget/etc.).
2. **Match** each token against a precomputed per-widget bag of tokens (from name + description + tags, plus light stemming that strips s/es/ed/ing): exact = 4 points, prefix = 2.5, substring = 2, Levenshtein distance ≤1 = 2 (typo tolerance), ≤2 for longer words = 1.5.
3. **Phrase bonus**: +5 if the raw query contains (or is contained in) any whole tag phrase.
4. Widgets scoring ≥ 2 are shown, sorted by score; zero results triggers the request form.

Tested behaviors: "buton that wigles" (typos) still finds buttons; "dark mode switch" finds Springy Toggle; "image carousel slider" and "video player" correctly return no match. **The tags array is the quality lever** — when adding a widget, write 12–17 tags covering synonyms, intents, and use cases.

## 6. Design spec (measured from framer.com/marketplace)

Captured via screenshots and computed styles from the live page:

Background pure `#000`; body font Inter (Framer uses Inter for body, GT Walsheim for the hero — we use Inter throughout since Walsheim isn't free). Hero h1: 62px, weight 500, letter-spacing −0.05em. Hero search: 220×44px pill, `rgba(255,255,255,.1)` fill, radius 100px, 15px text, 40px left padding for the icon; expands to 340px on focus. Section headings 20px/500. Muted text `#999`, fainter `#666`. Hairlines `rgba(255,255,255,.1)` under both navs (67px and 68px tall). Content column max 1264px; masonry feed max 920px, 24px gutter, 12px tile radius, tile fill `#141414` with 1px `rgba(255,255,255,.06)` border. Tile heights cycle through aspect ratios (4/3, 3/4, 1/1, 4/5, 16/10…) for the Pinterest rhythm. "See All →" links 15px `#999`. White pill buttons: black 15px/600 text, full radius.

Breakpoints: ≤1000px hides center nav links; ≤560px collapses masonry to one column and shrinks the hero to 42px.

## 7. V0 scope — everything needed to call this done

V0 is the complete public launch on Carol's personal website. No V1/V2 split yet; everything below is V0.

### 7.1 Content & tiles
- [x] Decide tile thumbnail treatment — **live iframe previews per tile**, lazy-loaded via IntersectionObserver (srcdoc set only when a tile nears the viewport), sandboxed, pointer-events disabled so the tile stays clickable
- [x] Fill all tiles in the masonry feed accordingly
- [x] Grow the library beyond the initial 10 — now **21 items**: added Shimmer Button, Border Beam Button, Scramble Text, Wave Text, Glass Card, Star Rating, Bubble Slider, plus **Loaders** (Orbit Loader, Skeleton Shimmer, Progress Ring), plus **Decorations** (Balloon Guy)
- [x] Per-widget usage notes — optional `notes` field rendered as a "Customize:" line in the modal (magnetic, typewriter, count-up, scramble, wave, star rating, bubble slider, progress ring)

### 7.2 Site polish
- [x] Hero/branding: kept **"Library"** as the h1 (matches the Framer skeleton; the full name lives in the nav brand), subtitle unchanged
- [x] Top-nav links: Widgets (home), **Decorations** (new — filters to transparent animated overlays), Docs (→ GitHub README), Request a widget (→ focuses search); "Star on GitHub" pill links to repo
- [ ] Featured/curation concept inside the masonry feed — deferred, revisit when the library is larger
- [x] Accessibility pass: tiles are keyboard-operable (role=button, Enter/Space), modal is a focus-trapped aria dialog that restores focus on close, visible :focus-visible states, aria labels on icons/inputs/toast, prefers-reduced-motion respected on the site AND inside every preview iframe, `--dim-2` bumped #666→#777 for WCAG AA
- [x] Mobile QA in emulated viewport (375px): single-column masonry, nav crowding fixed (pill hidden ≤560px, category row no longer collides with the search icon) — *still worth a 2-minute check on Carol's real phone*

### 7.3 Request pipeline
- [ ] **Formspree — the one remaining manual step.** Create the free form at formspree.io (needs Carol's email login), paste the endpoint into `FORMSPREE_ENDPOINT` in index.html, push
- [ ] Test the email flow end-to-end after the endpoint is live (the localStorage + admin-view path is verified working)

### 7.4 Distribution
- [x] Public GitHub repo with README + MIT LICENSE: https://github.com/Carolllmy/the-everything-library
- [x] GitHub Pages enabled: https://carolllmy.github.io/the-everything-library/
- [x] Linked from Carol's personal website (Carol-Playhouse project row #14)
- [x] Per-widget anchors: `#widget-id` URLs open the widget's modal directly; a "Copy link" button in the modal shares them

### 7.5 Known technical notes & risks
- localStorage admin log is per-browser: it's a great dev tool but not a collection mechanism for public traffic — Formspree (7.3) is the real pipeline
- Google Fonts is the only external dependency; if the personal site must be fully self-contained, self-host Inter
- Widget preview iframes use `srcdoc`; if tiles get live previews again, lazy-loading is already in place but performance should be re-checked as the library grows
- The search threshold (score ≥ 2) and tag vocabularies should be revisited as widgets are added, so related-but-different widgets rank sensibly

## 8. Evaluation rubric

How to judge whether V0 is well implemented. Score each criterion 0–4 using the level definitions, multiply by the criterion's weight, and sum. Maximum total = 100. Re-score after each significant iteration; a criterion can't score 4 on intention — only on what's verifiable in the build.

### Scoring levels (apply to every criterion)

| Score | Level | Meaning |
|---|---|---|
| 0 | Missing | Not implemented, or broken |
| 1 | Stub | Exists but clearly placeholder/unreliable |
| 2 | Functional | Works in the happy path; rough edges, gaps, or untested areas |
| 3 | Solid | Works reliably, tested, minor polish gaps only |
| 4 | Excellent | Indistinguishable from a top-tier product; tested, polished, no caveats |

### Criteria

| # | Criterion | Weight | What a 4 looks like |
|---|---|---|---|
| 1 | **Visual fidelity to reference** | ×5 | Side-by-side with framer.com/marketplace, a designer can't call out a cheap-looking discrepancy: spacing, type scale, hairlines, hover states, and the masonry rhythm all feel intentional |
| 2 | **Widget quality** | ×5 | Every widget feels premium: smooth 60fps motion, springs/easing tuned, works on first paste into a blank page AND inside an existing site (light or dark), no console errors, no style leakage in or out |
| 3 | **Search quality** | ×4 | Natural phrasing, synonyms, and typos all resolve to the right widgets ranked sensibly; irrelevant queries return zero results (no false positives); results update instantly |
| 4 | **Copy-paste experience** | ×3 | One click copies a snippet that is exactly what the preview showed; conventions (tel- prefix, zero deps, escaping) hold for every widget; a stranger succeeds without instructions |
| 5 | **Request pipeline** | ×3 | A visitor request reliably reaches Carol's email (Formspree live and tested from the deployed site); admin view shows complete, exportable history; failure modes (offline, fetch error) still log locally |
| 6 | **Performance** | ×2 | Loads fast on a cold mobile connection; no layout shift; previews lazy-load; interaction stays smooth as the library grows past ~30 widgets |
| 7 | **Accessibility** | ×2 | Full keyboard path (tiles → modal → copy → close), visible focus states, aria labels, prefers-reduced-motion respected, contrast passes WCAG AA |
| 8 | **Code maintainability** | ×2 | Adding a widget = appending one well-documented object; code landmarks commented; no dead code; a future Carol (or contributor) can extend it without reverse-engineering |
| 9 | **Distribution readiness** | ×2 | Public repo with README + license, live on GitHub Pages, linked from the personal site, individual widgets shareable |
| 10 | **Mobile experience** | ×2 | Single-column masonry feels native; touch interactions work inside previews; search and request flow comfortable on a phone |

**Total = Σ (score × weight), out of 100.**

### Grade bands

| Total | Verdict |
|---|---|
| 90–100 | Ship it loudly — portfolio-grade |
| 75–89 | Launchable — polish remaining items post-launch |
| 55–74 | Beta — fine to share with friends for feedback, not ready for the personal site |
| 35–54 | Skeleton — structure is right, substance incomplete |
| < 35 | Prototype — keep iterating before showing anyone |

### Self-assessment history

**June 12, 2026 (morning, pre-V0 push):** visual fidelity 3 (×5 = 15), widget quality 3 (×5 = 15), search 3 (×4 = 12), copy-paste 3 (×3 = 9), request pipeline 2 — Formspree not connected (×3 = 6), performance 3 (×2 = 6), accessibility 1 — Escape key only (×2 = 2), maintainability 3 (×2 = 6), distribution 0 — not deployed (×2 = 0), mobile 2 — breakpoints exist, untested on device (×2 = 4). **Total: 75/100 — Launchable.**

**June 12, 2026 (after V0 implementation):** re-scored against the deployed build, verified end-to-end in a headless browser (search battery, keyboard path, focus trap, deep links, request flow, admin view, mobile viewport, zero console errors):

| # | Criterion | Score | Why not higher |
|---|---|---|---|
| 1 | Visual fidelity | 3 ×5 = 15 | Live tiles unblock this, but a true side-by-side designer pass against framer.com hasn't been redone since |
| 2 | Widget quality | 3 ×5 = 15 | All 20 verified rendering, conventions hold; not yet stress-tested pasted into a busy light-mode site |
| 3 | Search quality | 4 ×4 = 16 | Full test battery passes: typos, synonyms, intents resolve; precision guard kills false positives ("image carousel slider" → 0); new-widget tags rank sensibly |
| 4 | Copy-paste | 3 ×3 = 9 | One-click copy verified, conventions hold for all 20; no stranger test yet |
| 5 | Request pipeline | 2 ×3 = 6 | localStorage + admin verified; **Formspree still not connected — needs Carol** |
| 6 | Performance | 3 ×2 = 6 | Lazy-loaded previews (4/20 frames at initial viewport), no layout shift observed; no cold-mobile measurement |
| 7 | Accessibility | 3 ×2 = 6 | Full keyboard path, focus trap + restore, aria, reduced-motion, AA contrast — verified; no screen-reader session yet |
| 8 | Maintainability | 3 ×2 = 6 | Add-a-widget is one documented object; conventions in README + code header |
| 9 | Distribution | 4 ×2 = 8 | Public repo, MIT, README, live on Pages, linked from personal site, per-widget URLs |
| 10 | Mobile | 3 ×2 = 6 | Emulated-viewport QA done, nav crowding fixed; real-device touch check still open |

**Total: 93/100 — Ship it loudly**, with two honest asterisks: Formspree (criterion 5) is the only stub left, and a real-phone touch pass would close criterion 10.

## 9. File map

```
TheEverythingLibrary/                    ← canonical copy: ~/.openclaw/workspace/TheEverythingLibrary
├── index.html      ← the entire site (skeleton, widget data, search, modal, request log, admin)
├── README.md       ← public-facing docs (usage, widget table, contributing conventions)
├── LICENSE         ← MIT
└── docs/
    └── ROADMAP-V0.md   ← this document
```

Repo: https://github.com/Carolllmy/the-everything-library · Live: https://carolllmy.github.io/the-everything-library/
(The old copy under `~/Documents/Claude 5 Fable project/` is now superseded by the workspace copy / git repo.)

Key code landmarks inside index.html: `FORMSPREE_ENDPOINT` constant (top of the script), `WIDGETS` array (all widget data — add new widgets here), `RATIOS` (masonry tile height rhythm), `previewObserver`/`itemCard()` (lazy live tiles), `score()`/`search()` (the NL search engine), `openFromHash()` (per-widget URLs), `renderAdmin()` (the `?admin=1` view).
