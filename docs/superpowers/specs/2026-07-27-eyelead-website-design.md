# Eyelead Company Website — Design

## Problem

Eyelead is repositioning as a company that builds and sells AI-powered apps,
starting with MyCofree (a personal finance app, already live as a Windows
desktop build and a web version at
`gustavotinelli.github.io/mycofree-web`). There is no public-facing home for
the company today — no place that presents Eyelead as a brand, explains what
it does, and lets visitors discover and reach its apps. Eyelead owns the
domain `eyelead.com.br` but nothing is deployed there yet.

This spec covers a single deliverable: a public marketing/landing website at
`eyelead.com.br` that introduces Eyelead and links out to its apps (MyCofree
today, more later). It does not cover LGPD hardening of MyCofree itself, or
any other app — that is separate, future work the user has explicitly
deferred ("first things first").

## Scope

**In scope:**
- A single-page static website: hero, "Sobre a Eyelead", "Nossos Apps"
  showcase, an EyePod (company podcast) mention, and a footer.
- A minimal privacy policy page, sufficient for a site that collects no
  personal data itself (see LGPD section below).
- Hosting on GitHub Pages at the custom domain `eyelead.com.br`.
- All content in Brazilian Portuguese.

**Explicitly out of scope:**
- LGPD compliance work for MyCofree or any other app — separate future
  project.
- Any data collection on the site itself (no contact form, no analytics, no
  newsletter signup). Confirmed with the user: v1 is purely informational.
- A blog, CMS, or any dynamically-updated content.
- EyePod getting its own dedicated section or page — it's a footer/nav-level
  mention linking out, not a product card (explicitly not to be presented
  the same way as MyCofree).
- Multi-language support (Portuguese only for v1).

## Architecture

### 1. Tech stack: plain static HTML/CSS/JS, no framework, no build step

The site is a single `index.html` plus `styles.css` and (if needed for small
interactions like a mobile nav toggle) a small `script.js` — no bundler, no
package.json, no build pipeline.

This was chosen over two alternatives:
- **Flutter Web** (same stack as MyCofree) — rejected. Flutter Web renders
  to canvas, which means weak SEO and a heavy JS/WASM payload before
  anything paints. Both are real costs for a page whose entire job is a
  fast, crawlable first impression — the opposite of what MyCofree's app
  logic needs from Flutter.
- **Next.js/React** — rejected for v1. Adds a Node build toolchain and real
  complexity that only pays off once there's a genuine need for dynamic
  content, a CMS, or server logic — none of which exist in the confirmed v1
  scope. Revisit if/when the site needs a blog or a real contact form.

A different stack from MyCofree's Flutter app is expected and normal —
marketing sites and product apps are commonly built differently even within
the same company.

### 2. Page structure

One page (`index.html`), sections top to bottom:

1. **Hero** — the existing Eyelead logo image (`assets/eyelead-logo.png`),
   which already contains the "EYELEAD" wordmark and the "Looking for the
   future" slogan baked into the artwork. The page does **not** repeat the
   company name or slogan as separate text — confirmed explicitly with the
   user, avoiding visual redundancy with the logo. Below the logo, a single
   CTA button ("Ver nossos apps") anchor-scrolls to the apps section.
2. **Sobre a Eyelead** — a short paragraph: what Eyelead is, that it builds
   AI-powered apps, and that MyCofree is the first of them.
3. **Nossos Apps** (`id="apps"`) — a card grid. Each card: app icon, name,
   one-line description, and a link button ("Acessar") to that app's live
   URL. v1 has one real card (MyCofree → `gustavotinelli.github.io/mycofree-web`)
   plus one visibly-empty "Em breve" placeholder card, signaling more apps
   are coming without overpromising specifics.
4. **EyePod mention** — a small pill-shaped element ("🎙 Ouça também o
   EyePod, nosso podcast") linking to wherever the podcast is hosted. Not
   styled like an app card — it's a different kind of thing (media, not a
   product) and should read that way.
5. **Footer** — copyright line, a link to `privacidade.html`, and a contact
   email/link.

### 3. Adding future apps

No data-driven abstraction (no JSON file, no JS array + template renderer)
for the app grid. With one or two apps, a hand-written HTML block per card
is simpler and more transparent than an abstraction built for a scale that
doesn't exist yet. Adding the next app later means copying the MyCofree
card's markup, swapping the icon/name/description/link, and removing the
"Em breve" placeholder (or moving it further down). Revisit this decision
if/when the grid grows past ~5 apps and hand-editing becomes tedious.

### 4. Visual design

Palette pulled directly from the existing logo artwork (sampled, not
guessed):
- Background: `#191919` (near-black)
- Primary text: `#FFFFFF`
- Accent (CTAs, highlights, section labels): `#E2A300` (gold)

Layout, spacing, typography choices (font family, sizing scale), and any
additional polish are explicitly deferred — the user asked to approve the
overall shape now and iterate on cosmetics later. This spec does not
prescribe those details; they'll be decided during implementation and can
change freely without re-opening this design.

The site must be responsive (mobile-first, since landing page traffic skews
mobile) using flexbox/grid and standard breakpoints — this is a baseline
requirement, not a deferred cosmetic.

### 5. LGPD scope for this site

The site itself collects no personal data: no forms, no analytics, no
cookies beyond what a static HTML page inherently has (none). Given that,
the privacy policy page (`privacidade.html`) only needs to:
- State plainly that the Eyelead website collects no personal data.
- Identify Eyelead as the data controller, with a contact email for privacy
  inquiries.
- Note that each Eyelead app (starting with MyCofree) has its own privacy
  policy governing that app's own data practices, and link to it once one
  exists.

This is intentionally minimal and scoped to what this specific deliverable
needs to be honest and compliant — it is not the broader LGPD audit the user
flagged as upcoming work for the apps themselves.

### 6. Hosting & deploy

- New public repo: `GustavoTinelli/eyelead-website`.
- GitHub Pages serves directly from the repo's default branch (root) — no
  GitHub Actions workflow needed, since there is no build step. Pushing to
  `main` is the deploy.
- A `CNAME` file in the repo root containing `eyelead.com.br` tells GitHub
  Pages which custom domain to serve.
- DNS: `eyelead.com.br` is an apex/root domain, so GitHub Pages requires
  **A records** (not a CNAME record — apex domains can't use CNAME per DNS
  spec) pointing at GitHub Pages' IPs:
  `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`.
  User confirmed they have DNS access at their registrar to add these.
- GitHub Pages auto-provisions an HTTPS certificate for the custom domain
  once DNS resolves correctly (no action needed beyond enabling "Enforce
  HTTPS" in repo settings once available).

## Testing approach

No automated test suite — there's no logic to test in a static informational
page. Verification is manual: open the deployed site in a real browser
(mobile and desktop viewport), confirm the hero/about/apps/footer render
correctly, confirm the MyCofree link and EyePod link both resolve, and
confirm the privacy policy page is reachable from the footer. A broken-link
check (all `<a href>` targets resolve) is worth doing by hand before calling
a deploy done, given how small the page is.

## Rollout

Single phase — this is small enough not to need sub-phases:
1. Scaffold the repo (`index.html`, `styles.css`, `assets/`, `privacidade.html`, `CNAME`).
2. Build the page per the structure above.
3. Push to `main`, enable GitHub Pages, add the `CNAME` file.
4. Add DNS A records at the registrar; wait for propagation and GitHub's
   HTTPS provisioning.
5. Manual verification pass (links, mobile layout, HTTPS working).
