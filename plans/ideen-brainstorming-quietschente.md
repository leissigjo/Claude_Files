# Quietschente – Ideen-Sammlung

**Status:** Brainstorming | Nicht Teil der offiziellen Architektur oder Use-Case-Dokumente

---

## Idee 1: Probability Engine — Top-5-Events-Empfehlung

**Kernfrage:** Welche fünf Events würden eine bestimmte Person mit der höchsten Wahrscheinlichkeit interessieren?

### Konzept

Eine Engine, die anhand von Verhaltensdaten und Präferenzen eines Profils eine gewichtete Rangliste offener Events berechnet.

### Eingabe-Signale (Features)

| Signal | Herkunft | Gewicht (Idee) |
|--------|----------|----------------|
| Vergangene Teilnahmen (Event-Typ) | `Registration`-History | hoch |
| Explizite Präferenzen im Profil | `Profile.preferences` | hoch |
| Saisonalität / Wochentag | `Event.date` | mittel |
| Soziales Signal: wer sonst ist angemeldet | `Registration`-Liste | mittel |
| Zeit seit letztem Event | `EmailLog` / Attendance | mittel |
| Event-Popularität (Auslastung) | `maxParticipants` vs. angemeldet | niedrig |
| Badges des Profils | `Profile.badges` | niedrig |

### Mögliche Scoring-Formel

```
score(event, profile) =
  w1 * typeSimilarity(event.type, profile.history)
  + w2 * preferenceMatch(event.typeConfig, profile.preferences)
  + w3 * socialProximity(event.registrations, profile.network)
  + w4 * recencyBoost(profile.lastAttendance)
  - w5 * penaltyIfTooSoon(event.date, profile.lastAttendance)
```

### Offene Fragen

- Wie baut man ein Netzwerk-Graph, wenn keine expliziten Freundschaften existieren? (Ko-Teilnahme als Proxy?)
- Cold-Start-Problem: neue Profile ohne History — Fallback auf Gruppen-Default oder onboarding-Quiz?
- Threshold: ab welchem Score wird ein Event überhaupt vorgeschlagen?
- Delivery: Push-Notification, E-Mail-Digest, oder In-App-Feed?

---

## Idee 2: Skalierte Persönlichkeitsmatrix

**Kernfrage:** Wie beschreibt man eine Person mehrdimensional, um Empfehlungen, Ansprachen und Erwartungen zu verfeinern?

### Dimensionen

#### 2a) Big Five Persönlichkeitsskalen (OCEAN)

Jede Dimension als kontinuierliche Skala 0–100, nicht als Kategorie.

| Dimension | Niedrig (0) | Hoch (100) | Relevanz für Events |
|-----------|-------------|------------|---------------------|
| **Openness** | Routine, vertraut | Neugier, Experiment | Hoch: neue Event-Typen vs. bekannte Formate |
| **Conscientiousness** | Spontan, flexibel | Geplant, strukturiert | Mittel: Vorlaufzeit, Verbindlichkeit |
| **Extraversion** | Introvertiert | Extravertiert | Hoch: Gruppen-Setting, Lautstärke, Crowd-Größe |
| **Agreeableness** | Direkt, kompetitiv | Kooperativ, harmonisch | Mittel: Wettbewerbs-Events vs. Community-Events |
| **Neuroticism** | Stabil, gelassen | Sensitiv, stressanfällig | Mittel: Unsicherheits-Toleranz bei Outdoor/Wetter |

#### 2b) Präferenz-Dimensionen

Drei Typen von Präferenzen, die sich inhaltlich überschneiden können aber semantisch verschieden sind:

| Typ | Definition | Beispiel |
|-----|-----------|---------|
| **Attained** | Was die Person bereits hat / kann / erlebt hat | "Ich gehe regelmäßig zum BBQ" |
| **Desired** | Was die Person anstrebt / lernen / erleben will | "Ich will endlich mal Bouldern ausprobieren" |
| **Possible** | Was realistisch machbar wäre, aber noch nicht bewusst angestrebt | "Wäre gut verträglich mit meinem Profil, liegt aber unter dem Radar" |

Die dritte Kategorie (*Possible*) ist besonders wertvoll: sie erlaubt Entdeckungsempfehlungen jenseits der Komfortzone, aber innerhalb des Wahrscheinlichkeitsraums.

#### 2c) Skills-Matrix nach Persönlichkeit

**Grundthese:** Aus dem OCEAN-Profil lassen sich Fähigkeiten ableiten, die wahrscheinlich vorhanden sind (*expected*) und solche, die unwahrscheinlich, aber möglich sind (*unexpected*).

| Persönlichkeitsprofil (dominant) | Expected Skills | Unexpected Skills (Überraschungspotenzial) |
|----------------------------------|-----------------|---------------------------------------------|
| Hohe Openness | Kreativität, Neues aufgreifen, Improvisation | Strukturiertes Durchhalten, Routineexzellenz |
| Hohe Conscientiousness | Vorbereitung, Pünktlichkeit, Detailgenauigkeit | Spontane Leadership, Flow-Zustände |
| Hohe Extraversion | Netzwerken, Moderation, Energie im Raum halten | Tiefe 1:1-Verbindungen, Contemplation |
| Hohe Agreeableness | Konfliktmediation, Teamkitt, Empathie | Klare Konfrontation, Entscheidungen durchsetzen |
| Niedriges Neuroticism (stabil) | Krisenmanagement, Ruhepol | Emotionale Tiefe kommunizieren, Verletzlichkeit zeigen |

### Wie man die Matrix befüllt

**Option A — Explizit (Onboarding-Quiz)**
Kurze, spielerische Selbsteinschätzung beim ersten Login. Problem: Selbstwahrnehmung weicht oft von Fremdwahrnehmung ab.

**Option B — Implizit (Verhaltensbeobachtung)**
Aus Teilnahme-Patterns, Anmeldezeitpunkt (spontan vs. früh), Absage-Verhalten, Antwort-Texten in Formularfeldern ableiten.

**Option C — Hybrid**
Onboarding-Quiz gibt Startwert; Verhalten über Zeit passt den Wert an (bayesisches Update).

### Offene Fragen

- Wie granular soll die Skala sein? 5 Stufen, 10 Stufen, oder Kontinuum?
- Datenschutz: Persönlichkeitsdaten sind sensibel — opt-in, lokal gespeichert, oder nie explizit gezeigt?
- Wer nutzt die Matrix? Nur die Engine intern, oder auch der Admin / Eventorganisator?
- Soll die Person ihr eigenes Profil sehen können ("Du bist 72/100 Extraversion")? Gamification vs. Stigmatisierung?
- Zeitliche Stabilität: Big Five sind recht stabil, aber Präferenzen ändern sich — wie oft neu kalibrieren?

---

## Idee 3: E-Mail-basiertes Onboarding- und Trainingsprogramm für Einsteiger

**Kernfrage:** Wie bringt man neue Nutzer ohne Handbuch, Webinar oder Support-Aufwand dazu, Quietschente wirklich zu verstehen und zu nutzen?

### Konzept

Eine automatisierte E-Mail-Sequenz, die sich nach der ersten Anmeldung oder Registrierung eines neuen Profils auslöst — kein Newsletter, sondern ein kurzes Lernprogramm in kleinen Dosen.

### Phasen der Sequenz

| Tag | E-Mail | Inhalt |
|-----|--------|--------|
| 0 | Willkommen | Bestätigung + "Was ist Quietschente?" in 3 Sätzen |
| 2 | Dein erstes Event | Wie man sich anmeldet, was passiert danach |
| 5 | Dein Profil | Präferenzen setzen, was beeinflusst das? |
| 10 | Absagen & Änderungen | Wie man Anmeldungen verwaltet, ohne Schaden anzurichten |
| 20 | Hast du teilgenommen? | Re-Engagement: Einladung zum nächsten Event + Feedback-Frage |

### Mechanik

- Sequenz startet automatisch bei `Profile`-Erstellung (neues Trigger-Event im `EmailLog`)
- Jede E-Mail enthält **eine Aktion** (kein Informationsüberfluss)
- Sequenz pausiert, wenn Nutzer aktiv ist (z. B. bereits 2 Events besucht hat) — kein Training, das nervt
- Opt-out jederzeit möglich, aber kein globales Unsubscribe (nur Training-Sequenz)

### Inhalts-Prinzipien

- **Situativ**: "Du hast dich gerade angemeldet — hier ist, was als nächstes passiert" statt abstrakte Erklärungen
- **Kurz**: Max. 3 Absätze pro Mail, eine klare CTA
- **Progressiv**: Jede Mail baut auf der vorherigen auf; keine Wiederholungen

### Offene Fragen

- Wo werden die E-Mail-Templates verwaltet? In `SheetEmailTemplateRepository` wie die anderen?
- Trigger-Logik: reicht `Profile`-Erstellung, oder erst nach erster Anmeldung?
- Sprache: automatisch aus Profil-Sprache ableiten oder manuell je Gruppe konfigurierbar?
- Messung: Öffnungsrate und Click-Through als Signal für Kalibrierung der Sequenz?

---

## Idee 4: Quietschente für Vereine und Firmen

**Kernfrage:** Was muss sich ändern, damit Quietschente nicht nur für lockere Community-Gruppen, sondern auch für strukturierte Organisationen wie Vereine, NGOs oder Firmenabteilungen funktioniert?

### Unterschiede gegenüber dem Standard-Use-Case

| Dimension | Community-Gruppe | Verein / Firma |
|-----------|-----------------|----------------|
| Mitgliedschaft | Offen, selbst gewählt | Formell, oft mit Mitgliedsliste |
| Rollen | Admin + Teilnehmer | Admin, Vorstand, Mitglied, Gast, Extern |
| Teilnahme-Pflicht | Freiwillig | Teils verpflichtend (z. B. GV, Pflichtschulung) |
| Abrechnung | Keine | Rechnungsstellung, Budgetverantwortung |
| Datenschutz | Tolerant | Strikt (DSGVO-Compliance oft wichtiger) |
| Kommunikation | Locker | Protokollartig, archivierbar |

### Zusätzliche Features, die nötig wären

**Mitgliederverwaltung**
- Import bestehender Mitgliederlisten (CSV / API)
- Mitgliedsstatus: aktiv, passiv, ausgetreten
- Pflichtfeld "Mitgliedsnummer" in Registrierung

**Rollen & Berechtigungen**
- Mehrere Admins mit unterschiedlichen Rechten (z. B. "kann Events erstellen" vs. "kann Mitglieder verwalten")
- Vorstandsrolle: sieht Statistiken, kann aber keine Events bearbeiten

**Verpflichtende Events**
- Event-Flag `mandatory: true` → Abwesenheit wird protokolliert
- Reminder-Eskalation: 1. Erinnerung → 2. Erinnerung → Vorstand-Meldung

**Abrechnung & Gebühren**
- Teilnahmegebühr pro Event konfigurierbar
- Export für Buchhaltung (CSV mit Name, Event, Betrag, Datum)
- Integration mit Zahlungsanbieter (z. B. Stripe, TWINT) als spätere Erweiterung

**Compliance & Protokoll**
- Automatisches Protokoll jeder GV / Sitzung: Wer war da, Traktanden, Beschlüsse (als ausfüllbares Formularfeld im Event)
- Datenschutzerklärung als Pflicht-Checkbox bei Erstregistrierung
- Export aller Personendaten eines Mitglieds auf Anfrage (DSGVO Art. 15)

### Deployment-Optionen für Vereine / Firmen

| Option | Beschreibung | Aufwand |
|--------|-------------|---------|
| **Shared** | Verein erstellt eine Gruppe im bestehenden Quietschente | Minimal |
| **White-Label** | Eigene Domain (`events.meinverein.ch`), eigene Farben | Mittel |
| **Self-Hosted** | Verein hostet eigene Instanz (eigene Google-Sheets, eigenes GAS-Projekt) | Hoch |

### Monetarisierungsansatz

- Shared: kostenlos bis X Mitglieder, danach Abo
- White-Label: monatliche Flat Fee
- Self-Hosted: einmalige Lizenz oder Support-Vertrag

### Offene Fragen

- Welche Vereine / Firmen sind die primäre Zielgruppe? (Sportvereine, NGOs, KMU-Teams?)
- Wie komplex darf das Rollenmodell werden, bevor es die Architektur sprengt?
- Lässt sich die bestehende `Group`-Klasse erweitern, oder braucht es eine `Organization`-Entität?
- Wie geht man mit gemischten Events um (Vereinsmitglieder + externe Gäste)?

---

## Querverbindungen zwischen den Ideen

- Idee 1 (Probability Engine) könnte Idee 2 (Persönlichkeitsmatrix) als Feature-Vektor nutzen
- *Possible Preferences* aus Idee 2 könnten die Discovery-Komponente der Empfehlungs-Engine speisen
- *Unexpected Skills* könnten als Grundlage für "Trau dich"-Empfehlungen dienen (Events, die Komfort-Zone leicht dehnen)
- Idee 3 (E-Mail-Training) teilt die Template- und Trigger-Infrastruktur mit dem bestehenden `EmailService` — geringe Zusatzkosten
- Idee 4 (Vereine/Firmen) profitiert von Idee 3 direkt: Neue Vereinsmitglieder können automatisch ongeboardet werden
- Idee 1 + 2 sind für Vereine/Firmen weniger relevant (keine Discovery nötig), aber die Skills-Matrix (Idee 2) könnte für Firmenabteilungen interessant sein (Kompetenz-Mapping)
