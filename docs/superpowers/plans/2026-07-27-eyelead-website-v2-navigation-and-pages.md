# Eyelead Website v2 — Navigation and Pages Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Convert the Eyelead site from a single page to a real multi-page site (Início, Sobre a Eyelead, Nossos Apps, Contato) with a nav bar, reframe the home page around four pillars (apps/podcast live, training/automation as vision), add a site-wide social-links footer, and switch to a self-hosted Poppins typeface — per `docs/superpowers/specs/2026-07-27-eyelead-website-v2-navigation-and-pages.md`.

**Architecture:** Still plain static HTML/CSS/JS, no framework, no build step. Five HTML pages now share one `styles.css`/`script.js`: `index.html`, `sobre.html` (new), `apps.html` (new), `contato.html` (new), `privacidade.html` (updated). Nav bar and footer markup is hand-duplicated identically across all five files — consistent with the project's existing "don't build abstractions for a scale that doesn't exist" principle.

**Tech Stack:** HTML5, CSS3 (flexbox, media queries, `@font-face`), vanilla JS (one new `addEventListener`/`classList.toggle` for the mobile nav). Self-hosted Poppins (woff2, latin subset — covers all Portuguese accented characters since they fall within the Latin-1 Supplement Unicode block).

**Note on "testing":** Same as v1 — no automated test suite (static informational site). Every task substitutes concrete structural/visual verification (grep for expected markup, or an exact browser check) for the write-test/run-test cycle.

**Note on deploying safely:** This is a live, already-public site (`eyelead.com.br`). To avoid exposing a half-finished state (e.g. a nav bar linking to pages that don't exist yet) to real visitors, **do not push to `origin` until Task 10** — every earlier task commits locally only. Task 10 is the single point where everything gets pushed together, once local verification (Task 9) has confirmed the whole site works as a unit.

---

## File Structure

```
eyelead-website/
├── index.html              # MODIFIED: adds nav+footer, restructures into hero/pillars/apps-teaser/sobre-teaser
├── sobre.html               # NEW: About page (mission + pillars in depth)
├── apps.html                 # NEW: Full apps catalog page
├── contato.html               # NEW: Contact info page
├── privacidade.html            # MODIFIED: adds nav, replaces old footer with new shared footer
├── styles.css                   # MODIFIED: Poppins @font-face + font-family, nav/hamburger, footer social links, pillars, teaser-link, page-title
├── script.js                     # MODIFIED: adds hamburger toggle handler
├── assets/
│   ├── eyelead-logo.png        # unchanged (hero)
│   ├── eyelead-logo-nav.png   # NEW: cropped wordmark-only logo for the nav bar
│   ├── mycofree-icon.png     # unchanged
│   ├── favicon.png             # unchanged
│   └── fonts/
│       ├── Poppins-Regular.woff2   # NEW: self-hosted, weight 400
│       ├── Poppins-SemiBold.woff2  # NEW: self-hosted, weight 600
│       └── Poppins-Bold.woff2      # NEW: self-hosted, weight 700
└── docs/superpowers/
    ├── specs/2026-07-27-eyelead-website-v2-navigation-and-pages.md   # already exists
    └── plans/2026-07-27-eyelead-website-v2-navigation-and-pages.md    # this file
```

---

### Task 1: Prepare new assets — nav logo crop and self-hosted Poppins fonts

**Files:**
- Create: `D:\EyeLead\eyelead-website\assets\eyelead-logo-nav.png`
- Create: `D:\EyeLead\eyelead-website\assets\fonts\Poppins-Regular.woff2`
- Create: `D:\EyeLead\eyelead-website\assets\fonts\Poppins-SemiBold.woff2`
- Create: `D:\EyeLead\eyelead-website\assets\fonts\Poppins-Bold.woff2`

- [ ] **Step 1: Generate the cropped nav logo**

Run this exact Python script (Pillow already installed in this environment):

```bash
python3 -c "
from PIL import Image
img = Image.open(r'D:\EyeLead\Eyelead_wall.png').convert('RGB')
crop = img.crop((100, 280, 960, 460))
crop.save(r'D:\EyeLead\eyelead-website\assets\eyelead-logo-nav.png')
print('saved', crop.size)
"
```

Expected output: `saved (860, 180)`

- [ ] **Step 2: Create the fonts directory and download the three self-hosted Poppins files**

These exact URLs were verified working during this plan's design (the "latin" subset, which covers all Portuguese accented characters since they fall in the Latin-1 Supplement Unicode block U+0080–00FF):

```bash
mkdir -p "D:/EyeLead/eyelead-website/assets/fonts"
curl -s -o "D:/EyeLead/eyelead-website/assets/fonts/Poppins-Regular.woff2" "https://fonts.gstatic.com/s/poppins/v24/pxiEyp8kv8JHgFVrJJfecg.woff2"
curl -s -o "D:/EyeLead/eyelead-website/assets/fonts/Poppins-SemiBold.woff2" "https://fonts.gstatic.com/s/poppins/v24/pxiByp8kv8JHgFVrLEj6Z1xlFQ.woff2"
curl -s -o "D:/EyeLead/eyelead-website/assets/fonts/Poppins-Bold.woff2" "https://fonts.gstatic.com/s/poppins/v24/pxiByp8kv8JHgFVrLCz7Z1xlFQ.woff2"
```

- [ ] **Step 3: Verify all four new files exist and are valid**

```bash
ls -la "D:/EyeLead/eyelead-website/assets/eyelead-logo-nav.png" "D:/EyeLead/eyelead-website/assets/fonts/"
file "D:/EyeLead/eyelead-website/assets/fonts/"*.woff2
```

Expected: `eyelead-logo-nav.png` exists (~30-60KB range is plausible, just confirm non-zero); all three `.woff2` files each report `Web Open Font Format (Version 2)` via the `file` command, each roughly 7-8KB.

- [ ] **Step 4: Commit (local only — do not push)**

```bash
cd "D:/EyeLead/eyelead-website"
git add assets/eyelead-logo-nav.png assets/fonts/
git commit -m "chore: add nav logo crop and self-hosted Poppins font files"
```

---

### Task 2: Update styles.css — fonts, nav bar, hamburger menu, footer social links, pillars, teasers

**Files:**
- Modify: `D:\EyeLead\eyelead-website\styles.css`

- [ ] **Step 1: Add `@font-face` rules right after the `:root` block, and update `body`'s `font-family`**

Find the existing `:root { ... }` block (the one with `--bg`, `--accent`, etc. — now has 11 variables after v1's fixes) and insert this immediately after its closing `}`:

```css
@font-face {
  font-family: 'Poppins';
  font-style: normal;
  font-weight: 400;
  font-display: swap;
  src: url('assets/fonts/Poppins-Regular.woff2') format('woff2');
}

@font-face {
  font-family: 'Poppins';
  font-style: normal;
  font-weight: 600;
  font-display: swap;
  src: url('assets/fonts/Poppins-SemiBold.woff2') format('woff2');
}

@font-face {
  font-family: 'Poppins';
  font-style: normal;
  font-weight: 700;
  font-display: swap;
  src: url('assets/fonts/Poppins-Bold.woff2') format('woff2');
}
```

Then find the existing `body { ... }` rule and change its `font-family` line from:
```css
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
```
to:
```css
  font-family: 'Poppins', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
```
(Poppins first, same system-font stack kept as fallback in case the font file fails to load — don't remove the fallback chain.)

- [ ] **Step 2: Add nav bar styles**

Append to the end of `styles.css`:

```css
/* Navbar */
.navbar {
  position: sticky;
  top: 0;
  z-index: 100;
  background: var(--bg);
  border-bottom: 1px solid var(--border);
}

.navbar-inner {
  position: relative;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 14px 24px;
  max-width: 1100px;
  margin: 0 auto;
}

.navbar-brand {
  display: block;
  line-height: 0;
}

.navbar-logo {
  height: 28px;
  width: auto;
  display: block;
}

.navbar-toggle {
  display: none;
  flex-direction: column;
  gap: 5px;
  background: none;
  border: none;
  cursor: pointer;
  padding: 4px;
}

.navbar-toggle span {
  display: block;
  width: 22px;
  height: 2px;
  background: var(--text);
}

.navbar-links {
  display: flex;
  align-items: center;
  gap: 24px;
  font-size: 0.85rem;
}

.navbar-links a {
  text-decoration: none;
  color: var(--text-light);
}

.navbar-links a:hover {
  color: var(--accent);
}

@media (max-width: 768px) {
  .navbar-toggle {
    display: flex;
  }

  .navbar-links {
    display: none;
    position: absolute;
    top: 100%;
    left: 0;
    right: 0;
    background: var(--bg);
    border-bottom: 1px solid var(--border);
    flex-direction: column;
    align-items: flex-start;
    padding: 16px 24px;
    gap: 16px;
  }

  .navbar-links.open {
    display: flex;
  }
}
```

- [ ] **Step 3: Add footer social-links styles**

Append:

```css
/* Footer social links */
.social-links {
  display: flex;
  justify-content: center;
  gap: 20px;
  margin-bottom: 12px;
  font-size: 0.8rem;
}

.social-links a {
  color: var(--link-muted);
  text-decoration: none;
}

.social-links a:hover {
  color: var(--accent);
}
```

- [ ] **Step 4: Add pillars ("O que fazemos") styles**

Append:

```css
/* Pillars ("O que fazemos") */
.pillars-section {
  padding: 24px 24px 48px;
}

.pillars {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  justify-content: center;
  max-width: 900px;
  margin: 0 auto;
}

.pillar-card {
  background: var(--bg-elevated);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 20px;
  width: 100%;
  max-width: 200px;
  text-align: center;
  text-decoration: none;
  color: inherit;
  display: block;
}

.pillar-card h3 {
  font-size: 0.9rem;
  color: var(--accent);
  margin-bottom: 6px;
}

.pillar-card p {
  font-size: 0.8rem;
  color: var(--text-muted);
}

.pillar-card-soon {
  border: 1px dashed var(--border-placeholder);
  background: transparent;
}

.pillar-card-soon h3 {
  color: var(--text-placeholder);
}

.pillar-card-soon .badge {
  display: inline-block;
  font-size: 0.7rem;
  color: var(--text-placeholder);
  margin-top: 6px;
}
```

- [ ] **Step 5: Add teaser-link and page-title styles**

Append:

```css
/* "See more" links under home page teaser sections */
.teaser-link {
  text-align: center;
  margin-top: 16px;
  font-size: 0.85rem;
}

.teaser-link a {
  color: var(--accent);
  text-decoration: none;
}

/* Page title header, used by sobre.html and apps.html */
.page-title {
  text-align: center;
  padding: 48px 24px 24px;
}

.page-title h1 {
  font-size: 1.75rem;
  margin-bottom: 8px;
}

.page-title p {
  font-size: 0.9rem;
  color: var(--text-muted);
}
```

- [ ] **Step 6: Verify**

```bash
grep -c "@font-face" "D:/EyeLead/eyelead-website/styles.css"
grep -c "'Poppins'" "D:/EyeLead/eyelead-website/styles.css"
grep -c "\.navbar\b" "D:/EyeLead/eyelead-website/styles.css"
grep -c "\.pillar-card\b" "D:/EyeLead/eyelead-website/styles.css"
grep -c "\.page-title" "D:/EyeLead/eyelead-website/styles.css"
```
Expected: `3`, at least `4` (3 in @font-face + 1 in body), at least `1`, at least `3`, at least `2`.

- [ ] **Step 7: Commit (local only)**

```bash
cd "D:/EyeLead/eyelead-website"
git add styles.css
git commit -m "feat: add Poppins typeface, nav bar, pillars, and teaser-link styles"
```

---

### Task 3: Update script.js — mobile nav toggle

**Files:**
- Modify: `D:\EyeLead\eyelead-website\script.js`

- [ ] **Step 1: Append the toggle handler**

Current content is one line (the year-fill script with its null-check guard from v1). Append this to the end of the file (don't touch the existing line):

```js

const navToggle = document.getElementById('navToggle');
const navLinks = document.getElementById('navLinks');
if (navToggle && navLinks) {
  navToggle.addEventListener('click', () => {
    const isOpen = navLinks.classList.toggle('open');
    navToggle.setAttribute('aria-expanded', isOpen ? 'true' : 'false');
  });
}
```

- [ ] **Step 2: Verify**

```bash
grep -c "navToggle" "D:/EyeLead/eyelead-website/script.js"
grep -c "getElementById('year')" "D:/EyeLead/eyelead-website/script.js"
```
Expected: at least `3` (declaration + 2 usages), `1` (the original year line is untouched).

- [ ] **Step 3: Commit (local only)**

```bash
cd "D:/EyeLead/eyelead-website"
git add script.js
git commit -m "feat: add mobile nav hamburger toggle"
```

---

### Task 4: Restructure index.html

**Files:**
- Modify: `D:\EyeLead\eyelead-website\index.html`

- [ ] **Step 1: Replace the entire file content**

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Eyelead — Looking for the future</title>
  <meta name="description" content="A Eyelead cria aplicativos incríveis com inteligência artificial e empodera empreendedores através de apps, podcast, treinamentos e automação.">
  <meta property="og:title" content="Eyelead — Looking for the future">
  <meta property="og:description" content="A Eyelead cria aplicativos incríveis com inteligência artificial.">
  <meta property="og:image" content="https://eyelead.com.br/assets/eyelead-logo.png">
  <meta property="og:url" content="https://eyelead.com.br/">
  <meta property="og:type" content="website">
  <link rel="icon" type="image/png" href="assets/favicon.png">
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <nav class="navbar">
    <div class="navbar-inner">
      <a href="index.html" class="navbar-brand"><img src="assets/eyelead-logo-nav.png" alt="Eyelead" class="navbar-logo"></a>
      <button type="button" class="navbar-toggle" id="navToggle" aria-label="Abrir menu" aria-expanded="false" aria-controls="navLinks">
        <span></span><span></span><span></span>
      </button>
      <div class="navbar-links" id="navLinks">
        <a href="sobre.html">Sobre a Eyelead</a>
        <a href="apps.html">Nossos Apps</a>
        <a href="contato.html">Contato</a>
        <a href="apps.html" class="btn btn-primary btn-small">Ver Apps</a>
      </div>
    </div>
  </nav>

  <header class="hero">
    <h1><img src="assets/eyelead-logo.png" alt="Eyelead — Looking for the future" class="hero-logo"></h1>
    <a href="apps.html" class="btn btn-primary">Ver nossos apps</a>
  </header>

  <section class="pillars-section">
    <h2 class="label">O QUE FAZEMOS</h2>
    <div class="pillars">
      <a href="apps.html" class="pillar-card">
        <h3>Apps de IA</h3>
        <p>Aplicativos inteligentes para o dia a dia.</p>
      </a>
      <a href="https://www.youtube.com/@EyePod_Cast" target="_blank" rel="noopener noreferrer" class="pillar-card">
        <h3>Podcast</h3>
        <p>EyePod: conversas sobre empreendedorismo, tecnologia e o futuro.</p>
      </a>
      <div class="pillar-card pillar-card-soon">
        <h3>Treinamentos</h3>
        <p>Capacitação prática para empreendedores.</p>
        <span class="badge">Em breve</span>
      </div>
      <div class="pillar-card pillar-card-soon">
        <h3>Automação</h3>
        <p>Soluções para simplificar processos e escalar negócios.</p>
        <span class="badge">Em breve</span>
      </div>
    </div>
  </section>

  <section class="apps">
    <h2 class="label">NOSSOS APPS</h2>
    <div class="app-grid">
      <div class="app-card">
        <img src="assets/mycofree-icon.png" alt="MyCofree" class="app-icon">
        <div class="app-info">
          <h3>MyCofree</h3>
          <p>Controle financeiro pessoal, simples e inteligente.</p>
        </div>
        <a href="https://gustavotinelli.github.io/mycofree-web/" target="_blank" rel="noopener noreferrer" class="btn btn-primary btn-small">Acessar</a>
      </div>
    </div>
    <div class="teaser-link">
      <a href="apps.html">Ver todos os apps →</a>
    </div>
  </section>

  <section class="about">
    <h2 class="label">SOBRE A EYELEAD</h2>
    <p>
      Somos uma empresa voltada para o futuro — criamos apps, formamos
      empreendedores e simplificamos negócios através da tecnologia.
    </p>
    <div class="teaser-link">
      <a href="sobre.html">Conheça nossa história →</a>
    </div>
  </section>

  <footer class="site-footer">
    <div class="social-links">
      <a href="https://www.youtube.com/@EyePod_Cast" target="_blank" rel="noopener noreferrer">YouTube</a>
      <a href="https://open.spotify.com/show/6eXKSgk2ZopXclbmYRGQb7" target="_blank" rel="noopener noreferrer">Spotify</a>
      <a href="https://www.instagram.com/eyeleadbr/" target="_blank" rel="noopener noreferrer">Instagram</a>
    </div>
    <p>&copy; <span id="year"></span> Eyelead · <a href="privacidade.html">Política de Privacidade</a> · <a href="contato.html">Contato</a></p>
  </footer>

  <script src="script.js"></script>
</body>
</html>
```

- [ ] **Step 2: Verify**

```bash
grep -c "navbar-toggle" "D:/EyeLead/eyelead-website/index.html"
grep -c "pillar-card" "D:/EyeLead/eyelead-website/index.html"
grep -c "Em breve" "D:/EyeLead/eyelead-website/index.html"
grep -c "sobre.html\|apps.html\|contato.html" "D:/EyeLead/eyelead-website/index.html"
grep -c "eyepod-pill" "D:/EyeLead/eyelead-website/index.html"
```
Expected: `1`, `4` (2 live pillar cards + 2 soon variants), `2` (Treinamentos + Automação badges), at least `6` (nav links + CTA + teaser links), `0` (the old standalone EyePod pill is fully removed, per spec).

- [ ] **Step 3: Commit (local only)**

```bash
cd "D:/EyeLead/eyelead-website"
git add index.html
git commit -m "feat: restructure home page with nav, pillars, and teaser sections"
```

---

### Task 5: Update privacidade.html — add nav bar and shared footer

**Files:**
- Modify: `D:\EyeLead\eyelead-website\privacidade.html`

- [ ] **Step 1: Replace the entire file content**

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Política de Privacidade — Eyelead</title>
  <link rel="icon" type="image/png" href="assets/favicon.png">
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <nav class="navbar">
    <div class="navbar-inner">
      <a href="index.html" class="navbar-brand"><img src="assets/eyelead-logo-nav.png" alt="Eyelead" class="navbar-logo"></a>
      <button type="button" class="navbar-toggle" id="navToggle" aria-label="Abrir menu" aria-expanded="false" aria-controls="navLinks">
        <span></span><span></span><span></span>
      </button>
      <div class="navbar-links" id="navLinks">
        <a href="sobre.html">Sobre a Eyelead</a>
        <a href="apps.html">Nossos Apps</a>
        <a href="contato.html">Contato</a>
        <a href="apps.html" class="btn btn-primary btn-small">Ver Apps</a>
      </div>
    </div>
  </nav>

  <div class="page">
    <a href="index.html" class="back-link">← Voltar</a>
    <h1>Política de Privacidade</h1>
    <p>Última atualização: Julho de 2026</p>

    <h2>Este site não coleta dados pessoais</h2>
    <p>
      O site institucional da Eyelead (eyelead.com.br) é puramente
      informativo. Não utilizamos formulários, cookies de rastreamento
      ou ferramentas de análise de visitas. Nenhum dado pessoal é
      coletado, armazenado ou compartilhado através deste site.
    </p>

    <h2>Controlador dos dados</h2>
    <p>
      A Eyelead é a controladora responsável por este site. Em caso de
      dúvidas sobre privacidade, entre em contato pelo e-mail
      <a href="mailto:contato@eyelead.com.br">contato@eyelead.com.br</a>.
    </p>

    <h2>Nossos aplicativos</h2>
    <p>
      Cada aplicativo da Eyelead, incluindo o MyCofree, possui sua
      própria política de privacidade, que descreve especificamente
      quais dados são coletados e como são utilizados dentro daquele
      aplicativo. Consulte a política de privacidade do aplicativo
      específico para mais informações.
    </p>
  </div>

  <footer class="site-footer">
    <div class="social-links">
      <a href="https://www.youtube.com/@EyePod_Cast" target="_blank" rel="noopener noreferrer">YouTube</a>
      <a href="https://open.spotify.com/show/6eXKSgk2ZopXclbmYRGQb7" target="_blank" rel="noopener noreferrer">Spotify</a>
      <a href="https://www.instagram.com/eyeleadbr/" target="_blank" rel="noopener noreferrer">Instagram</a>
    </div>
    <p>&copy; <span id="year"></span> Eyelead · <a href="privacidade.html">Política de Privacidade</a> · <a href="contato.html">Contato</a></p>
  </footer>

  <script src="script.js"></script>
</body>
</html>
```

Note: this changes nothing about the page's actual privacy content (still word-for-word identical to v1's text) — only the nav bar is added above it and the footer is replaced with the new shared version. No LGPD-relevant change, per the spec.

- [ ] **Step 2: Verify**

```bash
grep -c "navbar-toggle" "D:/EyeLead/eyelead-website/privacidade.html"
grep -c "social-links" "D:/EyeLead/eyelead-website/privacidade.html"
grep -c "Nenhum dado pessoal é" "D:/EyeLead/eyelead-website/privacidade.html"
```
Expected: `1`, `1`, `1` (confirms the privacy content text is untouched).

- [ ] **Step 3: Commit (local only)**

```bash
cd "D:/EyeLead/eyelead-website"
git add privacidade.html
git commit -m "feat: add nav bar and shared footer to the privacy policy page"
```

---

### Task 6: Create sobre.html

**Files:**
- Create: `D:\EyeLead\eyelead-website\sobre.html`

- [ ] **Step 1: Write the file**

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Sobre a Eyelead</title>
  <meta name="description" content="A Eyelead nasceu para empoderar empreendedores através da tecnologia — apps de IA, podcast, treinamentos e automação.">
  <meta property="og:title" content="Sobre a Eyelead">
  <meta property="og:description" content="A Eyelead nasceu para empoderar empreendedores através da tecnologia.">
  <meta property="og:image" content="https://eyelead.com.br/assets/eyelead-logo.png">
  <meta property="og:url" content="https://eyelead.com.br/sobre.html">
  <meta property="og:type" content="website">
  <link rel="icon" type="image/png" href="assets/favicon.png">
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <nav class="navbar">
    <div class="navbar-inner">
      <a href="index.html" class="navbar-brand"><img src="assets/eyelead-logo-nav.png" alt="Eyelead" class="navbar-logo"></a>
      <button type="button" class="navbar-toggle" id="navToggle" aria-label="Abrir menu" aria-expanded="false" aria-controls="navLinks">
        <span></span><span></span><span></span>
      </button>
      <div class="navbar-links" id="navLinks">
        <a href="sobre.html">Sobre a Eyelead</a>
        <a href="apps.html">Nossos Apps</a>
        <a href="contato.html">Contato</a>
        <a href="apps.html" class="btn btn-primary btn-small">Ver Apps</a>
      </div>
    </div>
  </nav>

  <div class="page-title">
    <h1>Sobre a Eyelead</h1>
    <p>Looking for the future — empoderando empreendedores através da tecnologia.</p>
  </div>

  <section class="about">
    <p>
      A Eyelead nasceu com o propósito de empoderar empreendedores através
      da tecnologia. Acreditamos que o futuro pertence a quem sabe usar
      inteligência artificial, automação e conhecimento para simplificar
      e escalar seus negócios — e é isso que construímos, em quatro
      frentes.
    </p>
  </section>

  <section class="pillars-section">
    <h2 class="label">O QUE FAZEMOS</h2>
    <div class="pillars">
      <a href="apps.html" class="pillar-card">
        <h3>Apps de IA</h3>
        <p>Desenvolvemos aplicativos que usam inteligência artificial
        para resolver problemas reais do dia a dia. O primeiro deles, o
        MyCofree, já está disponível para controle financeiro pessoal.</p>
      </a>
      <a href="https://www.youtube.com/@EyePod_Cast" target="_blank" rel="noopener noreferrer" class="pillar-card">
        <h3>Podcast</h3>
        <p>O EyePod é o nosso podcast, onde discutimos
        empreendedorismo, tecnologia e as tendências que vão moldar o
        futuro dos negócios.</p>
      </a>
      <div class="pillar-card pillar-card-soon">
        <h3>Treinamentos</h3>
        <p>Estamos preparando treinamentos práticos para ajudar
        empreendedores a dominar inteligência artificial e automação
        em seus negócios.</p>
        <span class="badge">Em breve</span>
      </div>
      <div class="pillar-card pillar-card-soon">
        <h3>Automação</h3>
        <p>Em breve, ofereceremos soluções de automação para
        simplificar processos e ajudar negócios a escalar com mais
        eficiência.</p>
        <span class="badge">Em breve</span>
      </div>
    </div>
  </section>

  <footer class="site-footer">
    <div class="social-links">
      <a href="https://www.youtube.com/@EyePod_Cast" target="_blank" rel="noopener noreferrer">YouTube</a>
      <a href="https://open.spotify.com/show/6eXKSgk2ZopXclbmYRGQb7" target="_blank" rel="noopener noreferrer">Spotify</a>
      <a href="https://www.instagram.com/eyeleadbr/" target="_blank" rel="noopener noreferrer">Instagram</a>
    </div>
    <p>&copy; <span id="year"></span> Eyelead · <a href="privacidade.html">Política de Privacidade</a> · <a href="contato.html">Contato</a></p>
  </footer>

  <script src="script.js"></script>
</body>
</html>
```

- [ ] **Step 2: Verify**

```bash
grep -c "pillar-card" "D:/EyeLead/eyelead-website/sobre.html"
grep -c "<h1>Sobre a Eyelead</h1>" "D:/EyeLead/eyelead-website/sobre.html"
grep -c "Em breve" "D:/EyeLead/eyelead-website/sobre.html"
```
Expected: `4`, `1`, `2`.

- [ ] **Step 3: Commit (local only)**

```bash
cd "D:/EyeLead/eyelead-website"
git add sobre.html
git commit -m "feat: add the Sobre a Eyelead page"
```

---

### Task 7: Create apps.html

**Files:**
- Create: `D:\EyeLead\eyelead-website\apps.html`

- [ ] **Step 1: Write the file**

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Nossos Apps — Eyelead</title>
  <meta name="description" content="Conheça os aplicativos da Eyelead, começando pelo MyCofree, para controle financeiro pessoal.">
  <meta property="og:title" content="Nossos Apps — Eyelead">
  <meta property="og:description" content="Conheça os aplicativos da Eyelead.">
  <meta property="og:image" content="https://eyelead.com.br/assets/eyelead-logo.png">
  <meta property="og:url" content="https://eyelead.com.br/apps.html">
  <meta property="og:type" content="website">
  <link rel="icon" type="image/png" href="assets/favicon.png">
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <nav class="navbar">
    <div class="navbar-inner">
      <a href="index.html" class="navbar-brand"><img src="assets/eyelead-logo-nav.png" alt="Eyelead" class="navbar-logo"></a>
      <button type="button" class="navbar-toggle" id="navToggle" aria-label="Abrir menu" aria-expanded="false" aria-controls="navLinks">
        <span></span><span></span><span></span>
      </button>
      <div class="navbar-links" id="navLinks">
        <a href="sobre.html">Sobre a Eyelead</a>
        <a href="apps.html">Nossos Apps</a>
        <a href="contato.html">Contato</a>
        <a href="apps.html" class="btn btn-primary btn-small">Ver Apps</a>
      </div>
    </div>
  </nav>

  <div class="page-title">
    <h1>Nossos Apps</h1>
    <p>Aplicativos inteligentes para simplificar o seu dia a dia.</p>
  </div>

  <section class="apps">
    <div class="app-grid">
      <div class="app-card">
        <img src="assets/mycofree-icon.png" alt="MyCofree" class="app-icon">
        <div class="app-info">
          <h3>MyCofree</h3>
          <p>Controle financeiro pessoal, simples e inteligente.</p>
        </div>
        <a href="https://gustavotinelli.github.io/mycofree-web/" target="_blank" rel="noopener noreferrer" class="btn btn-primary btn-small">Acessar</a>
      </div>
      <div class="app-card app-card-placeholder">
        <p>Em breve, novos apps</p>
      </div>
    </div>
  </section>

  <footer class="site-footer">
    <div class="social-links">
      <a href="https://www.youtube.com/@EyePod_Cast" target="_blank" rel="noopener noreferrer">YouTube</a>
      <a href="https://open.spotify.com/show/6eXKSgk2ZopXclbmYRGQb7" target="_blank" rel="noopener noreferrer">Spotify</a>
      <a href="https://www.instagram.com/eyeleadbr/" target="_blank" rel="noopener noreferrer">Instagram</a>
    </div>
    <p>&copy; <span id="year"></span> Eyelead · <a href="privacidade.html">Política de Privacidade</a> · <a href="contato.html">Contato</a></p>
  </footer>

  <script src="script.js"></script>
</body>
</html>
```

- [ ] **Step 2: Verify**

```bash
grep -c "app-card" "D:/EyeLead/eyelead-website/apps.html"
grep -c "<h1>Nossos Apps</h1>" "D:/EyeLead/eyelead-website/apps.html"
grep -c "Em breve, novos apps" "D:/EyeLead/eyelead-website/apps.html"
```
Expected: `2`, `1`, `1`.

- [ ] **Step 3: Commit (local only)**

```bash
cd "D:/EyeLead/eyelead-website"
git add apps.html
git commit -m "feat: add the Nossos Apps page"
```

---

### Task 8: Create contato.html

**Files:**
- Create: `D:\EyeLead\eyelead-website\contato.html`

- [ ] **Step 1: Write the file**

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Contato — Eyelead</title>
  <meta name="description" content="Fale com a Eyelead por e-mail ou nas redes sociais.">
  <meta property="og:title" content="Contato — Eyelead">
  <meta property="og:description" content="Fale com a Eyelead por e-mail ou nas redes sociais.">
  <meta property="og:image" content="https://eyelead.com.br/assets/eyelead-logo.png">
  <meta property="og:url" content="https://eyelead.com.br/contato.html">
  <meta property="og:type" content="website">
  <link rel="icon" type="image/png" href="assets/favicon.png">
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <nav class="navbar">
    <div class="navbar-inner">
      <a href="index.html" class="navbar-brand"><img src="assets/eyelead-logo-nav.png" alt="Eyelead" class="navbar-logo"></a>
      <button type="button" class="navbar-toggle" id="navToggle" aria-label="Abrir menu" aria-expanded="false" aria-controls="navLinks">
        <span></span><span></span><span></span>
      </button>
      <div class="navbar-links" id="navLinks">
        <a href="sobre.html">Sobre a Eyelead</a>
        <a href="apps.html">Nossos Apps</a>
        <a href="contato.html">Contato</a>
        <a href="apps.html" class="btn btn-primary btn-small">Ver Apps</a>
      </div>
    </div>
  </nav>

  <div class="page">
    <h1>Contato</h1>
    <p>
      Tem alguma dúvida, sugestão ou quer conversar com a gente? Estamos
      à disposição pelos canais abaixo.
    </p>

    <h2>E-mail</h2>
    <p><a href="mailto:contato@eyelead.com.br">contato@eyelead.com.br</a></p>

    <h2>Redes sociais</h2>
    <p>
      <a href="https://www.youtube.com/@EyePod_Cast" target="_blank" rel="noopener noreferrer">YouTube</a> ·
      <a href="https://open.spotify.com/show/6eXKSgk2ZopXclbmYRGQb7" target="_blank" rel="noopener noreferrer">Spotify</a> ·
      <a href="https://www.instagram.com/eyeleadbr/" target="_blank" rel="noopener noreferrer">Instagram</a>
    </p>
  </div>

  <footer class="site-footer">
    <div class="social-links">
      <a href="https://www.youtube.com/@EyePod_Cast" target="_blank" rel="noopener noreferrer">YouTube</a>
      <a href="https://open.spotify.com/show/6eXKSgk2ZopXclbmYRGQb7" target="_blank" rel="noopener noreferrer">Spotify</a>
      <a href="https://www.instagram.com/eyeleadbr/" target="_blank" rel="noopener noreferrer">Instagram</a>
    </div>
    <p>&copy; <span id="year"></span> Eyelead · <a href="privacidade.html">Política de Privacidade</a> · <a href="contato.html">Contato</a></p>
  </footer>

  <script src="script.js"></script>
</body>
</html>
```

- [ ] **Step 2: Verify**

```bash
grep -c "contato@eyelead.com.br" "D:/EyeLead/eyelead-website/contato.html"
grep -c "<h1>Contato</h1>" "D:/EyeLead/eyelead-website/contato.html"
grep -c "open.spotify.com\|instagram.com/eyeleadbr" "D:/EyeLead/eyelead-website/contato.html"
```
Expected: at least `2` (mailto link + display text), `1`, at least `4` (each appears once in the body content plus once in the footer).

- [ ] **Step 3: Commit (local only)**

```bash
cd "D:/EyeLead/eyelead-website"
git add contato.html
git commit -m "feat: add the Contato page"
```

---

### Task 9: Local verification pass

**Files:** none (verification only)

- [ ] **Step 1: Serve the site locally**

```bash
cd "D:/EyeLead/eyelead-website"
python -m http.server 8080
```
(run in the background)

- [ ] **Step 2: Headless-browser check of all five pages, desktop and mobile**

Use `puppeteer-core` (already installed at `C:\Users\Tina\AppData\Local\Temp\claude\D--MyCofree\75ff24bc-b763-43d4-b9c8-6b406a917530\scratchpad\webcheck\node_modules` — write scripts there and run from that directory, or `npm install puppeteer-core` fresh elsewhere if unavailable). Chrome at `C:\Program Files (x86)\Google\Chrome\Application\chrome.exe`.

```js
const puppeteer = require('puppeteer-core');

(async () => {
  const browser = await puppeteer.launch({
    executablePath: 'C:\\Program Files (x86)\\Google\\Chrome\\Application\\chrome.exe',
    headless: 'new',
    args: ['--no-sandbox', '--disable-gpu'],
  });

  const pages = ['/', '/sobre.html', '/apps.html', '/contato.html', '/privacidade.html'];

  for (const viewport of [{ name: 'desktop', width: 1280, height: 800 }, { name: 'mobile', width: 375, height: 812 }]) {
    for (const path of pages) {
      const page = await browser.newPage();
      await page.setViewport({ width: viewport.width, height: viewport.height });
      const issues = [];
      page.on('pageerror', (err) => issues.push(`[pageerror] ${err.message}`));
      page.on('response', (res) => {
        if (res.status() >= 400) issues.push(`[http${res.status()}] ${res.url()}`);
      });
      await page.goto(`http://localhost:8080${path}`, { waitUntil: 'networkidle2', timeout: 15000 });
      const fname = `v2_${viewport.name}_${path === '/' ? 'home' : path.replace(/\W/g, '_')}.png`;
      await page.screenshot({ path: fname, fullPage: true });
      console.log(`--- ${viewport.name} ${path} ---`);
      console.log(issues.join('\n') || '(no issues)');
      await page.close();
    }
  }

  await browser.close();
})();
```

Expected: `(no issues)` for every one of the 10 combinations (5 pages × 2 viewports). Report every screenshot's exact file path.

- [ ] **Step 3: Verify the mobile hamburger toggle actually works**

Add a focused check (new script or extend the one above) that on mobile viewport:
1. Navigates to `http://localhost:8080/`
2. Confirms `#navLinks` is NOT visible by default (e.g. `page.evaluate(() => getComputedStyle(document.getElementById('navLinks')).display)` should be `'none'`)
3. Clicks `#navToggle` (`await page.click('#navToggle')`)
4. Confirms `#navLinks` now IS visible (`display` should be `'flex'`) and `#navToggle`'s `aria-expanded` attribute is now `'true'`

Report the exact before/after `display` values and the `aria-expanded` value.

- [ ] **Step 4: Verify all internal nav links resolve (no 404s)**

```bash
for path in / /sobre.html /apps.html /contato.html /privacidade.html; do
  code=$(curl -s -o /dev/null -w "%{http_code}" "http://localhost:8080$path")
  echo "$path -> $code"
done
```
Expected: `200` for all five.

- [ ] **Step 5: Stop the local server**

Find and stop the `python -m http.server` process from Step 1.

No commit for this task (verification only).

---

### Task 10: Push everything to origin

**Files:** none (git operation only)

- [ ] **Step 1: Confirm the full local commit history for this phase**

```bash
cd "D:/EyeLead/eyelead-website"
git log --oneline -10
git status --short
```
Expected: a clean working tree, and the 8 commits from Tasks 1-8 all present, none of them pushed yet (this is the first push of this batch).

- [ ] **Step 2: Push**

```bash
cd "D:/EyeLead/eyelead-website"
git push origin main
```
Expected: push succeeds, shows the range of new commits landing on `main`.

- [ ] **Step 3: Wait briefly for GitHub Pages to rebuild, then confirm the live site picked up the change**

```bash
i=0; until curl -s "http://eyelead.com.br/sobre.html" | grep -q "Sobre a Eyelead"; do i=$((i+1)); if [ $i -gt 12 ]; then echo "timeout waiting for deploy"; break; fi; sleep 10; done
curl -s -o /dev/null -w "sobre.html: %{http_code}\n" "http://eyelead.com.br/sobre.html"
curl -s -o /dev/null -w "apps.html: %{http_code}\n" "http://eyelead.com.br/apps.html"
curl -s -o /dev/null -w "contato.html: %{http_code}\n" "http://eyelead.com.br/contato.html"
```
Expected: all three return `200` (these are brand-new pages that didn't exist on the live site before this push — a 404 here means the deploy hasn't finished yet, not necessarily a real problem; retry the wait loop once before treating it as an issue).

---

### Task 11: Final live verification

**Files:** none (verification only)

- [ ] **Step 1: Headless-browser check of all five live pages, desktop and mobile**

Adapt Task 9 Step 2's script to point at `http://eyelead.com.br` (or `https://` if HTTPS enforcement has completed by now — check with `curl -sI https://eyelead.com.br/` first; use whichever scheme currently returns `200`) instead of `localhost:8080`. Report exact console output and screenshot paths for all 10 combinations.

- [ ] **Step 2: Confirm the nav bar's mobile hamburger toggle works on the live site**

Repeat Task 9 Step 3's check against the live URL instead of localhost.

- [ ] **Step 3: Confirm every outbound link still resolves**

```bash
curl -s -o /dev/null -w "MyCofree: %{http_code}\n" "https://gustavotinelli.github.io/mycofree-web/"
curl -s -o /dev/null -w "YouTube: %{http_code}\n" "https://www.youtube.com/@EyePod_Cast"
curl -s -o /dev/null -w "Spotify: %{http_code}\n" "https://open.spotify.com/show/6eXKSgk2ZopXclbmYRGQb7"
curl -s -o /dev/null -w "Instagram: %{http_code}\n" "https://www.instagram.com/eyeleadbr/"
```
Expected: all four return `200`.

- [ ] **Step 4: Report completion**

Summarize: which pages are live, confirmation all links/toggle work, and a reminder that further visual/cosmetic polish (beyond nav/font/pages) remains explicitly out of scope for this phase.
