# Quietschente – Software-Architektur: Definition & Entscheidungen

**Rolle:** Senior Software Architekt | **Datum:** 2026-06-28 | **Status:** Zur Review

---

## Context

Quietschente ist eine Anlass-Koordinations-App für kleine Privatgruppen (Pizza-Abend, BBQ, Spielrunde, Wanderung). Der aktuelle Prototyp läuft vollständig auf **Google Apps Script (GAS) + Google Sheets** und deckt Phase 3–4 des Event-Lebenszyklus ab.

Die Produkt-Vision (Dokument `ich-habe-hier-einen-idempotent-reddy.md`) beschreibt einen Weg von v1.0 (Pizza-only) bis v3.0 (Template Engine, Multi-Gruppe). Dieser Plan definiert die **Architekturentscheidungen**, die notwendig sind, um diesen Weg ohne Sackgassen zu gehen — und priorisiert klar, was **jetzt** entschieden werden muss vs. was **später** relevant wird.

**Kern-Constraint:** App bleibt kostenlos oder nahezu kostenlos (~15 CHF/Jahr Domainkosten). Single developer.

---

## Systemkontext (C4 Level 1)

```
┌─────────────────────────────────────────────────────────────┐
│                      Quietschente                           │
│                                                             │
│  ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   │
│  │   Frontend   │   │   Backend    │   │  Datenspeicher│  │
│  │  (HTML/JS)   │──▶│  (GAS/exec) │──▶│ (Google Sheets│  │
│  │              │   │              │   │    5 Tabs)    │   │
│  └──────────────┘   └──────────────┘   └──────────────┘   │
└─────────────────────────────────────────────────────────────┘
         │                   │
         ▼                   ▼
   Teilnehmer          Gmail/MailApp
   Organisatoren       (E-Mail-Versand)
```

**Externe Systeme:** Google Sheets (DB), GAS (Runtime+Hosting), Gmail (E-Mail), DNS-Provider (Domain), (zukünftig: Cloudflare Workers, Brevo)

**Nutzerrollen:**
- **Teilnehmer:** erhalten Link, tragen sich ein, kein Account
- **Organisatorin:** erstellt Anlässe, moderiert, verwaltet Admin-Dashboard
- **Admin (technisch):** betreibt die Instanz (derzeit eine Person = Organisatorin)

---

## Architekturentscheidungen (ADRs)

### ADR-1: Runtime-Plattform — GAS beibehalten oder migrieren?

**Entscheidung: GAS durch v1.x und v2.0. Migration erst bei konkretem Trigger.**

| Option | Vorteile | Nachteile |
|--------|----------|-----------|
| **A: GAS beibehalten** | Null Infrastrukturkosten, kein Ops-Aufwand, Admin sieht Daten direkt in Sheets | Cold Start 3–5s; 6-Min-Execution-Limit; keine Pfad-Segmente in URLs; manuelle Joins; keine Transaktionen |
| B: Hybrid (GAS + externes DB) | Echte Queries, Indices | Externe Abhängigkeit, GAS→DB-Verbindung via UrlFetchApp |
| C: Vollmigration (Workers + D1) | Alle GAS-Limits fallen weg | Ops-Overhead, Kostenrisiko, bricht "Admin-sieht-Sheets"-Workflow |

**Empfehlung A** bis zum Migrations-Trigger (s. Roadmap). Die GAS-Limits sind bei der angestrebten Nutzerzahl (<50 Personen/Anlass) kein Problem.

**Migrations-Trigger (explizit definieren):**
- >500 Gesamtregistrierungen im Einträge-Sheet
- >3 gleichzeitig offene Anlässe
- Bedarf an komplexen Joins (>3 Sheets gleichzeitig)
- Zweite unabhängige Gruppe wird aufgeschaltet

---

### ADR-2: URL-Routing-Strategie

**Entscheidung: Query-Parameter jetzt; Cloudflare-Redirect bei v1.2.**

GAS unterstützt keine Pfad-Segmente. `/bbq` ist nicht möglich, `?type=bbq` schon.

| Variante | Umsetzbar wann | Aussehen |
|----------|----------------|----------|
| Query-Parameter (jetzt) | sofort | `quietschente.ch?type=bbq` |
| Cloudflare-Redirect (v1.2) | wenn Branding wichtig wird | `bbq.quietschente.ch` → Redirect |
| Pfad-Routing | nach GAS-Migration | `quietschente.ch/bbq/new` |

**Implementierung Cloudflare-Redirect (v1.2):**
```javascript
// Cloudflare Worker (Free Tier: 100k Req/Tag)
const slugMap = { bbq: 'bbq', pizza: 'pizza', wandern: 'wanderung', spiel: 'spielrunde' };
// Wildcard DNS *.quietschente.ch → Worker → 301 zu GAS-URL mit ?type=
```

Der Cloudflare-Worker ist auch der spätere **API-Gateway-Einstiegspunkt** nach einer GAS-Migration — diese Investition zahlt sich doppelt aus.

---

### ADR-3: Identität & Authentifizierung

**Entscheidung: Persistenter Profil-Token pro E-Mail-Adresse (Zero-Passwort-Prinzip beibehalten).**

Das Zero-Passwort-Prinzip ist ein Produkt-Kern-Prinzip und darf nicht aufgeweicht werden.

| Modell | Heute | v2.0+ |
|--------|-------|-------|
| Aktuell | Token pro Anmeldung (einmalig) | — |
| **Empfehlung** | beibehalten | Persistenter Profil-Token pro E-Mail → ermöglicht Profilseite, Eventhistorie, Präferenzen |
| Alternative: Magic Link | — | Höhere Sicherheit, aber zusätzlicher Schritt bei jeder Session → widerspricht dem Produkt-Prinzip |

**Persistenter Profil-Token (v2.0):**
- 1 Token pro E-Mail, dauerhaft gültig (Bearer Credential)
- Gespeichert in `Profile`-Sheet
- "Meinen Link erneut senden"-Flow als Wiederherstellung
- Alle personalisierten Links verwenden `?profile_token=<token>`
- Bestehende Anmelde-Token (`eintrag_token`) bleiben erhalten für Einzel-Aktionen

**Sicherheitshinweis:** Token = Identität. Nur via HTTPS übertragen (GAS und Cloudflare erzwingen HTTPS). Nie in Logs schreiben.

---

### ADR-4: Datenschema für die gesamte Vision

**Entscheidung: Entity-Modell mit JSON-Extension-Feldern; direkt migrations-kompatibel zu Postgres.**

Jede Entität = ein Sheet-Tab. JSON-Felder (`settings_json`, `fields_json`) ermöglichen Erweiterung ohne Schema-Änderung und mappen direkt auf Postgres JSONB.

```
Gruppen          (group_id, name, slug, settings_json, created_at)
Profile          (profile_id, email, token, display_name, preferences_json, badges_json, created_at)
Anlässe          (event_id, group_id, type, template_id, title, date, start, phase [0-7], location,
                  max_participants, settings_json, fields_snapshot_json, created_at, closed_at)
Anlass-Typ-Felder (type, field_key, field_label, field_type, required, sort_order)
Einträge         (entry_id, event_id, profile_id, status, fields_json, action_token, created_at, updated_at)
Email-Log        (log_id, profile_id, event_id, template_key, sent_at, status)
Spam-Filter      (rule_id, pattern, type, action, active, created_at)
System-Logs      (timestamp, level, function_name, event_id, profile_id, message, error_json)
```

**Wichtige Designentscheidungen im Schema:**
- `phase` als Integer (0–7) auf Anlässen: ermöglicht geordnete Lifecycle-Logik
- `group_id` ab sofort auf allen Entitäten: vorbereitet Multi-Tenancy ohne Daten-Migration
- `profile_id` auf Einträgen (nicht nur E-Mail): ermöglicht Historienabfragen auch nach Namensänderungen
- Archiv: kein separater Tab → `status`-Feld + Zeitstempel; historische Versionen via `version`-Feld
- `template_id` (nullable): null für v2-Events; ab v3 Referenz auf Template-Objekt (→ ADR-10)
- `fields_snapshot_json`: wird beim Übergang zu Phase 2 (INVITATION) gesetzt und danach nicht mehr verändert; sichert Vorwärtskompatibilität bei v3-Upgrade (→ ADR-10)

**Kritische Dateien für Schema-Anpassung:**
- [assets/Code_pizza_1.gs](assets/Code_pizza_1.gs) — `initSheets()` ab Zeile ~50 definiert alle Sheet-Strukturen
- Neue Sheets müssen in `initSheets()` ergänzt werden, bevor sie im Code verwendet werden

---

### ADR-5: Multi-Tenancy

**Entscheidung: Separate GAS-Deployment pro Gruppe (Status quo beibehalten).**

Mehrere Gruppen in einem Sheets-File zu betreiben (row-level filtering nach `group_id`) schafft ein Datenleck-Risiko, das in GAS/Sheets schwer abzusichern ist. Separate Deployments sind einfacher, sicherer und wartbarer für eine einzelne Entwicklerin.

`group_id` wird trotzdem ab sofort in alle Entitäten eingepflegt — als Vorarbeit für eine spätere Migration zu einem geteilten Backend, ohne Datentransformation.

---

### ADR-6: E-Mail-Infrastruktur

**Entscheidung: GAS MailApp jetzt → Brevo (externer Dienst) ab v1.2.**

| Phase | Dienst | Limit | Kosten |
|-------|--------|-------|--------|
| v1.0 | GAS MailApp | 100/Tag (Free) | 0 |
| v1.2+ | Brevo | 300/Tag (Free-Tier) | 0 |

**E-Mail-Lebenszyklus (6 Touchpoints pro Anlass):**
1. Bestätigung + Bearbeitungslink (sofort nach Anmeldung)
2. Erinnerung T-3 Tage (inkl. "Du bringst: X" + prominenter Absage-Link)
3. Tag-des-Events (Treffpunkt, Kontakt, Wetter bei Outdoor)
4. Post-Event Danke + 1-Klick-Feedback
5. Foto-Upload-Einladung (24h nach Event)
6. Re-Engagement bei nächstem Anlass der Gruppe

**Template-Strategie:** Templates als HTML-Strings in einem `Email-Templates`-Sheet-Tab, mit `{{variable}}`-Platzhaltern. Kein Template-Framework nötig — einfaches `replace()` in GAS. Ermöglicht Copy-Änderungen ohne Code-Deployment.

**Brevo-Integration (v1.2):**
```javascript
// GAS: UrlFetchApp.fetch("https://api.brevo.com/v3/smtp/email", { ... })
// API-Key in Script Properties (nicht im Code)
```

---

### ADR-7: Event-Typ-Erweiterungsmodell

**Entscheidung: Konfigurationsgetrieben (Option B) ab v2.0; bis dahin zwei hartcodierte Typen.**

Neue Anlass-Typen sollen Dateneingaben sein, keine Code-Änderungen. Der `Anlass-Typ-Felder`-Tab (ADR-4) ist das Herzstück.

**Rollout-Plan:**
- **v1.0–v1.1:** nur Pizza (Status quo)
- **v1.2:** BBQ + Pizza hartcodiert (schnelle Umsetzung)
- **v2.0:** `Anlass-Typ-Felder` einführen; BBQ + Wanderung + Spielrunde konfigurationsgetrieben
- **v3.0:** Organisatorin definiert eigene Felder via UI (Template Engine)

**Felder pro Typ (Vorschlag):**

| Typ | Zusatzfelder |
|-----|-------------|
| Alle | Name, E-Mail, Antwort (Ja/Nein/Vielleicht), Kommentar |
| Pizza/BBQ | Mitbringsel (required), Wetter-Plan-B (BBQ) |
| Wanderung | Fitnesslevel, Hund dabei, Treffpunkt-Info bestätigt |
| Spielrunde | Spielerfahrung, Spielpräferenz (kooperativ/kompetitiv/party) |
| Restaurant | Menüwahl, Allergien |

---

### ADR-8: Frontend-Architektur

**Entscheidung: Alpine.js als Drop-in-Erweiterung; Query-Parameter als Client-Router.**

Alpine.js löst das Problem der dynamischen Formularfelder (ADR-7) ohne Build-Pipeline. Es ist CDN-ladbar, erfordert keine npm, und bleibt bei einer GAS-Migration nutzbar.

```html
<!-- Drop-in CDN -->
<script src="https://cdn.jsdelivr.net/npm/alpinejs@3/dist/cdn.min.js" defer></script>
```

**Client-seitiges Routing via Query-Parameter:**
```javascript
const view = new URLSearchParams(window.location.search).get('view');
// 'register' | 'edit' | 'profile' | 'admin' | 'event-detail'
```

**Kritische Dateien:**
- [index.html](index.html) — Hauptformular (ab Zeile 1 bis ~470); Emoji-Map Zeile 313–323; hardcoded API-URL Zeile 138
- [admin_pizza.html](admin_pizza.html) — Admin-Dashboard; hardcoded Titel Zeile 6, 118, 129, 143, 146

---

### ADR-9: Admin-Interface

**Entscheidung: Admin als geschützte View in der Haupt-App; Admin-Token in GAS Script Properties.**

Admin-Token in URL (`?view=admin&token=...`) ist für eine private Community-App akzeptabel. Der Token darf nie im HTML-Source erscheinen — er wird client-seitig an jede API-Anfrage angehängt.

```javascript
// Token in sessionStorage speichern nach erstem Login
sessionStorage.setItem('admin_token', inputToken);
// Nie in URL-History schreiben — stattdessen via POST oder Header senden
```

---

### ADR-10: Vorwärtskompatibilität v2→v3 (Template Engine Migration)

**Entscheidung: Field-Snapshot auf Event-Entität beim Phase-2-Übergang; automatisches Migrations-Skript via Schema-Versionskennung.**

**Problem:** v3.0 macht `Anlass-Typ-Felder` zur admin-definierbaren Ressource (Template Engine). V2-Events referenzieren `type = "bbq"` — nach einem v3-Upgrade könnte dieser Typ verändert oder der Tab restrukturiert worden sein. Laufende Events würden ihre Feld-Konfiguration verlieren; Anmeldeformulare wären nicht mehr darstellbar.

---

**Teil 1 – Field-Snapshot bei Phasenübergang (präventiv, ab v2 einzubauen)**

Beim Übergang zu Phase 2 (INVITATION) legt `EventLifecycleService` einen unveränderlichen JSON-Snapshot der Felder an:

```javascript
advanceToNextPhase(eventId) {
  const event = this.eventRepo.findById(eventId);
  event.advancePhase();

  // Snapshot einmalig beim Übergang zu INVITATION setzen
  if (event.phase === EventPhase.INVITATION && !event.fieldsSnapshot) {
    const fields = this.typeFieldRepo.findByType(event.type);
    event.fieldsSnapshot = { version: 2, fields };
  }

  this.eventRepo.save(event);
  return event;
}
```

Der Snapshot wird in `fields_snapshot_json` (ADR-4) gespeichert und danach nie mehr überschrieben.

---

**Teil 2 – Dualer Lesepfad in v3**

v3-Code prüft beim Rendern eines Events immer zuerst den Snapshot:

```javascript
function resolveFields(event) {
  // Events ab Phase 2: Snapshot ist einzige Wahrheitsquelle
  if (event.fieldsSnapshot && event.phase >= EventPhase.INVITATION) {
    return event.fieldsSnapshot.fields;
  }
  // v3-Event noch im Entwurf (Phase 0–1) mit Template
  if (event.templateId) {
    return templateRepo.getFields(event.templateId);
  }
  // v2-Fallback: Event noch in Phase 0–1, kein Snapshot, kein templateId
  return typeFieldRepo.findByType(event.type);
}
```

---

**Teil 3 – Selbstauslösendes Migrations-Skript (damit es nicht vergessen werden kann)**

Das Skript wird nicht manuell aufgerufen, sondern ist Teil der Applikations-Initialisierung. Ein `schema_version`-Eintrag im Config-Tab steuert, ob und welche Migrationen noch ausstehen:

```javascript
// Wird bei jedem GAS-Start in doGet/doPost aufgerufen
function runPendingMigrations() {
  const currentVersion = configRepo.getSchemaVersion(); // z.B. 2
  if (currentVersion < 3) {
    migrateV2FieldSnapshots();
    configRepo.setSchemaVersion(3);
  }
}

function migrateV2FieldSnapshots() {
  const events = eventRepo.findAll();
  for (const event of events) {
    if (!event.fieldsSnapshot && event.phase >= EventPhase.INVITATION) {
      event.fieldsSnapshot = {
        version: 2,
        fields: typeFieldRepo.findByType(event.type)
      };
      eventRepo.save(event);
    }
  }
  Logger.log(`[Migration v3] Field-Snapshots für ${events.length} Events geprüft.`);
}
```

`schema_version` liegt als Zeile im bestehenden System-Logs-Tab oder in einem dedizierten Config-Tab. Das Skript ist **idempotent**: Wenn es ein zweites Mal läuft, überspringt es Events, die bereits einen Snapshot haben.

---

**Schema-Ergänzung für Config-Tab (neu, ADR-4 Erweiterung):**

```
Config   (key, value)
         z.B.: ("schema_version", "3"), ("migration_v3_ran_at", "2026-xx-xx")
```

---

**Garantien nach dem Upgrade:**

| Szenario | Verhalten |
|----------|-----------|
| v2-Event in Phase 3 (Anmeldung läuft), Deploy von v3 | Migrations-Skript läuft automatisch beim nächsten Request; Snapshot rückwirkend erstellt → keine Unterbrechung |
| Admin ändert BBQ-Template in v3 | Laufende BBQ-Events verwenden unverändert ihren Snapshot |
| Migrations-Skript "vergessen" zu deployen | Nicht möglich — es ist in `doGet/doPost` eingebettet und läuft selbst |
| Neues Event in v3 erstellt | Kein Snapshot bis Phase 1; Snapshot-Erstellung bei Phase-2-Übergang wie gewohnt |
| Skript läuft versehentlich zweimal | Keine Wirkung — idempotent |

---

## Migrations-Roadmap

```
v1.0 (jetzt)
  └── Stabilisierung: System-Logs vervollständigen, Fehlerbehandlung verbessern
      Datenschema: group_id auf alle Entitäten vorbereiten (auch wenn noch 1 Gruppe)
      Keine Architektur-Änderung nötig

v1.2 (nächste Version)
  ├── Cloudflare als DNS-Provider einrichten (für ADR-2 Subdomain-Redirect)
  ├── Brevo als E-Mail-Dienst integrieren (ADR-6)
  ├── Alpine.js in Frontend einführen (ADR-8)
  ├── event_type-Feld in Anlässe-Schema ergänzen
  ├── Anlass-Typ-Felder-Tab anlegen (Vorarbeit für v2.0)
  └── bbq.quietschente.ch → Cloudflare Worker → GAS ?type=bbq

v2.0 (Feature-Ausbau)
  ├── Profile-Tab + persistenter Profil-Token (ADR-3)
  ├── Profilseite (?view=profile&token=)
  ├── Voller E-Mail-Lebenszyklus (6 Touchpoints via Brevo)
  ├── Konfigurationsgetriebene Anlass-Typen (ADR-7 Option B)
  └── Evaluiere GAS-Migration bei definiertem Trigger (ADR-1)

v3.0 (Template Engine)
  ├── Wenn hier: GAS-Migration ist wahrscheinlich notwendig
  ├── Cloudflare Workers + D1 oder Supabase als DB
  ├── Template-Editor für Organisatorinnen im Admin
  ├── Config-Tab anlegen mit schema_version = 3
  ├── runPendingMigrations() in doGet/doPost einbetten (→ ADR-10)
  └── Field-Snapshot-Backfill läuft automatisch beim ersten v3-Request (→ ADR-10)
```

---

## Kostenmodell

| Komponente | v1.x | v2.0 |
|------------|------|-------|
| GAS Hosting | Free | Free |
| Google Sheets | Free | Free |
| Cloudflare Workers (Redirect) | Free (100k Req/Tag) | Free |
| Brevo E-Mail | Free (300/Tag, 9k/Monat) | Free |
| Domain quietschente.ch | ~15 CHF/Jahr | ~15 CHF/Jahr |
| Supabase (wenn migriert) | Free (500MB) | Free |
| **Total** | **~15 CHF/Jahr** | **~15 CHF/Jahr** |

---

## Querschnittsthemen

**Sicherheit:**
- Alle Tokens: kryptografisch zufällig (32+ Bytes). GAS: `Utilities.getUuid()` + Hashing
- Tokens nie loggen, nie in HTML-Source einbetten
- Admin-Token in GAS Script Properties (nicht im Code)
- Cloudflare WAF-Regeln für Rate-Limiting (Free-Tier) bei Subdomain-Setup

**DSGVO / Datenschutz:**
- Minimale Datenhaltung: nur E-Mail + Anzeigename
- E-Mails nur für Event-Kommunikation, kein Marketing
- Löschrecht: via persönlichem Bearbeitungslink bereits implementiert
- Datenlöschung nach Event-Ende: automatisierbar via GAS-Trigger (Archivierung nach X Tagen)

**Observability:**
- System-Logs-Tab: append-only, für alle write-Operationen
- Log-Format: `timestamp | level | function | event_id | profile_id | message | error_json`
- Bei v2.0: Sentry Free-Tier via GAS UrlFetchApp für Error-Tracking

**Lokalisation:**
- Nur Deutsch. Kein i18n-Framework.
- Alle UI-Strings in einer `Texte`-Konstanten-Datei oder Sheet-Tab → Copy-Änderungen ohne Code-Deployment

---

## Aufgeschobene Entscheidungen

Diese Themen sind **bewusst nicht entschieden** — sie werden getriggert, nicht geplant:

| Thema | Trigger für Entscheidung |
|-------|--------------------------|
| WebSockets / Echtzeit | Wenn Live-Koordination am Event-Tag gewünscht wird |
| Push-Notifications | Wenn Mobile-App oder PWA in Scope kommt |
| Zahlungsintegration | Wenn Kostenteilung-Feature umgesetzt wird |
| Open Graph / Social Preview | Sobald Einladungs-UX optimiert wird (v1.2 kandidat) |
| Analytics | Sobald A/B-Tests aus dem Design-Thinking-Dokument durchgeführt werden |
| CoC-Enforcement | `violations_json` auf Profile-Entität bereits vorgesehen; Logik bei v2.0 |

---

## Verifikation / Nächste Schritte

1. **Schema validieren:** `initSheets()` in [assets/Code_pizza_1.gs](assets/Code_pizza_1.gs) gegen das neue Datenschema prüfen — welche Felder fehlen noch?
2. **GAS-Limits messen:** Aktuelle Response-Zeiten und Execution-Dauer loggen — Baseline für Migrations-Trigger
3. **Cloudflare einrichten:** Domain quietschente.ch auf Cloudflare NS übertragen (auch wenn Subdomains erst bei v1.2 genutzt werden)
4. **Brevo-Account anlegen:** API-Key in GAS Script Properties hinterlegen, E-Mail-Versand testen
5. **Alpine.js Proof-of-Concept:** Dynamisches Formularfeld (Anlass-Typ-Auswahl) in index.html prototypisch einbauen
