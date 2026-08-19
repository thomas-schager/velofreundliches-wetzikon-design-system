# velofreundliches-wetzikon-design-system

Canonical design tokens (colors, spacing, type scale, radii, shadows, motion) for the
**velofreundliches Wetzikon** projects. This repo is the single source of truth — consuming
repos vendor a copy rather than depending on this repo at build/deploy time.

Consumers today:

- **[velofreundliches-wetzikon-contao](https://github.com/thomas-schager/velofreundliches-wetzikon-contao)** — the public Contao CMS site
  (`files/themes/velo/css/colors_and_type.css`) and the standalone VeloMelder feedback tool
  (`public/additions/velo-melder.html`, local `--vw-*`-prefixed mirror of the same values).
- The backend/admin app (Symfony/Laravel) — not yet created.

See [`design-system-sharing-strategy.md`](design-system-sharing-strategy.md) for the full
reasoning behind this repo's existence and the sync approach.

## What's here

- **`tokens.css`** — the canonical `:root` custom-properties block. This *is* what
  `colors_and_type.css`'s `:root` block should contain; everything below `:root` in that file
  (buttons, cards, modal, nav, etc.) is site-specific component CSS built on top of these tokens
  and is **not** shared — see "What moves, what doesn't" in the sharing-strategy doc.
- **`tokens.json`** — the same values as plain JSON, for non-CSS consumers (a PHP/Symfony admin
  app, a JS build step, a design tool). Structured 1:1 with `tokens.css`'s sections.
- **`CHANGELOG.md`** — every value change, dated. This is what a drift check in a consuming repo
  diffs against, so keep it honest: **one entry per PR that touches `tokens.css`/`tokens.json`.**

No build step, no package manager, no dependencies. These are meant to be read and copied by
hand or by a small script — keep it that way; the moment this needs `npm install` to use is the
moment it stops being the easy option for a static, buildless consumer like the public site.

## How to propose a change

1. Open a PR here changing `tokens.css` **and** `tokens.json` together (keep them in sync in the
   same PR — nothing enforces that automatically, so it's on the reviewer to check both changed).
2. Add a `CHANGELOG.md` entry: date, token(s) changed, old → new value, why.
3. Once merged, go update the vendored copy in each consuming repo (see below). A token change
   isn't "done" until the copies actually match again — the change here is the proposal, not the
   deployment.

**Naming:** all tokens use the `--color-*` / `--text-*` / `--space-*` (no site-specific prefix)
convention, matching what's live on the public site today. `velo-melder.html`'s local
`--vw-*`-prefixed block is the one legacy exception (see the sharing-strategy doc); don't
introduce a third naming convention anywhere else.

## How a consuming repo stays in sync (no package manager)

Each consumer keeps a **vendored copy** of the relevant token values, updated by hand when this
repo changes:

- **VeloWetzikon_Contao**: copy the `:root` block into
  `files/themes/velo/css/colors_and_type.css`, and the values that file's local
  `COMPILE:TOKENS` block mirrors into `public/additions/velo-melder.html` (translating
  `--color-*` names to that file's `--vw-*` names — the values must match, the names don't have
  to).
- **Backend/admin app**: either vendor a copy the same way, or — since it has real tooling
  already — pull `tokens.css`/`tokens.json` in via whatever dependency mechanism is convenient
  (a Composer VCS repository pointing here, a simple fetch in a build step, etc.). Start with a
  manual copy for consistency with the other consumer unless the shared surface grows enough to
  justify real package management — see the sharing-strategy doc's recommendation.

A copy that's out of sync is a bug, not a style choice — each consuming repo should run a CI
check that fetches `tokens.css` from here and diffs it against its own vendored copy, so drift
gets caught on a PR instead of noticed by a human months later. (This is why the `#dc3406` vs.
`#ff5544` primary-color mismatch between the public site and `velo-melder.html` went unnoticed
for as long as it did before this repo existed.)
