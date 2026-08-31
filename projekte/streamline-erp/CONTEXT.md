---
datum: 2026-08-20
projekt: streamline-erp
tags: [erp, zeiterfassung, hlks, pitch, kontext]
status: Pitch-Vorbereitung — kein Auftrag
---

# Streamline Engineering — Projektkontext ERP

## 0. Wie diese Datei zu nutzen ist

**Zweck:** Einmal Kontext festhalten, damit jede Claude-Session sofort weiss, worum es
geht — ohne dass du es neu erklären musst.

**Ablageort:** Diese Datei gehört ins **ERP-Projekt-Repo**, nicht ins `second-brain`.
Das folgt aus der eigenen Regel in `second-brain/CLAUDE.md`:

> Nicht hier: Website-Code, Kunden-Assets, projektspezifische `CLAUDE.md`-Dateien —
> die bleiben im jeweiligen Projekt-Repo.

Sobald das Repo existiert: diese Datei dort als `CLAUDE.md` ablegen. Ins `second-brain`
fliessen später nur die **allgemeinen** Learnings zurück.

**Kennzeichnung nach den Wissensklassen des Brains:**

| Marke | Bedeutung |
|---|---|
| **[F]** | Fakt — von dir bestätigt oder technisch eindeutig |
| **[A]** | Annahme — plausibel, aber unbestätigt. Vor dem Bauen verifizieren. |
| **[?]** | Offene Frage — muss beantwortet werden |

⚠️ **Diese Datei enthält bewusst mehr [A] und [?] als [F].** Das ist der aktuelle
Stand, nicht ein Mangel. Jede Annahme, die du beim Chef klärst, wird zu einem Fakt —
Abschnitt 10 ist die Liste zum Abarbeiten.

---

## 1. Die Firma

| | |
|---|---|
| **Name** | Streamline Engineering |
| **Branche** | Gebäudetechnik / HLKS-Planung **[F]** |
| **Grösse** | 1–5 Personen **[F]** |
| **Standort** | Zentralschweiz? **[A]** — relevant für Sprache (de-CH) und Kantonsbezug |
| **Deine Rolle** | Angestellt, Beteiligung geplant **[F]** |
| **Projektstatus** | **Pitch.** Kein Auftrag, kein Budget, keine Zusage. **[F]** |
| **Rechtsform** | **[?]** — AG/GmbH/Einzelfirma. Relevant für Buchhaltungspflichten |
| **Gründungsjahr** | **[?]** |
| **Anzahl Projekte/Jahr** | **[?]** — bestimmt, ob 20 oder 200 Projekte im System landen |

### Was das Pitch-Ziel ist

Du willst **abklären, was möglich ist, was du brauchst und wie lange es dauert** —
und das dann dem Chef vorlegen. Das heisst für diese Datei: Sie muss nicht nur
technisch tragen, sondern auch die drei Fragen beantworten, die dein Chef stellen wird:

1. *Was kostet das?*
2. *Wie lange dauert es?*
3. *Warum nicht einfach eine fertige Software kaufen?*

Frage 3 ist die kritische. Abschnitt 8 behandelt sie ehrlich — **inklusive der Fälle,
in denen Kaufen die bessere Antwort ist.** Ein Pitch, der diese Frage nicht vorwegnimmt,
fällt in der ersten Minute auseinander.

---

## 2. Was ein HLKS-Planungsbüro fürs ERP besonders macht

Das ist der Teil, der ein generisches Zeiterfassungstool von einem passenden
unterscheidet.

### Projekte laufen in Phasen, nicht am Stück **[A — verifizieren]**

Planungsleistungen in der Gebäudetechnik werden üblicherweise nach dem
**SIA-Phasenmodell** (SIA 112) strukturiert, Honorare nach **SIA 108** (Ordnung für
Leistungen und Honorare der Ingenieure der Bereiche Gebäudetechnik):

```text
1  Strategische Planung
2  Vorstudien
3  Projektierung        (31 Vorprojekt · 32 Bauprojekt · 33 Bewilligungsverfahren)
4  Ausschreibung
5  Realisierung         (51 Ausführungsprojekt · 52 Ausführung · 53 Inbetriebnahme)
6  Bewirtschaftung
```

**Warum das fürs ERP zentral ist:** Wenn Stunden nur auf „Projekt X" gebucht werden,
weisst du am Ende nicht, ob die Projektierung oder die Bauleitung das Budget gefressen
hat. Die Phase muss eine eigene Dimension im Datenmodell sein — nicht ein Textfeld.

**[?] Beim Chef klären:** Arbeitet ihr nach SIA-Phasen? Nach welchen genau? Oder habt
ihr ein eigenes Raster? **Wenn ihr ein eigenes verwendet, ist das die wichtigste
Einzelinformation für das ganze Projekt** — davon hängt die Struktur der Datenbank ab.

### Honorarmodelle **[A — verifizieren]**

Typisch in der Branche sind drei Varianten, oft gemischt im selben Betrieb:

| Modell | Was das ERP dafür braucht |
|---|---|
| **Nach Zeitaufwand** | Stundensätze pro Person oder Funktion, Erfassung nach Tätigkeit |
| **Pauschal / Globalhonorar** | Stundenbudget pro Phase + Nachkalkulation (war es kostendeckend?) |
| **Nach Baukosten-Prozent** | Bausumme am Projekt, daraus abgeleitetes Honorar |

**[?] Welche Modelle nutzt ihr?** Wenn Pauschalen vorkommen, ist die **Nachkalkulation
das eigentliche Killer-Feature** — nicht die Zeiterfassung selbst. Dann lautet das
Pitch-Argument nicht „wir erfassen Zeit", sondern „wir sehen endlich, welche
Pauschalprojekte Geld verlieren".

### Jeder arbeitet an mehreren Projekten gleichzeitig

Bei 1–5 Personen und parallel laufenden Projekten ist die Kernfrage nicht „wer hat wann
gearbeitet", sondern **„haben wir nächsten Monat noch Kapazität für einen weiteren
Auftrag?"**. Genau das nennst du mit „Auslastungserfassung" — und genau das können die
meisten einfachen Zeiterfassungs-Tools **nicht**.

Das ist dein stärkstes Build-Argument (siehe Abschnitt 8).

---

## 3. Zielbild

### Version 1 — was gebaut wird **[F]**

- **Zeiterfassung** — Stunden auf Projekt + Phase + Tätigkeit buchen
- **Auslastungserfassung** — Soll-Kapazität vs. geplante vs. tatsächliche Stunden
- **Projektführung** — Stundenbudget pro Phase, Ist-Stand, Rest, Nachkalkulation

### Version 2 — später, ausbaubar **[F]**

- Offerten erstellen
- Rechnungen (inkl. Schweizer QR-Rechnung)

### Ausdrücklich **nicht** in v1

Lohnbuchhaltung · Finanzbuchhaltung · Material- und Lagerwirtschaft · Spesen ·
Ferien-/Absenzenverwaltung · Dokumentenablage · CRM · Kundenportal

> **Scope-Disziplin ist hier der wichtigste Erfolgsfaktor.** Der häufigste Grund,
> warum selbstgebaute ERPs scheitern, ist nicht Technik — es ist ein Scope, der
> während des Bauens wächst. Wenn im Pitch-Gespräch „und könnte man nicht auch noch…"
> kommt: aufschreiben, auf die v2-Liste, nicht ins v1 aufnehmen.

### Ausdrücklicher Charakter: Programm, nicht Automation **[F]**

Deine Formulierung: *„nicht als automation sondern als zugängliches programm"*.

Das ist eine wichtige und richtige Abgrenzung, und sie hat konkrete Folgen:

- **Es ist eine Web-App mit Login**, die man täglich öffnet — kein n8n-Workflow.
- Das Agency-Prinzip aus `entscheidungen/2026-08-n8n-automationsprinzip.md`
  (deterministische Logik nicht per LLM) gilt hier verschärft: **In einem ERP hat
  kein LLM etwas zu suchen.** Stundensummen, Budgetberechnungen und Auslastungen sind
  deterministisch und müssen jedes Mal exakt dasselbe Ergebnis liefern.
- Falls später KI: nur an unkritischen Rändern (z. B. Textvorschlag für eine
  Offertenbeschreibung), nie in der Berechnung.

---

## 4. Datenmodell (Startpunkt zum Bauen)

Bewusst klein gehalten. Alles, was v1 nicht braucht, fehlt hier absichtlich.

```text
mitarbeiter
  id · vorname · nachname · email · aktiv
  pensum_prozent            -- 100 = Vollzeit, Basis der Auslastung
  stundensatz_intern        -- Kostensatz für Nachkalkulation
  stundensatz_extern        -- Verrechnungssatz
  rolle                     -- admin | mitarbeiter

kunde
  id · name · ort · aktiv
  -- bewusst schlank: kein CRM in v1

projekt
  id · nummer · bezeichnung · kunde_id
  status                    -- offerte | aktiv | pausiert | abgeschlossen
  honorarmodell             -- aufwand | pauschal | baukosten_prozent
  bausumme                  -- nur bei baukosten_prozent
  budget_stunden_total
  start_datum · end_datum_geplant

projekt_phase
  id · projekt_id
  code                      -- z.B. "32" (SIA) oder eigenes Raster
  bezeichnung               -- z.B. "Bauprojekt"
  budget_stunden            -- ← die entscheidende Zahl für Nachkalkulation
  sortierung
  abgeschlossen

taetigkeitsart
  id · bezeichnung          -- z.B. Planung, Bauleitung, Sitzung, Fahrzeit, Administration
  verrechenbar              -- true/false ← wichtig: nicht jede Stunde ist fakturierbar
  aktiv

zeiteintrag
  id · mitarbeiter_id · projekt_id · projekt_phase_id · taetigkeitsart_id
  datum · stunden           -- dezimal (7.25), nicht Start/Ende — schneller zu erfassen
  bemerkung
  gesperrt                  -- true nach Monatsabschluss
  erstellt_am · geaendert_am

kapazitaet_planung          -- Grundlage der Auslastungssicht
  id · mitarbeiter_id · projekt_id
  kalenderwoche · jahr
  geplante_stunden
```

### Vier Entscheidungen darin, die bewusst so getroffen sind

1. **`stunden` als Dezimalzahl statt Start-/Endzeit.** Erfassung dauert Sekunden statt
   einer halben Minute. Für Planungsbüros ist das der Standard.
   ⚠️ **Aber:** Falls die Arbeitszeiterfassung nach Arbeitsgesetz mit abgedeckt werden
   soll, braucht es zusätzlich Tagesbeginn, Tagesende und Pausen (siehe Abschnitt 6).
   **Diese Frage vor dem Bauen klären — sie ändert das Datenmodell.**
2. **`verrechenbar` an der Tätigkeitsart.** Ohne dieses Feld kannst du den
   Verrechnungsgrad nicht berechnen — die wichtigste Kennzahl eines Planungsbüros.
3. **`gesperrt` am Zeiteintrag.** Ohne Monatssperre ändert jemand rückwirkend Stunden,
   die schon fakturiert sind.
4. **`kapazitaet_planung` wochenweise, nicht tageweise.** Tagesgenaue Planung wird in
   einem 5-Personen-Betrieb erfahrungsgemäss nie gepflegt. Wochen sind der Punkt, an
   dem der Aufwand den Nutzen noch rechtfertigt.

---

## 5. Die drei Kernfunktionen

### 5.1 Zeiterfassung — hier entscheidet sich alles

> **Die härteste Anforderung des ganzen Projekts:** Eine Zeiterfassung, die pro Eintrag
> länger als etwa 15 Sekunden dauert, wird nicht benutzt. Dann wird freitags aus dem
> Gedächtnis nachgetragen, die Daten werden ungenau, und das System ist wertlos —
> technisch einwandfrei und trotzdem gescheitert.

Was daraus folgt:

- **Wochenansicht als Hauptbildschirm**, nicht ein Formular pro Eintrag
- **„Letzte Woche kopieren"** — bei wiederkehrenden Projekten der meistgenutzte Knopf
- **Zuletzt verwendete Projekte zuoberst**, nicht alphabetisch
- **Mobil bedienbar** — Stunden werden auf der Baustelle oder im Auto erfasst,
  nicht am Bürorechner **[A — bei euch prüfen]**
- **Nachträgliche Korrektur erlaubt**, bis der Monat gesperrt ist
- **Tastaturbedienbar**: Projekt tippen → Tab → Stunden → Enter

**[?] Wie wird heute erfasst?** Excel? Papier? Gar nicht? Das ist die wichtigste
Frage für die Akzeptanz: Wenn heute nichts erfasst wird, ist das Projekt eine
**Verhaltensänderung** und nicht eine Softwareeinführung — und dann ist der Widerstand
das Hauptrisiko, nicht der Code.

### 5.2 Auslastungserfassung

Zwei Sichten, die zusammengehören:

- **Rückblick:** Wie viele Stunden hat wer letzte Wochen tatsächlich gebucht?
  Verhältnis verrechenbar zu nicht verrechenbar.
- **Vorausschau:** Was ist für die nächsten 4–8 Wochen geplant, verglichen mit dem
  Pensum? Ampel: unter 80 % → Kapazität frei, über 110 % → Überlast.

Genau die Vorausschau beantwortet die Frage, die der Chef wöchentlich hat: *Können wir
den Auftrag noch annehmen?*

### 5.3 Projektführung

Pro Projekt eine Tabelle über die Phasen:

```text
Phase              Budget    Ist      Rest     Verbrauch
32 Bauprojekt      120 h     94 h     26 h     78 %  ▓▓▓▓▓▓▓▓░░
52 Ausführung      200 h    218 h    −18 h    109 %  ▓▓▓▓▓▓▓▓▓▓ ⚠
```

Plus **Nachkalkulation** bei Pauschalprojekten: Honorar gegen (Ist-Stunden ×
Kostensatz) — verdient oder draufgelegt? Das ist die Zahl, die den ganzen Aufwand
rechtfertigt.

---

## 6. Schweizer Rahmenbedingungen

> **[A] Alle Punkte hier auf aktuellen Stand prüfen** — Gesetze und Sätze ändern.
> Bei den arbeitsrechtlichen Punkten im Zweifel Treuhänder fragen, nicht mich.

### Arbeitszeiterfassung ist grundsätzlich Pflicht — dein stärkstes Pitch-Argument

Das Arbeitsgesetz (ArG Art. 46, ArGV 1 Art. 73) verpflichtet Arbeitgeber, Verzeichnisse
über die Arbeitszeit zu führen. Es gibt Erleichterungen und Verzichtsmöglichkeiten
(ArGV 1 Art. 73a/73b) unter bestimmten Bedingungen — etwa bei höheren Einkommen mit
grosser Autonomie oder gestützt auf einen GAV.

**Warum das für den Pitch zählt:** Es verschiebt die Frage von *„nice to have"* zu
*„müssen wir ohnehin"*. Falls heute nicht sauber erfasst wird, ist das kein
Software-Wunsch mehr, sondern das Schliessen einer Lücke.

⚠️ **Aber ehrlich bleiben:** Ob und in welcher Form die Pflicht bei euch greift, hängt
von Anstellungsverhältnissen und Löhnen ab. **Nicht als Druckmittel im Pitch verwenden,
bevor es geprüft ist** — wenn der Chef die Ausnahme kennt und du nicht, verlierst du
Glaubwürdigkeit für den ganzen Rest.

**[?] Klären:** Muss das ERP die arbeitsgesetzliche Zeiterfassung abdecken (Tagesbeginn,
Tagesende, Pausen) oder nur die Projektstunden? Das ändert das Datenmodell.

### Weiteres (relevant ab v2)

| Thema | Stand **[A]** | Betrifft |
|---|---|---|
| MWST-Normalsatz | 8.1 % seit 1.1.2024 | Rechnungen (v2) |
| QR-Rechnung | Pflicht, alte Einzahlungsscheine abgelöst (Ende Sept. 2022) | Rechnungen (v2) |
| Aufbewahrungspflicht | 10 Jahre für Geschäftsunterlagen (OR 958f) | **Backup-Konzept ab v1** |
| revDSG | Mitarbeiterdaten sind Personendaten | Datenhaltung, Zugriffsrechte |

**Praktische Folge für v1:** Datenhaltung in der Schweiz oder EU wählen, und ein
Backup, das ihr auch dann noch lesen könnt, wenn das System eines Tages abgeschaltet
wird. Ein CSV-Export aller Zeiteinträge reicht dafür — aber er muss **von Anfang an**
existieren, nicht „später mal".

---

## 7. Technischer Stack

### Ableitung aus der bestehenden Agency Decision

`second-brain/entscheidungen/2026-08-tech-stack-auswahlprinzip.md` gibt den
Entscheidungsbaum vor. Erste Frage darin:

> Braucht es Login / Datenbank / personalisierte App-Logik? → **Ja: Next.js prüfen**

Ein ERP erfüllt alle drei Kriterien. Der Stack folgt also aus deiner eigenen
verbindlichen Entscheidung, er ist nicht neu erfunden.

### Konkreter Vorschlag **[A]**

| Schicht | Wahl | Warum |
|---|---|---|
| Framework | **Next.js (App Router) + TypeScript** | Aus dem Entscheidungsbaum |
| Datenbank | **PostgreSQL** (Supabase oder Neon, Region EU/CH) | Relationale Daten, saubere Auswertungen |
| Auth | Supabase Auth oder Auth.js | 5 Nutzer, keine Exotik nötig |
| UI | Tailwind + shadcn/ui | Schnell, konsistent, kein eigenes Designsystem nötig |
| Tabellen/Charts | TanStack Table + Recharts | Auswertungen sind der Kern der App |
| Hosting | Vercel + verwaltete DB | Kein Serverbetrieb |
| Export | CSV/Excel serverseitig | Pflicht ab Tag 1 (siehe Abschnitt 6) |

**Laufende Kosten [A]:** Bei 5 Nutzern realistisch **CHF 0–50 pro Monat**. Auf den
kostenlosen Stufen ist das anfangs oft ganz gratis — aber im Pitch **mit Kosten
rechnen**, nicht mit null. Ein Chef, dem du „gratis" versprichst und der später eine
Rechnung sieht, erinnert sich an das Versprechen.

### Übertrag aus dem immo-otti.ch-Review — nicht denselben Fehler machen

Bei der Analyse von immo-otti.ch war der grösste offene Sicherheitspunkt genau dieses
Muster: **Supabase + browserseitiges Admin-Panel ohne belegte Row Level Security.**

Für dieses Projekt heisst das, verbindlich:

1. **RLS auf jeder Tabelle von der ersten Migration an**, nicht nachgerüstet.
   Der Grund, warum es so oft fehlt: Ohne RLS funktioniert lokal alles — der Fehler
   fällt erst auf, wenn es zu spät ist.
2. Jeder Mitarbeitende sieht **nur eigene Zeiteinträge**; Auswertungen über alle nur
   für die Admin-Rolle. Bei Lohn- und Kostensätzen im System ist das keine Kür.
3. **Keine Berechnung im Browser, die man manipulieren könnte** — Summen, Budgets und
   Sperren serverseitig.
4. Der Service-Role-Key gehört **ausschliesslich** in Server-Umgebungsvariablen.

---

## 8. Build vs. Buy — die Frage, die dein Chef stellen wird

Das ist der ehrlichste Abschnitt dieser Datei, und er ist absichtlich unbequem.

### Der Fall gegen Selberbauen

Für 1–5 Personen gibt es fertige Lösungen mit Projektzeiterfassung — im Schweizer
Umfeld etwa **Bexio**, **MOCO**, **Vertec** (auf Ingenieur- und Planungsbüros
ausgerichtet), **Proffix**, **Clockodo**. **[A — Funktionsumfang und Preise selbst
prüfen, ich kann sie hier nicht verifizieren.]**

Deren Vorteile sind real und dürfen nicht kleingeredet werden:

- Morgen einsatzbereit statt in drei Monaten
- Support, wenn etwas nicht geht
- Rechtliche Updates (MWST-Sätze, QR-Rechnung) kommen automatisch
- **Funktioniert weiter, wenn du das Unternehmen verlässt**

Bei fünf Personen sind die Lizenzkosten überschaubar. Nüchtern gerechnet ist gekaufte
Software in den ersten ein bis zwei Jahren fast sicher die günstigere Variante, wenn man
deine Arbeitszeit realistisch bewertet.

### Der Fall fürs Selberbauen

Er trägt nur, wenn mindestens einer dieser Punkte zutrifft:

1. **Die SIA-Phasenstruktur passt in keine Standardsoftware.** Wenn ihr eure
   Nachkalkulation heute in Excel macht, *weil* das Tool die Phasen nicht kann — dann
   ist das ein echtes Argument. **Das lässt sich in einer Stunde Testzugang prüfen.**
2. **Auslastungsvorausschau.** Genau das, was du willst, ist in einfachen
   Zeiterfassungs-Tools selten gut gelöst. Prüfenswert.
3. **Datenhoheit und Ausbaubarkeit.** Später Offerten und Rechnungen dranzubauen ist
   im eigenen System eine Erweiterung, in fremder Software ein Anbieterwechsel.
4. **Dein Anteil.** Du wirst Mitinhaber. Ein System, das ihr besitzt, ist ein
   Firmenwert; eine Lizenz ist eine Kostenposition. Das ist ein legitimes Argument —
   aber es ist ein **unternehmerisches**, kein technisches. Sag es als solches.

### Was ich dir empfehle

**Geh nicht mit „ich baue ein ERP" ins Gespräch, sondern mit einer Abklärung.**

> „Ich habe angeschaut, was uns bei Zeiterfassung und Auslastung fehlt. Bevor wir
> Lizenzen kaufen, würde ich in zwei Wochen zwei Dinge machen: zwei fertige Tools
> testen, und einen lauffähigen Prototyp bauen. Danach entscheiden wir mit echten
> Zahlen statt aus dem Bauch."

Das ist ein deutlich stärkerer Pitch als ein Dreimonatsversprechen, weil:

- er den Chef **nichts kostet ausser zwei Wochen deiner Zeit**,
- er die Kaufen-Frage nicht umgeht, sondern beantwortet,
- er dich als jemanden zeigt, der Optionen prüft statt sein Lieblingsprojekt zu
  verkaufen — und das ist genau die Eigenschaft, die man bei einem künftigen
  Mitinhaber sehen will.

**Falls der Test zeigt, dass ein fertiges Tool passt: dann ist das ein Erfolg, kein
Scheitern.** Du hast der Firma drei Monate Arbeit und ein Wartungsrisiko erspart.

---

## 9. Aufwand, Mittel und Zeitplan

**[A] Alle Schätzungen unter der Annahme: du arbeitest ungefähr einen Tag pro Woche
daran, mit Claude als Umsetzungshilfe. Bei weniger Zeit verlängert es sich linear.**

| Phase | Inhalt | Aufwand |
|---|---|---|
| **0 · Abklärung** | Fragen aus Abschnitt 10 klären, 2 fertige Tools testen, Entscheid Build/Buy | **1–2 Wochen** |
| **1 · Fundament** | Repo, Datenmodell, Migrationen, RLS, Auth, Stammdaten-Verwaltung | 2 Wochen |
| **2 · Zeiterfassung** | Wochenansicht, schnelle Eingabe, Korrektur, Monatssperre, CSV-Export | 2–3 Wochen |
| **3 · Auswertung** | Projektübersicht mit Budget/Ist, Auslastungssicht, Nachkalkulation | 2 Wochen |
| **4 · Pilot** | Echte Daten, parallel zum bisherigen Vorgehen, Korrekturen | 4 Wochen |

**Bis produktiv nutzbar: rund 3 Monate nebenberuflich.**
Bis „fertig" gibt es nicht — ein ERP wird laufend angepasst. Das gehört in den Pitch,
nicht in die Fussnote.

### Was du brauchst

| | |
|---|---|
| **Zeit** | ~1 Tag/Woche über 3 Monate ≈ 12–15 Arbeitstage |
| **Geld** | CHF 0–50/Monat Betrieb + dein Claude-Abo |
| **Vom Chef** | Freigabe deiner Arbeitszeit · Zugang zu echten Projektdaten für den Test · **eine Person, die den Piloten wirklich mitmacht** |
| **Nicht nötig** | Server, Lizenzen, externe Dienstleister |

### Risiken — offen benennen, das stärkt den Pitch

| Risiko | Wie ernst | Gegenmassnahme |
|---|---|---|
| **Niemand erfasst die Stunden** | **Hoch** — häufigster Grund für das Scheitern | Erfassung unter 15 Sekunden; Pilot mit einer Person, bevor alle umstellen |
| **Bus-Faktor 1** | **Hoch** | Standard-Stack ohne Exotik, dokumentiertes Setup, CSV-Export jederzeit. **Diesen Punkt selbst ansprechen** — der Chef denkt ihn ohnehin |
| **Scope wächst** | Mittel | v1-Liste aus Abschnitt 3 schriftlich fixieren |
| **Aufwand unterschätzt** | Mittel | Nach Phase 2 neu bewerten, statt drei Monate blind durchziehen |
| **Datenverlust** | Mittel | Verwaltete DB mit automatischem Backup + wöchentlicher CSV-Export |

---

## 10. Was du beim Chef abklären musst

Diese Liste ist das eigentliche Werkzeug für dein nächstes Gespräch. Jede Antwort
verwandelt ein **[A]** oben in ein **[F]**.

### Zum Geschäft
1. Wie werden Stunden heute erfasst — Excel, Papier, gar nicht?
2. Wie wird heute abgerechnet: nach Aufwand, pauschal, nach Baukosten-Prozent?
3. Arbeitet ihr nach SIA-Phasen? Nach welchen genau, oder nach einem eigenen Raster?
4. Wie viele Projekte laufen gleichzeitig, wie viele pro Jahr?
5. Gibt es unterschiedliche Stundensätze pro Person oder pro Tätigkeit?
6. Welche Tätigkeiten sind nicht verrechenbar (Akquise, interne Sitzungen, Fahrzeit)?

### Zum Schmerzpunkt — die wichtigste Frage
7. **Was genau geht heute schief?** Wisst ihr am Monatsende nicht, was fakturierbar
   ist? Kommen Pauschalprojekte defizitär raus? Nimmt jemand Aufträge an, obwohl kein
   Platz mehr ist?

> Wenn es auf diese Frage keine klare Antwort gibt, ist das ein **Warnsignal**. Ein ERP
> ohne konkreten Schmerz wird gebaut und dann nicht benutzt. Dann ist die richtige
> Empfehlung, das Projekt zu verschieben — auch wenn das nicht die Antwort ist, die du
> hören willst.

### Zum Rechtlichen
8. Muss die arbeitsgesetzliche Zeiterfassung mit abgedeckt werden, oder nur
   Projektstunden?
9. Gibt es einen Treuhänder, dessen Anforderungen an die Daten wir kennen sollten?

### Zum Rahmen
10. Darfst du Arbeitszeit dafür verwenden, und wie viel?
11. Wurde schon einmal ein Tool evaluiert? Woran ist es gescheitert?
12. Wer ausser dir müsste es täglich benutzen — und wie steht die Person dazu?

---

## 11. Nächste Schritte

1. **Abschnitt 10 mit dem Chef durchgehen** — das ist der Pitch, nicht eine
   Vorbereitung darauf. Die Fragen zeigen mehr Kompetenz als eine fertige Lösung.
2. **Antworten hier eintragen**, [A] durch [F] ersetzen.
3. **Zwei fertige Tools testen** (Testzugang, echte Projektstruktur nachbauen,
   Zeitbedarf messen).
4. **Dann entscheiden** — und wenn gebaut wird: Repo anlegen, diese Datei als
   `CLAUDE.md` hineinlegen, mit Phase 1 starten.

### Was ich beisteuern kann, sobald die Antworten da sind

- Datenmodell als lauffähige SQL-Migrationen inklusive RLS-Policies
- Prototyp der Zeiterfassung, damit du im Pitch etwas zeigen statt beschreiben kannst
- Vergleichsraster für die Tool-Evaluation
- Eine Pitch-Unterlage aus den Zahlen dieser Datei

**Ein Prototyp der Wochenansicht ist der grösste Hebel für dein Gespräch.** Etwas, das
man anfassen kann, überzeugt anders als eine Aufwandschätzung — und er lässt sich in
wenigen Stunden bauen, lange bevor über drei Monate entschieden werden muss.
