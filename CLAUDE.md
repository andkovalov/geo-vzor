# Delta G — design system & project conventions

B2C/B2B web for Delta G s.r.o. (geodetická firma, Praha). Static HTML/CSS/vanilla JS, no build step.
Live: https://deltag.vercel.app/ (auto-deploys from GitHub `andkovalov/deltag`, branch `main`).

**After every change: `git add -A && git commit && git push`** (user's standing instruction; commit messages in English + Claude co-author line).

## Files

| File | Purpose |
|---|---|
| `index.html` | B2C landing (hero, services scroll-stack, why-us, process, tech scanner, partners marquee, B2C reviews, lead form + quiz modal, contact, cookie banner) |
| `pro-firmy.html` | B2B catalog page (industry terminology allowed), own inline lead form (#poptavka) |
| `kariera.html` | Careers page + application form |
| `sluzby-*.html` | 6 B2C SEO service pages (rozdeleni-pozemku, vytyceni-hranic, zamereni-pro-projekt, vytyceni-stavby, geometricky-plan-kolaudace, pasport-stavby) — shared compact template: breadcrumb, H1=query, kdy-grid, 4 steps, price-box (TODO prices), FAQ + FAQPage JSON-LD, CTA to index.html#poptavka. Regenerate all via one script when the template changes. |
| `kontakt.html` | Contact page (header "Kontakt" points here on every page): contact cards, ÚOZI badge, B2C/B2B CTA cards, map |
| `zasady-ochrany-osobnich-udaju.html` | GDPR privacy policy (noindex, excluded from sitemap, disallowed in robots.txt) |
| `sitemap.xml`, `robots.txt` | 8 indexable URLs on vercel domain; regenerate when adding pages |
| `favicon.svg`, `apple-touch-icon.png`, `og-image.png` | brand assets (white swoosh on blue #0033FB) |
| `assets/logos/*` | partner logos (12, used in marquee) |

Each page is **self-contained**: own `<style>` block (copy tokens + needed components), own `<script>`. No shared CSS file — keep it that way unless the user asks.

## Tokens

```css
:root{
  --blue:#0033FB;     /* accent, buttons, links */
  --yellow:#FBED51;   /* hover/highlight accent, ::selection */
  --ink:#101828;      /* primary text */
  --ink-2:#344054;    /* secondary text */
  --g500:#667085;     /* muted text, labels */
  --g300:#D0D5DD;     /* borders */
  --g100:#F2F4F7;     /* section/page tint backgrounds */
  --radius:14px;
  --header-h:72px;
}
```

- Font: **Rajdhani** 400/500/600/700 — **self-hosted** in `assets/fonts/` (8 woff2: latin + latin-ext per weight). Every page: 4 `<link rel="preload" as="font" crossorigin>` (400/700 × latin/latin-ext) + inline `@font-face` block with unicode-range at the top of `<style>`. Never link fonts.googleapis.com (render-blocking, was a PSI finding). Body 18px, line-height 1.55.
- `::selection{background:var(--yellow);color:var(--ink)}` on every page.
- Sections alternate white / `--g100`. Section padding `clamp(3.5rem,9vh,6.5rem)` (more for hero-like).
- Container: `.wrap{width:min(1180px,100% - 2.5rem);margin-inline:auto}`.

## Type scale

- h1 hero: `clamp(2.4rem,5.6–6.6vw,4–4.6rem)`; accent word wrapped in `<em>` colored `--blue`.
- `.h2`: `clamp(1.9rem,4vw,2.8rem)` (landing uses up to 3.1rem).
- `.eyebrow`: uppercase 0.82rem, letter-spacing .22em, blue, with 26px dash `::before`.
- `.lead`: `clamp(1.05rem,1.6vw,1.25rem)`, color `--ink-2`, max-width ~46–52ch.
- Forced line breaks in leads: wrap lines in `<span class="line">` + `@media(min-width:860px){.lead:has(.line){max-width:none}.lead .line{display:block}}` (mobile falls back to natural wrap).

## Components (copy from index.html)

- **Buttons**: `.btn` uppercase 700; `.btn-primary` blue→**yellow bg + ink text** on hover (no shadows!); `.btn-ghost` 1.5px `--g300` border.
- **Header (corporate, identical on every page)**: fixed, 72px, `rgba(255,255,255,.86)` + backdrop blur. Nav: `Služby ▾` (dropdown with the 4 service pages + "Přehled všech služeb →"), `Pro firmy a investory`, `Kariéra`, `Kontakt`; current page gets `aria-current="page"` (blue). Dropdown: CSS-only `:hover/:focus-within`, invisible bridge via `.nav-item::after`. `.header-actions` = phone (filled icon, ≥1240px) + page-appropriate CTA. Burger <980px; mobile menu uses `.menu-group` headers + indented `.menu-sub` links.
- **Logo**: inline SVG, 3 paths — wordmark `#101828`, swoosh `class="logo-swoosh"` `#0033FB`, arc `#101828`. Hover: springy flick `translate(2px,-3px) rotate(-5deg)`, cubic-bezier(.34,1.56,.64,1). Never recolor swoosh to yellow on white.
- **Cards**: 1px `--g300` border, `--radius`, white bg. No hover lift on info cards (user removed it).
- **Icon chips**: 46–48px rounded square, blue bg, white strokes only (no yellow strokes inside icons).
- **Forms** (`.lead-form`): 2-col grid ≥640px, `.full` spans; inputs 1.5px border, focus blue ring `0 0 0 3px rgba(0,51,251,.12)`; consent checkbox `align-items:center`; honeypot `.hp-field` input name="company"; success block `.quiz-success`/`.form-success`.
- **Lead delivery**: Web3Forms. `const WEB3FORMS_KEY = ''` on each form-bearing page (index shared `sendLead`, pro-firmy `b2bForm`, kariera `careerForm`) → empty = demo mode (logs payload to console); paste the access key (from web3forms.com, tied to andkovalov@gmail.com) into all three to go live. POST JSON to `https://api.web3forms.com/submit` with `{access_key, subject, from_name, ...payload}`, expect `data.success`. Always `preventDefault`, `reportValidity`, honeypot check (`company` field).
- **Cookie banner**: opt-in GDPR; analytics load ONLY inside `loadAnalytics()` after consent; localStorage key `dg-cookie-consent` = `all|necessary`; footer link reopens it.
- **Partners marquee**: `.logo-track` = two identical `.logo-set`s (set has `padding-right` equal to gap, track gap 0) animated `translateX(-50%)` 40s linear; imgs uniform cells `clamp(120px,14vw,150px)×60`, `object-fit:contain`, grayscale→color, `loading="eager"`; no pause on hover; reduced-motion → static wrap, second set hidden.

## Animation rules (GSAP only — no Three.js, user rejected it)

- CDN: gsap 3.12.5 + ScrollTrigger from cdnjs.
- Guards on every page: `prefersReduced` media check + `hasGsap` typeof check; fallback = everything visible/static.
- Reveal pattern: `.reveal{opacity:0;transform:translateY(28px)}` + `ScrollTrigger.batch('.reveal',{start:'top 88%',once:true,...})`; reduced-motion sets visible immediately.
- Services scroll-stack (landing): cards `position:sticky` with cascading `top` (+0.9rem per card) and z-index 1..n; drawing layers `#svc-g{i}` toggled by per-card ScrollTriggers `start:'top center' end:'bottom center'` (non-overlapping!) with `overwrite:'auto'` tweens; `.draw` paths animate stroke-dashoffset.
- Tech scanner (landing): beam sweep ±56° yoyo 4.2s; readouts must not cause layout shift — fixed grid columns + `font-variant-numeric:tabular-nums`, value updates throttled (≥450ms).
- Subtle one-shot progressions (e.g. process line): toggle `.lit` classes via `gsap.delayedCall` stagger, CSS transitions do the drawing; numbers `--g300`→`--blue`.
- Tasteful, non-looping, never blocks reading. Hidden-tab note: rAF freezes GSAP — fine in real browsers, but preview screenshots may catch frame 0.

## Drawing/figure style (SVG)

Plot figure: light `--g100` panel, faint grid strokes `#D0D5DD/#EDF0F4`, parcel/houses stroked `--ink`/`--blue` 2.2–2.5px, dims `.dim` 15px `#344054` with `marker arr`, laser points `.pulse-pt` (scale+fade keyframe), yellow only for "stamp"/instrument accents with ink outline.

## Copy & tone

- Czech, natural business language. B2C pages: no jargon, benefits-first, fears addressed (úřady, ceny). B2B (`pro-firmy.html`): industry terminology OK (AZI, DTMŽ, vytyčovací sítě…).
- Facts: Delta G s.r.o., od 2004, IČO 02362538, DIČ CZ02362538, sídlo Tiskařská 10 / kancelář Polygrafická 262/3, Praha 10; +420 604 206 176; posta@deltag.cz; ÚOZI; map pin 50.078833,14.522639 (Google embed with `hl=cs`).
- Placeholders must be marked `<!-- TODO: ... -->` (prices to confirm, real reviews).

## SEO / meta (every page)

- Unique `<title>` + `meta description`; `<link rel="canonical" href="https://deltag.vercel.app/...">`; OG (`og:url`, `og:image` → absolute vercel URLs, locale `cs_CZ`) + `twitter:card summary_large_image`. Never use github.io URLs.
- JSON-LD: `LocalBusiness` (@id `…/#firma`, IČO, geo 50.078833/14.522639) on every page; service pages add `Service` with `provider:{"@id":"…#firma"}`. New page → add to sitemap.xml.
- Nav hover = selection style: `background:var(--yellow);color:var(--ink)` on `.main-nav` links and `.drop` items (no underline).
- **Footer (corporate, identical on every page incl. privacy)**: bg `#E4E7EC` (one step darker than `--g100`), 4 columns ≥1000px — logo+about / Služby (all 6 service pages) / Společnost / Kontakt — plus `.footer-bottom` bar (©, ÚOZI, index also keeps `#cookieSettingsLink`). When adding a service page: update footer Služby column, header dropdown + mobile menu on ALL pages, and sitemap.xml.
- favicon links: `favicon.svg` + `apple-touch-icon.png`. Privacy page: `noindex,follow`.

## New page checklist

1. Copy head boilerplate (fonts, favicon, meta) + tokens + needed component CSS from index.html.
2. Header with logo (full inline SVG + swoosh hover), `.header-actions`, back/nav links; footer with © line, Kariéra / Pro firmy / Zásady links as relevant.
3. GSAP CDN + guards + `.reveal` batch.
4. Forms → FORM_ENDPOINT pattern + GDPR consent linking to zásady.
5. Verify: no horizontal overflow at 375/768/1024/1440; header fits at ~920–1140; console clean.
6. Commit + push (auto-deploy).
