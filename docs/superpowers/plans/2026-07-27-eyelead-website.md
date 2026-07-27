# Eyelead Company Website Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Ship the v1 Eyelead company website — a single-page static site at `eyelead.com.br` introducing Eyelead, showcasing MyCofree, mentioning EyePod, and linking a minimal privacy policy — per `docs/superpowers/specs/2026-07-27-eyelead-website-design.md`.

**Architecture:** Plain static HTML/CSS/JS, no framework, no build step. Three files at the repo root (`index.html`, `privacidade.html`, `styles.css`) plus a tiny `script.js` (one line: fill in the copyright year) and an `assets/` folder for images. Hosted on GitHub Pages, serving directly from `main` — pushing to `main` is the deploy, no CI needed.

**Tech Stack:** HTML5, CSS3 (flexbox, media queries), vanilla JS. GitHub Pages + custom domain via DNS A records. Python 3 + Pillow for one-time image asset prep (already installed and used earlier in this session).

**Note on "testing" in this plan:** This spec has no automated test suite (there's no logic to unit-test in a static informational page — see the spec's Testing section). Every task below substitutes a concrete, structural verification step (grep for expected markup, or an exact visual check in a real browser) in place of the write-test/run-test cycle used for code with a test framework.

---

## File Structure

```
eyelead-website/
├── index.html              # The whole landing page
├── privacidade.html         # Privacy policy page
├── styles.css                # All styling for both pages
├── script.js                  # One line: sets the footer's copyright year
├── CNAME                       # GitHub Pages custom domain file (contains "eyelead.com.br")
├── assets/
│   ├── eyelead-logo.png    # Hero logo (copied from D:\EyeLead\Eyelead_wall.png)
│   ├── favicon.png           # Cropped "E + arrow" mark, no text (generated in Task 1)
│   └── mycofree-icon.png  # MyCofree app card icon (copied from the MyCofree repo)
├── .gitignore                  # Already exists (excludes .superpowers/)
└── docs/superpowers/
    ├── specs/2026-07-27-eyelead-website-design.md   # Already exists
    └── plans/2026-07-27-eyelead-website.md            # This file
```

---

### Task 1: Prepare image assets

**Files:**
- Create: `D:\EyeLead\eyelead-website\assets\eyelead-logo.png`
- Create: `D:\EyeLead\eyelead-website\assets\mycofree-icon.png`
- Create: `D:\EyeLead\eyelead-website\assets\favicon.png`

- [ ] **Step 1: Create the assets folder and copy the two source images**

Run:
```bash
mkdir -p "D:/EyeLead/eyelead-website/assets"
cp "D:/EyeLead/Eyelead_wall.png" "D:/EyeLead/eyelead-website/assets/eyelead-logo.png"
cp "D:/MyCofree/mycofree/web/icons/Icon-192.png" "D:/EyeLead/eyelead-website/assets/mycofree-icon.png"
```

Expected: both files exist. Verify with:
```bash
ls -la "D:/EyeLead/eyelead-website/assets"
```
Expected output lists `eyelead-logo.png` and `mycofree-icon.png` with non-zero sizes.

- [ ] **Step 2: Generate the favicon — a square crop of just the "E" mark + gold arrow, no wordmark text**

Run this exact Python script (uses Pillow, already installed in this environment):

```python
from PIL import Image
img = Image.open(r"D:\EyeLead\Eyelead_wall.png").convert("RGB")
bg = (25, 25, 25)
# Precise crop bounds for the E + arrow, excluding the "YELEAD" wordmark text
# (determined by pixel-scanning the source logo — see brainstorming session).
crop = img.crop((92, 270, 290, 641))
size = max(crop.size)
canvas = Image.new("RGB", (size, size), bg)
offset = ((size - crop.width) // 2, (size - crop.height) // 2)
canvas.paste(crop, offset)
canvas = canvas.resize((512, 512), Image.LANCZOS)
canvas.save(r"D:\EyeLead\eyelead-website\assets\favicon.png")
print("saved", canvas.size)
```

Save it to a temp file and run, e.g.:
```bash
python3 -c "
from PIL import Image
img = Image.open(r'D:\EyeLead\Eyelead_wall.png').convert('RGB')
bg = (25, 25, 25)
crop = img.crop((92, 270, 290, 641))
size = max(crop.size)
canvas = Image.new('RGB', (size, size), bg)
offset = ((size - crop.width) // 2, (size - crop.height) // 2)
canvas.paste(crop, offset)
canvas = canvas.resize((512, 512), Image.LANCZOS)
canvas.save(r'D:\EyeLead\eyelead-website\assets\favicon.png')
print('saved', canvas.size)
"
```

Expected output: `saved (512, 512)`

- [ ] **Step 3: Verify all three assets exist with reasonable file sizes**

Run:
```bash
ls -la "D:/EyeLead/eyelead-website/assets"
```
Expected: three `.png` files, each larger than 1KB (none are empty/corrupt).

- [ ] **Step 4: Commit**

```bash
cd "D:/EyeLead/eyelead-website"
git add assets/
git commit -m "chore: add logo, favicon, and MyCofree icon assets"
```

---

### Task 2: Write the landing page (index.html)

**Files:**
- Create: `D:\EyeLead\eyelead-website\index.html`

- [ ] **Step 1: Write the file**

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Eyelead — Looking for the future</title>
  <meta name="description" content="A Eyelead cria aplicativos incríveis com inteligência artificial. Conheça o MyCofree, nosso primeiro app.">
  <meta property="og:title" content="Eyelead — Looking for the future">
  <meta property="og:description" content="A Eyelead cria aplicativos incríveis com inteligência artificial.">
  <meta property="og:image" content="assets/eyelead-logo.png">
  <meta property="og:type" content="website">
  <link rel="icon" type="image/png" href="assets/favicon.png">
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <header class="hero">
    <img src="assets/eyelead-logo.png" alt="Eyelead — Looking for the future" class="hero-logo">
    <a href="#apps" class="btn btn-primary">Ver nossos apps</a>
  </header>

  <section class="about">
    <div class="label">SOBRE A EYELEAD</div>
    <p>
      A Eyelead cria aplicativos incríveis com inteligência artificial,
      pensados para simplificar o dia a dia das pessoas. Nosso primeiro
      app, o MyCofree, já está disponível — e é só o começo.
    </p>
  </section>

  <section class="apps" id="apps">
    <div class="label">NOSSOS APPS</div>
    <div class="app-grid">
      <div class="app-card">
        <img src="assets/mycofree-icon.png" alt="MyCofree" class="app-icon">
        <div class="app-info">
          <h3>MyCofree</h3>
          <p>Controle financeiro pessoal, simples e inteligente.</p>
        </div>
        <a href="https://gustavotinelli.github.io/mycofree-web/" class="btn btn-primary btn-small">Acessar</a>
      </div>
      <div class="app-card app-card-placeholder">
        <p>Em breve, novos apps</p>
      </div>
    </div>
  </section>

  <section class="eyepod">
    <a href="https://www.youtube.com/@EyePod_Cast" class="eyepod-pill">
      🎙 Ouça também o <strong>EyePod</strong>, nosso podcast →
    </a>
  </section>

  <footer class="site-footer">
    <p>&copy; <span id="year"></span> Eyelead · <a href="privacidade.html">Política de Privacidade</a> · <a href="mailto:contato@eyelead.com.br">Contato</a></p>
  </footer>

  <script src="script.js"></script>
</body>
</html>
```

- [ ] **Step 2: Verify the file has the expected structure**

Run:
```bash
grep -c "app-card" "D:/EyeLead/eyelead-website/index.html"
grep -c "youtube.com/@EyePod_Cast" "D:/EyeLead/eyelead-website/index.html"
grep -c "contato@eyelead.com.br" "D:/EyeLead/eyelead-website/index.html"
```
Expected: `2` (two `.app-card` elements — MyCofree + placeholder), `1`, `1`.

- [ ] **Step 3: Commit**

```bash
cd "D:/EyeLead/eyelead-website"
git add index.html
git commit -m "feat: add the landing page markup"
```

---

### Task 3: Write the privacy policy page (privacidade.html)

**Files:**
- Create: `D:\EyeLead\eyelead-website\privacidade.html`

- [ ] **Step 1: Write the file**

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
    <p>&copy; <span id="year"></span> Eyelead · <a href="index.html">Início</a></p>
  </footer>

  <script src="script.js"></script>
</body>
</html>
```

- [ ] **Step 2: Verify the file has the expected structure**

Run:
```bash
grep -c "contato@eyelead.com.br" "D:/EyeLead/eyelead-website/privacidade.html"
grep -c "back-link" "D:/EyeLead/eyelead-website/privacidade.html"
```
Expected: `1`, `1`.

- [ ] **Step 3: Commit**

```bash
cd "D:/EyeLead/eyelead-website"
git add privacidade.html
git commit -m "feat: add the privacy policy page"
```

---

### Task 4: Write the stylesheet and the year-fill script

**Files:**
- Create: `D:\EyeLead\eyelead-website\styles.css`
- Create: `D:\EyeLead\eyelead-website\script.js`

- [ ] **Step 1: Write styles.css**

```css
:root {
  --bg: #191919;
  --bg-elevated: #242424;
  --text: #ffffff;
  --text-muted: #aaaaaa;
  --text-dim: #666666;
  --accent: #E2A300;
  --border: #2a2a2a;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  background: var(--bg);
  color: var(--text);
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  line-height: 1.6;
}

a {
  color: inherit;
}

.label {
  font-size: 0.75rem;
  color: var(--accent);
  letter-spacing: 1px;
  text-align: center;
  margin-bottom: 12px;
}

.btn {
  display: inline-block;
  padding: 10px 24px;
  border-radius: 6px;
  font-weight: bold;
  font-size: 0.9rem;
  text-decoration: none;
}

.btn-primary {
  background: var(--accent);
  color: var(--bg);
}

.btn-small {
  padding: 8px 16px;
  font-size: 0.8rem;
}

/* Hero */
.hero {
  text-align: center;
  padding: 60px 20px 40px;
  border-bottom: 1px solid var(--border);
}

.hero-logo {
  max-width: 320px;
  width: 100%;
  height: auto;
  margin-bottom: 24px;
}

/* About */
.about {
  max-width: 560px;
  margin: 0 auto;
  padding: 48px 24px;
  text-align: center;
}

.about p {
  color: #cccccc;
  font-size: 1rem;
}

/* Apps */
.apps {
  padding: 24px 24px 48px;
}

.app-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  justify-content: center;
  max-width: 900px;
  margin: 0 auto;
}

.app-card {
  background: var(--bg-elevated);
  border: 1px solid var(--border);
  border-radius: 12px;
  padding: 20px;
  display: flex;
  align-items: center;
  gap: 16px;
  width: 100%;
  max-width: 420px;
}

.app-icon {
  width: 56px;
  height: 56px;
  border-radius: 12px;
  flex-shrink: 0;
}

.app-info {
  flex: 1;
}

.app-info h3 {
  font-size: 1rem;
  margin-bottom: 4px;
}

.app-info p {
  font-size: 0.85rem;
  color: var(--text-muted);
}

.app-card-placeholder {
  border: 1px dashed #3a3a3a;
  background: transparent;
  justify-content: center;
  color: #555555;
  font-size: 0.85rem;
  text-align: center;
}

/* EyePod */
.eyepod {
  text-align: center;
  padding: 16px 24px 48px;
}

.eyepod-pill {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  background: var(--bg-elevated);
  border-radius: 20px;
  padding: 10px 20px;
  font-size: 0.85rem;
  color: #cccccc;
  text-decoration: none;
}

.eyepod-pill strong {
  color: var(--accent);
}

/* Footer */
.site-footer {
  border-top: 1px solid var(--border);
  padding: 20px 24px;
  text-align: center;
  font-size: 0.75rem;
  color: var(--text-dim);
}

.site-footer a {
  color: #888888;
}

/* Privacy page */
.page {
  max-width: 640px;
  margin: 0 auto;
  padding: 48px 24px;
}

.page h1 {
  font-size: 1.5rem;
  margin-bottom: 8px;
}

.page h2 {
  font-size: 1.1rem;
  margin-top: 32px;
  margin-bottom: 8px;
  color: var(--accent);
}

.page p {
  color: #cccccc;
  font-size: 0.95rem;
}

.back-link {
  display: inline-block;
  margin-bottom: 24px;
  font-size: 0.85rem;
  color: #888888;
  text-decoration: none;
}

/* Responsive */
@media (max-width: 480px) {
  .hero {
    padding: 40px 16px 32px;
  }
  .app-card {
    flex-direction: column;
    text-align: center;
  }
}
```

- [ ] **Step 2: Write script.js**

```js
document.getElementById('year').textContent = new Date().getFullYear();
```

- [ ] **Step 3: Verify both files exist and are non-empty**

Run:
```bash
wc -l "D:/EyeLead/eyelead-website/styles.css" "D:/EyeLead/eyelead-website/script.js"
```
Expected: `styles.css` has 150+ lines, `script.js` has 1 line.

- [ ] **Step 4: Commit**

```bash
cd "D:/EyeLead/eyelead-website"
git add styles.css script.js
git commit -m "feat: add stylesheet and copyright-year script"
```

---

### Task 5: Local verification pass

**Files:** none (verification only)

- [ ] **Step 1: Serve the site locally**

Run in the background:
```bash
cd "D:/EyeLead/eyelead-website"
python -m http.server 8080
```

- [ ] **Step 2: Check both pages load with no broken links, using a headless browser**

Write and run this Node script (uses `puppeteer-core` — already installed at `C:\Users\Tina\AppData\Local\Temp\claude\D--MyCofree\75ff24bc-b763-43d4-b9c8-6b406a917530\scratchpad\webcheck\node_modules`, or install fresh with `npm install puppeteer-core` in a scratch directory if that path is unavailable):

```js
const puppeteer = require('puppeteer-core');

(async () => {
  const browser = await puppeteer.launch({
    executablePath: 'C:\\Program Files (x86)\\Google\\Chrome\\Application\\chrome.exe',
    headless: 'new',
    args: ['--no-sandbox', '--disable-gpu'],
  });

  for (const path of ['/', '/privacidade.html']) {
    const page = await browser.newPage();
    const issues = [];
    page.on('pageerror', (err) => issues.push(`[pageerror] ${err.message}`));
    page.on('response', (res) => {
      if (res.status() >= 400) issues.push(`[http${res.status()}] ${res.url()}`);
    });
    await page.goto(`http://localhost:8080${path}`, { waitUntil: 'networkidle2', timeout: 15000 });
    await page.screenshot({ path: `check${path.replace(/\W/g, '_')}.png`, fullPage: true });
    console.log(`--- ${path} ---`);
    console.log(issues.join('\n') || '(no issues)');
    await page.close();
  }

  await browser.close();
})();
```

Expected output: `(no issues)` for both `/` and `/privacidade.html` — no 4xx/5xx responses, no page errors. Two screenshot files are produced; view them and confirm the hero/about/apps/eyepod/footer sections render correctly on `/`, and the privacy page text renders on `/privacidade.html`.

- [ ] **Step 3: Check mobile viewport rendering**

Add `await page.setViewport({ width: 375, height: 812 });` before the `page.goto` call in the script above, re-run, and confirm in the new screenshots that the app card stacks vertically (per the `@media (max-width: 480px)` rule in styles.css) and nothing overflows horizontally.

- [ ] **Step 4: Stop the local server**

Find and stop the `python -m http.server` process started in Step 1.

No commit for this task — it's verification only, no files changed.

---

### Task 6: Create the GitHub repo and push

**Files:**
- Create: `D:\EyeLead\eyelead-website\CNAME`

- [ ] **Step 1: Write the CNAME file**

```
eyelead.com.br
```

(Exactly that one line, no `http://`, no trailing slash — this is GitHub Pages' custom domain config file.)

- [ ] **Step 2: Commit the CNAME file**

```bash
cd "D:/EyeLead/eyelead-website"
git add CNAME
git commit -m "chore: add CNAME for the eyelead.com.br custom domain"
```

- [ ] **Step 3: Create the GitHub repository**

Run (using the `gh` CLI at `D:/Tools/gh/gh.exe`, per this environment's established path):
```bash
D:/Tools/gh/gh.exe repo create GustavoTinelli/eyelead-website --public --description "Eyelead company website (eyelead.com.br)"
```
Expected output includes the new repo's URL, e.g. `https://github.com/GustavoTinelli/eyelead-website`.

- [ ] **Step 4: Add the remote and push**

```bash
cd "D:/EyeLead/eyelead-website"
git remote add origin https://github.com/GustavoTinelli/eyelead-website.git
git push -u origin main
```
Expected: push succeeds, output shows `main -> main`.

- [ ] **Step 5: Verify on GitHub**

```bash
D:/Tools/gh/gh.exe repo view GustavoTinelli/eyelead-website --web
```
(Or just confirm via `gh api repos/GustavoTinelli/eyelead-website --jq '.html_url'` if opening a browser isn't convenient.) Confirm the repo shows all committed files: `index.html`, `privacidade.html`, `styles.css`, `script.js`, `CNAME`, `assets/`, `docs/`.

---

### Task 7: Enable GitHub Pages and configure DNS

**Files:** none (repo/DNS configuration only)

- [ ] **Step 1: Enable GitHub Pages via the API**

```bash
D:/Tools/gh/gh.exe api repos/GustavoTinelli/eyelead-website/pages -X POST -f "source[branch]=main" -f "source[path]=/"
```
Expected: a JSON response describing the new Pages site, including a `"status"` field (typically `"building"` initially).

- [ ] **Step 2: Set the custom domain via the API**

```bash
D:/Tools/gh/gh.exe api repos/GustavoTinelli/eyelead-website/pages -X PUT -f "cname=eyelead.com.br"
```
Expected: `204` no-content response (success) or a JSON body confirming the CNAME was set. If this errors because Pages hasn't finished its first build yet, wait ~30 seconds and retry.

- [ ] **Step 3: Tell the user to add DNS A records**

This step cannot be automated — the user must add these at their domain registrar for `eyelead.com.br` (apex domain, so A records, not CNAME):

```
Type: A    Host: @    Value: 185.199.108.153
Type: A    Host: @    Value: 185.199.109.153
Type: A    Host: @    Value: 185.199.110.153
Type: A    Host: @    Value: 185.199.111.153
```

Report to the user: "Please add these four A records at your registrar for eyelead.com.br, then let me know so I can verify DNS propagation and enable HTTPS."

- [ ] **Step 4: After the user confirms DNS is added, verify propagation**

```bash
nslookup eyelead.com.br
```
Expected (once propagated — can take anywhere from minutes to a few hours): the four A records above appear in the output. If they don't yet, wait and retry rather than treating it as a failure — DNS propagation timing is outside this project's control.

- [ ] **Step 5: Enable HTTPS enforcement once DNS resolves correctly**

```bash
D:/Tools/gh/gh.exe api repos/GustavoTinelli/eyelead-website/pages -X PUT -F "https_enforced=true"
```
Note: GitHub needs to auto-provision a certificate for the custom domain first, which only starts after DNS resolves — this step may need to be retried after DNS has propagated and GitHub has had time to issue the cert (can take up to a few hours after DNS is correct).

---

### Task 8: Final live verification

**Files:** none (verification only)

- [ ] **Step 1: Confirm the live site loads over HTTPS with no console errors**

Adapt the Task 5 Step 2 script to point at `https://eyelead.com.br/` and `https://eyelead.com.br/privacidade.html` instead of `localhost:8080`. Run it and confirm `(no issues)` for both pages.

- [ ] **Step 2: Confirm the MyCofree link and EyePod link both resolve**

```bash
curl -s -o /dev/null -w "MyCofree: %{http_code}\n" "https://gustavotinelli.github.io/mycofree-web/"
curl -s -o /dev/null -w "EyePod: %{http_code}\n" "https://www.youtube.com/@EyePod_Cast"
```
Expected: both return `200`.

- [ ] **Step 3: Report completion to the user**

Summarize: repo URL, live site URL, confirmation that both pages load cleanly, and a reminder that font/spacing/visual polish was explicitly deferred to a follow-up pass (per the approved design).
