# Quellcode kopieren und analysieren lassen

Kurzanleitung, wie du den Quellcode einer Website an Claude gibst, damit die
`site-teardown`-Skill daraus ein komplettes Build-Blueprint macht.

## Variante 1 — nur die URL (am einfachsten)

Schreib einfach:

```
/site-teardown https://beispiel.ch
```

oder "Analysier mir diese Site: https://beispiel.ch".

Claude holt HTML, JS und CSS selber (WebFetch). Nachteil: bei sehr grossen
Seiten wird der Inhalt beim Abholen teilweise zusammengefasst, also weniger
Detail.

## Variante 2 — HTML selber kopieren (bestes Resultat)

Im Browser auf der Zielseite:

1. `Ctrl + U` (Mac: `Cmd + Option + U`) — zeigt den Seitenquelltext
2. `Ctrl + A` — alles markieren
3. `Ctrl + C` — kopieren
4. In den Chat einfügen, mit einem Satz davor, z.B.
   "Das ist der Quellcode von https://beispiel.ch — mach mir ein Teardown."

Tipp: den eingefügten Code in einen Code-Block packen (drei Backticks davor
und danach), dann bleibt die Formatierung sauber.

### Wenn der Code zu lang für den Chat ist

Als Datei ablegen statt einfügen:

```
# Seite speichern
curl -sL https://beispiel.ch -o /tmp/seite.html

# dann im Chat:
"Analysier /tmp/seite.html"
```

Oder die Datei im Repo ablegen (z.B. `sources/beispiel-ch.html`) und den Pfad
nennen. Claude liest Dateien direkt — das hat kein Längenlimit wie der Chat.

## Variante 3 — HTML + JS + CSS (vollständig)

Für maximale Tiefe zusätzlich die Hauptdateien mitliefern. Im DevTools-Tab
"Network" oder im HTML nach diesen Zeilen suchen:

```html
<link rel="stylesheet" href="/assets/main.css">
<script src="/assets/app.js"></script>
```

Diese URLs mitschicken oder herunterladen:

```bash
curl -sL https://beispiel.ch/assets/main.css -o /tmp/main.css
curl -sL https://beispiel.ch/assets/app.js  -o /tmp/app.js
```

Dritt-Skripte (Google Analytics, Cookie-Banner, reCAPTCHA, CDN-Libraries)
braucht es nicht — die gehören nicht zum eigenen Code der Seite.

## Was du zurückbekommst

Ein Markdown-Dokument unter `research/YYYY-MM-DD-{site}-teardown.md` mit:

- Tech-Stack (belegt aus dem Quellcode, nicht geraten)
- Design-System: Farben, Fonts, Spacing, Breakpoints
- Effekt-Tabelle: jeder Effekt, wie er umgesetzt ist, wie schwer nachbaubar
- Implementierungs-Details mit echten Code-Snippets
- Asset-Liste und Build-Plan Section für Section

## Die Skill

Liegt in `.claude/skills/site-teardown/SKILL.md` und wird automatisch aktiv,
sobald du eine URL oder rohes HTML schickst und nach einer Analyse fragst.
