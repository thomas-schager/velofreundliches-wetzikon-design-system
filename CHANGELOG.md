# Changelog

All notable token value changes. One entry per change, newest first.

## 2026-08-17 — Initial import

Repository created to become the canonical source of design tokens for the
velofreundliches-wetzikon projects (public Contao site + a planned backend/admin app), per
`design-system-sharing-strategy.md` in `VeloWetzikon_Contao`.

- `tokens.css` and `tokens.json` populated **verbatim from the public site's live
  `files/themes/velo/css/colors_and_type.css` `:root` block** — no values changed, this is a
  copy of what was already deployed, now given a canonical home.
- Confirmed during the audit that led to this repo's creation:
  `public/additions/velo-melder.html`'s local `--vw-primary` was `#ff5544`, not the public
  site's `#dc3406` — an existing, real drift between the two hand-kept copies, not introduced by
  this import. **Not yet fixed** in `velo-melder.html` as of this entry; tracked as a known
  follow-up (see that repo's `design-system-sharing-strategy.md`, §2 "Naming: pick one prefix,
  retire the other").
