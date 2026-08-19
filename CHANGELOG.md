# Changelog

All notable token value changes. One entry per change, newest first.

## 2026-08-19 — Repo renamed, sharing-strategy doc moved in

- Repo renamed on GitHub from `velofreundliches-wetzikon-design-tokens` to
  `velofreundliches-wetzikon-design-system` (reflects that this repo now holds more than raw
  tokens — see the new `design-system/brand-voice.html` page).
- `design-system-sharing-strategy.md` moved here from `VeloWetzikon_Contao`'s
  `claude-design-specs/` (which has been retired — see below) — it documents this repo's own
  reason for existing, so it belongs here, not in the repo it was written from.
- Added `design-system/brand-voice.html`: voice/language rules, iconography, imagery, footer, and
  UX guidelines migrated from `VeloWetzikon_Contao`'s `claude-design-specs/design-system-summary.md`
  (now deleted there — its token/component content was already covered by `tokens.html` and
  `contao-implementation.html`; this page captures what wasn't: the non-code guidance).
- `VeloWetzikon_Contao/claude-design-specs/` removed entirely: the three remaining planning docs
  (`contao-5-implementation-spec.md`, `contao-theme-spec.md`,
  `navigation/navigation-desktop-mobile-2-levels-spec.html`) were pre-implementation drafts already
  marked superseded by the live code and this repo's docs, so deleted rather than migrated.
  `velo-melder-data-contract.md` moved to `VeloWetzikon_Contao`'s repo root instead of here — it's
  an API/data contract for a future backend, not design-system content.

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
