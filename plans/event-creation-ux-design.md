# Event-Erstellung: Brainstorming `quietschente.ch/create`

## Context

Der Eventorganisator soll auf `/create` einen neuen Event einfach und schrittweise erstellen können. Die Kernhypothese: Erst Termin finden → dann Koordination (wer bringt was). Die Seite muss für alle 5 Event-Typen funktionieren und benutzerfreundlich sein, ohne Account oder Passwort.

**Bestehende Typen:** `pizza 🍕` | `bbq 🔥` | `wandern 🥾` | `spiel 🎲` | `default 🎉`  
**Bestehende Phasen:** Idee → Terminsuche → Einladung → Anmeldung → Vorbereitung → Durchführung → Nachbereitung → Wiederengagement

---

## Brainstorming: Fragestellungen pro Event-Typ

### Legende
- 🔴 **MUSS** — Pflichtangabe bei Erstellung (ohne das kein Event möglich)
- 🟡 **SPÄTER** — Kann nach Erstellung ergänzt werden (via Organizer-Link)
- 🟢 **SYSTEM** — Kann das System vorhersehen/ableuten/vorbefüllen

---

### Universelle Fragen (alle Event-Typen)

| # | Frage | Priorität | Begründung |
|---|-------|-----------|------------|
| 1 | Was für ein Event ist es? (Typ-Auswahl) | 🔴 MUSS | Bestimmt alle nachfolgende Defaults |
| 2 | Wie heisst der Anlass? (Titel) | 🔴 MUSS | Identifikation des Events |
| 3 | Wann findet er statt? (Datum + Uhrzeit) | 🔴 MUSS* | *Oder: mehrere Terminvorschläge für Terminsuche |
| 4 | Wo findet er statt? (Ort/Adresse) | 🔴 MUSS | Ohne Ort können Leute nicht kommen |
| 5 | Wie erreiche ich dich? (Organizer-Email) | 🔴 MUSS | Für Organizer-Link, Benachrichtigungen |
| 6 | Wieviele Personen maximal? (Kapazität) | 🟡 SPÄTER | Kann auch offen bleiben; bei BBQ/Wandern wichtiger |
| 7 | Kurze Beschreibung / Einladungstext | 🟡 SPÄTER | Nice to have; System hat Default-Text pro Typ |
| 8 | Sollen Gäste sagen was sie mitbringen? | 🟡 SPÄTER | Nicht alle Events brauchen das |
| 9 | Welche Phase startet der Event? | 🟢 SYSTEM | Default: `Terminsuche` wenn kein fixes Datum, sonst `Einladung` |
| 10 | Emoji für den Anlass | 🟢 SYSTEM | Aus Event-Typ abgeleitet |
| 11 | Mitbringsel-Prompt-Text | 🟢 SYSTEM | Typ-spezifisch vorausgefüllt |
| 12 | Event-Token (Organizer-Link) | 🟢 SYSTEM | Automatisch generiert |

---

### 🍕 Pizza / Mittag-Events

| Frage | Priorität | Begründung |
|-------|-----------|------------|
| Wie viele Pizzas werden bestellt? | 🟡 SPÄTER | Hängt von Anmeldezahl ab |
| Wer bringt Getränke / Vorspeise / Dessert? | 🟡 SPÄTER | Koordination nach Anmeldung |
| Vegetarisch/Vegan berücksichtigen? | 🟡 SPÄTER | Erst nach Anmeldungen sinnvoll |
| Liefern oder selber backen? | 🟡 SPÄTER | Details |
| Pizza-Service: welcher Laden? | 🟡 SPÄTER | Irrelevant für Erstellung |
| Indoor oder Outdoor? | 🟢 SYSTEM | Default: Indoor (Pizza = meist drinnen) |
| Bezahlart (split/gratis)? | 🟡 SPÄTER | Kann später kommuniziert werden |

**System kann vorhersehen:**
- Mitbringsel-Prompt: *"Was bringst du mit? (z.B. Getränke, Dessert, Servietten)"*
- Empfohlene Kapazität-Warnschwelle: 12 Personen

---

### 🔥 BBQ / Grill-Events

| Frage | Priorität | Begründung |
|-------|-----------|------------|
| Garten/Park vorhanden? Wo genau? (Ort-Detail) | 🔴 MUSS | BBQ braucht präzisen Treffpunkt |
| Ist ein Grill vorhanden oder muss jemand mitbringen? | 🟡 SPÄTER | Aber früh kommunizieren! |
| Welche Fleisch-/Veggie-Optionen? | 🟡 SPÄTER | Nach Anmeldung klarer |
| Wer bringt Kohle/Anzünder mit? | 🟡 SPÄTER | Koordination |
| Soll Regen-Absage-Regel definiert werden? | 🟡 SPÄTER | Wetterabhängig |
| Parkplatzsituation? | 🟡 SPÄTER | Info für Teilnehmer |

**System kann vorhersehen:**
- Mitbringsel-Prompt: *"Was grillierst du? (z.B. Würste, Veggie-Burger, Salat)"*
- Outdoor = Default
- Kapazität: offener als Pizza (keine automatische Schwelle)
- Wenn Datum Sommer (Juni–August): Priorität-Hinweis "Frühzeitig einladen"

---

### 🥾 Wanderung

| Frage | Priorität | Begründung |
|-------|-----------|------------|
| Treffpunkt (exakt, z.B. Bushaltestelle) | 🔴 MUSS | Wandern ohne klaren Treffpunkt funktioniert nicht |
| Schwierigkeitsgrad (leicht/mittel/schwer) | 🔴 MUSS | Entscheidungsrelevant für Teilnehmer |
| Streckenlänge/Dauer (ca.) | 🔴 MUSS | Teilnehmer müssen einschätzen können |
| Mit ÖV erreichbar? (Linie/Haltestelle) | 🟡 SPÄTER | Wichtig für CH, aber nachträgbar |
| Ausrüstungs-Empfehlung | 🟡 SPÄTER | System hat Default |
| Einkehr geplant? (Restaurant am Ende) | 🟡 SPÄTER | Schön zu wissen aber nicht notwendig |
| Rückweg gleich oder andere Route? | 🟡 SPÄTER | Detail |
| Hunde erlaubt? | 🟡 SPÄTER | Für manche relevant |

**System kann vorhersehen:**
- Mitbringsel-Prompt: *"Was nimmst du mit? (z.B. Rucksack, Wanderschuhe, Lunchpaket)"*
- Standard-Ausrüstungs-Hinweis: "Festes Schuhwerk, Wasser, Sonnenschutz"
- Startphase: `Terminsuche` (Wanderungen sind oft auf Verfügbarkeit angewiesen)

---

### 🎲 Spielrunde

| Frage | Priorität | Begründung |
|-------|-----------|------------|
| Wo findet sie statt? (Wohnung? Vereinsraum?) | 🔴 MUSS | Lokation entscheidend |
| Welche Art Spiele? (Brett/Karten/Rollenspiel) | 🟡 SPÄTER | Hilft beim Mitbringsel, aber nicht kritisch |
| Maximale Spieler-Anzahl (Kapazität) | 🔴 MUSS | Spiele haben feste Spielerzahlen |
| Anfänger-freundlich oder erfahrene Spieler? | 🟡 SPÄTER | Interessant für Einladungstext |
| Welche Spiele soll jemand mitbringen? | 🟡 SPÄTER | Koordination nach Anmeldung |
| Essen/Trinken während Spielen? | 🟡 SPÄTER | Nice to have |
| Wie lange dauert der Abend ca.? | 🟡 SPÄTER | Für Planung der Teilnehmer |

**System kann vorhersehen:**
- Mitbringsel-Prompt: *"Bringst du ein Spiel mit? (Welches?)"*
- Kapazität: PFLICHT für Spielrunden (Spiele haben Limits → System sollte Hinweis geben)
- Startphase: direkt `Einladung` (Spielrunden sind meist fix geplant)

---

## Phasen-Flow: Wann was

```
PHASE 0 (Idee)
  └─ /create öffnen, Typ auswählen

PHASE 1 (Terminsuche) — wenn kein fixes Datum
  └─ 2–4 Terminvorschläge eingeben
  └─ Einladungslink teilen → Teilnehmer stimmen ab
  └─ Organizer wählt final den Termin

PHASE 2 (Einladung) — wenn fixes Datum
  └─ Event-Seite ist live
  └─ SPÄTER: Beschreibung, Kapazität, besondere Hinweise ergänzen

PHASE 3 (Anmeldung)
  └─ SPÄTER: Mitbringsel-Koordination freischalten
  └─ SPÄTER: "Wer bringt was mit"-Übersicht für Organizer

PHASE 4+ (Vorbereitung, Durchführung...)
  └─ Via Organizer-Link verwaltbar
```

---

## Geplante Create-Seite: Schritt-Struktur

### Schritt 1 — Event-Typ wählen (visuell, Kacheln)
```
🍕 Pizzamittag   🔥 Sommer-BBQ
🥾 Wanderung     🎲 Spielrunde
🎉 Anderer Anlass
```
→ Kein Zurück-Button nötig, direkt anklicken

### Schritt 2 — Datum-Strategie
```
○ Ich habe ein fixes Datum     → direkt zu Schritt 3
○ Ich suche noch einen Termin  → Terminvorschläge eingeben (2–4)
```

### Schritt 3 — Pflichtangaben (alle typen)
```
Titel des Anlasses:    [________________]
Datum & Uhrzeit:       [__/__/____] [__:__]
Ort / Treffpunkt:      [________________]
Deine E-Mail:          [________________]
```
+ Typ-spezifische MUSS-Felder (z.B. Schwierigkeitsgrad für Wandern, Kapazität für Spielrunde)

### Schritt 4 — Vorschau & Veröffentlichen
```
[Event-Karte Vorschau]
Dein Organizer-Link: quietschente.ch/admin/TOKEN  (kopieren / per Mail senden)
Einladungslink:      quietschente.ch/event/TOKEN   (teilen)
[ Event erstellen → ]
```

---

## System-Defaults pro Typ (Code-Referenz)

Bereits in `event.html` + `register.html` implementiert:
```javascript
const EVENT_TYPES = {
  pizza:   { emoji: '🍕', label: 'Pizzamittag', bringPrompt: 'Was bringst du mit?' },
  bbq:     { emoji: '🔥', label: 'Sommer-BBQ',  bringPrompt: 'Fürs Grillieren?' },
  wandern: { emoji: '🥾', label: 'Wanderung',   bringPrompt: 'Ausrüstung mitbringen?' },
  spiel:   { emoji: '🎲', label: 'Spielrunde',  bringPrompt: 'Bringst du Spiele?' },
  default: { emoji: '🎉', label: 'Anlass',       bringPrompt: 'Was bringst du mit?' }
};
```
→ Direkt wiederverwendbar auf `/create`

---

## Entschiedene Design-Entscheide

1. **Terminsuche-Flow**: Doodle-ähnlich. Organizer gibt 2–4 Terminvorschläge ein. Teilnehmer stimmen direkt auf der Event-Seite ab. Organizer wählt final den Termin (im Admin-Bereich).

2. **Organizer-Zugang**: Zwei-Token-System:
   - `person_token` — identifiziert den Organizer als Person
   - `event_token` — identifiziert den spezifischen Anlass
   - Beide müssen übereinstimmen → Zugang zu Admin-Funktionen
   - Nach Schritt 1 der Erstellung werden beide Tokens per E-Mail zugesendet
   - Kein Account, kein Passwort — konsistent mit dem Gesamt-Konzept

3. **Mitbringsel-Koordination**: Erst Freitext (wie heute). Organizer kann später im Admin-Bereich Kategorien/Vorschläge definieren, die als Autofill-Vorschläge beim Teilnehmer erscheinen. System bleibt offen, wird aber schrittweise strukturierter.

4. **Backend-Erweiterung**: Neue Action `action=anlass_erstellen` im Google Apps Script nötig (Anlässe als First-Class Objects mit eigenen Tokens und Metadaten).

---

## Implementierungs-Schritte (nach Plan-Freigabe)

1. `create.html` — Neue Seite mit 4-Schritt-Flow (HTML + CSS aus bestehendem Design-System)
2. Typ-Auswahl-Komponente (Kacheln mit Emoji, klickbar)
3. Datum-Strategie-Entscheidung (fix vs. Terminsuche)
4. Pflichtfelder-Formular mit Typ-spezifischen Feldern
5. Preview-Schritt mit generierter Event-Karte
6. Backend-Erweiterung: `action=anlass_erstellen` im Google Apps Script
7. Organizer-Token generieren + per E-Mail versenden
8. Tests (Vitest Unit + Playwright E2E für den Create-Flow)

---

## Verifikation

- Alle 5 Typ-Kacheln klickbar und führen korrekte Defaults ein
- Pflichtfelder werden validiert (Name, Datum, Ort, Email)
- Typ-spezifische MUSS-Felder erscheinen/verschwinden korrekt
- Preview zeigt dieselbe Karte wie spätere Event-Seite
- Organizer erhält E-Mail mit Admin-Link
- Einladungslink öffnet bestehende `event.html` korrekt
