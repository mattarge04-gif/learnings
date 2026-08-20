# Code- und Sicherheitsanalyse: immo-otti.ch

**Analysiert:** Startseite (`/`) und Team-Seite (`/ueber-uns/team/`), ausgelieferter Quellcode
**Datum:** 2026-08-20
**Grundlage:** Nur das, was der Server an den Browser sendet. Kein Repository-Zugriff, keine
aktive Prüfung (Netzwerk-Egress zu immo-otti.ch war in dieser Umgebung blockiert). Aussagen zu
Backend-Konfiguration sind daher als **Prüfauftrag** formuliert, nicht als Befund.

---

## 1. Gesamturteil

| Bereich | Note | Kurzfassung |
|---|---|---|
| Architektur-Entscheidungen | **8/10** | Astro statisch, ein einziges JS-Island, cookieloses Analytics — richtig gewählt |
| SEO / strukturierte Daten | **8/10** | Überdurchschnittlich, bewusst gebaut, kein Plugin-Output |
| Code-Qualität / Wartbarkeit | **3/10** | Drei Generationen desselben CSS koexistieren, `!important`-Eskalation, toter Code |
| Performance | **4/10** | Ungrösste Originalfotos aus Supabase, CSS auf jeder Seite neu, animierte Blur-Flächen |
| Barrierefreiheit | **3/10** | Die Markenfarbe selbst ist kontrastuntauglich, Autoplay-Slider, kein Fokus-Konzept |
| Sicherheit | **unklar — potenziell kritisch** | Hängt vollständig an Supabase-RLS, von aussen nicht verifizierbar |

### Ist die Seite „vibegecodet"?

Ja, mit hoher Sicherheit — und der Code sagt es an mehreren Stellen selbst:

1. **`key="immobilien-kanton-schwyz-kaufen-2026"`** auf einem `<article>` in einer Astro-Datei.
   `key` ist ein React-Konzept, in Astro bedeutungslos und in HTML ungültig. Klassischer
   Fall von „Modell schreibt React-Gewohnheiten in ein anderes Framework".
2. **Ein Kommentar im ausgelieferten HTML:**
   `<!-- Removed duplicate founders photo per improvement prompt -->`
   Das Wort *prompt* steht wörtlich im Produktivcode.
3. **Drei Generationen desselben Modal-CSS** liegen gleichzeitig im Dokument — jede neue
   Iteration hat die alte nicht ersetzt, sondern mit `!important` überschrieben.
4. **`.swiper`, `.swiper-wrapper`, `.swiper-slide`-Regeln** ohne jede Swiper-Bibliothek —
   Überrest eines abgebrochenen Versuchs, den niemand aufgeräumt hat.
5. **`h3:contains("Datenschutz")`** — `:contains()` existiert in CSS nicht, nur in jQuery.
   Eine Halluzination, die nie jemand im Browser überprüft hat.
6. **Vier verschiedene `.btn-secondary`** mit vier verschiedenen Aussehen, je nach Sektion.

**Mein eigentlicher Kritikpunkt ist aber nicht, dass KI im Spiel war.** Die
Architekturentscheidungen sind besser als bei vielen handgebauten KMU-Seiten, und
funktional läuft das Ding. Das Problem ist, dass **nie ein Aufräum-Durchgang stattgefunden
hat**. Jede Iteration hat Code hinzugefügt und keine hat welchen entfernt. Das ist eine
Prozesslücke, keine Werkzeuglücke — und sie ist in ein bis zwei Tagen behebbar.

---

## 2. Was wirklich gut gemacht ist

Damit die Kritik unten einzuordnen ist — diese Entscheidungen sind richtig und
überdurchschnittlich:

- **Astro statisch mit einem einzigen hydrierten Island.** Genau eine Komponente (der Chat)
  lädt React, und die erst per IntersectionObserver, wenn sie sichtbar wird. Die meisten
  Agenturen hätten hier Next.js mit 300 KB JS ausgeliefert.
- **Plausible als einziger Tracker.** Cookielos, deshalb kein Consent-Banner nötig. Bewusste,
  saubere Entscheidung.
- **Self-hosted Inter** mit explizitem Kommentar gegen das Google-Fonts-CDN. DSG-freundlich.
- **`font-display: optional`** — verhindert Layout-Shift durch Font-Swap. Ungewöhnlich
  durchdachte Wahl.
- **JSON-LD auf Agenturniveau.** `RealEstateAgent` mit `areaServed`, `openingHours`,
  `hasOfferCatalog`, dazu eine per `@id` verknüpfte `Person`-Entität mit `alumniOf` und
  `knowsAbout`. Das ist ein durchdachtes E-E-A-T-Setup.
- **`{passive:true}`** auf dem Scroll-Listener.
- **Der Statistik-Zähler** (`100+ Verkäufe` etc.) ist sauber gebaut: IntersectionObserver mit
  `threshold:.25`, `unobserve` nach dem ersten Trigger, Cubic-Ease-Out über
  `requestAnimationFrame`. Kein Timer, kein Speicherleck.
- **Der Scroll-Lock im YouTube-Modal** merkt sich die Scroll-Position in `body.style.top` und
  stellt sie beim Schliessen wieder her. Das machen viele falsch.

---

## 3. Sicherheit

### 3.1 Der zentrale Punkt: Supabase + Admin-Bereich

Neu aus der Startseite:

```html
<img src="https://oahupthtqqbcgesiddhp.supabase.co/storage/v1/object/public/property-images/
          property-1786390083970-0-D1C0E410-0FBF-4E5D-8D52-5A5EE271D3E8.jpg">
```
```html
<a href="/admin/login" class="admin-login-link">Admin</a>
```

Daraus folgt die Architektur: **statische Astro-Seite + Supabase als Datenbank und
Dateispeicher + ein browserseitiges Admin-Panel.**

Und daraus folgt die entscheidende Konsequenz:

> Bei einer statischen Seite gibt es **keinen Server**, der Zugriffe prüfen kann. Das
> Admin-Panel läuft vollständig im Browser des Nutzers. Jede Absicherung im Frontend ist
> reine Kosmetik — man umgeht sie mit der Browser-Konsole. **Row Level Security (RLS) in
> Supabase ist die einzige echte Verteidigungslinie.** Gibt es sie nicht oder ist sie zu
> weit gefasst, kann jeder Besucher der Website Immobilien anlegen, ändern und löschen.

**Wichtige Klarstellung, damit hier nichts falsch verstanden wird:** Der Supabase-Anon-Key,
der im Admin-Bundle steht, ist **kein Leck**. Er ist per Design öffentlich und für den
Browser gedacht. Ihn zu „verstecken" bringt exakt nichts. Die Sicherheit entsteht
ausschliesslich aus den RLS-Policies. Wer behauptet, der Anon-Key sei die Schwachstelle,
hat das Modell nicht verstanden — die Schwachstelle wären fehlende Policies.

**Was konkret geprüft werden muss** (im Supabase-Dashboard, dauert 15 Minuten):

1. **Ist RLS auf jeder Tabelle aktiviert?** Table Editor → jede Tabelle → „RLS enabled"
   muss stehen. Eine Tabelle ohne RLS ist mit dem Anon-Key voll les- und schreibbar.
2. **Gibt es eine INSERT/UPDATE/DELETE-Policy, die für `anon` gilt?** Das wäre der
   Totalschaden. Schreibrechte dürfen ausschliesslich an `authenticated` gehen — und auch
   dort besser an eine Rollen- oder Besitzprüfung gebunden.
3. **Wie lautet die SELECT-Policy auf der Immobilien-Tabelle?** Sie muss `anon` lesen
   lassen, sonst könnte die Seite nicht bauen. Steht dort `USING (true)`, sind auch
   unveröffentlichte, verkaufte oder gesperrte Objekte samt allen Feldern öffentlich
   abrufbar. Richtig wäre `USING (status = 'published')`.
   Dass es Zustände jenseits von „öffentlich" gibt, belegt das CSS:
   ```css
   .property-card.unavailable { opacity:.7; filter:grayscale(30%) }
   .property-card.locked      { cursor:not-allowed }
   .locked-notice             { background:#ef44441a; color:#dc2626 }
   .sold-banner               { background:#ef4444f2; transform:rotate(-15deg) }
   ```
   Es gibt also gesperrte Objekte. Die Sperre ist im ausgelieferten HTML rein visuell.
   Wenn die Detailseiten dieser Objekte trotzdem statisch gebaut werden, sind sie über
   ihre UUID-URL vollständig erreichbar — die Sperre hält niemanden auf.
4. **Wie ist die Anmeldung am Admin-Panel abgesichert?** Supabase Auth mit E-Mail/Passwort
   ist in Ordnung, sofern MFA aktiv ist und keine Registrierung offensteht
   („Enable email signups" muss aus sein, sonst legt sich jeder selbst ein Konto an — und
   wenn die Policies auf `authenticated` statt auf eine Rolle prüfen, ist er damit Admin).
5. **Wird der Service-Role-Key irgendwo im Frontend oder im Build-Output verwendet?**
   Er darf ausschliesslich in CI-Umgebungsvariablen leben. Ein Service-Role-Key im
   Browser-Bundle hebelt RLS komplett aus. Suchbefehl im Repo:
   `grep -r "service_role\|SUPABASE_SERVICE" src/ dist/`

**Meine Einschätzung:** Ich habe keinen Beweis für eine Lücke — ich konnte nichts testen und
habe es bewusst auch nicht versucht. Aber das Muster „KI-gebautes Admin-Panel auf statischer
Seite mit Supabase" ist genau das Muster, bei dem RLS erfahrungsgemäss am häufigsten fehlt,
weil im lokalen Test alles funktioniert, solange RLS **aus** ist. Ich würde das als **offenen
kritischen Punkt** behandeln, bis das Gegenteil belegt ist.

### 3.2 Öffentlicher Storage-Bucket

Der Bucket `property-images` ist auf `public` gestellt. Für Inseratsfotos ist das
grundsätzlich in Ordnung. Zwei Anmerkungen:

- Die Objektnamen enthalten die **Originaldateinamen der Kamera**:
  `IMG_0406.jpeg`, `20260526_220249450_iOS.jpg`,
  `WhatsApp_Image_2026-04-09_at_17.01.13.jpeg`. Das verrät Aufnahmezeitpunkte und die
  Herkunft (iPhone, WhatsApp-Weiterleitung). Harmlos, aber unnötig — beim Upload umbenennen.
- **Zu prüfen:** Ob der `list`-Endpunkt für `anon` offensteht. Wenn ja, kann man den
  gesamten Bucket-Inhalt auflisten — inklusive Fotos zu Objekten, die auf der Website gar
  nicht (mehr) sichtbar sind. Storage → Policies auf `storage.objects` kontrollieren:
  eine SELECT-Policy für `anon` reicht zum Auflisten aus, nicht nur zum Abrufen.
- **Wichtig:** Sollten je Dokumente (Grundbuchauszüge, Verträge, Bewertungen) in denselben
  oder einen anderen `public`-Bucket wandern, ist das sofort ein ernstes Problem. Für so
  etwas gehört ein privater Bucket mit signierten URLs verwendet.

### 3.3 Keine strenge Content-Security-Policy möglich

Auf der Startseite stehen drei Inline-Event-Handler:

```html
<div class="youtube-placeholder" onclick="openYouTubeModal('B243ArNKau4')">
<button class="modal-close" onclick="closeYouTubeModal()">
<button onclick="openYouTubeModal('B243ArNKau4')" class="btn-primary watch">
```

Dazu kommen mehrere Inline-`<script>`-Blöcke und ein sehr grosser Inline-`<style>`-Block.
Konsequenz: Eine CSP müsste `script-src 'unsafe-inline'` erlauben — und damit ist sie
als XSS-Schutz wertlos.

Die Handler sind zudem der Grund für zwei globale Variablen:
```js
typeof window<"u" && (window.openYouTubeModal=n, window.closeYouTubeModal=d);
```

**Fix:** `addEventListener` statt `onclick`, Video-ID über `data-video-id`. Danach lässt
sich eine echte CSP setzen. Für Astro gibt es dafür Hash-basierte CSP-Unterstützung.

### 3.4 Fehlende Security-Header (zu verifizieren)

Header sieht man im HTML nicht — bitte mit `curl -I https://immo-otti.ch/` prüfen. Auf
Netlify gehören in eine `_headers`- oder `netlify.toml`-Datei mindestens:

```
/*
  X-Frame-Options: DENY
  X-Content-Type-Options: nosniff
  Referrer-Policy: strict-origin-when-cross-origin
  Permissions-Policy: geolocation=(), camera=(), microphone=()
  Strict-Transport-Security: max-age=31536000; includeSubDomains
```

`X-Frame-Options` ist hier nicht akademisch: Ohne ihn lässt sich die Seite in einen iFrame
einbetten und mit einem Overlay zum Clickjacking auf die Kontaktformulare missbrauchen.

### 3.5 Formular-Spam

Die sieben Netlify-Formulare haben nur den `bot-field`-Honeypot. Netlifys eingebauter
Akismet-Filter greift zusätzlich, aber bei einem Makler mit sichtbarer Telefonnummer würde
ich das im Auge behalten. Netlify bietet optional reCAPTCHA — das kostet allerdings die
cookiefreie Bilanz und damit womöglich die Consent-Freiheit. Ich würde es beim Honeypot
belassen und erst nachrüsten, wenn tatsächlich Spam ankommt.

---

## 4. Datenschutz (Schweizer DSG / DSGVO)

Hier sehe ich den grössten Widerspruch der Seite: Sie ist mit erkennbarer Sorgfalt
cookiefrei gebaut — und unterläuft das an einer Stelle selbst.

### 4.1 YouTube-Vorschaubild lädt bei jedem Seitenaufruf

```html
<img src="https://img.youtube.com/vi/B243ArNKau4/hqdefault.jpg" loading="lazy">
```

Das Bild kommt direkt von Google, ohne Zutun des Besuchers und ohne Einwilligung. Damit
gehen bei **jedem Aufruf der Startseite** IP-Adresse, User-Agent und Referrer an Google in
die USA — und, falls der Besucher bei Google eingeloggt ist, die entsprechenden Cookies
gleich mit. Der ganze Aufwand mit self-hosted Fonts und cookielosem Analytics wird durch
dieses eine `<img>` teilweise entwertet.

**Fix:** Das Vorschaubild einmalig herunterladen und über Cloudinary ausliefern — es ändert
sich ohnehin nur, wenn eine neue Folge kommt. Kostet nichts und löst das Problem vollständig.

### 4.2 YouTube-Embed statt No-Cookie-Variante

```js
t.src = `https://www.youtube.com/embed/${o}?autoplay=1&rel=0`;
```

Sobald jemand auf Play klickt, setzt YouTube Tracking-Cookies. Immerhin geschieht das erst
nach einer aktiven Handlung — das ist die vertretbare Variante. Trotzdem:
`https://www.youtube-nocookie.com/embed/…` ist ein Einzeiler und deutlich sauberer. Ideal
wäre ein kurzer Hinweis „Beim Abspielen werden Daten an YouTube übertragen" beim
Play-Button.

### 4.3 Formulardaten liegen bei Netlify

Die Formulare erheben Name, E-Mail, Telefon, **Adresse der Immobilie**, Zimmerzahl,
Fläche und Budget. Das ist bei einer Bewertungsanfrage schon eine recht aussagekräftige
Sammlung. Netlify Forms speichert das auf US-Infrastruktur. Nötig ist damit:

- ein Auftragsbearbeitungsvertrag (DPA) mit Netlify,
- eine namentliche Nennung von Netlify **und Supabase** in der Datenschutzerklärung,
- eine Aussage zur Aufbewahrungsdauer.

Die Einwilligungs-Checkbox in Schritt 4 des Modals ist vorhanden und richtig gebaut
(`input[name=privacy]`, ohne Vorauswahl) — das ist bereits gut gelöst.

### 4.4 Google-Rezensionen im Wortlaut

Die drei Testimonials sind offensichtlich aus Google-Rezensionen übernommen, inklusive
Metadaten („4 reviews · 7 photos", „Local Guide"). Namen wie „Matthias Loch" sind
Personendaten. Das ist üblich und in der Praxis selten ein Problem, aber sauber wäre eine
Einwilligung oder eine Abkürzung („Matthias L."). Die englischen Metadaten auf einer
deutschsprachigen Seite wirken zudem unfertig.

---

## 5. Performance

### 5.1 Das grösste Problem: ungrösste Originalfotos

```html
<img src="https://oahupthtqqbcgesiddhp.supabase.co/storage/v1/object/public/
          property-images/property-1783718380043-0-IMG_0406.jpeg"
     loading="lazy" width="400" height="300">
```

Sechs Immobilienbilder auf der Startseite, alle direkt aus dem Storage, **ohne jede
Transformation**. `width="400"` ist nur ein HTML-Attribut — es sagt dem Browser, wie gross
er das Bild *darstellen* soll, nicht, wie gross er es *herunterladen* soll. Geladen wird die
Originaldatei. Bei iPhone-Aufnahmen sind das erfahrungsgemäss 3–8 MB pro Stück; sechs Stück
ergeben grob 20–50 MB, die im Browser auf 400 px heruntergerechnet werden.

Das ist umso ärgerlicher, als **auf derselben Seite alles richtig gemacht wird**: Die
Cloudinary-Bilder tragen brav `f_webp,q_auto,w_800,h_600,c_fit`. Nur die Supabase-Bilder
nicht. Zwei Bildpipelines, eine optimiert, eine gar nicht.

**Fix — der mit Abstand grösste Hebel der ganzen Seite:**
- Supabase Image Transformation nutzen (`?width=400&quality=75`, ab Pro-Plan), **oder**
- die Supabase-URL durch Cloudinary Fetch schleifen, **oder**
- beim Upload im Admin-Panel serverseitig auf max. 1600 px verkleinern und als WebP ablegen.

Die dritte Variante ist die beste: löst das Problem an der Wurzel und kostet nichts.

### 5.2 Kaputtes `srcset` im Hero

```html
srcset="…f_webp,q_auto,w_400,h_300,c_fit/v1755771655/Gruppenbild….jpg 400w,
        ,q_auto,w_800,h_600,c_fit/v1755771655/Gruppenbild….jpg 800w"
```

Nach `400w,` folgt ein **zweites Komma** und danach ein Fragment ohne Domain. Beim
Zusammenbauen des Strings ist die Basis-URL für den zweiten Kandidaten verlorengegangen. Der
Browser interpretiert `,q_auto,w_800,…` als relativen Pfad auf immo-otti.ch — der 404 gibt.
Praktisch heisst das: **jedes Gerät bekommt die 400-px-Version**, auch der 4K-Desktop.
Deshalb wirkt das Gründerbild vermutlich unscharf.

### 5.3 Textleiche im Hero-Markup

Direkt nach demselben `<img>`, als sichtbarer Textknoten im DOM:

```html
</img>
    https://res.cloudinary.com/dphbnwjtx/image/upload/f_webp
</div>
```

Ein abgeschnittenes URL-Fragment, das als Text im `.founder-frame` landet. Sichtbar wird es
nur deshalb nicht, weil das Bild mit `position:absolute; inset:0` darüberliegt. Es ist der
Rest desselben verunglückten String-Zusammenbaus wie beim `srcset` — beide Fehler stammen
offensichtlich aus derselben Bearbeitung.

### 5.4 CSS wird auf jeder Seite neu geladen

Das gesamte CSS steckt inline im `<head>` — auf **jeder** Seite. Grob geschätzt 60–80 KB pro
Dokument, davon allein rund 40 % Modal-CSS, das auf den meisten Seiten nie gebraucht wird.

Beim ersten Aufruf ist das ein Vorteil (kein blockierender Request). Ab der zweiten Seite ist
es ein reiner Verlust: Eine externe CSS-Datei wäre ab da aus dem Cache gekommen, inline muss
sie jedes Mal neu übertragen werden. Für eine Seite mit Blog, Immobilienlisting und mehreren
Unterseiten ist das der falsche Kompromiss.

**Fix:** In `astro.config.mjs` `build.inlineStylesheets: 'auto'` setzen (Astros Standard).
Kleine Blöcke bleiben dann inline, grosse werden ausgelagert und cachebar.

### 5.5 Vier gleichzeitig laufende Endlos-Animationen

```css
.wave           { animation: wave 1.5s ease-in-out infinite }        /* 4 Elemente */
.float          { animation: float 20s ease-in-out infinite alternate }  /* 3 Elemente */
.bewertung-icon { animation: pulse-glow 2s ease-in-out infinite }
.richard-photo  { animation: subtle-pulse 2s infinite }
```

Neun Elemente animieren permanent, auch ausserhalb des Viewports. Besonders teuer:

```css
.float { background:#ffffff0a; backdrop-filter:blur(40px); border-radius:999px;
         animation: float 20s ease-in-out infinite alternate }
.f1    { width:220px; height:220px }
```

`backdrop-filter: blur(40px)` bedeutet, dass der Browser den Hintergrund hinter der Fläche
in jedem Frame neu weichzeichnet. Drei solche Flächen dauerhaft animiert sind auf
Mittelklasse-Android spürbar — und der visuelle Effekt ist bei 4 % Weiss auf hellem Grund
praktisch unsichtbar. Ich würde die drei `.float`-Kreise ersatzlos streichen: maximale
Kosten, minimaler Ertrag.

Und `pulse-glow` animiert `box-shadow`, was ein Repaint pro Frame erzwingt. Auf
`opacity` eines Pseudo-Elements umbauen.

### 5.6 Vier Drittanbieter-Origins, ein Preconnect

Die Startseite lädt von `res.cloudinary.com`, `plausible.io`, `img.youtube.com` und
`oahupthtqqbcgesiddhp.supabase.co`. Vorbereitet ist nur Cloudinary. Supabase liefert die
sechs grössten Bilder der Seite und braucht dringend einen eigenen Preconnect.

---

## 6. Barrierefreiheit

### 6.1 Der Grundsatzfehler: weisse Schrift auf dem Markengradienten

Das ist der schwerwiegendste und am weitesten verbreitete Fehler der Seite.

| Kombination | Kontrast | WCAG AA (4.5:1) |
|---|---|---|
| Weiss auf `#f2994a` (Gold-Start) | **≈ 2,2 : 1** | ✗ deutlich verfehlt |
| Weiss auf `#f2c94c` (Gold-Ende) | **≈ 1,6 : 1** | ✗ klar verfehlt |
| Weiss auf `#b45309` (Miet-Badge) | ≈ 5,0 : 1 | ✓ bestanden |
| `#9ca3af` auf Weiss (`.blog-meta`) | ≈ 2,5 : 1 | ✗ verfehlt |

Weiss auf dem Gold-Gradienten betrifft: **jeden `btn-primary`**, die „ZU VERKAUFEN"-Badges,
`.blog-category`, `.ai-badge`, den kompletten `.cta-banner` (goldene Fläche, weisser Text,
über eine ganze Bildschirmhöhe), das `.search-profile-banner` und alle `.option-icon` im
Modal. Also praktisch jede Handlungsaufforderung der Website.

Bemerkenswert: An zwei Stellen wurde das Problem bereits erkannt und gelöst —
`.property-badge.rental` nutzt das dunklere `#b45309`, und `.btn-search-profile` dreht die
Farben um (`background:#fff; color:#b35a14`). Es fehlt nur die Konsequenz, das überall zu tun.

**Fix ohne Markenverlust:** Ein dunkleres Gold als Textträger-Variante einführen
(`--gold-text: #b35a14`, ist ja schon im Code) und für Text auf goldenem Grund
`color: #141e30` statt Weiss verwenden. Das Blau auf Gold sieht gut aus und erreicht
über 7:1.

### 6.2 Autoplay-Karussell ohne Pause

```js
const v = 4e3;                                     // 4 Sekunden
const o = () => { i && clearInterval(i);
                  i = setInterval(() => n(t+1), v) };
```

Die Testimonials wechseln alle vier Sekunden automatisch, ohne Pause-Möglichkeit und ohne
Stopp bei Hover oder Fokus. Das verstösst gegen **WCAG 2.2.2 (Pause, Stop, Hide)**: Was
sich länger als fünf Sekunden automatisch bewegt und Text enthält, muss anhaltbar sein.
Vier Sekunden reichen zudem nicht, um eine dreizeilige Rezension zu Ende zu lesen.

Zwei Zeilen beheben das:
```js
slider.addEventListener('mouseenter', () => clearInterval(i));
slider.addEventListener('mouseleave', o);
```
Plus eine `prefers-reduced-motion`-Abfrage, die den Autoplay ganz abschaltet.

### 6.3 Die Pfeile des Karussells sind auf dem Desktop unsichtbar

```css
.slider     { position:relative; overflow:hidden; max-width:780px }
.nav        { position:absolute; top:50% }
.nav.prev   { left:-56px }
.nav.next   { right:-56px }
```

Die Navigationsbuttons liegen 56 px **ausserhalb** ihres Elternelements — das aber
`overflow:hidden` hat. Sie werden also abgeschnitten und sind auf Desktop-Breite nicht zu
sehen. Nur im Mobil-Breakpoint (`left:8px` / `right:8px`) tauchen sie auf. Die Tastatur
erreicht sie trotzdem, was noch verwirrender ist: fokussierbare, aber unsichtbare
Bedienelemente.

**Fix:** `overflow:hidden` auf einen inneren Wrapper verschieben oder die Buttons per
`padding` innerhalb des Sliders platzieren.

### 6.4 Emoji als Benutzeroberfläche

```html
<span>📐 203 m²</span> <span>🏠 1914</span> <p>📍 5727 Oberkulm</p>
<h3>🔍 Nichts Passendes dabei?</h3> <div class="icon">✓</div> <div class="icon">❤</div>
<div class="stars"><span>⭐⭐⭐⭐⭐</span></div>
```

Screenreader lesen das vor: „Dreieck-Lineal 203 Quadratmeter", „Lupe Nichts Passendes
dabei", und bei den Sternen fünfmal „weisser mittelgrosser Stern" — statt „Bewertung: 5 von
5". Zusätzlich rendert jedes System die Emoji anders, was das Design zerlegt.

Die Seite lädt ohnehin bereits Lucide-Icons für das Modal und hat massenhaft
Inline-SVG-Icons im Footer. Die Werkzeuge sind also da, sie werden nur an diesen Stellen
nicht benutzt.

Bei den Sternen kommt eine Kuriosität dazu:
```css
.stars { color:transparent; background:linear-gradient(…); background-clip:text }
```
Emoji sind farbige Glyphen — der Versuch, sie per `background-clip:text` einzufärben, ist
browserabhängig und fragil. Fünf SVG-Sterne mit `aria-label="5 von 5 Sternen"` wären
robuster und barrierefrei.

### 6.5 Kein Fokus-Konzept, kein `prefers-reduced-motion`

- Auf der ganzen Seite kommt **kein einziges `:focus-visible`** vor. Im Modal steht dafür
  reihenweise `outline:none !important`. Tastaturnutzer sehen nicht, wo sie sind.
- **Kein `prefers-reduced-motion`** — bei vier Endlos-Animationen, einem Autoplay-Slider,
  einem Zähler-Effekt und `scroll-behavior:smooth` ist das für Menschen mit vestibulären
  Beschwerden unangenehm bis unbenutzbar.
- Das YouTube-Modal hat **keinen Fokus-Trap**, kein `role="dialog"`/`aria-modal`, und der
  Fokus kehrt beim Schliessen nicht zum Auslöser zurück. Immerhin funktioniert Escape.
- **Kein Skip-Link** zum Hauptinhalt.

Ein Basispaket, das viel davon abdeckt:
```css
@media (prefers-reduced-motion: reduce){
  *{ animation-duration:.01ms!important; animation-iteration-count:1!important;
     transition-duration:.01ms!important; scroll-behavior:auto!important }
}
:focus-visible{ outline:3px solid #b35a14; outline-offset:2px }
```

---

## 7. Code-Qualität und Wartbarkeit

### 7.1 `.btn-secondary` bedeutet vier verschiedene Dinge

| Definiert in | Aussehen |
|---|---|
| Global | Transparent, blauer Rand `#1E3A8A`, blaue Schrift |
| Hero | Glas-Effekt, weisser Rand, weisse Schrift, `backdrop-filter` |
| Blog | Transparent, oranger Rand, orange Schrift |
| CTA-Banner | Transparent, weisser 2-px-Rand, weisse Schrift |

Welche Version gewinnt, hängt von der Reihenfolge im inline-CSS und von Astros
Scoping-Attributen ab. Für jeden, der die Seite später anfasst, ist das eine Falle: Eine
Änderung an „dem Button" trifft nie alle vier. Dasselbe gilt für `.section-title`
(dreimal, einmal blau, einmal gold, einmal grau) und `.modal-content` (im Podcast-Modal
und im Formular-Modal etwas völlig anderes).

### 7.2 `!important`-Eskalation

Im Modal-CSS steht `!important` auf hunderten Deklarationen, teilweise auf
Selbstverständlichkeiten wie `visibility:visible!important` und `opacity:1!important`. Das
ist kein Stilproblem, sondern ein **Symptom**: Weil derselbe Block dreimal existiert, haben
sich die Versionen gegenseitig überschrieben und wurden mit Gewalt gewonnen. Ab diesem Punkt
kann man das Modal nicht mehr sinnvoll umgestalten, ohne alles anzufassen.

### 7.3 Zwei Generationen Formular-Steuerelemente nebeneinander

Alte Variante (JS setzt eine Klasse, alles `!important`):
```css
#dynamicModal .checkbox-item.selected .checkbox-square{ background:#f2994a!important }
```
Neue Variante (rein deklarativ, modernes CSS):
```css
#dynamicModal .checkbox-input:checked + .checkbox-visual{ background:#f2994a }
#dynamicModal .radio-button-card:has(.radio-input:checked){ background:#f2994a1a }
```

Die neue ist deutlich besser: natives Input bleibt im DOM (Tastatur, `required`,
Formular-Serialisierung funktionieren), kein JS nötig. Sie ist offenbar als Ersatz gebaut
worden — nur wurde die alte nie gelöscht.

### 7.4 Toter Code (unvollständige Liste)

`.team-hero`, `.hero-content` (globale Variante), `.page-hero`, `.section-nav`,
`.dropdown-menu` + `.mobile-submenu` (kein Dropdown-Markup vorhanden), `.about-image`,
`.image-overlay`, `.play-button` (Version im About-Block), `.values-section`,
`.footer-certifications`, `.swiper*`, `.element-gap` / `.small-gap` / `.large-gap`,
`@keyframes checkmark-draw`, `.hero` **doppelt definiert** (einmal im Layout-Scope, einmal
im Hero-Component-Scope, mit minimal abweichender `background-position`).

### 7.5 Datenformatierung im Podcast-Block

```html
<div class="duration-badge">19</div>
<span class="meta-item">19</span>
<span class="meta-item">#52</span>
<div class="stat"><span class="num">26+</span><span class="label">Folgen</span></div>
```

Die Dauer wird als nacktes „19" ausgegeben — ohne Einheit, an zwei Stellen. Vermutlich sind
19 Minuten gemeint. Gleichzeitig behauptet die Statistik „26+ Folgen", während die
angezeigte Folge die Nummer **#52** trägt. Eine der beiden Zahlen ist falsch, und die 26 ist
offensichtlich hartcodiert, während die Foldennummer aus den Daten kommt.

Ebenso: Beim Mietobjekt steht `CHF 1'900` ohne „/Monat" — neben Kaufpreisen von
`CHF 1'580'000`. Formatierungslogik, die den Objekttyp berücksichtigt, fehlt.

### 7.6 Kleinigkeiten

- `key="…"` auf `<article>` — ungültiges HTML-Attribut, React-Überbleibsel.
- `© 2026` hartcodiert statt `{new Date().getFullYear()}`.
- Logo: `width="120" height="60"` im Attribut, `height:132px` im CSS → Layout-Sprung beim
  Anwenden des Stylesheets, und ein 132-px-Logo in einer 80-px-Leiste ist der Grund für das
  ganze `overflow:visible`/`z-index`-Gebastel im Header.
- `?cache_bust=2025` am Team-Banner — Cloudinary versioniert bereits über den `v…`-Pfad; der
  Query-Parameter verhindert teilweise CDN-Caching.
- Titel-Tippfehler im Podcast: „trozdem" (aus YouTube übernommen, aber zweimal ausgegeben).
- `<h3>` für Testimonial-Namen innerhalb einer Karte — semantisch fragwürdig, besser `<p>`
  mit `<strong>`.

---

## 8. Priorisierte Massnahmenliste

### Sofort (Sicherheit — vor allem anderen)
1. **Supabase-RLS auf allen Tabellen prüfen**, insbesondere Schreib-Policies für `anon`.
2. **Öffentliche Registrierung in Supabase Auth deaktivieren**, MFA für Admin-Konten aktivieren.
3. **SELECT-Policy auf der Immobilien-Tabelle** auf `status = 'published'` einschränken.
4. **Prüfen, ob der Service-Role-Key im Frontend-Bundle liegt.**

### Diese Woche (grosse Wirkung, kleiner Aufwand)
5. **Supabase-Bilder transformieren** — mit Abstand grösster Performance-Hebel.
6. **Kaputtes `srcset` und die Textleiche im Hero reparieren** — zwei Zeilen.
7. **YouTube-Thumbnail über Cloudinary spiegeln** — löst das Datenschutzproblem ganz.
8. **Kontrast der CTA-Buttons korrigieren** — dunkles Blau statt Weiss auf Gold.
9. **Security-Header** in `netlify.toml` ergänzen.
10. **Karussell-Pfeile sichtbar machen**, Autoplay bei Hover/Fokus anhalten.

### Diesen Monat (Aufräumen)
11. **CSS-Duplikate auflösen:** eine Quelle der Wahrheit pro Komponente, alte
    Modal-Generation und toten Code löschen, danach `!important` entfernen.
12. **`inlineStylesheets: 'auto'`** setzen, damit CSS über Seiten hinweg cachebar wird.
13. **`prefers-reduced-motion` und `:focus-visible`** global ergänzen.
14. **Emoji durch SVG-Icons ersetzen**, Sterne barrierefrei auszeichnen.
15. **Inline-`onclick` entfernen**, danach eine echte CSP setzen.
16. **`.float`-Kreise streichen**, `pulse-glow` auf `opacity` umstellen.
17. **Podcast-Daten korrigieren:** Dauer mit Einheit, Folgenzahl dynamisch, Mietpreis
    mit „/Monat".

---

## 9. Was ich nicht prüfen konnte

- Die beiden JS-Bundles (`DynamicModal…js`, `ImmoOttiChat…js`) — Egress war blockiert.
  Darin steckt die gesamte Formularvalidierung, der Absende-Code und das Chat-Backend.
- Der Admin-Bereich unter `/admin/login` und dessen Supabase-Anbindung.
- Die HTTP-Response-Header.
- Die Supabase-Policies selbst — nur im Dashboard oder Repo einsehbar.
- Die Unterseiten (Immobilien-Listing, Detailseiten, Blog, Bewertungsformular, Kontakt).

Ich habe bewusst **keine aktiven Tests** gegen die Website oder das Supabase-Projekt
durchgeführt. Sämtliche Sicherheitsaussagen beruhen ausschliesslich auf dem Lesen des
Codes, den der Server ohnehin an jeden Browser ausliefert.
