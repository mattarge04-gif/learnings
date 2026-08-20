# Site Teardown: Immo Otti

**URL:** https://immo-otti.ch/ueber-uns/team/ (analysierte Seite — nicht die Startseite)
**Built by:** loaded.ch (Footer: „Webdesign & Entwicklung — loaded.ch")
**Betreiber:** Inka AG / immo otti, Hüttenort 2, 6365 Kehrsiten NW
**Platform:** Astro v5.13.2, statisch gebaut, auf Netlify gehostet
**Date analyzed:** 2026-08-20
**Quelle:** Vom Nutzer eingefügtes HTML inkl. vollständig inlinetem CSS. Die beiden JS-Bundles
waren aus der Analyse-Umgebung nicht abrufbar (Netzwerk-Egress blockiert) — alle Aussagen zu
JS-Verhalten aus diesen Dateien sind als *abgeleitet* gekennzeichnet.

---

## Wichtiger Hinweis zur Quelle

Der eingefügte Quellcode ist die **Team-Unterseite** (`<title>Unser Team – Immo Otti</title>`,
Canonical `https://immo-otti.ch/ueber-uns/team/`), nicht die Startseite. Da Astro pro Seite das
CSS *aller* eingebundenen Komponenten inline schreibt, enthält dieses eine Dokument trotzdem:

- das globale Designsystem (`:root`-Tokens, Typo-Skala, Buttons, Container)
- Header + Navigation (global)
- Footer (global)
- das komplette Modal-Formularsystem (global)
- den Otti-AI-Floating-Banner (global)
- den Chat-Widget-Einstiegspunkt (global)
- die Team-Seite selbst

Nicht enthalten: die Sektionen der Startseite (`.hero`-Regel ist vorhanden, das Markup fehlt),
Immobilien-Listing, Blog, Bewertungsseite.

---

## Tech Stack (Confirmed from Source)

| Technology | Evidence | Purpose |
|---|---|---|
| **Astro 5.13.2** | `<meta name="generator" content="Astro v5.13.2">`, `data-astro-cid-*` Scoping-Attribute auf jedem Element | Static Site Generator, Komponenten-Scoping, Zero-JS by default |
| **Astro Islands** | `<astro-island uid="elRyh" client="visible" component-url="/_astro/ImmoOttiChat.DuM2Jw_g.js">` + inline Custom-Element-Definition | Partielle Hydration — nur das Chat-Widget wird JS-hydriert |
| **React (o. Preact)** | `renderer-url="/_astro/client.CSQ4PDHQ.js"` — Dateiname entspricht `@astrojs/react`s Client-Renderer | Rendert die `ImmoOttiChat`-Komponente |
| **Netlify Forms** | 7 versteckte `<form>` mit `name="dynamic-*"`, `<input type="hidden" name="form-name">`, `<input name="bot-field">` (Honeypot) | Formular-Backend ohne eigenen Server; Netlify parst die versteckten Forms beim Build |
| **Cloudinary** (2 Accounts) | `res.cloudinary.com/dihw9b8ce/...` und `res.cloudinary.com/dphbnwjtx/...`, jeweils mit `f_webp,q_auto,w_,h_,c_fill` | Bild-CDN mit On-the-fly-Transformation |
| **Plausible Analytics** | `<script async src="https://plausible.io/js/pa-4Pmqrw7o2Xy7XLZTKsD42.js">` + `plausible.init()` | Cookieloses Analytics — **der einzige Tracker auf der Seite** |
| **Lucide Icons** | CSS-Selektor `[data-lucide]{width:28px;height:28px;stroke-width:2.5}` | Icons im Modal, per JS injiziert (*abgeleitet* — Loader steckt im Modal-Bundle) |
| **Inter (self-hosted)** | 4× `@font-face` inline, `/fonts/inter-400..700.woff2`, `font-display:optional` | Einzige Schrift; **kein** Google-Fonts-CDN (Kommentar im Code sagt das explizit) |
| **Schema.org JSON-LD** | 4 Blöcke: `RealEstateAgent`, `WebSite` (+`SearchAction`), `Person`, `BreadcrumbList` | SEO / E-E-A-T-Entitäten, stark ausgebaut |
| **anewera.ch** | Footer-Badge „◆ Für KI-Agenten erreichbar · anewera" | Verzeichnis-/Interop-Layer für KI-Agenten |

**Nicht vorhanden:** kein GSAP, kein Lenis/Locomotive, kein jQuery, kein Tailwind (handgeschriebenes
CSS mit BEM-artigen Klassennamen), kein Cookie-Banner, kein Google Analytics/GTM, kein
Framework-Router. Alles Scroll-/Hover-Verhalten ist reines CSS oder ein 3-Zeilen-Inline-Script.

---

## Design System

### Colors

Definiert als CSS Custom Properties auf `:root` (**zweimal identisch** deklariert, siehe „Notes"):

| Name/Usage | Token | Value |
|---|---|---|
| Gold/Orange — Primär-Akzent, CTA-Start | `--gold-start` | `#f2994a` |
| Gold/Gelb — CTA-Ende | `--gold-end` | `#f2c94c` |
| Dunkelblau — Footer/Overlay-Start | `--blue-start` | `#141e30` |
| Dunkelblau — Footer/Overlay-Ende | `--blue-end` | `#243b55` |
| Fliesstext-Primär | `--text-primary` | `#141e30` |
| Fliesstext-Sekundär | `--text-secondary` | `#666` |
| Heller Hintergrund | `--background-light` | `#f8f9fa` |
| Rahmen hell | `--border-light` | `rgba(0,0,0,.08)` |
| Browser-Theme | `<meta name="theme-color">` | `#f2994a` |

Gradients (durchgehend `135deg`):
```css
--primary-gradient: linear-gradient(135deg, #f2994a 0%, #f2c94c 100%);  /* CTA, Icons, Progress */
--dark-gradient:    linear-gradient(135deg, #141e30 0%, #243b55 100%);  /* Footer, AI-Banner */
/* Sektions-Hintergrund (hardcodiert, kein Token): */
background: linear-gradient(135deg, #f8fafc, #eef2ff);   /* team-values, team-intro-text, credentials */
/* Hero-Overlay über Bild: */
background: linear-gradient(135deg,#141e30b3,#243b5580), url(...);  /* 70%/50% Deckung */
background: linear-gradient(135deg,#141e3066,#243b554d);            /* banner-overlay, 40%/30% */
```

Nicht-Token-Farben, die trotzdem vorkommen (Inkonsistenz): `#1f2937`, `#374151`, `#6b7280`,
`#d1d5db`, `#e5e7eb` (Tailwind-Graustufen), `#10b981` / `#047857` (Erfolg-Grün),
`#1E3A8A` (`.btn-secondary` global — passt zu keinem Token).

### Typography

Basis-Skala (global, feste px-Werte — **keine** `clamp()`/fluid Typo):

| Role | Font | Weight | Size | Line-height | Farbe |
|---|---|---|---|---|---|
| h1 | Inter | 700 | 56px | 1.1 | `--text-primary` |
| h2 | Inter | 700 | 42px | 1.2 | `--text-primary` |
| h3 | Inter | 600 | 28px | 1.3 | `--text-primary` |
| h4 | Inter | 600 | 20px | 1.4 | `--text-primary` |
| p | Inter | 400 | 16px | 1.6 | `--text-secondary` |
| body | Inter, sans-serif | 400 | — | 1.6 | `#1f2937` |
| `.hero-title` | Inter | 700 | 3.5rem → 2.5rem @1024 | 1.1–1.2 | `#fff` + text-shadow |
| `.section-title` | Inter | 700 | 2.5rem → 2rem @768 | — | `#1f2937` |
| `.member-name` | Inter | 700 | 2rem → 1.75 @1024 → 1.5 @768 | — | `--text-primary` |
| `.member-title` | Inter | 600 | 1.125rem, `uppercase`, `letter-spacing:.5px` | — | `--gold-start` |
| Formular-Labels | Inter | 600 | 14px, `letter-spacing:.5px` | — | `#141e30` |
| Buttons | Inter | 600 | 16px, `letter-spacing:.5px` | — | `#fff` |

Font-Dateien (self-hosted, alle `font-display: optional`, nur 400 wird preloaded):
```
/fonts/inter-400.woff2
/fonts/inter-500.woff2
/fonts/inter-600.woff2
/fonts/inter-700.woff2
```

### Spacing System

Kein Token-System — feste Werte in einem groben 4/8-px-Raster, gemischt `rem` und `px`:

```css
.container       { max-width:1200px; margin:0 auto; padding:0 16px }  /* @768: 0 1rem */
.section         { padding:100px 0 }
.section-compact { padding:80px 0 }
.team-members    { padding:5rem 0 }   /* @768: 3rem 0 */
.members-grid    { gap:4rem }
.member-card     { padding:2rem; gap:3rem }   /* @768: 1.5rem */
.footer          { padding:5rem 0 2rem }      /* @768: 2rem 0 1rem */
/* Utility-Klassen (definiert, im Markup ungenutzt): */
.element-gap{gap:32px} .small-gap{gap:16px} .large-gap{gap:80px}
```

Border-Radius-Skala: `4px` (Checkbox) · `12px` (Buttons, Cards klein, Header-Icons) ·
`16px` (Header-Bar, Inputs, CTA-Buttons, credentials) · `20px` (Modal, Option-Cards, AI-Banner) ·
`24px` (member-card) · `999px`/`50%` (Pills, Social-Icons, Progress-Dots).

Shadow-Skala (als Utilities definiert):
```css
.shadow-sm{box-shadow:0 4px 12px #0000000d}   .shadow-md{box-shadow:0 12px 40px #00000014}
.shadow-lg{box-shadow:0 20px 60px #0000001f}  .shadow-xl{box-shadow:0 32px 80px #00000029}
/* Gold-getönte Schatten für alles Interaktive: */
box-shadow:0 8px 32px #f2994a4d;   /* btn-next Ruhezustand */
box-shadow:0 16px 48px #f2994a66;  /* btn-next Hover */
box-shadow:0 20px 60px #f2994a40, 0 0 0 4px #f2994a26;  /* option-card.selected */
```

### Responsive Approach

Zwei Breakpoints, mobile-last (Desktop ist Default, `max-width`-Queries):

| Breakpoint | Was passiert |
|---|---|
| `@media(max-width:1024px)` | Desktop-Nav + Desktop-CTA `display:none`, Burger erscheint; `.member-card` 2-spaltig → 1-spaltig + zentriert; Foto auf `max-width:300px`; Footer 1-spaltig, Footer-Links 2-spaltig; h1 3.5rem → 2.5rem |
| `@media(max-width:768px)` | `.values-grid` 3-spaltig → 1-spaltig; Modal auf 95 % Breite / 98vh; Modal-Actions als `column-reverse` und volle Breite; Option-Grid 1-spaltig; AI-Banner von zentriert-fix auf `left:12px;right:12px`; Footer-Bottom vertikal |

Zusätzlich ein WebKit-Hack:
```css
@media screen and (-webkit-min-device-pixel-ratio:0){
  .phone-number{color:#fff!important;-webkit-text-fill-color:white!important}
}
```
→ gegen iOS-Safaris automatische Blaufärbung von `tel:`-Links.

---

## Effects Breakdown

| Effect | Implementation | Complexity | Cloneable? |
|---|---|---|---|
| Nav-Unterstrich fährt ein | `a::after` mit `transform:scaleX(0)` → `scaleX(1)` bei `:hover`, `transform-origin:left`, `.25s` | Low | Ja |
| Dropdown-Menü | `opacity/visibility/translateY(-10px)` → `0`, `.3s`, per `:hover` auf `.nav-dropdown` | Low | Ja (Markup fehlt auf dieser Seite) |
| Mobile-Menü | Inline-Script togglet `.open`, CSS `display:none/block` + `backdrop-filter:blur(20px)` | Low | Ja |
| Fixed-Hintergrund im Hero | `background-attachment:fixed` — Pseudo-Parallax ohne JS | Low | Ja (mit Vorbehalt, s. Notes) |
| Karten-Lift bei Hover | `translateY(-4px)` + Schatten `0 8px 32px` → `0 16px 48px`, `.3s ease` | Low | Ja |
| Foto-Zoom bei Hover | `img{transform:scale(1.05)}` in `overflow:hidden`-Wrapper, mit `:first-child`-Ausnahme | Low | Ja |
| Back-to-Top | Inline-Modul: `scrollY>400` → `display:grid`, Klick → `scrollTo({behavior:'smooth'})` | Low | Ja |
| Otti-AI-Banner slide-up | Nach 2000 ms Klasse `.visible`, `translateY(120px)→0`, `cubic-bezier(.16,1,.3,1)`, Dismiss in `sessionStorage` | Low | Ja |
| Modal öffnen/schliessen | Overlay `opacity/visibility` + Container `scale(.9) translateY(20px)` → `scale(1) translateY(0)`, `.3s` | Low | Ja |
| Pulsierendes Avatar | `@keyframes subtle-pulse` — `scale(1)↔scale(1.05)`, 2 s, endlos | Low | Ja |
| Option-Card Auswahl | Hover `translateY(-8px) scale(1.02)`, Icon `scale(1.1) rotate(5deg)`, `::before`-Gradient `opacity 0→1` | Med | Ja |
| Button-Shimmer | `::before` mit `linear-gradient(90deg,transparent,rgba(255,255,255,.2),transparent)`, `left:-100%` → `100%` in `.5s` | Low | Ja |
| Progress-Dots | `.active` → Gold-Gradient + `scale(1.2)` + Glow; `.completed` → `#10b981` + `scale(1.1)` | Low | Ja |
| Scroll-Indikator | `@keyframes bounce` (Container) + `bounce-arrow` (Pfeil), Sichtbarkeit per `.show` aus JS | Low | Ja |
| Custom Checkbox/Radio | Native Inputs `opacity:0;position:absolute`, Optik über Geschwister-Element + `:checked +` / `:has()` | Med | Ja |
| Chat-Widget | Astro-Island `client:visible` — IntersectionObserver lädt React erst beim Sichtbarwerden | Med | Ja |
| Gradient-Text | `background-clip:text` + `color:transparent` auf Gold-Gradient | Low | Ja |
| Glassmorphism | `.glass`: `rgba(255,255,255,.15)` + `backdrop-filter:blur(20px)` + heller Rand | Low | Ja |

---

## Implementation Details

### 1. Der fixe Header, der 8 px unter der Kante schwebt (bestätigt aus CSS)

Der Header ist keine durchgehende Leiste, sondern eine schwebende, abgerundete Karte:

```css
.site-header   { position:fixed; top:8px; inset-inline:0; z-index:1000;
                 background:transparent; overflow:visible }
.header-content{ display:flex; justify-content:space-between; height:80px;
                 padding:0 16px; border-radius:16px; position:relative }
.white-bg      { background:#fffffffa;             /* 98 % opak, nicht ganz weiss */
                 border:1px solid rgba(0,0,0,.08);
                 box-shadow:0 2px 20px #00000014 }
```

**Der Trick:** `.site-header` selbst ist transparent, die weisse Karte ist das innere
`.header-content` im `.container` (max 1200 px). Dadurch entstehen links und rechts
automatisch Ränder, die Leiste „schwebt".

**Der Haken:** `.logo-img{height:132px}` — das Logo ist 132 px hoch in einer 80 px hohen
Leiste. Deshalb `overflow:visible` auf Header *und* `.header-content` und
`z-index:3` auf `.logo`. Das Logo ragt absichtlich oben und unten heraus.

### 2. Nav-Unterstrich (bestätigt aus CSS)

Der Standard-Trick, sauber umgesetzt:

```css
.nav-item > a{ position:relative }
.nav-item > a::after{
  content:""; position:absolute; left:0; right:0; bottom:-6px; height:2px;
  background:linear-gradient(135deg,var(--gold-start),var(--gold-end));
  transform:scaleX(0); transform-origin:left;
  transition:transform .25s ease;
}
.nav-item:hover > a::after{ transform:scaleX(1) }
```

**Die Einsicht:** `scaleX` statt `width` animieren — läuft auf dem Compositor, kein Reflow.
`transform-origin:left` gibt die Laufrichtung.

### 3. Back-to-Top — das gesamte Scroll-JS der Seite (bestätigt, inline im `<head>`)

Das ist wortwörtlich der komplette Scroll-Code der Website:

```js
const o = document.getElementById("backToTop"),
      e = () => { o && (o.style.display = window.scrollY > 400 ? "grid" : "none") };
window.addEventListener("scroll", e, { passive: true });
o && o.addEventListener("click", () => window.scrollTo({ top: 0, behavior: "smooth" }));
```

```css
.back-to-top{ position:fixed; bottom:24px; right:24px; display:none; place-items:center;
              width:44px; height:44px; border-radius:12px; border:none; font-weight:800 }
```

**Die Einsicht:** `display:none` ↔ `display:grid` (nicht `block`), weil `place-items:center`
den Pfeil zentriert. `{passive:true}` verhindert Scroll-Blocking. Kein Throttle/rAF — bei
einer reinen Property-Zuweisung vertretbar, aber es feuert bei jedem Scroll-Event.

### 4. Otti-AI-Floating-Banner (bestätigt, inline vor `</body>`)

```js
(function(){
  const e = document.getElementById("aiFloatingBanner"),
        s = document.getElementById("aiFloatingClose");
  if (!e || !s) return;
  if (sessionStorage.getItem("ai-banner-dismissed")) { e.style.display = "none"; return }
  setTimeout(() => { e.classList.add("visible") }, 2e3);
  s.addEventListener("click", () => {
    e.classList.remove("visible");
    sessionStorage.setItem("ai-banner-dismissed", "1");
    setTimeout(() => { e.style.display = "none" }, 400);
  });
})();
```

```css
.ai-floating-banner{ position:fixed; bottom:24px; left:50%;
  transform:translate(-50%) translateY(120px); opacity:0; pointer-events:none; z-index:999;
  transition:transform .4s cubic-bezier(.16,1,.3,1), opacity .4s ease }
.ai-floating-banner.visible{ transform:translate(-50%) translateY(0); opacity:1;
  pointer-events:auto }
```

**Die Einsichten:**
- `pointer-events:none` im Ruhezustand — der unsichtbare Banner fängt keine Klicks ab.
- Die 400 ms im zweiten `setTimeout` entsprechen exakt der CSS-Transitionsdauer: erst
  ausfahren, dann aus dem Layout nehmen.
- `cubic-bezier(.16,1,.3,1)` ist „easeOutExpo" — schneller Start, langes weiches Ausklingen.
- `sessionStorage`, nicht `localStorage`: der Banner kommt beim nächsten Besuch wieder.
- Auf Mobil wechselt die Zentrierung von `translate(-50%)` auf `translate(0)` mit
  `left:12px;right:12px` — sonst würde die `-50%`-Verschiebung die volle Breite ruinieren.

### 5. Der Shimmer über den CTA-Buttons (bestätigt aus CSS)

```css
.btn-next{ position:relative; overflow:hidden;
           background:linear-gradient(135deg,#f2994a,#f2c94c);
           box-shadow:0 8px 32px #f2994a4d }
.btn-next::before{ content:""; position:absolute; top:0; left:-100%; width:100%; height:100%;
  background:linear-gradient(90deg,transparent,rgba(255,255,255,.2),transparent);
  transition:left .5s ease }
.btn-next:hover:not(:disabled)::before{ left:100% }
.btn-next:hover:not(:disabled){ transform:translateY(-3px) scale(1.02);
                                box-shadow:0 16px 48px #f2994a66 }
.btn-next:active:not(:disabled){ transform:translateY(-1px) scale(.98) }
```

**Die Einsicht:** Ein 100 % breiter, halbtransparenter weisser Gradientstreifen wird von
`left:-100%` nach `left:100%` geschoben — der Parent hat `overflow:hidden`, also sieht man
nur den Durchlauf. Kein Keyframe nötig, eine `transition` genügt. Der `:active`-Zustand mit
`scale(.98)` gibt das haptische „Drücken".

### 6. Option-Cards im Modal (bestätigt aus CSS)

```css
.option-card{ border:2px solid rgba(242,153,74,.15); border-radius:20px; padding:24px 20px;
  min-height:140px; position:relative; overflow:hidden;
  transition:all .4s cubic-bezier(.4,0,.2,1) }
.option-card::before{ content:""; position:absolute; inset:0;
  background:linear-gradient(135deg,#f2994a0d,#f2c94c0d);
  opacity:0; transition:opacity .3s ease }
.option-card:hover{ border-color:#f2994a66; transform:translateY(-8px) scale(1.02);
                    box-shadow:0 20px 60px #f2994a33 }
.option-card:hover::before{ opacity:1 }
.option-card.selected{ border-color:#f2994a; background:#fff;
  box-shadow:0 20px 60px #f2994a40, 0 0 0 4px #f2994a26; transform:translateY(-4px) }
.option-card:hover .option-icon{ transform:scale(1.1) rotate(5deg) }
```

**Die Einsicht:** Der ausgewählte Zustand liegt *tiefer* (`-4px`) als der Hover-Zustand
(`-8px`) — ausgewählt heisst „eingerastet", nicht „schwebt am höchsten". Der zweite
Box-Shadow `0 0 0 4px #f2994a26` ist ein Fake-Focus-Ring, der nicht am Layout zieht.
`cubic-bezier(.4,0,.2,1)` ist Material Designs „standard easing".

### 7. Custom Checkbox/Radio ohne JS (bestätigt aus CSS)

Zwei Generationen im selben Stylesheet — die neuere ist die interessante:

```css
#dynamicModal .checkbox-input{ position:absolute; opacity:0; width:0; height:0 }
#dynamicModal .checkbox-input:checked + .checkbox-visual{ background:#f2994a;
                                                          border-color:#f2994a }
#dynamicModal .checkbox-input:checked + .checkbox-visual .checkbox-checkmark{
  opacity:1; transform:scale(1) }

/* Radio-Karte färbt sich komplett, per :has() */
#dynamicModal .radio-button-card:has(.radio-input:checked){
  border-color:#f2994a; background:#f2994a1a }
#dynamicModal .radio-input:checked + .radio-card-content .radio-button-circle{
  border-color:#f2994a; background:#f2994a }
#dynamicModal .radio-input:checked + .radio-card-content .radio-button-circle::after{
  content:""; position:absolute; inset:5px; background:#fff; border-radius:50% }
```

**Die Einsicht:** Das native Input bleibt im DOM (Tastatur, Formular-Serialisierung,
`required` funktionieren weiter), ist aber unsichtbar; die Optik übernimmt das
Geschwisterelement per `+`-Kombinator. `:has()` erlaubt es, die *ganze* Karte einzufärben,
ohne dass JS eine Klasse setzen muss. Der innere Punkt ist `inset:5px` statt
`top/left/transform` — kürzer und ohne Rundungsfehler.

⚠️ Die ältere Generation daneben macht dasselbe über eine JS-gesetzte Klasse
(`.checkbox-item.selected`, `.radio-button-card.selected`) mit `!important` auf jeder
Deklaration. Beim Nachbau nur die `:has()`-Variante übernehmen.

### 8. Modal-Öffnung (bestätigt aus CSS, Steuerung abgeleitet)

```css
.modal-overlay{ position:fixed; inset:0; background:#0009; backdrop-filter:blur(8px);
  z-index:1000; display:flex; align-items:center; justify-content:center;
  opacity:0; visibility:hidden; transition:all .3s ease }
.modal-overlay.active{ opacity:1; visibility:visible }
.modal-container{ background:#fffffff2; backdrop-filter:blur(20px); border-radius:20px;
  width:90%; max-width:650px; max-height:90vh; display:flex; flex-direction:column;
  transform:scale(.9) translateY(20px); transition:all .3s ease }
.modal-overlay.active .modal-container{ transform:scale(1) translateY(0) }
```

**Die Einsicht:** `visibility` wird mitanimiert, damit das geschlossene Modal keine Klicks
abfängt — `opacity:0` allein würde das nicht tun. Der Container skaliert *und* fährt hoch,
das liest sich als „springt hervor".

Struktur: fixer Header (`flex-shrink:0`) → scrollbarer Content
(`flex:1; overflow-y:auto; max-height:calc(90vh - 250px)`) → Progress → fixe Action-Leiste.
Auf Mobil wird `.modal-actions` `position:sticky;bottom:0` mit `background:#fffffffa`, damit
„Weiter" beim Scrollen immer erreichbar bleibt.

### 9. Der mehrstufige Formular-Flow (*abgeleitet* — JS-Bundle nicht abrufbar)

Belegbar aus dem HTML:

```html
<button class="cta btn-primary desktop-only" data-modal-trigger="beratung">Beratung</button>
...
<div class="modal-content" id="modalContent">
  <!-- Form steps will be inserted here dynamically -->
</div>
<div class="progress-dots">
  <div class="progress-dot active"></div><div class="progress-dot"></div>
  <div class="progress-dot"></div><div class="progress-dot"></div>
</div>
<span class="progress-text">Schritt 1 von 4</span>
<button class="btn-back" id="modalBack" style="display:none">Zurück</button>
<button class="btn-next" id="modalNext">Weiter</button>
<script type="module" src="/_astro/DynamicModal.astro_astro_type_script_index_0_lang.Dil1NVJ0.js">
```

Daraus rekonstruierbar:

- Ein `data-modal-trigger="<flow>"`-Attribut auf beliebigen Buttons öffnet das Modal mit dem
  benannten Flow. Sieben Flows sind über die versteckten Netlify-Forms belegt:
  `beratung`, `bewertung`, `verkauf`, `vermietung`, `kauf`, `miete`, `termin`.
- Der Content-Container ist beim Ausliefern **leer** — die Schritte werden komplett aus einem
  JS-Config-Objekt gerendert. Vier Dots = vier Schritte im Default-Flow.
- Schritt 4 ist die Datenschutz-Einwilligung (`.privacy-consent-container`,
  `input[name=privacy]`, eigenes 50×50-Checkbox-Design auf Gold-Gradient).
- Die Felder pro Flow stehen 1:1 in den versteckten Forms — das ist die verlässlichste Quelle
  für den Nachbau:

| Flow | Felder (über die Basis `firstName, lastName, email, phone, message` hinaus) |
|---|---|
| `beratung` | — |
| `bewertung` | `propertyType, address, rooms, area` |
| `verkauf` | `propertyType, address` |
| `vermietung` | `propertyType, address` |
| `kauf` | `propertyType, location, budget` |
| `miete` | `propertyType, location, budget` |
| `termin` | `date, time` |

- Absenden geht per POST an Netlify mit `form-name` + `flow-type` im Body und dem
  `bot-field`-Honeypot (Standard-Netlify-Pattern).
- `.form-step{display:none}` / `.form-step.active{display:block}` → das JS togglet nur die
  `active`-Klasse, alle Schritte liegen gleichzeitig im DOM.
- `#scrollIndicator` bekommt `.show`, wenn der Content-Bereich überläuft (*abgeleitet* aus
  `.scroll-indicator{display:none}` / `.show{display:flex}` + Text „Mehr Felder unten").

### 10. Chat-Widget als lazy Island (bestätigt aus HTML)

```html
<astro-island uid="elRyh" prefix="r1"
  component-url="/_astro/ImmoOttiChat.DuM2Jw_g.js"
  renderer-url="/_astro/client.CSQ4PDHQ.js"
  client="visible" opts='{"name":"ImmoOttiChat","value":true}' ssr await-children>
  <button style="position:fixed;bottom:20px;right:20px;z-index:9998;width:56px;height:56px;
                 border-radius:50%;background:linear-gradient(135deg,#f2994a,#f2c94c);...">
```

Der zugehörige Loader (ebenfalls inline im HTML):

```js
var a = (s, i, o) => {
  let r = async () => { await (await s())() },
      t = typeof i.value === "object" ? i.value : void 0,
      c = { rootMargin: t?.rootMargin },
      n = new IntersectionObserver(e => {
        for (let l of e) if (l.isIntersecting) { n.disconnect(); r(); break }
      }, c);
  for (let e of o.children) n.observe(e);
};
(self.Astro || (self.Astro = {})).visible = a;
```

**Die Einsicht:** Der Chat-Button ist serverseitig gerendertes HTML mit Inline-Styles — er ist
sofort sichtbar und sieht fertig aus, obwohl noch kein React geladen wurde. Erst wenn er in den
Viewport kommt, zieht der IntersectionObserver React + die Komponente nach und ersetzt den
statischen Button durch die echte Komponente. Der Nutzer merkt keinen Unterschied.
Der Preis: ein Klick in den ersten Millisekunden geht ins Leere.

Was hinter dem Chat steckt (Backend-Endpunkt, Streaming, Nachrichtendarstellung) konnte ich
nicht prüfen — dafür bräuchte ich `ImmoOttiChat.DuM2Jw_g.js`.

### 11. Fixed-Background-Pseudo-Parallax (bestätigt aus CSS)

```css
.hero-background{ position:absolute; inset:0; z-index:-1;
  background: linear-gradient(135deg,#141e30b3,#243b5580),
              url(https://res.cloudinary.com/dihw9b8ce/.../a-sleek-futuristic-abstract-art-piece...png);
  background-size:cover; background-position:center;
  background-attachment:fixed }
```

**Die Einsicht:** Zwei Layer in *einer* `background`-Deklaration — der Gradient liegt über dem
Bild, ohne dass ein Overlay-Element nötig wäre. `background-attachment:fixed` erzeugt den
Parallax-Eindruck mit null Zeilen JS.

⚠️ Diese Regel gehört zu `.team-hero`, das im Markup dieser Seite **gar nicht vorkommt** — die
Seite nutzt stattdessen `.team-banner` mit einem echten Overlay-Div (`.banner-overlay`) und
ohne `attachment:fixed`. Der `attachment:fixed`-Trick wird auf der Startseite verwendet.

### 12. Werte-Kacheln mit Text statt Icons (bestätigt aus HTML+CSS)

```html
<div class="value-card">
  <div class="value-icon">INTEGRITY</div>
  <h3>Integrität</h3>
  <p>Wir handeln stets ethisch und moralisch einwandfrei</p>
</div>
```
```css
.value-card .value-icon{ width:80px; height:80px;
  background:linear-gradient(135deg,var(--gold-start),var(--gold-end));
  color:#fff; border-radius:12px; display:flex; align-items:center; justify-content:center;
  font-size:.65rem; font-weight:700; text-align:center; line-height:1.1;
  letter-spacing:.5px; margin:0 auto .75rem }
```

In der 80×80-Gold-Kachel steht **englischer Text in 10,4 px** („INTEGRITY", „EXPERTISE",
„INNOVATION"), während die Überschrift darunter deutsch ist. Das sieht nach einem
Platzhalter aus, wo ursprünglich ein Icon geplant war. Beim Nachbau: Lucide-Icon einsetzen
(`shield-check`, `graduation-cap`, `sparkles`) — die Kachelgrösse passt schon.

---

## Assets Needed to Recreate

1. **Logo** (SVG) — `7031_immo-otti.ch_logo-01_zwybeg.svg`, wird in 3 Grössen genutzt:
   Header `height:132px`, Footer `height:48px` (mit `filter:brightness(0) invert(1)` → weiss
   gefärbt, ein einziges SVG reicht), JSON-LD `512×512` mit `c_pad,b_white`.
2. **Hero-Hintergrund abstrakt** (1920×1080, WebP) — der Dateiname verrät den Prompt:
   „a sleek futuristic abstract art piece with…". Midjourney-Prompt zum Nachbauen:
   `sleek futuristic abstract art, deep navy and warm amber gradient, flowing geometric forms,
   cinematic lighting --ar 16:9`.
3. **Startseiten-Hero** (1920×1080, WebP) — Dateiname:
   „a photograph of a modern cottage nestled…". Prompt:
   `photograph of a modern chalet nestled in swiss alpine landscape, lake lucerne, golden hour
   --ar 16:9`. Dient gleichzeitig als OG-Image in `1200×630`.
4. **Team-Gruppenbild** (Querformat, min. 1600 px breit) — `.team-banner` mit
   `min-height:500px`, `center/cover`. Original ist ein WhatsApp-Bild mit
   `?cache_bust=2025` an der URL.
5. **2 Portraitfotos** — dargestellt in `400px` Höhe, `object-fit:cover`, Radius 20 px.
   Die erste Karte ist auf `object-fit:contain` + `background:#f5f5f5` umgestellt und vom
   Hover-Zoom ausgenommen — das deutet auf ein freigestelltes Bild in abweichendem
   Seitenverhältnis hin.
6. **Rundes Portrait fürs Modal** — 160×160, Cloudinary macht `c_fill,g_face`
   (Gesichtserkennung beim Zuschnitt); im Markup als 80×80 mit 3 px Gold-Gradient-Rand.
7. **Inter-Woff2 in 4 Schnitten** — von rsms.me/inter oder Google Fonts herunterladen und
   selbst hosten.
8. **Favicon-Set** — `favicon.svg`, `favicon.ico`, `apple-touch-icon.png` (180),
   `favicon-32x32.png`, `favicon-16x16.png`, `site.webmanifest`.
9. **Social-Icons** — YouTube, Instagram, TikTok bereits als Inline-SVG-Pfade im Quellcode
   vorhanden, direkt kopierbar. Kontakt-Icons (Telefon, Mail, Pin) sind Heroicons-Outline-Pfade.
10. **Chat-Icon** — Inline-SVG (Sprechblase mit drei Punkten), im Quellcode enthalten.

Rein per Code erzeugbar, kein Asset nötig: alle Gradients, der Burger (`.hamb` aus drei
2-px-Pseudo-Elementen), Progress-Dots, Checkbox-/Radio-Optik, der Select-Pfeil (Data-URI-SVG
inline im CSS).

---

## Build Plan

### Recommended Stack

- **Astro 5** — genau richtig für diese Site: 95 % statischer Inhalt, JS nur an drei Stellen.
  Jede React-/Next-Alternative würde die Seite ohne Gegenwert schwerer machen.
- **Scoped CSS in `.astro`-Dateien** — kein Tailwind nötig; das Original ist handgeschriebenes
  CSS mit Astros automatischem Scoping (`data-astro-cid-*`).
- **Keine Animationsbibliothek** — jeder einzelne Effekt der Seite ist eine CSS-`transition`
  oder `@keyframes`. GSAP/Framer Motion hier hinzuzufügen wäre reiner Ballast.
- **React nur für das Chat-Widget**, geladen mit `client:visible`.
- **Netlify** — Forms funktionieren ohne Backend, aber nur auf Netlify. Bei anderem Hosting
  siehe „Notes".
- **Cloudinary** (oder Astros `<Image>` mit Sharp) für responsive Bilder.

### NPM Packages

```bash
npm create astro@latest
npm install @astrojs/react react react-dom
npm install lucide  # Icons im Modal; alternativ astro-icon
# optional statt Cloudinary:
npm install @astrojs/sitemap sharp
```

### Section-by-Section Build Order

**1. Layout + Designsystem** (`src/layouts/Base.astro`)
- `:root`-Tokens (Gold/Blau/Text/Border), die 4 `@font-face` inline, `<link rel="preload">` nur
  für Gewicht 400
- Globale Typo-Skala (h1 56 / h2 42 / h3 28 / h4 20 / p 16), `.container` 1200 px,
  `*{margin:0;padding:0;box-sizing:border-box}`, `html{scroll-behavior:smooth}`,
  `html,body{overflow-x:hidden}`
- `.btn-primary` / `.btn-secondary`, Shadow-Utilities, `.glass`, `.text-gradient-blue`
- Meta-Block + die 4 JSON-LD-Blöcke als Props-gesteuerte Komponente

**2. Header** (`src/components/Header.astro`)
- `position:fixed;top:8px`, innen `.header-content.white-bg` mit `border-radius:16px`,
  `height:80px`
- Nav-Liste mit `::after`-Unterstrich; Dropdown-CSS gleich mitnehmen (Untermenüs existieren
  auf anderen Seiten)
- Burger aus `.hamb` + `::before`/`::after` bei `top:∓6px`; Toggle-Script (3 Zeilen)
- Desktop-CTA mit `data-modal-trigger="beratung"`
- ⚠️ Logo-Höhe beim Nachbau auf ~56–64 px setzen statt 132 px (siehe Notes)

**3. Footer** (`src/components/Footer.astro`)
- Grid `1.5fr 2fr 1fr` → 1-spaltig @1024
- Dark-Gradient-Hintergrund, Logo weiss gefiltert, Social-Kreise (40 px, `rgba(255,255,255,.1)`,
  Hover → Gold-Gradient + `translateY(-2px)`)
- Kontaktblock mit Inline-SVG-Icons in `#f2c94c`; `tel:`-Links brauchen den
  `-webkit-text-fill-color`-Fix
- Bottom-Leiste mit Copyright, Rechts-Links, „Für KI-Agenten erreichbar"-Pill, Credits

**4. Team-Banner** (`.team-intro`)
- `min-height:500px`, Bild `center/cover`, absolutes `.banner-overlay` mit
  `linear-gradient(135deg,#141e3066,#243b554d)`, `z-index:1`
- Content `z-index:2`, `text-shadow` auf Titel und Subtitle
- Mobil: `min-height:400px` **und `padding-top:120px`** — sonst verdeckt der fixe Header
  die Überschrift

**5. Intro-Textblock** (`.team-intro-text`)
- `padding:4rem 0`, Hintergrund `linear-gradient(135deg,#f8fafc,#eef2ff)`
- Zentrierter 800-px-Block, `.section-title` 2.5rem, Fliesstext 1.125rem/1.7

**6. Team-Mitglieder** (`.team-members`)
- `.members-grid{display:grid;gap:4rem}` (einspaltig, Karten stapeln)
- `.member-card{grid-template-columns:1fr 2fr;gap:3rem;padding:2rem;border-radius:24px}`
- Hover: Karte `translateY(-4px)`, Bild `scale(1.05)`; @1024 einspaltig + zentriert
- Rechte Spalte: Name → Rolle (uppercase, gold) → Credentials-Box (Gradient, Radius 16) →
  Biografie-Absätze

**7. Werte-Kacheln** (`.team-values`)
- `grid-template-columns:repeat(3,1fr)` → 1-spaltig @768
- 80×80-Gold-Kachel: dort ein Lucide-Icon statt des englischen Textes einsetzen
- Karten-Hover wie bei den Member-Cards

**8. Modal-System** (`src/components/DynamicModal.astro`)
- Overlay + Container, Header mit rundem Portrait (`subtle-pulse`), Content-Div (leer),
  Scroll-Indikator, Progress-Dots, Action-Leiste
- Flow-Definitionen als JS-Objekt: Schritte → Felder, aus der Feld-Tabelle oben ableiten
- `document.querySelectorAll('[data-modal-trigger]')` → Flow starten
- Checkbox/Radio ausschliesslich in der `:has()`-Variante bauen, ohne `!important`
- Die versteckten Netlify-Forms in einem `<div style="display:none">` spiegeln — sonst
  akzeptiert Netlify die Übermittlung nicht
- Auf Mobil `.modal-actions{position:sticky;bottom:0}`

**9. AI-Banner** (`src/components/AiBanner.astro`)
- Markup + die 15 Zeilen Script exakt wie oben übernehmen
- Auf Mobil die Transform-Umschaltung nicht vergessen

**10. Chat-Widget** (`src/components/ImmoOttiChat.jsx`)
- Als React-Island mit `client:visible` einbinden
- Der SSR-gerenderte Platzhalter-Button (56 px, Gold-Gradient, `z-index:9998`) ergibt sich
  automatisch, wenn die Komponente im ersten Render den Button zeigt

**11. Back-to-Top + Plausible**
- Das 4-Zeilen-Script ins Base-Layout, Button mit `.glass`
- Plausible-Script-Tag mit eigener Site-ID

---

## Notes

### Was am Original wirklich gut ist

- **Nur ein Tracker.** Plausible, cookielos, kein Consent-Banner nötig. Deshalb lädt die Seite
  auch ohne Cookie-Overlay.
- **Self-hosted Fonts** mit explizitem Kommentar gegen Google Fonts CDN — DSGVO-sauber und
  eine Verbindung weniger.
- **JSON-LD auf hohem Niveau.** `RealEstateAgent` mit `openingHoursSpecification`, `areaServed`,
  `hasOfferCatalog`, dazu eine verknüpfte `Person`-Entität mit `@id`-Referenz
  (`founder`/`employee` zeigen per `@id` auf Richard Otto) und `alumniOf`. Das ist ein
  bewusst gebautes E-E-A-T-Setup, kein Plugin-Output.
- **Islands-Architektur konsequent genutzt** — genau eine Komponente wird hydriert, und die
  erst beim Sichtwerden.
- **`{passive:true}`** auf dem Scroll-Listener.

### Fehler und Altlasten, die man nicht mitkopieren sollte

1. **Massive CSS-Duplikation.** Der `#dynamicModal`-Block existiert **dreimal** — einmal global,
   einmal scoped (`data-astro-cid-tufcfiup`), einmal wieder global. Dazu ist `:root` mit den
   Tokens zweimal deklariert und `.btn-primary` ebenso. Grob geschätzt sind 30–40 % des
   ausgelieferten CSS redundant.
2. **`!important`-Wildwuchs.** Im Modal-CSS steht `!important` auf hunderten Deklarationen —
   die direkte Folge von Punkt 1: die Duplikate überschreiben sich gegenseitig und wurden
   mit Gewalt gewonnen. Beim Nachbau eine Quelle der Wahrheit pro Komponente führen.
3. **Ungültiges CSS:**
   ```css
   .form-step h3:contains("Datenschutz"){ color:#10b981; margin-bottom:16px }
   ```
   `:contains()` gibt es in CSS nicht (nur in jQuery). Die Regel wird vom Browser komplett
   verworfen — die grüne Datenschutz-Überschrift erscheint nie. Ersatz: eine Klasse setzen.
4. **Toter Code.** `.team-hero`, `.hero-content` und `.hero-background` sind vollständig
   ausformuliert, kommen im Markup dieser Seite aber nicht vor. Ebenso `.section-nav`,
   `.page-hero`, `.mobile-submenu`, `.dropdown-menu` (kein Dropdown-Markup) und die
   Utilities `.element-gap/.small-gap/.large-gap`.
5. **`.hero-title` ist zweimal definiert**, in derselben Datei, mit unterschiedlichen Werten
   (einmal `margin-bottom:1.5rem` ohne Farbe, dann `1rem` mit `color:#fff`). Die zweite gewinnt.
6. **CLS-Risiko am Logo.** Das `<img>` trägt `width="120" height="60"`, das CSS setzt
   `height:132px;width:auto`. Der Browser reserviert nach den Attributen, rendert nach dem CSS —
   das Layout springt beim Anwenden des Stylesheets. Und ein 132 px hohes Logo in einer 80 px
   hohen Leiste ist der Grund für das ganze `overflow:visible`/`z-index`-Gebastel.
7. **`background-attachment:fixed`** ist auf iOS Safari faktisch wirkungslos und erzwingt auf
   Android/Desktop bei jedem Scroll-Frame ein Repaint der Hintergrundfläche. Moderne
   Alternative: ein `position:sticky`-Layer oder `transform:translateZ()`-Parallax.
8. **Zwei Cloudinary-Accounts** (`dihw9b8ce` und `dphbnwjtx`) parallel im Einsatz — beim
   Nachbau auf einen konsolidieren. Immerhin liegen beide auf `res.cloudinary.com`, das
   `preconnect` deckt also beide ab.
9. **Gemischte Bildquellen.** Ein Portrait kommt von Cloudinary, das andere aus
   `/images/team/rubinho-granha.jpg` — lokal, ohne WebP-Konvertierung, ohne
   Grössen-Transformation, und der Dateiname passt nicht zum angezeigten Namen. Sieht nach
   einem Rest aus einer früheren Version aus.
10. **Cache-Busting per Query-String** (`?cache_bust=2025`) am Team-Banner — Cloudinary kann
    Versionierung über den `v…`-Pfadteil, der Query-Parameter ist unnötig und verhindert
    teilweise CDN-Caching.
11. **`font-display: optional`** bedeutet: Ist Inter beim ersten Rendern nicht innerhalb von
    ~100 ms verfügbar, zeigt der Browser die Fallback-Schrift für die **gesamte Seitenladung**
    an und tauscht nicht mehr. Ausgezeichnet für CLS, aber der erste Besuch sieht oft nicht wie
    beabsichtigt aus. Bewusste Entscheidung — `swap` wäre der übliche Kompromiss.
12. **Copyright steht auf „© 2026"** — hartcodiert. Besser `{new Date().getFullYear()}`.
13. **`/admin/login`-Link im Footer**, als „discrete" kommentiert. Wenn dahinter ein echtes
    Login steht, gehört es nicht sichtbar verlinkt.
14. **Kein `prefers-reduced-motion`.** Kein einziger Media-Query dafür, obwohl es eine
    Endlos-Animation gibt (`subtle-pulse`, 2 s, unbegrenzt) sowie den `bounce`-Indikator.
    Beim Nachbau ergänzen:
    ```css
    @media (prefers-reduced-motion: reduce){
      *{ animation-duration:.01ms!important; animation-iteration-count:1!important;
         transition-duration:.01ms!important; scroll-behavior:auto!important }
    }
    ```
15. **Kein `:focus-visible`** irgendwo. Bei durchgehend versteckten nativen Inputs
    (`opacity:0`) ist Tastaturbedienung dadurch praktisch blind — der Fake-Focus-Ring
    (`box-shadow:0 0 0 4px`) existiert nur für `:focus` auf Textfeldern, nicht für die
    Checkbox-/Radio-/Option-Cards.
16. **Werte-Kacheln mit englischem 10-px-Text** statt Icons auf einer deutschsprachigen Seite
    (siehe Effekt 12).

### Lizenzen und Abhängigkeiten

- **Inter** ist SIL Open Font License — frei nutzbar, auch kommerziell.
- **Lucide** ist ISC-Lizenz — frei.
- **Netlify Forms** ist ein Plattform-Feature: bei Vercel/Cloudflare-Hosting entfällt es
  ersatzlos. Alternativen: Formspree, Web3Forms, oder eine eigene Serverless-Funktion. Der
  `bot-field`-Honeypot funktioniert überall.
- **Cloudinary** hat ein grosszügiges Free-Tier; Astros eingebautes `<Image>` mit Sharp ist
  für eine statische Seite dieser Grösse aber völlig ausreichend und kostenlos.
- Der Chat („Otti AI") ist ein eigenes Produkt auf `otti-ai.ch` — ein Nachbau bräuchte ein
  eigenes LLM-Backend.
- Die Fotos, das Logo und die Texte sind selbstverständlich geschützt. Nachbaubar ist die
  Struktur, nicht der Inhalt.

### Was für einen vollständigen Teardown noch fehlt

Zwei Dateien, an die ich aus dieser Umgebung nicht herankomme:

```
https://immo-otti.ch/_astro/DynamicModal.astro_astro_type_script_index_0_lang.Dil1NVJ0.js
https://immo-otti.ch/_astro/ImmoOttiChat.DuM2Jw_g.js
```

Mit diesen wären zusätzlich belegbar: die exakten Schrittdefinitionen und Optionstexte je Flow,
die Validierungsregeln, der Submit-Code, das Chat-Backend samt Endpunkt und Begrüssungstext.
Beide sind im Browser über die Netzwerkanalyse oder direkt per URL erreichbar.
