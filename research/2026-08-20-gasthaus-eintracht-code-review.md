# Code- und Sicherheitsanalyse: gasthaus-eintracht.ch

**Analysiert:** Startseite, ausgelieferter Quellcode
**Datum:** 2026-08-20
**Bauart:** Einzelne statische HTML-Datei, kein Build-Schritt, kein Framework.
CSS und JS vollständig inline. Menüdaten per `fetch('daten.json')` zur Laufzeit,
Reservation über das Drittanbieter-Widget Localina.
**Nicht geprüft:** `speisekarte-print.html`, `getraenke-print.html`, `daten.json`,
der erwähnte Admin-Bereich, HTTP-Header. Egress zu der Domain war blockiert.

**Offenlegung:** Diese Seite ist in Zusammenarbeit mit mir entstanden. Ein Teil der
Befunde unten geht auf meine eigene Arbeit zurück. Ich habe sie genauso bewertet wie
fremden Code — ein Audit, das die eigene Arbeit schont, ist wertlos.

---

## 1. Gesamturteil

| Bereich | Note | Kurzfassung |
|---|---|---|
| Architektur-Entscheidung | **8/10** | Eine Datei, keine Abhängigkeiten, nichts zu warten — für ein Restaurant genau richtig |
| Rechtliches (Impressum) | **7/10** | UID, HR-Nummer, Rechtsform vorhanden — das machen die wenigsten |
| SEO / strukturierte Daten | **6/10** | Solide Basis, aber Canonical-Problem und ungenutztes Potenzial |
| Code-Qualität | **4/10** | ~25–30 % totes CSS, Inline-Styles gegen Stylesheet, doppelte Media Query |
| Performance | **6/10** | Sehr leicht, aber acht Bilder ohne Lazy Loading und 87 KB jQuery für einen Button |
| Barrierefreiheit | **3/10** | Fliesstext der ganzen Seite unter Kontrastminimum, Menükarten per Tastatur unerreichbar |
| Sicherheit | **6/10, ein offener Punkt** | Keine Nutzerdaten auf dem Server, aber eine XSS-Senke, deren Tragweite von `daten.json` abhängt |
| Datenschutz | **3/10** | Nennt den falschen Dienstleister und verschweigt Google Fonts |

### Die ehrliche Zusammenfassung

Die **Grundentscheidung ist richtig**: Ein Gasthaus braucht kein Astro, kein React und kein
CMS. Eine Datei, die man per FTP hochlädt, hat keine Sicherheitsupdates, keine
Dependency-Hölle und keine Build-Pipeline, die in zwei Jahren nicht mehr läuft. Das ist
langlebiger als 90 % dessen, was Agenturen für ein KMU bauen.

Die **Ausführung hat dieselbe Krankheit wie immo-otti.ch**, nur in kleinerem Massstab: Es
wurde iterativ ergänzt und nie aufgeräumt. Die auffälligste Spur davon ist ein komplettes,
selbstgebautes Reservationsformular mit Kalender — CSS für Modal, Formularfelder,
Kalendergitter, Zeitfenster, Erfolgsmeldung, rund 150 Zeilen — das im HTML **gar nicht mehr
existiert**, weil es durch Localina ersetzt wurde. Im JavaScript steht sogar noch der
Kommentar `// CALENDAR`, dem kein Kalender mehr folgt.

Und dann gibt es zwei Befunde, die mich wirklich stören, weil sie den Gast direkt betreffen:
**der Fliesstext der ganzen Seite ist zu hell zum Lesen**, und **die Öffnungszeiten sind die
am schlechtesten lesbare Information im Footer** — bei einem Restaurant ausgerechnet.

---

## 2. Was wirklich gut ist

- **Keine Abhängigkeiten ausser Localina.** Kein npm, kein Build, nichts, was verrottet.
- **Sauberes Design-System** über CSS Custom Properties. Die Gold-auf-Dunkel-Kombination
  (`#C9A84C` auf `#1A1814`) erreicht **≈ 7,8:1** — deutlich über dem Minimum, sehr gut gewählt.
- **Der Localina-Fallback ist richtig gedacht:**
  ```js
  if (window.Localina && typeof Localina.startBooking === 'function') { … }
  else { alert('Für Ihre Reservation rufen Sie uns bitte an: +41 41 450 12 52'); }
  ```
  Wenn das Widget nicht lädt, bekommt der Gast eine Telefonnummer statt eines toten Buttons.
  Genau so gehört das gemacht.
- **`font-size: 16px` für Formularfelder auf Mobil** — verhindert das automatische Zoomen von
  iOS Safari. Ein Detail, das fast alle übersehen.
- **`role="img"` + `aria-label` auf den Galeriebildern** — korrekt, da es
  Hintergrundbilder auf Divs sind und kein `alt` haben können.
- **Escape schliesst alle drei Modals**, zentral in einem Handler.
- **`{ passive: true }`** auf beiden Scroll-Listenern.
- **IntersectionObserver mit `unobserve`** nach dem Auslösen — kein Speicherleck.
- **Impressum mit UID (`CHE-142.398.016`), Handelsregisternummer und Rechtsform.** Für einen
  gewerblichen Schweizer Auftritt gesetzlich nötig und meist vergessen.
- **Keine Cookies, kein Tracking.** Deshalb kein Consent-Banner nötig.

---

## 3. Sicherheit

### 3.1 XSS-Senke: `innerHTML` mit ungefilterten Daten aus `daten.json`

Das ist der wichtigste technische Befund. An drei Stellen wandern Daten aus `daten.json`
ungeprüft in `innerHTML`:

```js
function renderPdfContent(data) {
  return data.map(s => `
    <div class="pdf-dish-section">
      <h4>${s.section}</h4>
      ${s.items.map(i => `
        <div class="pdf-dish-name">${i.name}</div>
        ${i.desc ? `<div class="pdf-dish-desc">${i.desc}</div>` : ''}
        <div class="pdf-dish-price">${i.price}</div>
      `).join('')}
    </div>`).join('');
}
```
```js
document.getElementById('pdf-modal-body').innerHTML = content + `<a href="${file}" …>`;
box.innerHTML = `… <span class="saisonal-item-name">${i.name}</span> …`;
```

Ein Gerichtname wie `<img src=x onerror="…">` in `daten.json` führt beliebiges JavaScript im
Kontext der Website aus — bei jedem Besucher, der die Karte öffnet. Das ist eine klassische
Stored-XSS-Kette.

**Wie schlimm das ist, hängt an einer Frage, die ich nicht beantworten kann:
Wer darf `daten.json` schreiben?**

- **Wenn die Datei nur per FTP/SFTP von Hand hochgeladen wird:** geringes Risiko. Dann müsste
  jemand bereits Server-Zugriff haben, und wer den hat, braucht kein XSS mehr.
- **Wenn es einen Admin-Bereich gibt, der `daten.json` über einen HTTP-Endpunkt schreibt**
  (der Kommentar `// SAISONAL MODAL – Inhalt live aus daten.json (synchron mit Admin)` deutet
  darauf hin): dann ist die Frage, wie dieser Endpunkt geschützt ist. Ist er es nicht oder
  nur schwach, kann ein Fremder Schadcode in die Website einschleusen, den jeder Besucher
  ausführt. **Das wäre kritisch.**

**Der Fix ist unabhängig davon sinnvoll und kostet fünf Zeilen** — die Werte werden über
`textContent` statt `innerHTML` gesetzt, oder man escapt vorher:

```js
const esc = s => String(s ?? '').replace(/[&<>"']/g,
  c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'}[c]));
```
und dann konsequent `${esc(i.name)}`, `${esc(i.desc)}`, `${esc(i.price)}`, `${esc(s.section)}`.

Das behebt nebenbei einen ganz unspektakulären Alltagsfehler: Ein Gericht mit `&` im Namen
(„Fisch & Chips") oder einem `<` in der Beschreibung zerlegt sonst das Layout.

### 3.2 jQuery vom fremden CDN ohne Subresource Integrity

```html
<script src="https://code.jquery.com/jquery-3.7.0.min.js"></script>
```

Kein `integrity`, kein `crossorigin`. Wird `code.jquery.com` kompromittiert oder die
DNS-Auflösung manipuliert, läuft fremder Code mit vollen Rechten auf der Seite. Fix:

```html
<script src="https://code.jquery.com/jquery-3.7.0.min.js"
        integrity="sha256-…" crossorigin="anonymous"></script>
```
Den Hash liefert die jQuery-Website. Noch besser: jQuery selbst hosten — dann fällt auch der
Drittanbieter-Verbindungsaufbau weg.

Beim Localina-Widget geht SRI nicht, weil das Skript per Query-Parameter konfiguriert wird
und sich ändern kann. Das ist ein bewusst eingekaufter Vertrauensvorschuss gegenüber einem
Schweizer Reservationsanbieter — vertretbar, aber man sollte wissen, dass man ihn gibt.

### 3.3 Keine strenge CSP möglich

Rund 30 Inline-`onclick`-Handler, mehrere Inline-`<script>`-Blöcke, ein grosser
Inline-`<style>`-Block und dutzende `style=""`-Attribute. Eine CSP müsste
`script-src 'unsafe-inline'` erlauben und wäre als XSS-Schutz wertlos — was in Kombination
mit 3.1 doppelt ärgerlich ist.

Realistisch: Der Umbau auf `addEventListener` lohnt sich hier nur, wenn ohnehin aufgeräumt
wird. Dann aber gleich mitmachen.

### 3.4 Was hier angenehm einfach ist

Kein eigenes Backend, keine Datenbank, keine Benutzerkonten, keine Formulardaten auf dem
Server. Die Reservationsdaten gehen direkt an Localina, das Angriffsfläche und Verantwortung
übernimmt. Das ist sicherheitstechnisch eine **gute** Entscheidung — es gibt schlicht sehr
wenig zu kompromittieren.

Empfehlung trotzdem, falls der Hoster es erlaubt (`.htaccess` oder Header-Konfiguration):
```
X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Strict-Transport-Security: max-age=31536000; includeSubDomains
```
`X-Frame-Options` verhindert, dass jemand die Seite in einen iFrame packt und Reservationen
abfängt.

---

## 4. Datenschutz — hier ist der grösste Handlungsbedarf

### 4.1 Die Datenschutzerklärung nennt den falschen Dienstleister

```html
<strong>Reservations- und Kontaktformular:</strong> Die von Ihnen eingegebenen Angaben
… werden zur Bearbeitung Ihrer Reservation über den Dienst <strong>Formspree</strong>
übermittelt …
```

**Auf der Seite gibt es kein Formspree-Formular.** Reservationen laufen über **Localina**
(`mylocalina.ch`) — dorthin gehen Name, Telefonnummer, E-Mail und Reservationsdetails.

Die Datenschutzerklärung beschreibt damit eine Datenbearbeitung, die nicht stattfindet, und
verschweigt diejenige, die tatsächlich stattfindet. Das ist der Kernpunkt einer
Datenschutzerklärung — nicht eine Formalie. Das gehört korrigiert, bevor irgendetwas anderes
angefasst wird.

Das ist übrigens ein Überbleibsel derselben Ablösung wie das tote Kalender-CSS: Erst gab es
ein eigenes Formular über Formspree, dann kam Localina, und der Rechtstext ist beim alten
Stand geblieben.

**Neuer Text (Entwurf, ersetzt den Formspree-Absatz):**

> **Reservationen:** Für Tischreservationen nutzen wir den Dienst Localina der Localina AG
> (Schweiz). Wenn Sie über unsere Website reservieren, werden die von Ihnen eingegebenen
> Angaben (Name, E-Mail, Telefonnummer, Datum, Uhrzeit, Personenzahl und allfällige
> Bemerkungen) direkt an Localina übermittelt und dort zur Verwaltung Ihrer Reservation
> bearbeitet. Es erfolgt keine Weitergabe an Dritte zu Werbezwecken.
> Beim Öffnen des Reservationsfensters wird zudem Ihre IP-Adresse an Localina übertragen.

### 4.2 Google Fonts wird geladen und nicht erwähnt — und der Code weiss das

```html
<link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond…" rel="stylesheet">
```

Bei jedem Seitenaufruf geht die IP-Adresse des Besuchers an Google in die USA, ohne
Einwilligung und ohne Hinweis. Das Bemerkenswerte: **Im Quellcode steht der Hinweis darauf,
und zwar von uns selbst geschrieben:**

```html
<!-- HINWEIS (intern, nicht öffentlich sichtbar): Datenschutztext vor Veröffentlichung
     durch eine fachkundige Person prüfen lassen. Bei Einsatz von Google Maps,
     Google Fonts oder Analyse-Tools muss dieser Abschnitt ergänzt werden. -->
```

Der Hinweis ist korrekt und wurde nie umgesetzt. Zwei Anmerkungen dazu:

1. **„intern, nicht öffentlich sichtbar" stimmt nicht.** HTML-Kommentare stehen für jeden im
   Seitenquelltext. Interne Notizen gehören nicht ins ausgelieferte Dokument.
2. **Der beste Fix ist nicht, den Text zu ergänzen, sondern das Problem zu entfernen:**
   Beide Schriften selbst hosten. Cormorant Garamond und Jost stehen unter der SIL Open Font
   License, dürfen also frei mitgeliefert werden.

```html
<!-- statt des Google-Links: -->
<style>
@font-face { font-family:'Jost'; src:url('fonts/jost-400.woff2') format('woff2');
             font-weight:400; font-display:swap; }
@font-face { font-family:'Cormorant Garamond'; src:url('fonts/cormorant-300.woff2') format('woff2');
             font-weight:300; font-display:swap; }
/* … je Schnitt einer */
</style>
```

Das löst gleichzeitig ein Performance-Problem (siehe 6.3) und ein Rechtsproblem. Beste
Massnahme der ganzen Liste, gemessen an Aufwand zu Wirkung.

### 4.3 Ein Detail zur Sorgfalt

Der eingebettete Google-Maps-**Link** (`target="_blank"`) ist unproblematisch — es wird
nichts geladen, bis der Gast klickt. Das ist die richtige Variante gegenüber einer
eingebetteten Karte. Gut gelöst.

---

## 5. Inhaltliche Fehler, die Gäste sehen

### 5.1 Die Fallback-Speisekarte zeigt erfundene Gerichte mit erfundenen Preisen

```js
const DEFAULT_SK=[{section:"Vorspeisen",items:[
  {name:"Bündner Gerstensuppe", …, price:"CHF 9.50"},
  …
  {name:"Zürcher Geschnetzeltes", …, price:"CHF 36.50"},
  {name:"Rindsfilet vom Grill", …, price:"CHF 52.00"},
  {name:"Forellenfilet", desc:"Aus dem Luzerner See, Mandelbutter, Dampfkartoffeln",
   price:"CHF 32.00"},
```

Diese Daten werden angezeigt, **wann immer `daten.json` nicht rechtzeitig lädt** — bei
langsamer Verbindung, Serverfehler, Tippfehler im JSON, oder schlicht wenn jemand die
Speisekarte anklickt, bevor der Fetch zurück ist.

Drei Probleme, aufsteigend nach Ernst:

1. **„Aus dem Luzerner See" gibt es nicht.** Der See bei Luzern heisst Vierwaldstättersee.
   Ein solcher Fehler auf der Karte eines Innerschweizer Gasthauses fällt jedem
   Einheimischen sofort auf. Das ist der deutlichste Beleg dafür, dass diese Daten
   Platzhalter sind und nie jemand gegengelesen hat.
2. **Kein einziges mediterranes oder italienisches Gericht** in einer Fallback-Karte, obwohl
   die ganze Seite mit „Schweizer & mediterrane Küche" und „feine italienische Akzente"
   positioniert ist.
3. **Falsche Preise sind ein geschäftliches Risiko.** Wenn ein Gast „CHF 36.50" liest und im
   Lokal 42 Franken bezahlt, ist das im besten Fall peinlich. Die Schweizer
   Preisbekanntgabeverordnung nimmt Preisangaben ernst.

**Fix:** Die Fallback-Konstanten durch eine ehrliche Meldung ersetzen.

```js
const DEFAULT_SK = [];
const DEFAULT_GK = [];
// und in openPdfModal(), wenn totalItems === 0:
'<p>Die Karte konnte gerade nicht geladen werden. Bitte rufen Sie uns an: ' +
'<a href="tel:+41414501252">+41 41 450 12 52</a> — oder versuchen Sie es kurz erneut.</p>'
```

Der Mechanismus dafür existiert bereits (`totalItems ? … : '…'`), er zeigt derzeit nur die
falsche Alternative.

### 5.2 Datum in der Vergangenheit, Text in der Zukunft

```html
<p>Ab 1. Juli 2026 übernimmt Familie Rondinelli das Gasthaus Eintracht in Root
   und führt es mit frischer Energie weiter.</p>
```

Heute ist der 20. August 2026. Das liest sich, als wäre die Seite seit Wochen nicht
angefasst worden. Besser:

> Seit dem 1. Juli 2026 führt Familie Rondinelli das Gasthaus Eintracht in Root …

Ebenso: Die saisonale Karte ist mit „Frühling / Sommer 2026" beschriftet — im August noch
knapp korrekt, ab September nicht mehr. Diese Beschriftung sollte aus `daten.json` kommen,
damit sie ohne HTML-Änderung aktualisierbar ist.

### 5.3 Öffnungszeiten in den strukturierten Daten sind irreführend

```json
"openingHoursSpecification": [{ "dayOfWeek": ["Monday",…,"Saturday"],
                                "opens": "09:00", "closes": "23:30" }]
```

Im Footer steht dagegen: Küche 11:30–14:00 und 17:30–21:30.

Google zeigt aufgrund dieser Daten „geöffnet" an, wenn jemand um 15:30 nach einem Restaurant
sucht — und der Gast steht vor einer Küche, die geschlossen ist. Für ein Restaurant ist das
die häufigste Ursache enttäuschter Erstbesucher.

Sauber wäre, die Betriebszeiten zu behalten und die Küchenzeiten explizit zu ergänzen:

```json
"openingHoursSpecification": [
  { "@type":"OpeningHoursSpecification",
    "dayOfWeek":["Monday","Tuesday","Wednesday","Thursday","Friday","Saturday"],
    "opens":"09:00","closes":"23:30" }
],
"hasMenu": "https://gasthaus-eintracht.ch/speisekarte-print.html",
"geo": { "@type":"GeoCoordinates","latitude":47.1123116,"longitude":8.3876533 },
"acceptsReservations": "https://www.mylocalina.ch/…"
```

Die Koordinaten stehen bereits im Google-Maps-Link im Footer — sie müssen nur übernommen
werden. `hasMenu` und `geo` sind zwei Zeilen mit spürbarem Local-SEO-Effekt.

---

## 6. Performance

### 6.1 Acht Bilder ohne Lazy Loading

Sämtliche Bilder sind CSS-`background-image` auf Divs — im Hero, in den Menükarten, in der
Galerie und in der Zoom-Ansicht. Folgen:

- **Kein `loading="lazy"` möglich.** Alle sechs Galeriebilder plus die Zoom-Kopien werden
  sofort geladen, obwohl sie beim ersten Bildschirm nicht sichtbar sind. Bei
  Hintergrundbildern greift die Browser-Optimierung nur, wenn das Element gar nicht
  dargestellt wird — die Galerie ist aber im Layout, nur ausserhalb des Viewports.
- **Kein `srcset`.** Ein Handy lädt dieselbe Datei wie ein 27-Zoll-Monitor.
- **Nicht in der Google-Bildersuche.** Für ein Restaurant mit guten Foodfotos ist das
  verschenkte Sichtbarkeit.

Dazu kommt: Die sechs Galeriebilder existieren **zweimal im Dokument** — einmal im Slider,
einmal in der Zoom-Ansicht. Selbe Dateien, also nur ein Download je Bild, aber doppeltes
Markup, das bei jeder Änderung an zwei Stellen gepflegt werden muss.

**Empfehlung, nach Aufwand sortiert:**
1. Alle Bilder als WebP konvertieren und auf max. 1600 px Breite bringen (Squoosh, kostenlos).
   Bei unoptimierten Handyfotos sind das typischerweise 80–90 % weniger Bytes.
2. Galerie auf echte `<img loading="lazy" alt="…">` umstellen, mit `object-fit: cover`.
   Löst Lazy Loading, Bildersuche und Alt-Texte in einem Schritt.
3. Das Hero-Bild vorladen:
   ```html
   <link rel="preload" as="image" href="img/interior.jpg" fetchpriority="high">
   ```

Ich kann die Dateigrössen von hier nicht messen — bitte einmal im Netzwerk-Tab der
Entwicklertools nachsehen. Wenn `interior.jpg` über 300 KB liegt, ist das der grösste
Einzelhebel der Seite.

### 6.2 87 KB jQuery für einen Button

```html
<script src="https://code.jquery.com/jquery-3.7.0.min.js"></script>
<script src="https://www.mylocalina.ch/script/widget.js?…"></script>
<script>jQuery.noConflict();</script>
```

jQuery wird ausschliesslich von Localina benötigt — im eigenen Code kommt kein einziges `$`
vor. Ändern lässt sich das nicht, ohne Localina zu ersetzen. Was geht:

```html
<script defer src="…jquery…"></script>
<script defer src="…widget.js…"></script>
```

`defer` bewirkt, dass beide das Rendern nicht mehr verzögern und in Reihenfolge nach dem
Parsen laufen. Der bestehende Fallback in `reservieren()` deckt genau den Fall ab, dass
jemand vorher klickt — die Absicherung ist also schon da.

### 6.3 Google Fonts kostet zwei Verbindungen und blockiert das Rendern

```html
<link href="https://fonts.googleapis.com/css2?family=Cormorant+Garamond:ital,wght@
      0,300;0,400;0,500;0,600;1,300;1,400;1,500&family=Jost:wght@300;400;500;600
      &display=swap" rel="stylesheet">
```

Das sind **elf Schriftschnitte** (sieben Cormorant inklusive vier Kursiven, vier Jost). Im
CSS verwendet werden davon: Cormorant in 300/400/500 (keine Kursive), Jost in 400/500/600.
Es werden also mindestens fünf Schnitte geladen, die nirgends vorkommen — darunter alle
vier kursiven.

Dazu ist es ein rendering-blockierender Request an eine fremde Domain, ohne `preconnect`.

Selbst hosten löst das komplett: nur die sechs tatsächlich genutzten Schnitte, keine fremde
Verbindung, kein Datenschutzproblem. Falls das kurzfristig nicht geht, wenigstens die
kursiven aus der URL streichen und ergänzen:
```html
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
```

### 6.4 Ein wirklich schlanker Rest

Positiv festgehalten: Ohne Bilder und jQuery ist das Dokument klein, es gibt keinen
Framework-Overhead, keine Hydration, keinen Router. Die Seite dürfte auf einem
Mittelklasse-Handy sehr schnell interaktiv sein. Die Probleme oben sind alle Zulieferung,
nicht Struktur.

---

## 7. Barrierefreiheit — der schwächste Bereich

### 7.1 Der Fliesstext der ganzen Seite ist zu hell

```css
--text-muted: #8A8070;
--cream: #F5F0E8;
```

| Kombination | Kontrast | WCAG AA (4.5:1) |
|---|---|---|
| `#8A8070` auf `#F5F0E8` (cream) | **≈ 3,4 : 1** | ✗ verfehlt |
| `#8A8070` auf `#FDFCF8` (weiss) | **≈ 3,8 : 1** | ✗ verfehlt |
| `rgba(255,255,255,.45)` auf `#1A1814` (Öffnungszeiten) | **≈ 4,4 : 1** | ✗ knapp verfehlt |
| `rgba(255,255,255,.30)` auf `#1A1814` (Wochentage) | **≈ 2,7 : 1** | ✗ klar verfehlt |
| `rgba(255,255,255,.25)` auf `#1A1814` (Impressum/Datenschutz-Links) | **≈ 2,2 : 1** | ✗ klar verfehlt |
| `rgba(255,255,255,.35)` auf `#1A1814` (Gerichtbeschreibungen im Modal) | **≈ 3,2 : 1** | ✗ verfehlt |
| `rgba(255,255,255,.40)` auf Hero-Overlay (`.hero-tag`) | **≈ 3,9 : 1** | ✗ verfehlt |
| `#C9A84C` auf `#1A1814` (Gold auf Dunkel) | ≈ 7,8 : 1 | ✓ sehr gut |

`--text-muted` trägt praktisch den gesamten Fliesstext: `.about-text p`, `.phil-text p`,
`.menu-card p`, die drei Werte-Absätze, `.saisonal-item-desc`, `.modal-note`. Bei 0,80–0,88 rem
(13–14 px) ist 3,4:1 für ältere Gäste — die Kernzielgruppe eines Gasthauses — schwer lesbar.

**Fix, eine Zeile:**
```css
--text-muted: #6E6558;   /* statt #8A8070 → ≈ 5,0:1 auf cream */
```
Optisch bleibt es ein warmes Grau, der Charakter geht nicht verloren.

**Und für den Footer** — hier ist es besonders ärgerlich, weil ausgerechnet die
Öffnungszeiten betroffen sind, die wichtigste Information einer Restaurantseite:
```css
.hours-row      { color: rgba(255,255,255,0.75); }  /* statt .45 */
.hours-row .day { color: rgba(255,255,255,0.55); }  /* statt .30 */
.footer-bottom-links a { color: rgba(255,255,255,0.55); }  /* statt .25 */
.pdf-dish-desc  { color: rgba(255,255,255,0.60); }  /* statt .35 */
```
Die Impressum- und Datenschutz-Links bei 2,2:1 sind zusätzlich heikel: Es sind
Pflichtangaben, und sie sind praktisch unsichtbar. Auf Bildschirmen unter 480 px werden sie
sogar ganz ausgeblendet (`.footer-bottom-links { display: none }`) — auf dem Handy gibt es
dann **gar keinen sichtbaren Weg zu Impressum und Datenschutz**. Das würde ich ändern.

### 7.2 Die vier Menükarten sind per Tastatur nicht erreichbar

```html
<div class="menu-card" onclick="openPdfModal('speisekarte')" style="cursor:pointer;">
  …
  <span class="menu-link">Zum Menü</span>
</div>
```

Ein `<div>` mit `onclick` bekommt keinen Fokus, reagiert nicht auf Enter oder Leertaste und
wird Screenreadern nicht als bedienbar angekündigt. `.menu-link` ist ein `<span>`, also
ebenfalls kein Bedienelement. Dasselbe gilt für die sechs Galeriebilder
(`<div onclick="openGalleryZoom(0)">`).

Das heisst: **Wer nicht mit der Maus arbeitet, kommt an die Speisekarte auf der Startseite
nicht heran.** Immerhin gibt es Auswege — die Buttons „Speisekarte ansehen & drucken" und
„Alle Bilder ansehen" sind echte Elemente und funktionieren. Aber die primäre
Interaktion der Seite ist es nicht.

**Fix:** `<div>` durch `<button type="button">` ersetzen, mit
`text-align:left; background:none; border:none; font:inherit; width:100%;`. Fokus, Enter,
Leertaste und die Screenreader-Ankündigung kommen dann von selbst.

### 7.3 Die Navigations-Markierung funktioniert überhaupt nicht

```html
<li><a href="javascript:void(0)" onclick="navScrollTo('speisekarte')">Speisekarte</a></li>
```
```js
const navMap = { '#start':'start', '#speisekarte':'speisekarte', … };
document.querySelectorAll('.nav-links a').forEach(a => {
  const href = a.getAttribute('href');            // → "javascript:void(0)"
  const target = navMap[href] || href.replace('#', '');  // → "javascript:void(0)"
  a.classList.toggle('active', target === current);      // → immer false
});
```

`navMap` erwartet Hash-Links wie `#speisekarte`, die Links liefern aber
`javascript:void(0)`. Der Lookup schlägt fehl, der Fallback `href.replace('#','')` gibt den
unveränderten String zurück, und `target === current` ist für jeden Link falsch.

Da `updateNav()` beim Laden sofort einmal läuft, wird sogar das im HTML gesetzte
`class="active"` auf „Start" wieder entfernt. **Die goldene Unterstreichung in der Navigation
erscheint also zu keinem Zeitpunkt.**

Das ist ein Fehler, der beim Umbau der Links auf `javascript:void(0)` entstanden ist —
`updateNav()` wurde nicht mitgezogen. Zwei mögliche Fixe:

*Minimal:* ein `data-target`-Attribut ergänzen.
```html
<li><a href="javascript:void(0)" data-target="speisekarte" onclick="navScrollTo('speisekarte')">…</a></li>
```
```js
const target = a.dataset.target;
```

*Besser (löst gleich mehrere Probleme):* echte Anker-Links verwenden.
```html
<li><a href="#speisekarte">Speisekarte</a></li>
```
Das `scroll-margin-top: 64px` im CSS erledigt den Versatz unter der fixen Navigation bereits —
`navScrollTo()` wird dann gar nicht mehr gebraucht. Vorteile: Rechtsklick → „In neuem Tab
öffnen" funktioniert, Screenreader kündigen ein Ziel an, und `#speisekarte` wird teilbar.

Der ursprüngliche Grund für `javascript:void(0)` war, die URL sauber zu halten
(`history.replaceState`). Bei einem Restaurant ist ein teilbarer Link zur Speisekarte
allerdings mehr wert als eine rautenfreie Adresszeile.

### 7.4 Modals ohne Fokus-Verwaltung

Alle drei Overlays (Speisekarte, Saisonal, Galerie-Zoom):

- kein `role="dialog"` / `aria-modal="true"`
- der Fokus springt beim Öffnen nicht ins Modal
- **kein Fokus-Trap** — mit Tab wandert man hinter dem Overlay durch die Seite
- der Fokus kehrt beim Schliessen nicht zum auslösenden Element zurück

Escape funktioniert, das ist die halbe Miete. Der Rest sind etwa 20 Zeilen für alle drei
gemeinsam.

### 7.5 Weitere Punkte

- **Kein `prefers-reduced-motion`** — bei `scroll-behavior: smooth`, den Fade-in-Animationen
  und den `scrollIntoView({behavior:'smooth'})`-Aufrufen.
- **Der Hamburger-Button** hat `aria-label="Menü"` (gut), aber kein `aria-expanded` und kein
  `aria-controls`. Screenreader sagen nicht, ob das Menü offen ist.
- **Kein Skip-Link** zum Hauptinhalt.
- **Kein `:focus-visible`-Stil** definiert. Auf echten Buttons greift die Browser-Vorgabe,
  auf den Div-Karten gibt es gar keinen Fokus (siehe 7.2).
- **`☎` und `✉` als Icons** werden vorgelesen („Schwarzes Telefon"). `aria-hidden="true"`
  darum setzen.
- **`🇨🇭` in der Navigation** — Windows stellt Flaggen-Emoji nicht als Flagge dar, sondern
  als Buchstaben „CH". Auf einer Seite mit sonst sehr durchdachter Typografie ein Bruch.
- **Die Teamnamen sind `<div>`**, während die Werte-Überschriften `<h3>` sind. Inkonsistent
  in der Dokumentstruktur.

---

## 8. Code-Qualität

### 8.1 Rund 25–30 % des CSS ist tot

Das gesamte selbstgebaute Reservationssystem existiert nur noch als Stylesheet:

`.modal-overlay`, `.modal`, `.modal-header`, `.modal-body`, `.modal-close`, `.form-row`,
`.form-group`, `.form-divider`, `.calendar-mini`, `.cal-header`, `.cal-nav`, `.cal-month`,
`.cal-days-header`, `.cal-day-name`, `.cal-grid`, `.cal-cell` (+ `.cal-empty`, `.cal-past`,
`.cal-closed`, `.cal-selected`, `.cal-today`), `.cal-legend`, `.cal-dot`, `.time-slots`,
`.time-slot` (+ `.selected`, `.unavailable`), `.modal-note`, `.btn-submit`, `.success-msg`,
`.success-icon`.

Kein einziges dieser Elemente kommt im HTML vor. Im JavaScript steht der verwaiste Kommentar:

```js
  // CALENDAR
  // Smooth scroll ohne URL-Raute
  function navScrollTo(id) { … }
```

Dazu die zugehörigen Mobile-Overrides in **zwei** `@media (max-width: 768px)`-Blöcken.

### 8.2 Die doppelte Media Query

Am Ende des Stylesheets steht:

```css
/* === MOBILE-FIX Reservation (muss NACH den Desktop-Regeln stehen) === */
@media (max-width: 768px) { .modal-overlay { … } .modal { … } … }
```

Der Kommentar benennt die Ursache korrekt: Der erste Mobile-Block steht **vor** der
`.modal`-Definition für den Desktop, wird also von ihr überschrieben. Die saubere Lösung
wäre gewesen, die Reihenfolge zu korrigieren — stattdessen wurde der Block dupliziert. Das
ist genau derselbe Mechanismus, der bei immo-otti.ch zum `!important`-Wildwuchs geführt hat,
hier zum Glück noch im Frühstadium. Da beide Blöcke ohnehin totes CSS betreffen, löst sich
das beim Aufräumen von selbst.

### 8.3 Inline-Styles gegen das Stylesheet

Die Abschnitte „Werte" und „Team" sind vollständig mit `style=""`-Attributen gestaltet — im
Gegensatz zum restlichen Dokument, das mit Klassen arbeitet. Das erzwingt dann:

```css
.werte-grid { grid-template-columns: 1fr !important; gap: 1.25rem !important; }
.team-grid  { grid-template-columns: 1fr !important; gap: 0.9rem !important; }
#ueber-uns > div { padding: 2rem 1.25rem !important; }
```

Drei `!important`, die es ohne Inline-Styles nicht bräuchte. Das Muster ist eindeutig: Diese
Abschnitte wurden später ergänzt, ohne das Stylesheet anzufassen — schnell im Ergebnis,
teuer in der Pflege.

Ähnlich bei `.menu-img`: Das Stylesheet setzt `height: 150px`, jedes einzelne Element
überschreibt inline auf `140px`, und die Mobile-Regel braucht dann `height: 80px !important`.

### 8.4 Doppelte ID

```html
<footer class="footer" id="kontakt">
<div id="reservation" style="position:absolute; margin-top:-64px;"></div>
  <div class="footer-info">
    <div class="footer-col" id="reservation">
```

`id="reservation"` kommt zweimal vor — ungültiges HTML. Der erste ist ein alter
Anker-Versatz-Hack, der durch `scroll-margin-top` überflüssig geworden ist. Ersatzlos
streichen. (Der `navMap`-Eintrag `'#reservation': 'kontakt'` gehört zur selben Altlast.)

### 8.5 Kleinere Fundstücke

- **Tippfehler im Funktionsnamen:** `closeSaionalOutside` (statt `Saisonal`). Funktioniert,
  weil er konsistent falsch geschrieben ist.
- **Veralteter Kommentar:** `<!-- Favicon: SVG inline als Data-URI -->` — es folgen fünf
  ganz normale Dateireferenzen, keine Data-URI.
- **Erledigtes TODO:** `<!-- TODO: telephone ergänzen, sobald bekannt -->` steht direkt über
  einem JSON-LD-Block, in dem `telephone` längst ausgefüllt ist.
- **Tote CSS-Eigenschaften:** `object-fit: cover` auf `.menu-img` und `.gallery-zoom-img` —
  beides sind `<div>`-Elemente mit Hintergrundbild, wo `object-fit` nichts tut.
  Ebenso `.hero-eyebrow::before { display: none; }`.
- **`body { overflow-x: hidden; max-width: 100vw; }`** — `100vw` schliesst die Breite der
  vertikalen Bildlaufleiste ein und kann selbst horizontales Überlaufen erzeugen. Es wird
  hier nur durch `overflow-x: hidden` kaschiert. Beides streichen und die eigentliche
  Ursache suchen wäre sauberer.
- **`height: 100vh` im Hero:** Auf Mobilgeräten ist `100vh` grösser als der sichtbare
  Bereich, solange die Adressleiste eingeblendet ist. Der Mobile-Breakpoint entschärft das
  mit `80vh`. Im Modal-CSS wird bereits `95dvh` verwendet — die moderne Einheit ist also
  bekannt, nur nicht überall eingesetzt. `100dvh` wäre konsistent.
- **Zwei Navigationseinträge, ein Ziel:** „Reservation" und „Kontakt" scrollen beide zu
  `kontakt`.
- **Drei Telefonnummern im Footer ohne Beschriftung** — der Gast weiss nicht, welche er
  wählen soll. Eine als „Restaurant", die anderen als „Mobil" kennzeichnen.

### 8.6 Zwei Interaktionsfehler in der Galerie

**Nach dem Ziehen öffnet sich das Bild.** Der Slider unterstützt Ziehen mit der Maus:
```js
slider.addEventListener('mousedown', e => { isDragging = true; … });
document.addEventListener('mouseup', () => { isDragging = false; … });
```
Nach dem Loslassen feuert der Browser aber zusätzlich ein `click`-Ereignis auf der Kachel,
unter der die Maus liegt — und damit `openGalleryZoom(i)`. Wer die Galerie verschiebt,
landet danach in der Vollbildansicht.

*Fix:* Beim Loslassen merken, ob tatsächlich gezogen wurde, und den folgenden Klick einmalig
unterdrücken:
```js
let moved = false;
document.addEventListener('mousemove', e => { if (isDragging) { moved = true; … } });
slider.addEventListener('click', e => { if (moved) { e.stopPropagation(); moved = false; } }, true);
```
Dazu `user-select: none` auf `.gallery-slider`, sonst markiert das Ziehen den Inhalt.

**Inkonsistentes Verhalten am Rand:** Der Slider ist begrenzt
(`Math.max(0, Math.min(index, slides.length - 1))`), die Zoom-Ansicht läuft im Kreis
(`((index % TOTAL) + TOTAL) % TOTAL`). Am letzten Bild passiert beim Slider-Pfeil nichts,
während der Zoom-Pfeil zum ersten springt. Zusätzlich werden die Pfeile am Rand nicht
ausgegraut, sie wirken also defekt.

### 8.7 Ein hartcodierter Wert, der aus dem Ruder laufen wird

```js
const TOTAL_ZOOM = 6;
```
Kommt ein siebtes Galeriebild dazu, muss es an **drei** Stellen ergänzt werden: im Slider,
in der Zoom-Ansicht und in dieser Konstante. Vergisst man die Konstante, zeigt der Zähler
„1 / 6" bei sieben Bildern und die Umlaufnavigation überspringt eines.

*Fix:* `const TOTAL_ZOOM = zoomSlider.children.length;` — und besser noch die Zoom-Slides aus
demselben Array erzeugen wie die Slider-Kacheln, damit die Bildliste nur einmal existiert.

---

## 9. SEO

### 9.1 Canonical zeigt auf eine andere Domain als die ausgelieferte

```html
<link rel="canonical" href="https://gasthaus-eintracht.ch/">
<meta property="og:url" content="https://gasthaus-eintracht.ch/">
```
Aufgerufen wird die Seite unter **`https://www.gasthaus-eintracht.ch/`**.

Wenn `www` nicht per 301 auf die Variante ohne `www` weiterleitet, liefern zwei Adressen
denselben Inhalt aus. Google folgt zwar in der Regel dem Canonical, aber Verlinkungen,
Suchkonsolen-Daten und Social-Vorschauen verteilen sich auf zwei Hosts.

**Bitte prüfen:** `curl -I https://www.gasthaus-eintracht.ch/` — kommt dort `301` mit
`Location: https://gasthaus-eintracht.ch/`, ist alles in Ordnung. Kommt `200`, gehört eine
Weiterleitung eingerichtet.

*Anmerkung:* Für ein lokales Restaurant ist die `www`-Variante die vertrautere. Man kann
auch andersherum kanonisieren — Hauptsache eine Richtung, konsequent.

### 9.2 Ungenutztes Potenzial

- **`hasMenu` und `geo`** fehlen im Restaurant-Schema (siehe 5.3). Beides ist relevant für
  das Google-Business-Umfeld.
- **`sameAs`** fehlt komplett — falls es Google-Business-, Instagram- oder Facebook-Profile
  gibt, gehören sie hier verlinkt. Das ist einer der wirksamsten Local-SEO-Hebel überhaupt.
- **`meta name="keywords"`** wird seit 2009 von Google ignoriert. Schadet nicht, nützt
  nichts; das Keyword-Stuffing darin wirkt bei manueller Prüfung unschön.
- **`og:type="restaurant.restaurant"`** ist ein alter Facebook-Typ. `website` ist die
  robustere Wahl.
- **`lang="de"`** — bei durchgehend Schweizer Schreibweise („Grüsse", „Strasse") und
  `og:locale="de_CH"` wäre `lang="de-CH"` konsequenter.
- **`robots.txt` und `sitemap.xml`** konnte ich nicht prüfen. Bei drei bis vier Seiten kein
  Drama, aber schnell gemacht.
- **Kein Analytics.** Datenschutzfreundlich, heisst aber auch: keine Information darüber, ob
  jemand die Seite findet und ob Reservationen darüber zustandekommen. Plausible ist
  cookielos und braucht kein Banner — wäre eine Überlegung wert.

---

## 10. Priorisierte Massnahmenliste

### Sofort — rechtlich und inhaltlich
1. **Datenschutzerklärung korrigieren:** Formspree raus, Localina rein (Entwurfstext in 4.1).
2. **Google Fonts selbst hosten** — löst Datenschutz und Performance in einem Zug.
3. **Fallback-Speisekarte durch eine ehrliche Fehlermeldung ersetzen** — keine erfundenen
   Preise mehr ausliefern.
4. **„Ab 1. Juli 2026" → „Seit 1. Juli 2026".**
5. **Internen HTML-Kommentar entfernen** — er ist öffentlich sichtbar.

### Diese Woche — kleine Eingriffe, grosse Wirkung
6. **`--text-muted` auf `#6E6558` abdunkeln** — eine Zeile, hebt die Lesbarkeit der ganzen
   Seite über den Schwellenwert.
7. **Footer-Kontraste anheben**, besonders Öffnungszeiten und die Rechts-Links; die Links auf
   Mobil nicht mehr ausblenden.
8. **`escapeHtml()` einführen** und in allen drei `innerHTML`-Stellen anwenden.
9. **SRI-Hash bei jQuery ergänzen**, beide Skripte auf `defer`.
10. **Menükarten und Galeriekacheln zu `<button>` machen** — Tastaturbedienung.
11. **Navigation reparieren** — echte `#`-Anker verwenden (behebt auch 7.3 und macht Links
    teilbar).
12. **Doppelte `id="reservation"` entfernen.**
13. **`www`-Weiterleitung prüfen** und, falls nötig, einrichten.
14. **`hasMenu`, `geo` und `sameAs`** ins JSON-LD ergänzen.

### Diesen Monat — aufräumen
15. **Totes CSS löschen:** kompletter Reservationsmodal- und Kalenderblock samt der beiden
    Mobile-Overrides. Danach ist auch die doppelte Media Query weg.
16. **Bilder optimieren:** WebP, max. 1600 px, Galerie auf `<img loading="lazy" alt="…">`
    umstellen.
17. **Inline-Styles der Werte- und Team-Abschnitte** in Klassen überführen, die drei
    `!important` entfernen.
18. **`prefers-reduced-motion`, `:focus-visible`, `aria-expanded`** ergänzen.
19. **Fokus-Verwaltung in den Modals** (Trap, Fokus setzen, Fokus zurückgeben).
20. **Galerie-Ziehen:** Klick nach dem Ziehen unterdrücken, `TOTAL_ZOOM` dynamisch ermitteln,
    Slider und Zoom im Randverhalten angleichen.
21. **Security-Header** setzen, falls der Hoster es zulässt.
22. **Kleinkram:** `closeSaionalOutside` umbenennen, veraltete Kommentare entfernen, tote
    `object-fit`-Regeln streichen, `100dvh` im Hero, Telefonnummern beschriften,
    `aria-hidden` auf die Symbol-Zeichen.

---

## 11. Offene Fragen

1. **Wie kommt `daten.json` auf den Server?** Von Hand per FTP, oder über einen
   Admin-Bereich mit Schreib-Endpunkt? Davon hängt ab, ob 3.1 eine Formalie oder ein echtes
   Einfallstor ist. Falls es einen Admin gibt: Wo liegt er, und wie ist er geschützt?
2. **Sind die Preise in `DEFAULT_SK` / `DEFAULT_GK` echt?** Falls ja, ist Punkt 3 weniger
   dringend — dann sollten sie aber trotzdem nur aus `daten.json` kommen, damit es eine
   einzige Quelle gibt.
3. **Wo läuft das Hosting?** Entscheidet, ob Security-Header und die `www`-Weiterleitung
   überhaupt konfigurierbar sind.
4. **Soll ich `speisekarte-print.html`, `getraenke-print.html` und den Admin-Bereich auch
   ansehen?** Die Druckseiten dürften dasselbe `daten.json` verwenden und damit dieselbe
   Escaping-Frage haben.
