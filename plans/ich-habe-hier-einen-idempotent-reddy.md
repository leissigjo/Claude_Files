# Quietschente – Event-Lebenszyklus: Design Thinking Übersicht

**Version:** v1.1-konzept | **Datum:** 2026-06-23 | **Basis:** Pizza-Abend als primärer Use Case

---

## Context

Der bestehende Prototyp deckt bereits die **mittlere Phase** des Event-Lebenszyklus ab (Anmeldung + Mitbringsel-Koordination für Pizzamittag). Die nächste Ausbaustufe soll:
1. Den **gesamten Lebenszyklus** abdecken – von der Idee bis zur Nachbereitung
2. Neue Rollen ermöglichen: **Teilnehmer können selbst zu Organisatorinnen werden** (E-Mail + Token, kein Account-Zwang)
3. **Generalisierbar** sein für andere Anlasstypen: Spielrunde, Wanderung, Brunch, BBQ

Primärer Use Case bleibt **Pizza-Abend** (v1.x). Erweiterung auf andere Typen erfolgt ab v1.2.

---

## Versionierungsplan

```
v1.0 — Pizza-Kern (aktueller Stand)
       Pizzamittag-only, GAS-Backend, Phasen 3–4 implementiert

v1.1 — Generalisierter Lifecycle (dieses Dokument)
       Kein neuer Code — Lifecycle & Hypothesen dokumentiert und typ-neutral abstrahiert

v1.2 — Konfigurierbarer Event-Typ (technisch)
       event_type-Parameter in URL/Anlass-Tabelle, Feldlabels dynamisch,
       Emoji-Map erweitert, "Was bringst du mit?" → kontextabhängiger Label

v2.0 — Multi-Event-Typ mit typ-spezifischen Feldern
       Wanderung: +Treffpunkt, +Route-Link, +Wetterplan-B
       Spielrunde: +Spieltyp, +Spielerzahl-Min/Max
       BBQ: +Grill-Slot, +Brunch-Kategorie
       Schema-Erweiterung im Backend, Admin-Filter nach Typ

v2.1 — Wetterintegration (Wanderung-Feature)
       Optionaler Plan-B-Block, Admin kann "Event gefährdet" flaggen

v3.0 — Template Engine
       Admin definiert Event-Typen selbst, vollständig datengetrieben
```

**Grenzziehung:** Major-Sprung = Breaking Schema Change. Minor = additiv ohne Backend-Break.

---

## Event-Typ-Clustering & URL-basierte Vorlagen

### Idee

Die Event-Typ-Matrix (siehe unten) definiert natürliche Cluster. Jeder Cluster soll als **URL-Einstiegspunkt** fungieren, der eine vorkonfigurierte Vorlage lädt – statt dass die Organisatorin alle Optionen manuell einstellt.

```
quietschente.ch/createevent/bbq       → BBQ-Vorlage: Mitbringsel-Fokus, Wetter-Plan-B, kein Datum fix
quietschente.ch/createevent/resto     → Restaurant-Vorlage: Terminsuche + Reservationsfeld
quietschente.ch/createevent/pizza     → Pizza-Vorlage: Datum bekannt, Mitbringsel-Koordination
quietschente.ch/createevent/spiel     → Spielrunde-Vorlage: Programm-Feld, kein Essen-Fokus
quietschente.ch/createevent/wandern   → Wanderungs-Vorlage: Route, Treffpunkt, Fitness, Wetter
```

### Clustering-Logik (aus der Matrix abgeleitet)

Nicht jeder Typ braucht eine eigene Vorlage – Clustering reduziert auf wesentliche Varianten:

| Cluster | Event-Typen | Gemeinsame Merkmale | URL-Slug |
|---|---|---|---|
| **Bring-etwas** | Pizza, BBQ, Brunch | Mitbringsel-Koordination zentral | `bbq`, `pizza`, `brunch` |
| **Geh-irgendwo** | Restaurant, Kulturanlass | Ort + Reservation zentral, kein Mitbringsel | `resto`, `kultur` |
| **Programm** | Spielrunde | Was gespielt wird, Spielerzahl | `spiel` |
| **Outdoor** | Wanderung, Veloausfahrt | Route, Fitness, Wetter, Treffpunkt | `wandern`, `velo` |

### Technische Machbarkeit (zwei Varianten)

**Variante A – Query-Parameter (GAS-nativ, v1.2 umsetzbar)**
GAS unterstützt keine Pfad-Segmente. Realisierung via URL-Parameter:
```
https://[GAS-URL]?type=bbq       → lädt BBQ-Vorlage
https://[GAS-URL]?type=resto     → lädt Restaurant-Vorlage
```
Vorteil: Kein zusätzliches Hosting. Nachteil: Lange GAS-URL, nicht merkfähig.
Kurzlink-Lösung: `quietschente.ch/bbq` als Redirect → GAS-URL mit `?type=bbq` (via einfaches HTML-Redirect auf eigenem Hosting).

**Variante B – Subdomain-Routing mit Tippfehler-Toleranz (v2.x, erfordert Cloudflare Workers)**
```
bbq.quietschente.ch      → BBQ-Vorlage
bqq.quietschente.ch      → fuzzy match → BBQ-Vorlage  (Levenshtein-Distanz ≤ 1)
resto.quietschente.ch    → Restaurant-Vorlage
restau.quietschente.ch   → fuzzy match → Restaurant-Vorlage
```

Technische Umsetzung:
1. Wildcard-DNS-Eintrag `*.quietschente.ch` → Cloudflare Worker
2. Worker extrahiert Subdomain, vergleicht per Levenshtein-Distanz gegen bekannte Slugs
3. Redirect auf GAS-URL mit `?type={erkannter-typ}`
4. Fallback bei kein Match: `/createevent` ohne Vorlage

Bekannte Slug-Alias-Map (Tippfehler + Synonyme):
```javascript
const slugAliases = {
  bbq: ["bqq", "grill", "grillieren", "barbecue"],
  pizza: ["piza", "pizze"],
  resto: ["restaurant", "restau", "auswärts", "essen"],
  spiel: ["spielen", "spielabend", "game"],
  wandern: ["wanderung", "hike", "hiking", "wander"],
};
```

### Was eine Vorlage vorkonfiguriert

Pro Cluster wird beim Laden der Seite voreingestellt:

| Parameter | BBQ | Resto | Pizza | Spiel | Wandern |
|---|---|---|---|---|---|
| Workflow | Datum erst wählen | Terminsuche zuerst | Datum bekannt → direkt Anmeldung | Datum wählen | Terminsuche + Fitness |
| Mitbringsel-Label | "Fürs Grillieren?" | – (ausgeblendet) | "Was bringst du mit?" | "Bringst du Spiele?" | "Ausrüstung OK?" |
| Pflichtfelder extra | Wetter-Plan-B | Restaurant + Reservation | – | Spieltyp (optional) | Route-Link + Treffpunkt |
| Emoji-Map | Grill, Fleisch, Salate | – | Pizza, Pasta, Wein | Würfel, Karten | Berge, Stiefel |
| Erinnerungs-E-Mail | +Wetterhinweis | +Reservationsinfo | Standard | Standard | +Wetterhinweis +Treffpunkt |
| Vorlaufzeit-Hinweis | 2–3 Wochen | 1–2 Wochen | 1–2 Wochen | 1 Woche | 3–4 Wochen |

### Hypothesen zu URL-Vorlagen

- Organisatorinnen, die mit einem typ-spezifischen URL starten, schliessen die Event-Erstellung öfter ab als solche, die mit einem leeren Formular beginnen, weil der Kontext bereits gesetzt ist. → **Test:** Completion-Rate `/createevent/bbq` vs. `/createevent` leer.
- Typ-spezifische URLs werden leichter per WhatsApp oder Signal weitergeleitet, weil der Empfänger sofort weiss, worum es geht ("bbq.quietschente.ch"). → **Test:** Klickrate auf geteilte Links mit vs. ohne sprechenden Slug.
- Tippfehler-Toleranz bei Subdomains ist kein relevanter Nutzengewinn, weil Links in der Praxis kopiert werden, nicht abgetippt. → **Test:** Anteil manuell eingetippter Subdomains vs. kopiierter Links in Analytics.
- Eine Vorlage, die den Workflow vorgibt ("Erst Terminsuche, dann Anmeldung" vs. "Direkt Anmeldung"), reduziert Rückfragen an die Organisatorin. → **Test:** Anzahl Rückfragen pro Event-Typ mit/ohne konfiguriertem Workflow.

---

## Technische Ausgangslage: Was ist bereits generisch?

Aus der Code-Analyse (`Code_pizza_1.gs`, `index.html`, `admin_pizza.html`):

| Komponente | Status | Details |
|---|---|---|
| Datenbankschema (Einträge) | ✅ Vollständig generisch | Keine pizza-spezifischen Spalten |
| Kern-Logik (Register/Edit/Delete) | ✅ Vollständig generisch | Funktioniert für jeden Anlasstyp |
| Spam-Filter | ✅ Vollständig generisch | Ignoriert Mitbringsel-Feld bewusst |
| Admin-Moderation | ✅ Vollständig generisch | – |
| "Mitbringsel"-Feldlabel | ✅ Wiederverwendbar | "Was bringst du mit?" – typ-neutral |
| Titel/Header "Pizzamittag" | ❌ Hardcoded | 10+ Stellen in HTML-Dateien |
| Emoji-Map | ❌ Hardcoded | Nur Essen/Getränke (index.html:313–323) |
| Startzeit "12:00" | ❌ Hardcoded | 5+ Stellen in Code und UI |
| E-Mail-Template | ❌ Teilweise hardcoded | Betreff + Body erwähnen Pizza |
| Event-Typ-Feld | ❌ Fehlt | Kein Kategorie-Feld im Datenmodell |

**Fazit:** ~70% generisch, ~30% Pizza-spezifisch. Aufwand für v1.2 ist überschaubar.

---

## Event-Typ-Matrix

### Analysierte Typen
Die primären Typen unterscheiden sich nach **Ort-Typ**, **Kern-Koordinationsaufgabe** und den externen Einflussfaktoren:

| # | Event-Typ | Kern-Koordinationsaufgabe | Bemerkung |
|---|---|---|---|
| 1 | **Pizza-Abend (Heimkochen)** | Mitbringsel-Koordination (Essen/Getränke) | Primärer Use Case; Ort fix (privat) |
| 2 | **Auswärtsessen** | Reservation + Ort + ggf. Bestellung im Voraus | Externer Ort, externe Kapazitätslimite |
| 3 | **BBQ** | Grill-Slots + Mitbringsel + Wetter-Plan-B | Outdoor, wetterabhängig |
| 4 | **Spielrunde** | Programm: Welche Spiele, Spielerzahl, Niveau | Kein Essen-Fokus; Spieltisch-Logistik |
| 5 | **Wanderung** | Ziel + Route + körperliche Anforderung + Wetter | Stärkstes Risikoprofil; Sicherheit relevant |
| 6 | **Kulturanlass** (Kino, Konzert) | Ticketkauf + fixer Termin extern | Geringster Koordinationsspielraum |

### Bestimmungsfaktoren: Was beeinflusst, was gefragt werden muss?

Diese Faktoren bestimmen, welche Felder und Fragen bei der Eventerfassung notwendig sind:

| Faktor | Ausprägungen | Ausgelöste Fragen / Felder |
|---|---|---|
| **Ort-Typ** | Privat indoor / Öffentlich indoor (Restaurant) / Outdoor | Bei öffentlich: Reservation? Ort-Details. Bei outdoor: Treffpunkt, Anreise-Optionen |
| **Reservierungsbedarf** | Ja / Nein | Ja → Platzzahl zwingend, Anmeldefrist nötig, Organizer-Kontakt prominent |
| **Wetterabhängigkeit** | Keine / Mittel (BBQ) / Hoch (Wanderung) | Mittel/Hoch → Plan-B-Feld pflichtartig, Wetterhinweis in Erinnerungs-E-Mail |
| **Mitbringsel-Konzept** | Essen/Getränke / Equipment / Spiele / Keins | Beeinflusst Label, Emoji-Map, Koordinationslogik (Duplikat-Warnung vs. Spieltyp-Filter) |
| **Körperliche Anforderung** | Keine / Niedrig / Mittel / Hoch | Mittel/Hoch → Anmeldefeld "Fitness-Hinweis", Organizer sieht Übersicht der Angaben |
| **Kapazität** | Flexibel / Begrenzt (Raum) / Sehr begrenzt (Tisch/Grill) | Begrenzt → Warteliste, Anmeldeschluss, Kapazitätsbalken prominent |
| **Terminunsicherheit** | Fix / Wetter-abhängig / Gruppen-abhängig (Mindest-TN) | Wetter → Kurzfrist-Absage-Mechanismus; Mindest-TN → Anmeldezähler mit Schwelle |
| **Kostenteilung** | Keine / Spontan / Vordefiniert (Restaurantpreis) | Vordefiniert → Betrag im Anlass anzeigen; spontan → Post-Event-Hinweis |
| **Vorlaufzeit** | <1 Woche / 1–2 Wochen / 3–4 Wochen | Kurz → kein Doodle nötig; Lang → Terminsuche + mehrstufige Erinnerungen |

### Event-Typ-Vergleich (erweitert)

| Attribut | Pizza (privat) | Auswärtsessen | BBQ | Spielrunde | Wanderung |
|---|---|---|---|---|---|
| **Kern-Koordination** | Mitbringsel | Ort / Reservation | Grill-Slots + Mitbringsel | Programm (Spiele) | Ziel + Route + Stärken |
| **Ort-Typ** | Privat indoor | Öffentlich indoor | Outdoor (privat/park) | Privat indoor | Outdoor |
| **Reservation** | Nein | Ja (kritisch) | Nein | Nein | Nein / Hütte |
| **Wetterabhängigkeit** | Keine | Keine | Mittel–hoch | Keine | Sehr hoch |
| **Körperl. Anforderung** | Keine | Keine | Keine | Keine | Mittel–hoch |
| **Kapazität** | Raumgrösse | Tischreservation | Grillkapazität | Spieltisch/Anzahl | 6–12 Optimal |
| **Terminunsicherheit** | Gering | Gering | Wetterrisiko | Gering | Wetter + Fitness |
| **Kostenteilung** | Keine | Ja (Restaurantpreis) | Ggf. Fleisch/Grillgut | Keine | Ggf. ÖV/Parkplatz |
| **Vorlaufzeit** | 1–2 Wochen | 1–2 Wochen | 2–3 Wochen | 1 Woche | 3–4 Wochen |
| **Mitbringsel-Label** | "Was bringst du mit?" | Nicht nötig / Wein | "Fürs Grillieren?" | "Bringst du Spiele?" | "Ausrüstung OK?" |
| **Nachbereitung** | Einfach | Einfach | Grill-Cleanup | Spielbericht opt. | Fotos/Route/Sicherheit |

---

## Kernpersonas

| Persona | Beschreibung |
|---|---|
| **Lotta (Organisatorin)** | Hat eine Idee, will keine Sekretärin der Gruppe sein |
| **Moritz (Stammteilnehmer)** | Kommt meistens, hasst Bürokratie |
| **Inge (Zurückhaltende)** | Würde gerne dabei sein, zu viele Hürden schrecken ab |
| **Felix (Gelegenheitsbrowser)** | Hat den Link geteilt bekommen, kennt den Kontext nicht |
| **Cem (Wiederkommer)** | War letztes Mal dabei, fragt sich: "Findet das nochmal statt?" |

---

## Phase 0 – Keimzelle: Die Idee entsteht

**Wer:** Lotta, Cem | **Zustand:** "Lass uns wieder eine Pizzarunde machen." Noch kein Datum.

**Hypothesen:**
- Potenzielle Organisatorinnen verwerfen Event-Ideen weil der Aufwand (WhatsApp + Doodle + Formular) abschreckt. → **Test:** Interview: "Hattest du mal eine Idee, die du nicht weiterverfolgt hast?"
- Minimal-Start (nur Titel + Beschreibung) senkt die Abbruchrate signifikant. → **Test:** A/B-Test Vollformular vs. Minimal-Start.
- Wiederkommer (Cem) werden Organisatoren, wenn ein vergangener Anlass als Vorlage dient ("Kopieren"). → **Test:** Nutzerbefragung nach Event.
- In Gruppen ohne Struktur fühlt sich niemand zuständig – das ist der eigentliche Blocker. → **Test:** Tiefeninterview: Kommt "Niemand fühlt sich verantwortlich" spontan?

---

## Phase 1 – Terminsuche (Doodle-Moment)

**Wer:** Lotta + alle Eingeladenen | **Zustand:** Termin offen. Aktuell nicht im System.

**Hypothesen:**
- Zwei separate Tools (Doodle + Anmeldung) frustrieren und führen zu Drop-off dazwischen. → **Test:** Usability-Test Zwei-Tool-Aufgabe: Frustrationsindikatoren messen.
- Kombinierter "Ich kann + ich bringe mit"-Schritt erhöht Abschlussrate. → **Test:** Prototype-Vergleich kombiniert vs. sequenziell.
- Zurückhaltende (Inge) bevorzugen anonyme Abstimmung vor offiziellem Start. → **Test:** Post-Nutzung-Befragung.
- Eine sichtbare Deadline ("Bis Freitag 20 Uhr") beschleunigt die Terminentscheidung. → **Test:** Events mit/ohne Deadline: Zeit bis Terminfix vergleichen.
- *(Wanderung-spezifisch)* Expliziter Plan-B bei Wetterrisiko erhöht Anmelde-Verbindlichkeit. → **Test:** Anmeldequote mit/ohne Plan-B-Angabe.

---

## Phase 2 – Einladung & Verbreitung

**Wer:** Lotta → Felix | **Zustand:** Termin steht. Wie erfährt die Gruppe davon?

**Hypothesen:**
- Ein einzelner Link als Einladung spart >60% Zeitaufwand vs. manuellem Schreiben. → **Test:** Zeitaufwand messen.
- Open-Graph-Preview (Titel, Datum, Kurzbeschreibung) erhöht Weiterleitungsrate in Messenger. → **Test:** A/B-Test mit/ohne OG-Tags.
- "Du kannst jemanden mitbringen"-Hinweis erhöht Neuanmeldungen aus Netzwerk. → **Test:** Herkunftstracking der Anmeldungen.

---

## Phase 3 – Anmeldephase *(Kernfeature, bereits vorhanden)*

**Wer:** Alle Eingeladenen | **Zustand:** Eintragen, Mitbringsel wählen.

**Hypothesen:**
- Mehr als 3 sichtbare Formularfelder erhöhen Abbruchrate. → **Test:** Heatmap + Abbruchanalyse.
- Sichtbare Teilnehmerliste ("Ach, Moritz ist auch da") erhöht Anmelderate bei Zurückhaltenden. → **Test:** Opt-in-Rate mit/ohne Liste.
- Klare Micro-Copy bei E-Mail-Feld erhöht Angabequote. → **Test:** A/B verschiedener Formulierungen.
- Mitbringsel-Doubletten (5x Wein) sind bekannter Pain Point. → **Test:** Interviews: Nennen >40% "Doubletten"?
- *(Spielrunde-spezifisch)* Spieltyp-Sichtbarkeit ("Kooperativ / Kompetitiv / Party") erhöht Zusagerate weil Fehlerwartungen sinken. → **Test:** A/B mit/ohne Spielkategorie-Filter.
- *(Spielrunde-spezifisch)* "Spiel schon vorhanden"-Flag verhindert Duplikate effektiver als beim Essen. → **Test:** Spielvielfalt vor/nach Einführung der Sichtbarkeitsliste.

---

## Phase 4 – Vorbereitungsphase (Koordination)

**Wer:** Lotta + Moritz | **Zustand:** Zwischen Anmeldeschluss und Event.

**Hypothesen:**
- Organisatorinnen erstellen manuell eine Mitbringsel-Zusammenfassung für WhatsApp – automatisierbar. → **Test:** Interviews: Haben >60% diese Zusammenfassung erstellt?
- Automatische Erinnerung (1 Tag vorher, inkl. "Du bringst: Getränke") wird als nützlich empfunden. → **Test:** Opt-in-Rate + NPS nach Event.
- Teilnehmer, die kurzfristig absagen, tun dies nicht aktiv – sie erscheinen einfach nicht. → **Test:** Verhältnis Anmeldungen zu Erschienenen.
- *(Wanderung-spezifisch)* Treffpunkt-Unklarheit erzeugt überproportional viele Rückfragen. Map-Link + ÖPNV-Hinweis reduziert Support-Aufwand. → **Test:** Rückfrage-Nachrichten vor Events mit/ohne strukturierten Treffpunkt-Block.

---

## Phase 5 – Durchführung

**Wer:** Alle Anwesenden | **Zustand:** Event findet statt.

**Hypothesen:**
- Teilnehmer greifen am Event-Tag auf die Event-Seite zu (Adresse nachschlagen, Wer kommt noch?). → **Test:** Page-Aufrufe am Event-Tag tracken.
- Ein Dankeschön-Micro-Feedback (1 Klick + 1 Satz via QR-Code) wird aktiv genutzt. → **Test:** Teilnahmequote messen.

---

## Phase 6 – Nachbereitung (Post-Event)

**Wer:** Lotta, Moritz, Inge (die nicht da war) | **Zustand:** Event vorbei.

**Hypothesen:**
- Teilnehmer teilen Fotos, wenn es einen accountfreien zentralen Ort gibt. → **Test:** Upload-Rate vs. WhatsApp-Baseline.
- Automatisches "Memory" (Datum, Teilnehmerzahl, Mitbringsel-Übersicht) wird als Anerkennung wahrgenommen und geteilt. → **Test:** Mockup-Interview.
- "Beim nächsten Mal dabei?"-Option in Dankes-E-Mail erzeugt Opt-in für Folge-Events. → **Test:** Conversion-Rate.
- Nicht-Teilnehmer (Inge) werden durch Fotos/Zusammenfassung zur nächsten Teilnahme motiviert (FOMO). → **Test:** Opt-in-Rate von Nicht-Teilnehmern.

---

## Phase 7 – Wiederengagement & Community

**Wer:** Alle, langfristig | **Zustand:** Wird aus einem Anlass ein Rhythmus?

**Hypothesen:**
- Gruppen mit 2+ Events über dieselbe Plattform zeigen deutlich höhere Wiederkehrrate. → **Test:** Kohorten-Analyse nach 6 Monaten.
- "Nächsten Anlass erstellen"-Prompt direkt beim Event-Abschluss nutzt vorhandene Energie. → **Test:** Conversion vs. spätere E-Mail.
- Teilnehmer wollen die Organisator-Rolle nicht zweimal hintereinander – Rotations-Wunsch. → **Test:** Bestätigen >50% in Interviews?

---

## Helfer-Konzept (Option)

### Idee

Für grössere oder aufwändigere Anlässe (BBQ, Wanderung, Spielrunde mit vielen Personen) werden **Helfer mit spezifischen Aufgaben** benötigt – nicht nur Teilnehmer, die etwas mitbringen. Das Helfer-Konzept erweitert die bestehende Anmeldung um eine zweite Registrierungsspur.

### Zusätzliche Use Cases

| Use Case | Beschreibung | Neu / Erweiterung |
|---|---|---|
| **Helfer-Slot-Definition** | Organisatorin definiert offene Aufgaben mit Personenzahl: "2× Grillmaster", "1× Einkaufen", "2× Auf-/Abbau" | Neu (Admin-Seite) |
| **Helfer-Anmeldung** | Teilnehmer meldet sich gezielt für einen Slot an – zusätzlich zur oder statt normaler Gast-Anmeldung | Erweiterung Phase 3 |
| **Offene Slots sichtbar** | Einladungsseite zeigt "Noch 1 Grillmaster gesucht" – soziale Sichtbarkeit offener Aufgaben | Erweiterung Phase 2+3 |
| **Helfer-Briefing** | Separate E-Mail an Helfer: frühere Ankunftszeit, spezifische Instruktionen, Kontakt der Organisatorin | Neu (E-Mail-Kanal) |
| **Helfer-Erinnerung** | Gesonderte Erinnerung 2 Tage vorher mit Aufgaben-Details (anders als Gast-Erinnerung) | Erweiterung Phase 4 |
| **Helfer-Anerkennung** | Post-Event: Helfer werden im "Memory"-E-Mail namentlich erwähnt; sichtbar im Profil | Erweiterung Phase 6 |
| **Helfer-Profil** | "Hat bei 5 Events geholfen" – Helfer-Geschichte im persönlichen Profil sichtbar | Erweiterung Profil-System |
| **Direkte Helfer-Einladung** | Organisatorin kann bekannte Helfer direkt per E-Mail für einen Slot anfragen (Token-basiert) | Neu (Phase 2) |

### Verknüpfung mit dem Lebenszyklus

- **Phase 0 – Idee:** Organisatorin kann beim Erstellen des Anlasses sofort Helfer-Slots definieren ("Ich brauche jemanden fürs Grillen").
- **Phase 2 – Einladung:** Der Einladungslink zeigt offene Helfer-Slots prominent: "Wir suchen noch: 1× Grillmaster". Kann als separater "Helfer gesucht"-Link geteilt werden.
- **Phase 3 – Anmeldung:** Formular mit Wahl: "Ich komme als Gast" / "Ich helfe gerne bei: [Dropdown der offenen Slots]" / Beides.
- **Phase 4 – Vorbereitung:** Helfer erhalten eigenen Kommunikationsstrang (früherer Ankunftszeitpunkt, Aufgaben-Checkliste).
- **Phase 6 – Nachbereitung:** Helfer werden im Post-Event-Memory namentlich hervorgehoben – stärker als normale Teilnehmer.
- **Phase 7 – Wiederengagement:** Profil zeigt Helfer-Geschichte. System kann aktiv fragen: "Lotta organisiert wieder – du hast letztes Mal geholfen. Dabei?"

### Verbindung zum URL-Vorlage-System

Event-Typ-Vorlagen können Standard-Helfer-Slots mitbringen:

| Event-Typ | Standard-Slots in Vorlage |
|---|---|
| BBQ | Grillmaster (1–2×), Einkaufen (1×), Auf-/Abbau (2×) |
| Wanderung | Tourenführer (1×), Erste-Hilfe-verantwortlich (1×) |
| Spielrunde | Spielauswahl kuratieren (1×), Getränke koordinieren (1×) |
| Pizza-Abend | Einkaufen (1×) – optional, da Mitbringsel-Koordination meist reicht |

### Designfragen (offen)

- Ist ein Helfer auch Gast? → **Standard: Ja**, Helfer ist gleichzeitig eingeladen; Ausnahme explizit markierbar.
- Können Helfer einander sehen? → Ja, sinnvoll für Koordination ("Cem macht das Grillieren, du nimmst den Auf-/Abbau").
- Gibt es eine Kapazitäts-Obergrenze pro Slot? → Ja, definiert beim Erstellen des Slots.

### Hypothesen zum Helfer-Konzept

- Organisatorinnen zögern, aktiv um Hilfe zu bitten ("Ich will niemanden belasten") – ein strukturiertes Slot-System senkt diese Hemmschwelle, weil das System fragt, nicht die Person. → **Test:** Interview: "Hättest du eher um Hilfe gebeten, wenn es ein Formular gegeben hätte?"
- Teilnehmer helfen bereitwilliger, wenn die Aufgabe klar und zeitlich begrenzt definiert ist ("30 Min. Einkaufen vor dem Event") vs. vage ("Kannst du irgendwie helfen?"). → **Test:** A/B-Test: Slot mit Zeitangabe vs. ohne – Slot-Besetzungsrate vergleichen.
- Öffentlich sichtbare offene Slots ("Noch niemand hat sich für Grillmaster gemeldet") erzeugen positiven sozialen Druck und erhöhen die Helfer-Quote. → **Test:** Sichtbare offene Slots vs. nur intern für Organizer: Besetzungsrate vergleichen.
- Helfer, die in der Post-Event-Kommunikation namentlich erwähnt werden, helfen beim nächsten Anlass häufiger wieder. → **Test:** Wiederholungs-Helfer-Rate mit/ohne namentlicher Erwähnung.
- Das Helfer-Konzept ist ein stärkerer Community-Bindungs-Mechanismus als die normale Teilnahme: "Ich bin Teil des Events, nicht nur Gast." → **Test:** NPS-Vergleich Helfer vs. reguläre Teilnehmer nach dem Event.
- Die Anzahl offener Helfer-Slots ist ein indirekter Indikator für Anlass-Komplexität: Events mit >3 Slots haben höhere Organisationsabbruchrate. → **Test:** Korrelation Slot-Anzahl und Erfolgsrate der Eventdurchführung.

---

## Gemeinsamer Kern (typ-invariant)

Diese Mechanismen gelten **für alle Event-Typen unverändert**:
- Token-System, Moderationsqueue, Edit/Delete-Link, Spam-Filter, E-Mail-Bestätigung (Phase 3)
- Persönlicher Bearbeitungslink reduziert Abmeldehürde
- Kein Pflicht-Account senkt Eintrittsbarriere
- Sichtbarkeit der Teilnehmerliste erhöht Commitment
- "Wollen wir das nochmal machen?" als Re-Engagement-Trigger (Phase 7)

---

## Designprinzipien

1. **Ein Link = die ganze Reise** – Vom Doodle bis zum Foto-Album, eine URL.
2. **Profil ohne Account** – E-Mail-Adresse + fixer Token ergeben eine persistente Identität ohne traditionelle Registrierung. Dies ermöglicht zwei Dimensionen:
   - *Intern (Systemseite):* Aufbau eines Nutzerprofils für Vorschläge (nächste Anlässe, bevorzugtes Mitbringsel, passende Gruppen)
   - *Extern (Nutzerseite):* Über den persönlichen Token-Link kann der Nutzer eigene Präferenzen einsehen und eine persönliche Liste vergangener und kommender Anlässe abrufen – kein Login nötig
3. **Null-Account-Prinzip** – Token-Links sind die Identität; kein Passwort, kein Login-Formular.
4. **Organisieren fühlt sich wie Einladen an** – Gastgeberin, nicht Projektmanagerin.
5. **Gemeinschaft vor Skalierung** – 10 Menschen, die sich kennen > 100 anonyme Nutzer.
6. **Die App verschwindet, die Erinnerung bleibt** – Post-Event-Inhalte laden zum nächsten ein.

### Profil-Prinzip: Auswirkungen auf den Lebenszyklus

Das Profil-Prinzip reichert mehrere Phasen an:

| Phase | Interner Nutzen (System) | Externer Nutzen (Nutzer) |
|---|---|---|
| 0 – Idee | Vorschlag: "Deine Gruppe hat lange nichts gemacht" | – |
| 1 – Terminsuche | Verfügbarkeitsmuster aus vergangenen Teilnahmen | Kalender-Sync möglich |
| 3 – Anmeldung | Vorausfüllen von bevorzugtem Mitbringsel | Eigene Präferenz sichtbar & editierbar |
| 6 – Nachbereitung | Event dem Profil-Verlauf hinzufügen | "Du warst bei 5 Pizzamittagen dabei" |
| 7 – Wiederengagement | Passende Folge-Events vorschlagen | Persönliche Event-Liste mit kommenden Anlässen |

**Was sieht der Nutzer im Profil? (Hypothesen)**

Neben angemeldeten und vergangenen Events gibt es weitere Profil-Inhalte – hier als testbare Hypothesen:

| Profilinhalt | Hypothese | Test |
|---|---|---|
| Vergangene Events (Liste) | Nutzer möchten ihren "Beitrag" zur Gemeinschaft sehen – nicht nur Datum, sondern was sie gebracht haben | Interview: "Was würdest du dir auf einem Rückblick-Bildschirm wünschen?" |
| Bevorstehende Events | Nutzer vergessen Anlässe, wenn sie nicht im Kalender sind; eine persönliche "Demnächst"-Liste reduziert No-Shows | Vergleich No-Show-Rate mit/ohne Profil-Erinnerung |
| Bevorzugtes Mitbringsel (Präferenzen) | Nutzer erinnern sich nicht immer, was sie letzte Mal gebracht haben; eine gespeicherte Präferenz beschleunigt Anmeldung | A/B-Test: vorausgefülltes vs. leeres Mitbringsel-Feld |
| Verlässlichkeitssignal (intern sichtbar) | Nutzer, die ihre eigene Erscheinungsquote sehen, empfinden das als Motivator, nicht als Sanktion | Befragung: "Motiviert dich diese Zahl zur Teilnahme?" |
| Organisationshistorie | Nutzer, die einmal organisiert haben, sind eher bereit, es wieder zu tun, wenn ihr Beitrag sichtbar ist | Vergleich Zweit-Organisationsrate mit/ohne Sichtbarkeit |
| Ernährungspräferenzen / Hinweise | Gespeicherte Hinweise (vegan, Nussallergie) ersparen wiederholtes Eintippen | Tracking wie oft Hinweis-Feld leer bleibt bei Nutzern mit Profil vs. ohne |
| Gruppenkontext ("Deine Kreise") | Nutzer möchten sehen, zu welchen Event-Gruppen sie gehören, um Einladungen besser einzuordnen | Befragung: "Weisst du, wer dich für diesen Anlass eingeladen hat?" |

---

## Code of Conduct

Klar sichtbare Verhaltenserwartungen sind Teil der Community-Pflege – sie schützen die Organisatorin und fördern Verlässlichkeit.

**Kern-Regeln (Entwurf):**
1. **Verbindlich anmelden** – Wer sich anmeldet, erscheint. Oder sagt rechtzeitig ab.
2. **Mitbringsel einhalten** – Was man einträgt, bringt man mit. Änderungen bitte vorher kommunizieren.
3. **Einladung beantworten** – Auch ein "Nein" ist besser als Schweigen.
4. **Organisator respektieren** – Die Person, die organisiert, leistet Freiwilligenarbeit.
5. **Datenschutz der Gruppe** – Fotos und Informationen aus dem Anlass verbleiben im Kreis der Beteiligten.

**Hypothesen zum Code of Conduct:**
- Explizit sichtbare Regeln (z.B. kurz bei Anmeldung eingeblendet) reduzieren No-Shows, weil Verbindlichkeit bewusst wird. → **Test:** No-Show-Rate vor/nach Einführung der Regeln.
- Nutzer empfinden einen kurzen "Ich habe verstanden"-Klick bei Anmeldung nicht als Hürde, sondern als Zugehörigkeitssignal. → **Test:** Abbruchrate mit vs. ohne kurzen CoC-Hinweis bei Anmeldung.
- Organisatorinnen fühlen sich sicherer, einen Anlass öffentlich zu teilen, wenn ein Code of Conduct sichtbar ist. → **Test:** Interview: "Würde ein CoC deine Bereitschaft zum Teilen des Links erhöhen?"

---

## Gamification & Malus-System

**Gamification für Organisieren** (Bonus-Anreize):

| Element | Mechanik | Hypothese |
|---|---|---|
| "Erstorganisator"-Badge | Wer zum ersten Mal organisiert, erhält ein sichtbares Abzeichen im Profil | Badge erhöht Stolz und Bereitschaft, es nochmals zu tun |
| "Zuverlässige Gastgeberin"-Status | Ab X organisierten Events, die erfolgreich durchgeführt wurden | Teilnehmer vertrauen Anlässen bekannter Organisatoren mehr |
| "Rotierender Organisator"-Anreiz | System schlägt aktiv nächste Person vor + würdigt die aktuell organisierende | Verhindert, dass immer dieselbe Person muss |
| Teilnahme-Streak | "Du warst bei 5 Events dabei" – keine Punkte, nur Sichtbarkeit | Soziale Bestätigung motiviert zur Kontinuität |

**Malus für fehlende Rückmeldungen:**

| Verhalten | Malus | Hypothese |
|---|---|---|
| Anmeldung + kein Erscheinen, keine Absage (No-Show) | Privates Signal: "Du warst angemeldet aber nicht da" im Profil; bei Wiederholung: Hinweis bei nächster Anmeldung | Sichtbarkeit eigenen Verhaltens motiviert zur Änderung stärker als externe Sanktion |
| Einladung zur Terminabstimmung ignoriert (kein Vote) | Kein harter Malus; aber Organizer sieht "X Personen haben noch nicht abgestimmt" | Transparenz erzeugt sozialen Druck ohne Bestrafung |
| Mitbringsel kurzfristig nicht mitgebracht (ohne Update) | Kein automatischer Malus; aber Post-Event-Feedback ermöglicht Organizer-Hinweis | Peer-Accountability ist effektiver als System-Sanktion |

**Hypothesen zum Gamification/Malus-System:**
- Ein privates, nicht-öffentliches No-Show-Signal ("Du warst 2x nicht erschienen") reduziert Wiederholungs-No-Shows, ohne Scham zu erzeugen. → **Test:** No-Show-Rate nach Einführung des privaten Signals.
- Badges für Organisatoren erhöhen die Rate von Erst-Organisatoren, wenn sie im Anmeldeformular sichtbar sind ("Organisiert von Lotta – Zuverlässige Gastgeberin"). → **Test:** Anmelderate bei Events mit/ohne Organizer-Badge.
- Teilnehmer empfinden Gamification als authentisch, wenn sie soziale Anerkennung (Zugehörigkeit, Beitrag) widerspiegelt – nicht als spielerische Punkte. → **Test:** Befragung: "Fühlt sich dieses Badge bedeutsam oder albern an?"
- Ein "Du hast noch nicht auf X Einladungen reagiert"-Hinweis im Profil erhöht die Abstimmungs-Beteiligung bei Terminsuchen. → **Test:** Vote-Rate vor/nach Einführung des Hinweises.

---

## E-Mail als strategischer zweiter Kanal

E-Mail ist nicht nur technische Benachrichtigung – sie ist der **primäre Beziehungsaufbau-Kanal** zwischen Plattform und Nutzer. Aktuell: einmalig (Edit-Link nach Anmeldung). Potenzial: vollständiger Kommunikationsstrang entlang des gesamten Lebenszyklus.

### E-Mail-Touchpoints entlang des Lebenszyklus

| Zeitpunkt | Inhalt | Zweck |
|---|---|---|
| **Sofort nach Anmeldung** | Bestätigung + persönlicher Edit-Link + kurze Plattform-Erklärung + CoC-Zusammenfassung | Onboarding, Vertrauen, Verbindlichkeit herstellen |
| **1–2 Tage vor Event** | Erinnerung: Datum, Uhrzeit, Ort, "Du bringst: X" + Absage-Link prominent sichtbar | No-Show-Prävention, Mitbringsel in Erinnerung rufen |
| **Tag des Events** | Optionale Kurzinfo: Treffpunkt, Kontakt der Organisatorin, Wetter (bei Outdoor) | Orientierung, letzte Sicherheit |
| **24h nach Event** | Dankeschön + Feedback-Frage (1–3 Klicks) + Fotos-Upload-Link + "Nächster Anlass?"-Hinweis | Feedbackloop, Community-Stärkung, Wiederengagement |
| **Bei neuem Anlass der Gruppe** | Personalisiertes "Deine Gruppe trifft sich wieder"-Reminder | Niedrigschwelliger Wiedereinstieg |

### Plattform-Erklärung per E-Mail (Onboarding)

Erstmalige Nutzer verstehen das Token-Konzept nicht intuitiv. Die Anmelde-Bestätigungs-E-Mail ist der ideale Moment für ein Mini-Onboarding:

**Entwurf (Kern-Inhalte der Willkommens-E-Mail):**
- "Du bist angemeldet für [Anlass] am [Datum]"
- "Dein persönlicher Link – damit kannst du jederzeit deine Anmeldung ändern oder absagen: [LINK]" (hervorgehoben)
- "Was ist Quietschente? – Eine einfache Plattform für private Anlässe. Kein Passwort, kein Account. Dein Link ist dein Zugang."
- Kurze CoC-Zusammenfassung: "Wir freuen uns auf dich. Bitte meld dich ab, wenn du doch nicht kommen kannst."
- Optional: "Möchtest du beim nächsten Anlass direkt benachrichtigt werden? [Ja, bitte]"

### Feedback-Loop per E-Mail (Post-Event)

Das Post-Event-E-Mail schliesst den Kreislauf und füttert das Profil:

**Feedback-Fragen (maximal 3, 1-Klick-Antworten):**
1. "Warst du dabei?" → Ja / Nein, hatte Grund / Nein, vergessen
2. "Wie war es?" → Sehr gut / Gut / Okay / Weniger gut *(nur wenn Ja bei Frage 1)*
3. "Möchtest du beim nächsten Mal dabei sein?" → Ja, benachrichtige mich / Ich melde mich selbst

**Interne Verwendung der Antworten:**
- Erschienen: trägt zu Verlässlichkeits-Signal im Profil bei
- Bewertung: aggregiert als Event-Qualitäts-Indikator für Organisatorin
- Interesse: füllt automatisch "Beim nächsten Mal"-Opt-in

### Hypothesen zur E-Mail-Kommunikation

- Eine kurze Plattform-Erklärung direkt in der Bestätigungs-E-Mail reduziert Rückfragen an die Organisatorin ("Wie bearbeite ich meine Anmeldung?"). → **Test:** Anzahl Direkt-Rückfragen vor/nach Einführung der Erklärung.
- Eine Erinnerungs-E-Mail 24h vor dem Event senkt die No-Show-Rate deutlich, wenn der Absage-Link prominent sichtbar ist. → **Test:** No-Show-Rate mit/ohne Erinnerungs-E-Mail.
- Nutzer, die per E-Mail über einen Folge-Anlass informiert werden, melden sich schneller an als solche, die aktiv den Link suchen müssen. → **Test:** Anmeldezeitpunkt (Tage vor Event) bei E-Mail-benachrichtigten vs. selbst-informierten Nutzern.
- 1-Klick-Post-Event-Feedback (keine Umleitung zu Formular) erzeugt eine Rücklaufquote >30% – bei mehrstufigem Formular wäre sie <10%. → **Test:** A/B-Test 1-Klick-Antwort vs. Link zu separatem Formular.
- Die Kombination von CoC-Hinweis + Absage-Link in der Erinnerungs-E-Mail erhöht aktive Absagen, weil beides zusammen Verbindlichkeit und einfachen Ausstieg kommuniziert. → **Test:** Quote aktiver Absagen vor/nach Einführung dieser Kombination.
- Nutzer, die "Ja, benachrichtige mich beim nächsten Anlass" geklickt haben, melden sich beim Folge-Event mit signifikant höherer Rate an als die Kontrollgruppe. → **Test:** Conversion-Rate Opt-in-Nutzer vs. Gesamt-Eingeladene.

---

## Nächste Schritte (Design Thinking Prozess)

| Schritt | Methode | Ziel |
|---|---|---|
| Empathize | 5 Interviews Organizer-Typ (Lotta) | Pain Points Phase 0–1 |
| Empathize | 5 Interviews Teilnehmer-Typ (Moritz/Inge) | Hürden Anmeldung & Engagement |
| Define | "How Might We"-Workshop | Kernproblem schärfen |
| Ideate | Crazy 8s für Phase 1 (Terminsuche) | Schnellste Time-to-Value |
| Prototype | Papier-Mockup "Alles-in-einem-Link" | Null-Account-Prinzip validieren |
| Test | 3 echte Gruppen, echter Anlass | Daten statt Annahmen |

---

## Kritische Dateien (für technische Umsetzung v1.2+)

- [index.html](index.html) – Hardcoded "Pizzamittag"-Strings (Zeilen 6, 147, 200, 230, 291), Emoji-Map (Zeilen 313–323)
- [admin_pizza.html](admin_pizza.html) – Hardcoded Titel/Header (Zeilen 6, 118, 129, 143, 146)
- [assets/Code_pizza_1.gs](assets/Code_pizza_1.gs) – E-Mail-Template (Zeilen 369–370), Startzeit "12:00" (Zeilen 451, 477), Anlässe-Schema (kein Typ-Feld)
