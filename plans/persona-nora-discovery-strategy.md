# Persona 6 – Nora (Die Suchende): Discovery-Feature & Strategische Entscheidungen

**Version:** v0.1-konzept | **Datum:** 2026-06-28 | **Basis:** Design Thinking Erweiterung

---

## Kontext

Die bestehenden 5 Kernpersonas (Lotta, Moritz, Inge, Felix, Cem) setzen alle voraus, dass ein Nutzer durch eine persönliche Einladung zur Plattform kommt. Persona 6 bricht dieses Modell auf: Sie kommt ohne Einladung und sucht aktiv nach lokalen Events basierend auf eigenen Interessen.

Diese Persona ist keine reine Feature-Erweiterung – sie stellt eine **strategische Weggabelung** für das Produkt dar.

---

## Persona-Steckbrief

| Attribut | Beschreibung |
|---|---|
| **Name** | Nora |
| **Typ** | Die Suchende / Die Neue |
| **Lebenssituation** | Neu in der Stadt, nach Umzug / Studienende / Trennung – sozialer Kreis noch klein |
| **Zitat** | *"Ich würde gerne an einem Spielabend mitmachen – aber ich weiss nicht, wo ich suchen soll, ohne auf einer Eventbrite-Massenveranstaltung zu landen."* |
| **Digitale Affinität** | Mittel bis hoch – nutzt Apps, aber will keinen weiteren Account anlegen |
| **Verhältnis zu Lotta** | Kennt Lotta (noch) nicht – kommt über Discovery, nicht über Einladung |

### Pain Points

- Eventbrite / Meetup fühlt sich anonym und kommerziell an
- Lokale Facebook-Gruppen sind unübersichtlich
- Angst, zu einem Event zu gehen, wo sie niemanden kennt
- Weiss nicht, wo sie nach kleinen, privaten Gruppen suchen soll

### Kernbedürfnisse

- Events nach Interessen-Kategorie filtern (Wandern, Kochen, Spiele, BBQ, Kultur)
- Gruppen-Grösse sehen – kleine Gruppen (6–12) wirken sicherer
- Profil mit Präferenzen anlegen, ohne Account-Registrierung
- Einen "sozialen Ankerpunkt" haben (kennt jemand von den Teilnehmern?)

---

## Profil-Seite: Interessen & Vorschläge

### Neue Profil-Elemente für Nora

Ergänzung zur bestehenden Profil-Tabelle im Hauptdokument:

| Profilinhalt | Noras Perspektive | Hypothese | Test |
|---|---|---|---|
| **Interessen-Tags** (Wandern, Kochen, Spiele, BBQ, Kultur) | "Ich will einmal einstellen, was mich interessiert" | Tags erhöhen Vorschlagsqualität – aber nur wenn ≥5 lokale Events/Woche in der Kategorie vorhanden sind, sonst wirkt das Feature leer | Aktivierungsrate nach Regionen mit <5 vs. >10 Events |
| **Umkreis-Präferenz** | "Nur Events in meiner Stadt / bis 20 km" | Räumliche Nähe ist stärkerer Entscheidungsfilter als Interessen-Match | A/B: Vorschläge sortiert nach Nähe vs. nach Interessen-Score |
| **Gruppen-Grösse-Präferenz** | "Lieber 6–10 Personen als 50" | Kleingruppen-Filter wird von Noras Typ öfter genutzt als von eingeladenen Teilnehmern | Nutzungs-Rate des Filters bei Neulinge vs. Eingeladene |
| **Bekannte Gesichter** | "Kenne ich jemanden, der auch dabei ist?" | 1 gemeinsame Verbindung erhöht Anmelde-Conversion bei Fremden stärker als der Event-Inhalt selbst | Anmelderate Events mit/ohne "Du kennst jemanden hier"-Hinweis |

### Profil-Anlage (Token-Modell beibehalten)

Noras Profil wird über denselben Mechanismus wie bestehende Nutzer angelegt:

```
1. Nora gibt E-Mail-Adresse ein
2. Sie erhält einen persönlichen Token-Link
3. Über diesen Link: Interessen-Tags wählen, PLZ / Stadtbereich angeben
4. Sofortige Vorschläge: "3 Events passen zu dir diese Woche"
```

**Kein Passwort, kein separates Registrierungsformular** – identisch mit dem bestehenden Null-Account-Prinzip.

---

## User Journey Nora (neu)

```
Phase -1 – Discovery (nicht im bestehenden Lifecycle):
  Nora findet Quietschente über Google ("Spielabend Zürich"),
  einen geteilten Social-Media-Post oder einen QR-Code
  → landet auf einer öffentlichen Event-Übersicht oder Landing Page

Phase 0 – Profil anlegen:
  Nora wählt 3–5 Interessen-Tags, gibt PLZ / Stadt an
  → E-Mail-Eingabe → Token-Link kommt per Mail
  → Sofort: "3 Events passen zu dir diese Woche"

Phase 1 – Browsing:
  Nora sieht Event-Cards mit:
    ✅ Typ (BBQ, Wanderung, Spielrunde...)
    ✅ Datum + Wochentag
    ✅ Gruppen-Grösse (z.B. "8–12 Personen")
    ✅ Offene Plätze
    ✅ Organisatoren-Verlässlichkeits-Badge
    ✅ Stadtteil / ungefährer Ort
    ❌ Vollständige Teilnehmerliste (Datenschutz)
    ❌ Private Kontaktdaten

Phase 3 – Anmeldung:
  Identisch mit bestehendem Anmelde-Formular
  + Optionales Feld: "Ich kenne bisher niemanden hier – kurze Vorstellung?"
  → Lotta sieht "Neuling"-Flag in Admin-Ansicht und kann aktiv willkommen heissen

Phase 7 – Wiederengagement:
  Noras Profil akkumuliert Events
  → nach 2 Events mit derselben Gruppe:
    "Möchtest du Anlässe dieser Gruppe direkt mitbekommen?"
  → Nora wechselt vom Discovery-Kanal in den Einladungs-Kanal
  → langfristig: Nora wird zur Inge, Moritz – oder sogar zur Lotta
```

---

## Strategische Weggabelung: 3 Wege

Dies ist die zentrale Entscheidung, bevor irgendein Feature gebaut wird.

### Weg A – Optionale Event-Öffentlichkeit *(Empfehlung)*

**Konzept:** Lotta kann beim Erstellen eines Events wählen: *"Nur mein Kreis"* oder *"Offen für Gleichgesinnte in der Region"*. Nora sieht ausschliesslich öffentlich markierte Events.

| | Details |
|---|---|
| **Passt zu** | "Gemeinschaft vor Skalierung", Null-Account-Prinzip, bestehendem Tech-Stack |
| **Lotta behält Kontrolle** | Ihre privaten Events bleiben privat – keine erzwungene Öffnung |
| **Risiko** | Wenige öffentliche Events in der Anfangsphase → Nora findet nichts ("Empty-Shelf-Problem") |
| **Vorteil** | Kein Plattform-Wandel – Discovery ist ein optionales Layer auf dem bestehenden Produkt |
| **Version** | v2.x (nach Multi-Event-Typ) |

**Offene Fragen:**
- Wer darf einem öffentlichen Event beitreten? Jeder mit Account / nur verifizierte Noras?
- Sieht Lotta vor der Genehmigung eine Kurzbeschreibung von Nora?
- Gibt es eine Kapazitätsgrenze für Externe (z.B. max. 2 Neulinge pro Event)?

---

### Weg B – Interessen-Matching innerhalb bekannter Kreise

**Konzept:** Kein Fremdentdecken. Nora hinterlegt Interessen und bekommt eine E-Mail, wenn jemand aus einem *bestehenden Quietschente-Netzwerk*, mit dem sie verbunden ist, einen passenden Event ankündigt.

| | Details |
|---|---|
| **Passt zu** | Vollständig zum bestehenden Modell |
| **Einschränkung** | Nora muss bereits Teil eines Kreises sein – hilft echten Neuen kaum |
| **Vorteil** | Datenschutz-einfach, kein Discovery-Infrastruktur-Aufwand |
| **Version** | v1.x als Profilseite-Erweiterung machbar |

**Realitätscheck:** Dieser Weg löst Noras Kernproblem nicht – sie hat keinen bestehenden Kreis. Nützlich als erste Stufe (Profilseite mit Interessen), aber unvollständig.

---

### Weg C – Kuratiertes lokales Verzeichnis

**Konzept:** Quietschente wird zu einer offenen Plattform mit öffentlichem Eventverzeichnis, ähnlich Meetup – aber privater und kuratierter.

| | Details |
|---|---|
| **Verlässt** | "10 Menschen die sich kennen"-Vision fundamental |
| **Erfordert** | Eigene Moderations-Infrastruktur, Community-Management, Trust & Safety |
| **Risiko** | Skalierungsprobleme, Qualitätsverlust, Produktidentitätsverlust |
| **Vorteil** | Grösste Reichweite, direkt für Noras Bedürfnis gebaut |
| **Version** | v3.x oder separates Produkt |

**Empfehlung:** Nicht vor Validierung von Weg A verfolgen. Weg C ist ein anderes Produkt.

---

## Entscheidungsmatrix

| Kriterium | Weg A | Weg B | Weg C |
|---|---|---|---|
| Löst Noras Kernproblem | ✅ Ja | ⚠️ Teilweise | ✅ Ja |
| Passt zu Designprinzipien | ✅ Ja | ✅ Ja | ❌ Nein |
| Umsetzbar ohne Backend-Rewrite | ✅ v2.x | ✅ v1.x | ❌ Nein |
| Lotta behält Kontrolle | ✅ Ja | ✅ Ja | ⚠️ Eingeschränkt |
| Datenschutz-Risiko | ⚠️ Mittel | ✅ Niedrig | ❌ Hoch |
| Chicken-Egg-Problem | ⚠️ Ja (wenig Events anfangs) | ✅ Nein | ❌ Stark |
| Strategischer Aufwand | Mittel | Niedrig | Sehr hoch |

---

## Hypothesen zu Noras Persona

| Hypothese | Test |
|---|---|
| Nora bricht bei der Anmeldung eher ab als Moritz, wenn sie keine einzige bekannte Person in der Teilnehmerliste sieht – ein "Neuling-willkommen"-Signal reduziert diesen Drop signifikant | Anmelderate Neulinge mit/ohne expliziten Willkommens-Hinweis von Organisatorin |
| Ein Interessen-Profil mit weniger als 5 verfügbaren Events in der Nähe wirkt kontraproduktiv ("Die App funktioniert nicht für mich") | Profil-Aktivierungsrate nach Regionen mit <5 vs. >10 verfügbaren Events |
| Nora akzeptiert das Token-Modell (kein Passwort) schneller als erwartet, wenn die Onboarding-E-Mail erklärt, dass ihr persönlicher Link ihr "Schlüssel" ist | Abbruchrate Onboarding-Flow Nora-Typ vs. Lotta-Typ |
| Lotta zögert, ihren Event öffentlich zu machen, wenn sie nicht weiss, wer Nora ist – ein "Verifizierter Neuling"-Status (hat bereits 1 Event auf Quietschente besucht) erhöht Lottas Bereitschaft zur Öffnung | Anteil öffentlicher Events bei Organisatorinnen mit/ohne Einsicht in Neuling-Status |
| Räumliche Nähe (Stadtteil) ist für Nora ein stärkerer Entscheidungsfilter als Interessen-Match | A/B-Test: Vorschläge sortiert nach Interessen-Score vs. nach Distanz |
| Noras grösste Hürde ist nicht die Anmeldung, sondern die erste physische Anwesenheit – eine Kurz-Vorstellung ("Ich komme alleine und kenne noch niemanden") senkt diese Hürde messbar | Vergleich No-Show-Rate Neulinge mit/ohne Vorstellungs-Option |

---

## Verbindung zum bestehenden Lifecycle

Noras Phase -1 und Phase 0 sind neu. Ab Phase 3 (Anmeldung) ist die Journey identisch mit bestehenden Personas.

| Phase | Noras Spezifika | Bestehendes Feature |
|---|---|---|
| -1 Discovery | Öffentliche Event-Übersicht (neu) | – |
| 0 Profil | Interessen-Tags + Umkreis (neu) | Token-Profil (vorhanden) |
| 1 Terminsuche | Sieht öffentliche Events (neu) | Bestehende Anmelde-Seite |
| 3 Anmeldung | + "Neuling"-Flag optional (minimal neu) | ✅ Vorhanden |
| 4 Vorbereitung | Lotta sieht Neuling-Flag in Admin (minimal neu) | ✅ Admin-Seite vorhanden |
| 6 Nachbereitung | Gleich wie andere Teilnehmer | ✅ Vorhanden |
| 7 Wiederengagement | Opt-in für Gruppen-Kanal (neu) | Teilweise vorhanden |

---

## Empfohlene nächste Schritte

| Schritt | Methode | Ziel | Priorität |
|---|---|---|---|
| Strategieentscheid | Team-Workshop: Weg A / B / C? | Klare Produktvision vor Entwicklung | **Zuerst** |
| Empathize | 3–5 Interviews mit "Nora-Typ" (neu in Stadt, kleiner Kreis) | Validieren: Ist Discovery tatsächlich ein Pain Point? | Hoch |
| Empathize | 3–5 Interviews mit Lottas: "Würdest du deinen Event öffentlich machen?" | Angebots-Seite validieren (Weg A) | Hoch |
| Define | HMW: "Wie könnten wir Nora einen sozialen Ankerpunkt geben?" | Kernproblem schärfen | Mittel |
| Prototype (Weg A) | Mockup: Öffentliche Event-Card + Neuling-Anmeldung | Low-Cost-Test der Discovery-Idee | Mittel |
| Test | Pilotregion mit 10 öffentlichen Events | Empty-Shelf-Problem validieren | Vor Launch |

---

## Kritische Designfrage (offen)

> *Ist Quietschente ein Werkzeug für bestehende Freundeskreise – oder eine Plattform für lokale Gemeinschaftsbildung?*

Beide Antworten sind valid – aber sie führen zu unterschiedlichen Produkten, unterschiedlichen Metriken und unterschiedlichen Wachstumsstrategien. **Dieser Entscheid sollte bewusst und explizit getroffen werden, bevor Noras Feature gebaut wird.**

Weg A ist der risikoärmste Einstieg: Er lässt Lotta entscheiden, ob sie öffnen will – und gibt dem Team Daten, ob Discovery überhaupt Bedarf hat, ohne die bestehende Produktvision zu verlassen.
