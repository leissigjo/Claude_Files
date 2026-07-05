# Quietschente – Design Patterns & Component Catalogue

**Version:** 1.0 | **Datum:** 2026-06-28 | **Basis:** Event-Lifecycle v1.1, ADR-8
**Zielgruppe:** Design Review, Frontend-Entwicklung

---

## Designprinzipien (nicht verhandelbar)

| # | Prinzip | Konsequenz für UI |
|---|---------|------------------|
| DP1 | **Zero-Account** – Token ist Identität | Kein Login-Formular, kein Passwort-Feld irgendwo |
| DP2 | **Max. 3 sichtbare Formularfelder** | Progressive Disclosure: Felder nach Bedarf einblenden |
| DP3 | **1-Klick für sekundäre Aktionen** | Feedback, Opt-in, CoC: nie mehr als ein Klick |
| DP4 | **Typ-Context = vorausgefüllter Kontext** | Label, Emoji-Map und Pflichtfelder aus Event-Typ ableiten |
| DP5 | **Soziale Sichtbarkeit ohne Druck** | Teilnehmerliste, Helfer-Slots: sichtbar, nie mahnend |
| DP6 | **Private Accountability** | No-Show-Signal, Zuverlässigkeit: nur für den Nutzer selbst |

---

## Interaction Patterns

### P01 – Token-Gate
**"Dein Link ist dein Zugang."**

Jede persönliche Aktion (Bearbeiten, Absagen, Profil, Feedback) ist an einen URL-Token gebunden.

```
URL-Struktur:
  ?profile_token=<uuid>       → Profil, Event-Liste, Präferenzen
  ?action_token=<uuid>        → Einzelne Anmeldung bearbeiten / absagen
  ?view=admin&token=<uuid>    → Admin-DashboardI have here a project description plan (ich-habe-hier-einen-idempotent-reddy.md) , her the architecture document plan(datenarchitektur-solid-quietschente), a a very rudimentary plan for design patterns. design-patterns-quietschente.md

Regeln:
  - Token NIEMALS im HTML-Source einbetten
  - Token NIEMALS in Logs schreiben
  - Nur via HTTPS übertragen
  - "Meinen Link erneut senden" als einziger Recovery-Flow
```

**Betroffene Components:** C11 (TokenActionBar), C19 (TokenRecoveryForm), C16 (ProfileCard)

---

### P02 – Progressive Disclosure
**Zeig nur, was jetzt gebraucht wird.**

```
Stufe 1 (immer sichtbar):   Name, E-Mail, Antwort (Ja/Nein/Vielleicht)
Stufe 2 (wenn Ja):          Mitbringsel-Auswahl (C03)
Stufe 3 (wenn Outdoor):     Wetter-Plan-B-Bestätigung (C20)
Stufe 3 (wenn Helfer):      Slot-Auswahl (C05)
```

Formular-Felder erscheinen animiert von unten; verschwinden wenn nicht mehr relevant.

**Betroffene Components:** C01 (RegistrationForm), C04 (DatePollWidget)

---

### P03 – Social Proof Feed
**"Ach, Moritz ist auch dabei."**

Teilnehmerliste und Mitbringsel sind vor der Anmeldung sichtbar. Ziel: Hemmschwelle bei Inge senken.

```
Reihenfolge auf Einladungsseite:
  1. Event-Info (Datum, Ort, Typ)
  2. Teilnehmerliste (C06) – öffentlich, Name + Mitbringsel
  3. Helfer-Slots (C08) – offene Slots prominent
  4. Dann: Anmeldeformular (C01)
```

**Betroffene Components:** C06 (ParticipantList), C08 (HelperSlotGrid), C07 (CapacityBar)

---

### P04 – Duplikat-Signal
**Verhindern, dass 5x Wein auf dem Tisch stehen.**

Beim Öffnen des Mitbringsel-Pickers (C03) sind bereits gewählte Kategorien markiert.

```
Visuelles Vokabular:
  ✅ Grün / Normal:   noch verfügbar
  🟡 Gelb / Gedimmt:  bereits 1x gewählt
  🔴 Rot / Gesperrt:  bereits voll (nur bei Helfer-Slots mit Kapazitäts-Limit)
  Für Mitbringsel (Essen/Getränke): nie hart sperren, nur signalisieren
```

**Betroffene Components:** C03 (EmojiMitbringselPicker), C05 (HelperSlotPicker)

---

### P05 – 1-Klick-Aktion
**Keine Weiterleitungen für sekundäre Aktionen.**

```
Gilt für:
  - Post-Event Feedback (3 Klick-Buttons inline in E-Mail)
  - Opt-in "Beim nächsten Mal" (1 Button)
  - CoC-Bestätigung (1 Checkbox oder Button)
  - Micro-Feedback am Event-Tag (1 Emoji-Klick)

Nicht erlaubt:
  - Link zu separatem Formular für Feedback
  - Modal-Dialog mit weiterem Schritt
  - Weiterleitung auf neue Seite für Opt-in
```

**Betroffene Components:** C13 (OneClickFeedback), C14 (ReengagementCard), C23 (CoCConfirmation)

---

### P06 – Contextual Template
**URL-Parameter setzt den Kontext, nicht der Nutzer.**

```
Input:  URL ?type=bbq  →  bbq.quietschente.ch (nach v1.2)
Output: Formular-Labels werden getauscht
        Emoji-Map wird gewechselt
        Pflichtfelder werden aktiviert/deaktiviert
        Workflow-Reihenfolge ändert sich

Mapping (Beispiele):
  type=pizza  →  Label: "Was bringst du mit?"    / Emojis: 🍕🍝🍷
  type=bbq    →  Label: "Fürs Grillieren?"       / Emojis: 🥩🌽🥗  + Plan-B Pflicht
  type=wandern →  Label: "Ausrüstung OK?"         / Emojis: 🥾🎒⛰️ + Fitness-Feld
  type=spiel  →  Label: "Bringst du Spiele?"     / Emojis: 🎲🃏🕹️
```

**Betroffene Components:** C02 (EventTypeCard), C03 (EmojiMitbringselPicker), C01 (RegistrationForm)

---

### P07 – Deadline Visibility
**Fristen erzeugen Handlung.**

```
Varianten:
  - Abstimmungs-Deadline (Phase 1): "Abstimmung endet Fr, 4. Juli 20:00"
  - Anmeldeschluss (Phase 3):       "Anmeldeschluss in 3 Tagen"
  - Event-Countdown (Phase 4/5):    "Das Event ist morgen"

Verhalten:
  - Ab >7 Tage:   statisches Datum
  - 3–7 Tage:     "in X Tagen" + gelbes Badge (C24)
  - <3 Tage:      "in X Stunden/Tagen" + rotes Badge (C24)
  - Abgelaufen:   Badge verschwindet, Aktion deaktiviert
```

**Betroffene Components:** C24 (CountdownBadge), C04 (DatePollWidget)

---

### P08 – Open Slot Social Pressure
**Öffentliche Leerstellen erzeugen sozialen Druck ohne direkte Aufforderung.**

```
Auf der Einladungsseite (Phase 2/3):
  "Noch gesucht: 🔥 Grillmaster (1 Platz frei)"
  "Noch gesucht: 🛒 Einkaufen (1 Platz frei)"
  "[Dein Name hier] – Slot übernehmen"

Regeln:
  - Nie mahnend formulieren ("Niemand hat sich gemeldet!")
  - Immer konstruktiv ("Noch frei: ...")
  - Bei 0 offenen Slots: Sektion ausblenden (nicht "Alles besetzt")
```

**Betroffene Components:** C08 (HelperSlotGrid), C05 (HelperSlotPicker)

---

### P09 – Phase Advancement (Admin only)
**Klarer nächster Schritt nach jeder abgeschlossenen Phase.**

```
Phasen-Übergänge → NextStepPrompt (C15):
  Phase 0 → 1:  "Termin noch offen? → Abstimmung starten"
  Phase 1 → 2:  "Termin fixiert → Einladungslink teilen"
  Phase 2 → 3:  "Anmeldung jetzt öffnen"
  Phase 3 → 4:  "Anmeldeschluss → Übersicht prüfen"
  Phase 5 → 6:  "Event vorbei → Feedback einholen"
  Phase 6 → 7:  "Nächsten Anlass planen?"

Verhalten:
  - Prompt erscheint prominent oben im Admin-Dashboard
  - Verschwindet nach Klick oder automatischem Phasenwechsel
  - Nie blockierend (kein Modal, kein Pflicht-Klick)
```

**Betroffene Components:** C15 (NextStepPrompt), C09 (PhaseStepper)

---

### P10 – Re-engagement Hook
**Energie nutzen, solange sie da ist.**

```
Trigger-Momente:
  Nach Event-Abschluss (sofort):  "Nächstes Event jetzt planen?"  →  A35
  Post-Event-E-Mail:              "Beim nächsten Mal dabei?"      →  A33
  Profil-Seite:                   "Deine Gruppe hat lange nichts gemacht" → A35

Ton: einladend, nie drängend. Optional, immer dismissbar.
```

**Betroffene Components:** C14 (ReengagementCard), C15 (NextStepPrompt)

---

### P11 – Adaptive Form
**Felder reagieren auf Antworten, nicht nur auf Event-Typ.**

```
Logik-Baum:
  Antwort = "Ja"          →  Mitbringsel-Feld einblenden (C03)
  Antwort = "Nein"        →  Nur Kommentar-Feld ("Warum nicht?" optional)
  Antwort = "Vielleicht"  →  Mitbringsel optional, Hinweis: "Meld dich ab wenn du weißt"
  Event-Typ = Outdoor     →  Wetter-Plan-B-Bestätigung einblenden (C20)
  Helfer-Slots offen      →  Helfer-Auswahl nach Pflichtfeldern einblenden (C05)
```

**Betroffene Components:** C01 (RegistrationForm)

---

### P12 – Private Accountability
**Sichtbar für sich selbst, nie für andere.**

```
Elemente (NUR auf der eigenen Profil-Seite):
  - Erscheinungsquote: "Du warst bei 4 von 5 Events dabei"
  - No-Show-Hinweis (wenn >1 No-Show): "Du warst 2x angemeldet aber nicht da"
  - Ton: sachlich, nicht beschämend, handlungsorientiert

Was NIE öffentlich sichtbar ist:
  - Zuverlässigkeits-Score
  - Absage-Gründe
  - No-Show-Count
```

**Betroffene Components:** C18 (PrivateReliabilitySignal)

---

## Component Catalogue

### Gruppe A – Formulare & Input

---

#### C01 – RegistrationForm
**Adaptives Anmeldeformular**

```
Zustände:
  neu        →  leeres Formular, Core-Felder
  bearbeiten →  vorausgefüllt mit bestehenden Daten, via action_token
  helfer     →  HelperSlotPicker (C05) nach Core-Feldern eingefügt

Core-Felder (immer):
  [Name*]  [E-Mail*]  [Antwort: Ja / Nein / Vielleicht]

Conditional:
  Antwort = Ja    →  Mitbringsel (C03) + Kommentar
  Outdoor-Event   →  Wetter-Plan-B bestätigen (C20 inline)
  Slots offen     →  Helfer-Option (C05) einblenden

Validierung:
  E-Mail:   Format-Check client-seitig (Email Value Object Logik)
  Name:     Min. 2 Zeichen
  Absenden: Button disabled bis Mindestfelder valide
  Spam:     Honeypot-Feld (unsichtbar), Timing-Check (>2.5s)

Verknüpfte Patterns: P02, P06, P11
```

---

#### C02 – EventTypeCard
**Auswahl-Karte für Event-Typ**

```
Aufbau einer Karte:
  [Emoji gross]
  [Typ-Label]
  [Kurzbeschreibung 1 Zeile]
  [Vorlaufzeit-Hinweis: "Empfohlen: 2–3 Wochen vorher"]

Zustände:
  default    →  neutral
  hover      →  leichtes Highlight
  selected   →  farbiger Border, Checkmark
  disabled   →  gedimmt (für nicht verfügbare Typen in v1.x)

Verfügbare Typen v1.x: Pizza, BBQ
Verfügbare Typen v2.0: + Wanderung, Spielrunde, Restaurant

Verknüpfte Patterns: P06
```

---

#### C03 – EmojiMitbringselPicker
**Emoji-Grid für Mitbringsel-Auswahl**

```
Aufbau:
  Scrollbares Grid, 4 Spalten
  Jede Zelle: [Emoji] + [Label]
  Freitexteingabe unten für "Anderes"

Emoji-Maps (typ-spezifisch):
  pizza/brunch:  🍕 🍝 🍷 🥗 🥤 🍰 🧀 🥖
  bbq:           🥩 🌽 🥗 🧅 🥤 🍺 🍰 🧂
  wandern:       🎒 🥾 ⛏️ 🧴 🥪 💧 🗺️ 🩹
  spiel:         🎲 🃏 🕹️ ♟️ 🎯 📝 🍿 🥤

Duplikat-Signal (Pattern P04):
  Bereits gewählt (von anderem TN): Zelle gedimmt + "1x gewählt"
  Eigene Wahl: Zelle hervorgehoben + Checkmark
  Kapazitätslimit (Helfer-Slots): Zelle gesperrt

Verknüpfte Patterns: P04, P06
```

---

#### C04 – DatePollWidget
**Terminabstimmungs-Widget**

```
Aufbau:
  Header:  [Abstimmungs-Deadline als CountdownBadge (C24)]
  Body:    Tabelle: Optionen (Zeilen) × Personen (Spalten)
           Zellen: ✅ kann / ❌ kann nicht / ❓ vielleicht
  Footer:  [Eigene Stimme abgeben / ändern]
           Optional: [+ Mitbringsel-Intention eingeben] (Pattern P08 kombiniert)

Zustände:
  offen        →  interaktiv, Deadline sichtbar
  abgeschlossen →  read-only, Gewinner-Datum hervorgehoben
  anonym       →  Personen-Spalten zeigen nur Anzahl, kein Name

Verknüpfte Patterns: P02, P07
```

---

#### C05 – HelperSlotPicker
**Helfer-Slot-Auswahl**

```
Aufbau:
  Liste der verfügbaren Slots:
    [Slot-Label] [Kapazität: X/Y] [Auswählen-Button]
  Gewählter Slot: hervorgehoben, änderbar

Verhalten:
  - Voller Slot: deaktiviert (aber sichtbar mit "Belegt")
  - Mehrfachauswahl: nur wenn explizit erlaubt
  - Optional kombiniert mit RegistrationForm (C01) als erweiterter Schritt

Verknüpfte Patterns: P04, P08
```

---

### Gruppe B – Listen & Status

---

#### C06 – ParticipantList
**Teilnehmerliste**

```
Öffentliche Variante (Einladungsseite):
  [Name] – [Mitbringsel-Emoji + Label]
  Sortiert: Neueste oben
  Anonym-Option: [Anonym] – [Mitbringsel]

Admin-Variante (Dashboard):
  [Name] | [E-Mail] | [Antwort] | [Mitbringsel] | [Status] | [Aktionen]
  Aktionen: Bestätigen / Ablehnen / E-Mail senden

Kompakt-Variante:
  Nur Zahl: "12 Personen angemeldet" (für Profil-Cards)

CapacityBar (C07) immer oben angehängt.

Verknüpfte Patterns: P03, P05
```

---

#### C07 – CapacityBar
**Kapazitätsbalken**

```
Aufbau:
  [Gefüllter Balken] [X / Y Plätze]

Farblogik:
  <60% voll:   grün
  60–85% voll: gelb + "Noch X Plätze frei"
  >85% voll:   orange + "Fast ausgebucht"
  100% voll:   rot + "Ausgebucht – Warteliste?" (wenn Warteliste aktiv)

Verknüpfte Patterns: P03
```

---

#### C08 – HelperSlotGrid
**Helfer-Slots Übersicht (Einladungsseite)**

```
Aufbau:
  Header: "Wir suchen noch Helfer:"
  Pro Slot:
    [Slot-Emoji + Label]  [Belegt: Name / Offen: "Noch frei"]  [Übernehmen]

Verhalten:
  - Alle Slots belegt: Sektion vollständig ausblenden
  - "Übernehmen"-Button → direkt zu C05 im Anmeldeformular scrollen

Verknüpfte Patterns: P03, P08
```

---

#### C09 – PhaseStepper
**Phasen-Indikator (Admin only)**

```
Phasen (0–7):
  0 Idee → 1 Terminsuche → 2 Einladung → 3 Anmeldung →
  4 Vorbereitung → 5 Durchführung → 6 Nachbereitung → 7 Wiederengagement

Aufbau (horizontal, Desktop):
  ● ── ● ── ◉ ── ○ ── ○ ── ○ ── ○ ── ○
  done  done  aktiv  nächste ...

Mobile: Kompakt als "Phase 3 von 7: Anmeldung"

Nicht sichtbar für Teilnehmer.

Verknüpfte Patterns: P09
```

---

#### C10 – EventCard
**Event-Zusammenfassungskarte**

```
Aufbau:
  [Typ-Emoji gross]  [Titel]
                     [Datum + Uhrzeit]
                     [Ort]
                     [Phase-Badge]  [TN-Anzahl]

Varianten:
  upcoming:  helles Design, Countdown (C24) unten
  past:      gedimmt, "Memory ansehen"-Link
  draft:     gestrichelte Border, "Noch nicht geteilt"

Verwendung: Profil-Seite, Admin-Dashboard
```

---

### Gruppe C – Aktionen & Navigation

---

#### C11 – TokenActionBar
**Persönliche Aktions-Leiste**

```
Erscheint:
  - Nach erfolgreicher Anmeldung (In-App, unten)
  - In der Bestätigungs-E-Mail (als Buttons)

Buttons:
  [✏️ Anmeldung bearbeiten]  [❌ Absagen]

Regeln:
  - Token nie sichtbar im Button-Label
  - Absage-Button immer sichtbar und gleichwertig prominent
    (Absagen ermöglichen = No-Shows reduzieren)
  - Nach Absage: Button-Zustand → "Abgesagt" + "Rückmeldung ändern"-Option

Verknüpfte Patterns: P01
```

---

#### C12 – SharePanel
**Event-Link teilen**

```
Aufbau:
  [Event-URL mit Copy-Button]
  [OG-Preview-Simulation: Titel, Datum, Kurzbeschr., Typ-Emoji]
  [QR-Code zum Herunterladen]
  Optional: [Helfer-Link separat kopieren]

OG-Preview zeigt wie der Link in WhatsApp/Signal erscheint.
Verwendung: Admin-Dashboard, Phase 2.

Verknüpfte Patterns: –
```

---

#### C13 – OneClickFeedback
**1-Klick-Feedback-Widget**

```
Fragen (max. 3, sequenziell):
  F1: "Warst du dabei?"
      [✅ Ja]  [❌ Nein, hatte Grund]  [😅 Nein, vergessen]

  F2 (nur wenn F1 = Ja): "Wie war's?"
      [😄 Sehr gut]  [🙂 Gut]  [😐 Okay]  [😕 Weniger gut]

  F3: "Beim nächsten Mal?"
      [🔔 Ja, benachrichtige mich]  [👤 Ich melde mich selbst]

Verhalten:
  - Antwort = sofortiger Klick, kein "Absenden"-Button
  - Nächste Frage erscheint direkt darunter (kein Seitenwechsel)
  - In E-Mail: mailto-Links mit Pre-filled Subject für Klick-Tracking

Varianten: E-Mail-inline, Auf Seite, Minimal (nur F1, am Event-Tag)

Verknüpfte Patterns: P05
```

---

#### C14 – ReengagementCard
**"Beim nächsten Mal dabei?"-Karte**

```
Aufbau:
  [Event-Memory-Emoji oder Foto-Thumbnail]
  "Hat Spaß gemacht! Sollen wir das wiederholen?"
  [🔔 Ja, informiere mich]  [Nicht jetzt]

Erscheint:
  - Post-Event-E-Mail (unterhalb Feedback)
  - Profil-Seite nach Event

"Nicht jetzt" dismisst dauerhaft für dieses Event, fragt beim nächsten neu.

Verknüpfte Patterns: P10
```

---

#### C15 – NextStepPrompt
**Kontextueller Nächster-Schritt-Hinweis (Admin only)**

```
Aufbau:
  [Info-Icon]  [Text: was als nächstes zu tun ist]  [CTA-Button]  [Dismiss ✕]

Beispiele:
  "Termin ist fixiert. Jetzt Einladung teilen →"  [Link kopieren]
  "Anmeldeschluss erreicht. Übersicht prüfen →"   [Teilnehmerliste]
  "Event vorbei. Wie war's? →"                    [Feedback senden]

Regeln:
  - Nie blockierend (kein Modal, kein Pflicht)
  - Dismissbar (bleibt weg bis nächste Phase)
  - Nur im Admin-Dashboard sichtbar

Verknüpfte Patterns: P09, P10
```

---

### Gruppe D – Profil & Identität

---

#### C16 – ProfileCard
**Profil-Karte (token-basiert)**

```
Aufbau:
  [Avatar-Platzhalter / Initiale]  [Anzeigename]
                                   [Mitglied seit: Datum des ersten Events]
  [Badges (C17)]
  [X vergangene Events]  [Y bevorstehende Events]
  [Präferenzen bearbeiten →]

Zugang: nur via ?profile_token= URL (kein Login)
Öffentliche Mini-Variante: auf Event-Seite neben Organizer-Name (nur Name + Badges)

Verknüpfte Patterns: P01
```

---

#### C17 – BadgeDisplay
**Abzeichen-Anzeige**

```
Verfügbare Badges:
  🎉 Erstanmeldung          – Erstes Event angemeldet
  🎪 Erstorganisator        – Erstes Event organisiert
  ⭐ Zuverlässige Gastgeberin – X Events erfolgreich organisiert
  🔄 Stammgast              – Bei Y Events dabei (Streak)
  🤝 Helfer                 – Bei Z Events als Helfer eingetragen
  🏆 Rotierende Organisatorin – War nicht in Folge zweimal Organisatorin

Aufbau: Emoji-Icon + Label (Tooltip mit Erklärung)
Ton: soziale Anerkennung, nie spielerische Punkte

Verknüpfte Patterns: –
```

---

#### C18 – PrivateReliabilitySignal
**Eigenes Verlässlichkeits-Signal (nur für den Nutzer selbst)**

```
Aufbau:
  "Du warst bei [4 von 5] Events dabei, zu denen du dich angemeldet hast."
  Optional (bei No-Show): "Du warst 1x angemeldet aber nicht erschienen."

Ton: sachlich, nicht beschämend
  ✅ "4 von 5 – stark!"    bei Quote ≥ 80%
  🟡 "3 von 5"             bei Quote 60–79%
  – kein negativer Ton     bei Quote <60% (nur Zahl, kein Emoji)

NIEMALS öffentlich sichtbar.
Verwendung: Nur Profil-Seite (C16).

Verknüpfte Patterns: P12
```

---

#### C19 – TokenRecoveryForm
**"Meinen Link erneut senden"**

```
Aufbau:
  [E-Mail-Eingabe]
  [Button: "Link zusenden"]
  → E-Mail mit Profil-Token-Link wird versendet
  Bestätigung: "Wenn diese Adresse registriert ist, bekommst du gleich eine E-Mail."

Regeln:
  - Kein Feedback ob E-Mail existiert (DSGVO / Enumeration-Schutz)
  - Rate-Limit: max. 3 Anfragen pro Stunde pro E-Mail
  - Ersetzt vollständig jeden Login-Flow

Verknüpfte Patterns: P01
```

---

### Gruppe E – Kontext-Blöcke

---

#### C20 – WeatherPlanBBlock
**Wetter-Plan-B (Outdoor-Events)**

```
Erscheint bei: type=bbq, type=wandern (conditional, Pattern P06)

Zustände:
  plan_defined:  "[☁️ Plan B: Pavillon vorhanden, falls Regen]"
  weather_flag:  "[⚠️ Wetter unsicher – Plan B gilt: ...]" (rot/orange, Admin gesetzt)
  no_plan:       Block ausgeblendet

In der Anmeldung (C01): kurze Bestätigung "Plan B zur Kenntnis genommen ✓"
In der Erinnerungs-E-Mail: Plan-B-Info mit Wetterhinweis wiederholen.

Verknüpfte Patterns: P06
```

---

#### C21 – MemorySummary
**Post-Event-Zusammenfassung**

```
Auto-generierter Inhalt:
  [Typ-Emoji]  "[Event-Titel]"
  Datum • Ort • [X] Personen dabei
  Mitgebracht: [Emoji-Liste der Mitbringsel]
  Helfer: [Namen der Helfer, namentlich hervorgehoben]
  [Foto-Galerie-Link wenn Fotos vorhanden]

Ton: warm, anerkennend, kein Bericht-Stil

Verwendung:
  - Post-Event-E-Mail (als E-Mail-Block)
  - Profil-Seite: vergangene Events als kompakte Memory-Cards

Verknüpfte Patterns: P10
```

---

#### C22 – OnboardingBanner
**Kontext-Erklärung für neue Nutzer (Felix)**

```
Trigger: Erster Besuch einer Event-Seite (keine bekannte Session)

Aufbau (kompakt, dismissbar):
  "Was ist Quietschente?  Einfache Anlass-Koordination – kein Account, kein Passwort.
   Dein persönlicher Link ist dein Zugang. [Verstanden ✕]"

Variante in der Bestätigungs-E-Mail (ausführlicher):
  - "Du bist angemeldet für ..."
  - "Dein Link: [LINK]" (hervorgehoben)
  - "Was ist Quietschente? ..."
  - "CoC-Zusammenfassung" (2 Sätze)

Regeln:
  - Nur einmalig anzeigen (Session-Storage-Flag)
  - Kein Modal, nur Banner oben

Verknüpfte Patterns: P01
```

---

#### C23 – CoCConfirmation
**Code of Conduct Bestätigung**

```
Aufbau (inline in C01, zwischen Formular und Submit-Button):
  [ ] "Ich melde mich ab, wenn ich doch nicht kommen kann."
  ℹ️ [Verhaltensregeln lesen +] (expandierbar, nicht standardmäßig offen)

Verhalten:
  - Checkbox ist Pflicht für Absenden
  - Kein Modal, kein separater Schritt
  - Nur bei Erstanmeldung; bei Bearbeitung (C01 Zustand "bearbeiten") weglassen

Verknüpfte Patterns: P05
```

---

#### C24 – CountdownBadge
**Deadline / Countdown-Badge**

```
Typen:
  abstimmung:   "Abstimmung bis Fr, 4. Juli 20:00"
  anmeldung:    "Anmeldeschluss in 3 Tagen"
  event:        "Event morgen"  /  "Event in 2 Stunden"

Farb-Schema:
  >7 Tage:    grau (neutral)
  3–7 Tage:   gelb (aufmerksam)
  <3 Tage:    rot (dringend)
  Abgelaufen: ausgeblendet

Format: Kleines Tag-Element, verwendbar inline in Text oder als eigenständiges Badge.

Verknüpfte Patterns: P07
```

---

#### C25 – ReminderBanner
**Kontextueller Erinnerungs-Hinweis auf Event-Seite**

```
Trigger: Nutzer ist angemeldet (via action_token oder profile_token bekannt)
         UND Event ist in ≤3 Tagen

Inhalt:
  "Das Event ist [morgen / in 2 Tagen]. Du bringst: [Mitbringsel-Emoji + Label]"
  [✏️ Ändern] [✅ Alles klar]

Verhalten:
  - "Alles klar" blendet Banner für diese Session aus
  - "Ändern" → springt direkt zu C01 (Bearbeiten-Zustand)
  - Nicht für Personen mit Antwort "Nein" anzeigen

Verknüpfte Patterns: P07, P01
```

---

## Version-Priorisierung

| Priorität | Components | Actions | Version |
|-----------|-----------|---------|---------|
| **Must-have** | C01, C06, C07, C11, C22, C23 | A16–A22 (Kern-Anmeldung) | v1.0 |
| **High** | C02, C03, C08, C12, C15, C20, C24 | A01–A05, A11–A15 (Typ + Teilen) | v1.2 |
| **Medium** | C04, C05, C09, C10, C13, C14, C16, C17, C18, C25 | A06–A10 (Terminsuche), A23–A29 (Vorbereitung/Event) | v2.0 |
| **Nice-to-have** | C19, C21 + Gamification vollständig | A30–A39 (Post-Event, Profil, Re-engagement) | v2.0+ |

---

## Offene Designfragen (für Review)

| # | Frage | Betroffene Components |
|---|-------|-----------------------|
| DQ1 | Wie sieht der Zustand aus, wenn kein action_token in der URL ist – aber Nutzer bearbeiten will? (TokenRecoveryForm einblenden oder hint?) | C11, C19 |
| DQ2 | Duplikat-Signal bei Mitbringsel: hard block oder nur Signal? (Hypothese: nie hart sperren bei Essen) | C03 |
| DQ3 | Anonyme Abstimmung (Phase 1): sehen alle Namen oder nur Zahl? Wählbar pro Event? | C04 |
| DQ4 | Sehen Helfer einander auf der Einladungsseite? (Koordinations-Argument: ja) | C08 |
| DQ5 | OnboardingBanner: Banner oder eingebetteter Erklärungsblock in der Formular-Sidebar? | C22 |
| DQ6 | Profil-Seite: separate URL (?view=profile) oder Modal über Event-Seite? | C16 |
