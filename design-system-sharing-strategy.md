# Design System Sharing Strategy — Public Site ↔ Backend/Admin App

Status: **proposal**, prepared before the new backend repository exists. Written from within
`VeloWetzikon_Contao` before this repo existed, then moved here on 2026-08-19 once it did (it
documents this repo's own reason for existing, so it belongs here, not in the repo it was
written from) — see this repo's `CHANGELOG.md`. Because of that history, **"this repository"
below still means `VeloWetzikon_Contao`** (the public site), not the repo you're reading this
file in now. The proposed name `velo-wetzikon-design-tokens` also didn't stick — the repo was
actually created and later renamed to `velofreundliches-wetzikon-design-system`, and its real
layout also includes a `design-system/` folder of human-readable HTML docs (`tokens.html`,
`contao-implementation.html`, `brand-voice.html`) beyond just `tokens.css`/`tokens.json`/
`README.md`/`CHANGELOG.md`. The reasoning and structure below otherwise still holds. Written for
the split agreed on:

- **This repository** (`VeloWetzikon_Contao`) stays the public-facing home: the Contao CMS site
  plus the frontend additions (VeloMelder map tool, embeddable Velonetz views). It stays
  **buildless** — static HTML/CSS/JS, deployed by FTP-ing files to shared hosting, no npm/composer
  install step at deploy time (see `DEPLOY.md`). This constraint is load-bearing for everything
  below and should not be casually broken.
- **A new repository** (name TBD) becomes the API + admin backend, in **PHP/Symfony or
  Laravel** — a real application with Composer, a database, and (likely) its own small
  build step for whatever renders its admin UI.

## 1. Where things stand today (the problem this solves)

There are already **two** copies of the same design values in *this one* repository, under two
different naming conventions:

| Where | Prefix | Example | Kept in sync how |
|---|---|---|---|
| `files/themes/velo/css/colors_and_type.css` | `--color-*` | `--color-primary: #dc3406` | Hand-edited, the "real" site's tokens |
| `public/additions/velo-melder.html` (`COMPILE:TOKENS` block) | `--vw-*` | `--vw-primary: #ff5544` ⚠ | Hand-edited, manually kept visually consistent — **and not even value-identical**, see below |

Auditing this while preparing this document surfaced that `velo-melder.html`'s local primary
color is actually `#ff5544` (a punchier red-orange), not `#dc3406` — the two token sets have
already drifted in at least one real value, not just naming. `VeloWetzikon_Contao`'s
`claude-design-specs/design-system-summary.md` also documented the wrong prefix (`--vw-*`) as if
it were what `colors_and_type.css` used; that error was corrected there, and that document has
since been retired — its "Farben" section content is now this repo's `tokens.css`/`tokens.html`.

**The lesson:** hand-kept copies drift, silently, even within a single repo under one person's
control. Adding a *third* copy (a new backend/admin app, in a different language, in a different
repo) without a deliberate mechanism will make this worse, not better — so the goal isn't just
"share the tokens," it's "stop drifting."

## 2. Proposed structure: a dedicated design-system repository

Per your preference, create a **third repository** — call it `velo-wetzikon-design-tokens` (or
similar) — that becomes the single source of truth for design values. Suggested contents:

```
velo-wetzikon-design-tokens/
├── tokens.css          # the canonical :root block — --color-*, --text-*, --space-*, etc.
├── tokens.json         # the same values as plain JSON (for non-CSS consumers: PHP, JS build tools, design tools)
├── README.md           # what this repo is, how to consume it, how to propose a change
└── CHANGELOG.md         # every value change, dated — this is what "drift detection" diffs against
```

Two files, one CSS and one JSON, both hand-generated from the same source values (or the JSON
generated from the CSS by a tiny script — either is fine at this scale; don't over-engineer a
build pipeline for ~40 values). `tokens.json` shape mirrors `tokens.css` 1:1, e.g.:

```json
{
  "color": {
    "primary": "#dc3406",
    "primary-hover": "#c32d05",
    "success": "#296310"
  },
  "space": { "1": "4px", "2": "8px", "3": "12px" },
  "rate": { "1": "#b91c1c", "2": "#ea580c", "3": "#ca8a04", "4": "#65a30d", "5": "#15803d" }
}
```

This repo does **not** need to be a published npm package, does **not** need a build step, and
does **not** need to be version-pinned via a package manager in either consumer — see §3. Keep it
as plain, hand-editable files; the moment it grows real tooling is the moment it stops being
approachable for quick edits, which defeats the point.

### Naming: pick one prefix, retire the other

While creating this repo, resolve the `--vw-*` vs `--color-*` split rather than canonizing
*either* existing name — both are historical accidents (`--vw-*` came from the standalone
VeloMelder tool being built independently; `--color-*` is what actually shipped on the main
site). Recommend adopting **`--color-*`** as canonical (it's what the live public site already
uses, so canonizing the other name would mean rewriting the CMS's CSS, the larger and riskier
surface) and updating `velo-melder.html`'s local block to match both the names *and* the values
(fixing the `#ff5544` vs `#dc3406` drift found above) the next time that file is touched.

## 3. How each repo consumes the tokens (no build step required anywhere)

### This repo (public, buildless)

Vendor a **copy** of `tokens.css` directly into `files/themes/velo/css/colors_and_type.css`, and
a copy of the relevant `--color-*` subset into `velo-melder.html`'s local block (it can't
`@import` an external stylesheet from a `file://`-served or FTP-deployed static page without a
build step, so a manual copy stays the only option there regardless of where the source of truth
lives). This is **the same manual-copy workflow this repo already uses everywhere** — matches
how `public/additions/velo-velonetz-embed-3.html` is kept in sync with `velo-melder.html` today
via the `compile-velo-views` skill's marker-comment convention. Concretely:

1. When a token value changes, edit it in `velo-wetzikon-design-tokens/tokens.css` first, commit,
   bump `CHANGELOG.md`.
2. Copy the changed lines into `colors_and_type.css` (and into `velo-melder.html`'s
   `COMPILE:TOKENS` block if the value is one it mirrors) by hand, in the same PR/session as the
   change is actually needed here.
3. No CI dependency required for this repo to build or deploy — it never talks to the tokens repo
   at deploy time, only at edit time.

### The new backend repo (Symfony/Laravel, has real tooling)

Since it already has Composer and presumably a small asset pipeline for its own admin UI, it can
consume `tokens.css` more directly than this repo can — e.g. a Composer path/VCS repository
pointing at `velo-wetzikon-design-tokens` (if you want it versioned/pinned), or, just as
reasonably, the same manual-copy approach as above for consistency and to avoid a second
integration pattern to maintain. Given the admin app's design bar is "consistent brand,
readable," not "pixel-identical to the public site," a manual copy kept honest by the drift check
in §4 is proportionate — don't reach for Composer-package machinery just for ~40 CSS variables
unless the backend app is going to consume many more shared assets over time (fonts, icon set,
component CSS) and the overhead starts paying for itself.

### Recommendation

Start with **manual copies in both directions**, protected by an automated drift check (§4) —
not a package manager dependency in either repo. Revisit only if the shared surface grows
significantly (e.g. you end up sharing whole component CSS, not just tokens) or if the backend
app's own frontend build makes a real dependency genuinely free to add.

## 4. Drift detection (the part that actually prevents this problem)

A manual-copy strategy is only trustworthy if drift gets caught automatically, not "whenever
someone happens to notice." Add a small CI check to **each consuming repo** (this repo and the
new backend repo), not to the tokens repo itself:

```
# pseudocode for a CI step (GitHub Actions or equivalent)
- fetch the canonical tokens.css from velo-wetzikon-design-tokens (raw GitHub URL, or a checked-
  out submodule/clone — no package manager needed, just an HTTP fetch or git clone)
- extract the --color-* custom properties from it
- extract the same properties from this repo's own vendored copy
  (files/themes/velo/css/colors_and_type.css, and/or velo-melder.html's COMPILE:TOKENS block)
- diff the two value sets
- if they differ: fail the check (or just post a warning comment, if you'd rather this stay
  advisory) and print exactly which token(s) drifted
```

This can be a ~30-line script in whatever language is convenient per repo (a `.php` script for
the Symfony/Laravel repo, a plain shell/`node` script for this repo — since this repo has no
Node/PHP CLI runtime assumption for its OWN deploy, a GitHub Actions workflow running the check
in CI is fine even if nothing runs it locally; developers only see it as a PR status check). This
repo already has no build step for deployment — the check runs in CI only, never at deploy time,
so it doesn't compromise the "FTP the files up" simplicity documented in `DEPLOY.md`.

## 5. What moves, what doesn't

**Moves to the shared tokens repo:** color palette, spacing scale, type scale (font sizes/
weights/line-heights), the 5-step rating-color scale, radii, shadows, motion tokens
(easing/duration) — i.e. everything currently in `colors_and_type.css`'s `:root` block and
`velo-melder.html`'s `COMPILE:TOKENS` mirror.

**Stays local to this repo, not shared:** the actual CSS *components* (button styles, card
styles, modal layout, nav markup/behavior) — those are implementations built on top of the
tokens, specific to the Contao Twig templates and the standalone VeloMelder HTML file
respectively. The new backend's admin UI will have its own component needs (data tables, forms,
login screen) that don't map onto this project's public-facing component library, and trying to
share components (not just tokens) across a Contao/Twig site and a Symfony/Laravel admin app
would be a much larger, more fragile undertaking than the value it'd add — scope this project to
**tokens only**.

**The `ROUTE_TYPES` registry and the `RATINGS` scale** (colors used specifically for map route
types and Meldung ratings, documented in `velo-melder-data-contract.md`) are a related but
distinct case: those are *data*, not design tokens — once the new backend owns the Meldungen/
routes API, it likely becomes the source of truth for those specific registries too (so the
admin UI and the public map agree on what a "Kantonale Veloroute" looks like), served over the
API rather than duplicated as a hand-kept array in multiple frontend files. That's a data-contract
concern (see `velo-melder-data-contract.md` §3.2), not a design-tokens-repo concern, even though
the hex values happen to overlap with the rating-scale tokens above.

## 6. Open items for you to decide (not blocking, but worth a conscious choice)

- **Repo name and visibility** for `velo-wetzikon-design-tokens` (public open-source alongside
  the other two, or private).
- **Who "owns" a token change** — i.e. does a value change get proposed/reviewed in the tokens
  repo first, or can either consuming repo change its local copy first and backport? Recommend
  "tokens repo first, always" to keep the drift check meaningful (a check that expects the copies
  to lag behind the source by design is much less useful than one that expects them to match).
- **Whether the drift-check CI step is blocking or advisory** in each repo — blocking is stronger
  but means an intentional, not-yet-backported change in the tokens repo could red-X unrelated
  PRs in a consuming repo until someone copies the update over.
