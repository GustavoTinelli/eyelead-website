# Eyelead Website v2 — Navigation and Pages — Design

## Problem

The v1 Eyelead website (`docs/superpowers/specs/2026-07-27-eyelead-website-design.md`)
shipped as a single page: hero, a short "about" blurb, an apps showcase, an
EyePod mention, and a footer. The user's own assessment: it "feels too
amateur" — not because of visual polish (already a known, deferred item),
but because the site is thin and single-page, with no real navigation, and
doesn't communicate the full scope of what Eyelead actually is.

Over the course of this design conversation, a more complete picture of
Eyelead emerged: it isn't just an AI-apps company. Its mission (tied to its
existing slogan, "Looking for the future") is to empower entrepreneurs
through four pillars — AI apps (live, starting with MyCofree), a podcast
(EyePod, live), and, longer-term, training/education and business
automation services (both explicitly not live yet — vision, not current
offerings). The site needs to communicate that broader mission, with real
navigation, without overpromising on the two pillars that don't exist yet.

The reference point the user provided, `volpiferraz.com.br`, is a
multi-page company site (separate URLs for About/Projects/Contact, reached
via a nav bar) — the user confirmed they want Eyelead to follow that same
multi-page pattern rather than a single scrolling page with anchor links.

## Scope

**In scope:**
- A real navigation bar, present on every page, linking to four pages:
  Início (home), Sobre a Eyelead, Nossos Apps, Contato.
- Converting the site from single-page to multi-page: `sobre.html`,
  `apps.html`, and `contato.html` as new top-level pages alongside the
  existing `index.html` and `privacidade.html` (privacy stays footer-only,
  not a nav item — no change to that from v1).
- A restructured Início (home) page: hero (unchanged), a new "O que
  fazemos" section presenting the four pillars, a condensed apps teaser
  linking to the new Apps page, a condensed about teaser linking to the new
  Sobre page, and a site-wide footer.
- A site-wide footer social-links row: YouTube, Spotify, Instagram —
  replacing v1's single standalone "Ouça também o EyePod" pill (which
  becomes redundant once EyePod has both a pillar card on the home page
  and a footer icon; see Architecture).
- A typography change: Poppins (self-hosted), replacing the v1 system-font
  stack, per explicit user feedback that the previous font "is too basic."
- A compact, nav-appropriate crop of the existing logo (wordmark only, no
  slogan script) for use in the nav bar, since the full hero logo asset is
  too tall for a compact nav.
- A mobile hamburger-menu toggle for the nav bar (the one new piece of
  JS this phase needs, beyond v1's single-line year script).

**Explicitly out of scope:**
- Any content or booking flow for Treinamentos or Automação — both are
  presented as future vision only, exactly like the apps grid's existing
  "Em breve" pattern. No pricing, no signup, no waitlist form.
- A dedicated EyePod page — it's represented via the "O que fazemos" pillar
  card (home page) and the footer's YouTube icon, not a standalone page.
- A contact form — the user explicitly chose to keep Contato as static
  info (email + socials) only. This means the "purely informational, no
  personal data collected" privacy stance from v1 is unchanged; the privacy
  policy page needs no content changes in this phase.
- Stats/metrics banners (e.g. "X apps, X years") — explicitly rejected by
  the user as premature for a brand-new company; revisit once there's a
  real number worth showing.
- Any further visual/CSS polish beyond the specific items this phase
  covers (nav bar, font, new page layouts). Colors, spacing conventions,
  and component styles (buttons, cards) carry over unchanged from v1's
  design tokens (`--bg`, `--accent`, etc. in `styles.css`).

## Architecture

### 1. Multi-page structure, no framework, no build step

Same foundational choice as v1: plain HTML/CSS/JS, no framework, no build
tooling. Four HTML pages now share one `styles.css` and one `script.js`:
`index.html`, `sobre.html`, `apps.html`, `contato.html` (plus
`privacidade.html`, unchanged, still not a nav item). Each page repeats the
same `<head>` boilerplate (meta tags, stylesheet link) and the same nav
bar and footer markup — with only four pages, hand-duplicating this
markup across files is simpler and more transparent than introducing any
templating mechanism, consistent with v1's "don't build abstractions for a
scale that doesn't exist" principle (Section 3 of the v1 spec, applied here
to pages instead of app cards).

### 2. Navigation bar

Fixed/sticky top bar, present identically on all five pages (including
`privacidade.html`, for consistent navigation even from that footer-linked
page). Layout (chosen from three mocked-up options): compact logo on the
left, page links in the middle/right, and a visually distinct CTA button
("Ver Apps," linking to `apps.html`) at the end of the link list — the
CTA uses the same `.btn.btn-primary` treatment already established in v1.

Logo: a new cropped asset, `assets/eyelead-logo-nav.png`, containing just
the "EYELEAD" wordmark and its gold arrow accent — cropped from the same
source (`Eyelead_wall.png`) used for the full hero logo and the favicon,
excluding the "Looking for the future" script text (which stays exclusive
to the hero, so the slogan doesn't feel diluted by repetition on every
page). Crop bounds: `(100, 280, 960, 460)` from the 1080×965 source image.

Mobile: below `768px` (a new breakpoint, distinct from v1's existing
`480px` one — the nav has to collapse well before phone-width, since four
links plus a logo and a CTA button will overflow on tablet-width screens
long before `480px`; `480px` stays exactly as-is for the app-card layout
switch it already governs), the link list collapses behind a hamburger
toggle button. This requires a small, self-contained addition to
`script.js`: a click handler that toggles a CSS class (e.g. `.nav-open`) on
the nav element, with the corresponding CSS showing/hiding the link list.
No new library — plain `addEventListener` and `classList.toggle`.

### 3. Typography

Poppins, weights 400/600/700, replacing the v1 system-font stack
(`-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif`) as
the `body` font-family in `styles.css`. Chosen over two alternatives shown
side-by-side: a Space Grotesk (headings) + Inter (body) pairing, and Sora
— Poppins was the user's pick after seeing real rendered comparisons.

**Self-hosted, not loaded from Google Fonts' CDN.** The v1 privacy policy
states the site "does not use tracking cookies or analytics tools" — proxying
every visitor's font request through Google's CDN is in tension with that
stance (Google can log the requesting IP), even though Google Fonts itself
sets no cookies. Self-hosting is a small one-time step (download the
`.woff2` files, add `@font-face` rules, no ongoing dependency) and keeps
the site's actual network behavior matching what its own privacy policy
already promises. Font files live at `assets/fonts/`.

### 4. Home page (`index.html`) restructuring

New top-to-bottom flow:
1. **Nav** (new, Section 2).
2. **Hero** — unchanged from v1: full logo image (with slogan), "Ver
   nossos apps" CTA anchor-scrolling to the apps teaser section below.
3. **"O que fazemos"** (new) — four pillar cards in a responsive grid:
   - **Apps de IA** (live): "Aplicativos inteligentes para o dia a dia."
   - **Podcast** (live): "EyePod: empreendedorismo e futuro." This card
     is the home page's only EyePod mention — it replaces v1's standalone
     "Ouça também o EyePod" pill entirely (see Scope).
   - **Treinamentos** (not live — styled like the existing "Em breve" app
     placeholder card: dashed border, muted text): "Em breve."
   - **Automação** (not live, same treatment): "Em breve."
4. **Nossos Apps teaser** — the same MyCofree card style from v1, plus a
   "Ver todos os apps →" link to the new `apps.html`.
5. **Sobre teaser** — a short paragraph (shorter than the full Sobre page)
   plus a "Conheça nossa história →" link to the new `sobre.html`.
6. **Footer** (new site-wide version, Section 5).

### 5. Site-wide footer with social links

Every page's footer gains a row of three social icons/links above the
existing copyright line, linking to:
- YouTube: `https://www.youtube.com/@EyePod_Cast`
- Spotify: `https://open.spotify.com/show/6eXKSgk2ZopXclbmYRGQb7`
- Instagram: `https://www.instagram.com/eyeleadbr/` (the URL the user
  provided included a `?igsh=...` share-tracking query parameter; stripped
  here since it's not needed for a direct link and the site otherwise
  avoids passing tracking parameters).

This is the only place any of these three platforms are linked from
site-wide chrome — consistent with Section 2's decision not to give EyePod
its own nav item.

### 6. Sobre a Eyelead page (`sobre.html`, new)

Expands on the home page's Sobre teaser: the fuller mission statement tied
to "Looking for the future," and the four pillars from "O que fazemos"
explained in more depth (what each one is, and for Treinamentos/Automação,
that they're upcoming rather than available today). No new factual claims
beyond what's already established in this spec — this page is where the
teaser's "conheça nossa história" promise gets paid off, not a place to
introdut any new commitments.

### 7. Nossos Apps page (`apps.html`, new)

The full apps catalog, replacing the home page as the definitive place to
browse Eyelead's apps. Same card component as v1/home (icon, name,
description, "Acessar" link), starting with MyCofree, plus the "Em breve,
novos apps" placeholder card — both carried over unchanged from v1's
existing markup, just relocated to their own page instead of living
directly on the home page.

### 8. Contato page (`contato.html`, new)

Static contact information only, per explicit user decision: the
`contato@eyelead.com.br` email (already used in the v1 footer and privacy
policy) and the same three social links from Section 5. No form, no new
data collection — the v1 privacy policy's "this site collects no personal
data" statement remains accurate and needs no edits in this phase.

## LGPD considerations

No change from v1's stance: the site still collects zero personal data
(no forms added in this phase, confirmed explicitly with the user for the
Contato page). The only new consideration surfaced during this phase is
the font-hosting decision (Section 3), resolved by self-hosting rather than
introducing a third-party font CDN — keeping the site's actual behavior
aligned with what `privacidade.html` already states, without needing to
change that page's content.

## Testing approach

Same as v1: no automated test suite (still a static informational site,
now just more pages). Verification is manual/structural: local
headless-browser checks (the established pattern from v1 — Chrome via
`puppeteer-core`, screenshots, console-error checks) against all five
pages, both desktop and mobile viewports, plus a check that the nav's
hamburger toggle actually opens/closes on mobile and that all internal
nav links resolve to the correct pages.

## Rollout

Single phase, following the same shape as v1's rollout (this is additive
to a working, deployed site — nothing needs to go offline while it's
built):
1. Prepare new assets: the cropped nav logo, the self-hosted Poppins font
   files.
2. Build the shared nav bar and footer markup, add the mobile-toggle JS,
   update `styles.css` for the new font and nav/footer styles.
3. Restructure `index.html` into the new flow (Section 4).
4. Build `sobre.html`, `apps.html`, `contato.html`.
5. Local verification pass across all five pages, both viewports.
6. Commit, push, and let the already-working GitHub Pages deploy pick it
   up (no CI changes needed — same as v1, pushing to `main` is the
   deploy).
7. Live verification pass, matching v1's approach.
