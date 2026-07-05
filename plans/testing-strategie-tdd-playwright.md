# Quietschente – Testing-Strategie: TDD + Playwright

**Datum:** 2026-06-28 | **Basis:** SOLID-Architektur + GAS-Constraint

---

## Kernproblem: GAS ist nicht lokal ausführbar

Google Apps Script läuft in Googles Cloud — es gibt keine lokale GAS-Runtime.
Das bedeutet: direktes Unit-Testing von GAS-Code (`SpreadsheetApp`, `MailApp`, etc.) ist ohne Mocks nicht möglich.

**Die Lösung liegt in der Architektur:** Die SOLID-Schichtentrennung aus dem Klassendiagramm macht TDD möglich, weil:

- **Domain-Schicht** (Email, Token, Event, Registration, SpamCheckService) = pures JavaScript, kein GAS-Import → direkt mit Vitest/Jest testbar
- **Service-Schicht** (RegistrationService, EmailService) = hängt nur an Interfaces → mit Mocks testbar
- **Infrastruktur-Schicht** (GoogleSheetsRepository, GasMailAppSender) = GAS-spezifisch → nur via E2E testbar

---

## Test-Pyramide für Quietschente

```
             ▲
            /█\
           / █ \          E2E Tests (Playwright)
          / ███ \         5–10 kritische User Journeys
         /───────\        Gegen staging GAS-Deployment
        / ███████ \
       /  ███████  \      Integrationstests (manuell / optional)
      /  █████████  \     GAS-Funktionen gegen echtes Staging-Sheet
     /───────────────\
    / ███████████████ \   Unit Tests (Vitest)
   /  ███████████████  \  Domain + Services mit Mocks
  /─────────────────────\ ~80% der Testabdeckung hier
```

**Faustregel:** Alles was kein GAS braucht → Unit Test. Alles was Browser + GAS braucht → Playwright.

---

## Projektstruktur (neu)

```
Quietschente/
├── src/
│   ├── domain/                    ← Pures JS, kein GAS
│   │   ├── Email.js
│   │   ├── Token.js
│   │   ├── EventPhase.js
│   │   ├── Event.js
│   │   ├── Registration.js
│   │   ├── Profile.js
│   │   └── SpamCheckService.js
│   ├── services/                  ← Hängt an Interfaces, mockbar
│   │   ├── RegistrationService.js
│   │   ├── EmailService.js
│   │   └── EventLifecycleService.js
│   └── infrastructure/            ← GAS-spezifisch, E2E-tested
│       ├── GoogleSheetsEventRepo.js
│       ├── GoogleSheetsRegistrationRepo.js
│       ├── GasMailAppSender.js
│       └── BrevoEmailSender.js
├── tests/
│   ├── unit/
│   │   ├── domain/
│   │   │   ├── Email.test.js
│   │   │   ├── Token.test.js
│   │   │   ├── EventPhase.test.js
│   │   │   ├── Registration.test.js
│   │   │   └── SpamCheckService.test.js
│   │   └── services/
│   │       ├── RegistrationService.test.js
│   │       └── EmailService.test.js
│   └── e2e/
│       ├── registration.spec.js   ← Anmeldung
│       ├── edit-delete.spec.js    ← Bearbeiten/Löschen via Token
│       ├── admin.spec.js          ← Moderation + Event-Verwaltung
│       └── spam.spec.js           ← Spam-Filter-Verhalten
├── package.json
├── vitest.config.js
├── playwright.config.js
└── Code.gs                        ← GAS Entry Point (via esbuild gebündelt)
```

---

## Build-Pipeline: src/ → Code.gs

GAS unterstützt kein `import/export`. Lösung: **esbuild** bündelt `src/` in eine einzige `Code.gs`-Datei.

```json
// package.json
{
  "scripts": {
    "build":      "esbuild src/main.js --bundle --outfile=Code.gs --format=iife --platform=browser",
    "test:unit":  "vitest run",
    "test:e2e":   "playwright test",
    "test":       "vitest run && playwright test",
    "deploy":     "npm run build && clasp push && clasp deploy --force"
  },
  "devDependencies": {
    "vitest": "^2.0.0",
    "@playwright/test": "^1.45.0",
    "esbuild": "^0.21.0",
    "@google/clasp": "^2.4.2"
  }
}
```

**Deployment-Flow:**
```
src/ (ES6 Module)
     │
     ▼ esbuild build
Code.gs (IIFE, kein import/export)
     │
     ▼ clasp push
GAS-Projekt in Google Cloud
     │
     ▼ clasp deploy
Neue Deployment-URL (Staging oder Prod)
```

---

## Teil 1: TDD — Unit Tests mit Vitest

### TDD-Zyklus

```
1. RED   → Test schreiben, der fehlschlägt (Klasse existiert noch nicht)
2. GREEN → Minimaler Code, der den Test besteht
3. REFACTOR → Code aufräumen, Test muss weiter grün bleiben
```

### Beispiel: Email Value Object

```javascript
// tests/unit/domain/Email.test.js
import { describe, it, expect } from 'vitest';
import { Email } from '../../../src/domain/Email.js';

describe('Email', () => {
  it('akzeptiert gültige Schweizer E-Mail-Adresse', () => {
    expect(() => new Email('moritz@example.ch')).not.toThrow();
  });

  it('wirft Fehler bei fehlendem @', () => {
    expect(() => new Email('keinat.ch')).toThrow('Ungültige E-Mail');
  });

  it('normalisiert Grossbuchstaben zu Kleinbuchstaben', () => {
    expect(new Email('MORITZ@EXAMPLE.CH').toString()).toBe('moritz@example.ch');
  });

  it('zwei gleiche E-Mails sind gleich', () => {
    expect(new Email('a@b.ch').equals(new Email('A@B.CH'))).toBe(true);
  });
});
```

### Beispiel: EventPhase Zustandsmaschine

```javascript
// tests/unit/domain/EventPhase.test.js
import { EventPhase } from '../../../src/domain/EventPhase.js';

describe('EventPhase', () => {
  it('startet bei IDEA (0)', () => {
    expect(EventPhase.IDEA.value).toBe(0);
  });

  it('erlaubt Übergang IDEA → SCHEDULING', () => {
    expect(EventPhase.IDEA.canTransitionTo(EventPhase.SCHEDULING)).toBe(true);
  });

  it('verbietet Rücksprung REGISTRATION → IDEA', () => {
    expect(EventPhase.REGISTRATION.canTransitionTo(EventPhase.IDEA)).toBe(false);
  });

  it('wirft Fehler bei ungültigem Phase-Wert', () => {
    expect(() => new EventPhase(99)).toThrow();
  });
});
```

### Beispiel: SpamCheckService mit Regeln

```javascript
// tests/unit/domain/SpamCheckService.test.js
import { SpamCheckService } from '../../../src/domain/SpamCheckService.js';
import { SpamRule } from '../../../src/domain/SpamRule.js';

describe('SpamCheckService', () => {
  const rules = [
    new SpamRule('casino', 'keyword', 'abgelehnt'),
    new SpamRule('http', 'url', 'in_pruefung'),
  ];
  const checker = new SpamCheckService(rules);

  it('akzeptiert normale Anmeldung', () => {
    const result = checker.check({ name: 'Moritz', hinweise: 'Vegan bitte' }, { submittedAt: Date.now() - 5000 });
    expect(result.status).toBe('angefragt');
  });

  it('lehnt Casino-Spam ab', () => {
    const result = checker.check({ name: 'Casino Winner', hinweise: '' }, { submittedAt: Date.now() - 5000 });
    expect(result.status).toBe('abgelehnt');
  });

  it('markiert URL-Einträge zur Prüfung', () => {
    const result = checker.check({ name: 'Link', hinweise: 'Schau mal: http://example.com' }, { submittedAt: Date.now() - 5000 });
    expect(result.status).toBe('in_pruefung');
  });

  it('markiert zu schnelle Eingaben als verdächtig', () => {
    const result = checker.check({ name: 'Bot', hinweise: '' }, { submittedAt: Date.now() - 1000 }); // < 2.5s
    expect(result.status).toBe('in_pruefung');
    expect(result.grund).toContain('timing');
  });
});
```

### Beispiel: RegistrationService mit gemockten Dependencies (DIP zahlt sich aus)

```javascript
// tests/unit/services/RegistrationService.test.js
import { vi, describe, it, expect, beforeEach } from 'vitest';
import { RegistrationService } from '../../../src/services/RegistrationService.js';

describe('RegistrationService', () => {
  let regRepo, profileRepo, spamChecker, emailService, service;

  beforeEach(() => {
    // Mocks für alle Interfaces — kein GAS, kein echtes Sheet
    regRepo = {
      findByToken: vi.fn(),
      save: vi.fn(),
      findByEventId: vi.fn().mockReturnValue([]),
    };
    profileRepo = {
      findByEmail: vi.fn().mockReturnValue(null),
      save: vi.fn(),
    };
    spamChecker = {
      check: vi.fn().mockReturnValue({ status: 'angefragt', passed: true }),
    };
    emailService = {
      sendConfirmation: vi.fn(),
    };

    service = new RegistrationService(regRepo, profileRepo, spamChecker, emailService);
  });

  it('erstellt neues Profil wenn E-Mail noch unbekannt', async () => {
    await service.register({
      eventId: 'evt_1', email: 'neu@example.ch',
      name: 'Inge', fields: { mitbringsel: 'Salat' },
    });
    expect(profileRepo.save).toHaveBeenCalledOnce();
  });

  it('sendet Bestätigungs-E-Mail nach erfolgreicher Anmeldung', async () => {
    await service.register({
      eventId: 'evt_1', email: 'moritz@example.ch',
      name: 'Moritz', fields: { mitbringsel: 'Würste' },
    });
    expect(emailService.sendConfirmation).toHaveBeenCalledOnce();
  });

  it('sendet keine E-Mail wenn Spam erkannt', async () => {
    spamChecker.check.mockReturnValue({ status: 'abgelehnt', passed: false });
    await service.register({
      eventId: 'evt_1', email: 'spam@example.com',
      name: 'Spammer', fields: { mitbringsel: 'casino' },
    });
    expect(emailService.sendConfirmation).not.toHaveBeenCalled();
  });
});
```

---

## Teil 2: Playwright — E2E Tests

### Konfiguration

```javascript
// playwright.config.js
import { defineConfig } from '@playwright/test';

export default defineConfig({
  testDir: './tests/e2e',
  use: {
    baseURL: process.env.GAS_STAGING_URL, // Staging-Deployment-URL
    locale: 'de-CH',
    timezoneId: 'Europe/Zurich',
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'chromium', use: { browserName: 'chromium' } },
    { name: 'mobile',   use: { ...devices['Pixel 7'] } },
  ],
  reporter: [['html', { outputFolder: 'test-results' }]],
});
```

### Testdaten-Reset (kritisch für GAS)

Da Playwright gegen ein echtes GAS-Staging-Deployment testet, braucht es einen Reset-Mechanismus:

```javascript
// tests/e2e/helpers/reset.js
export async function resetTestData(request) {
  // Spezieller Admin-Endpoint nur im Staging aktiv:
  // ?action=test_reset&token=TEST_ADMIN_TOKEN
  // Löscht alle Einträge mit name="[TEST]..." aus dem Staging-Sheet
  await request.post(process.env.GAS_STAGING_URL, {
    data: { action: 'test_reset', token: process.env.TEST_ADMIN_TOKEN }
  });
}
```

### E2E Test: Anmeldungsflow (Hauptstrecke)

```javascript
// tests/e2e/registration.spec.js
import { test, expect } from '@playwright/test';
import { resetTestData } from './helpers/reset.js';

test.beforeEach(async ({ request }) => {
  await resetTestData(request);
});

test('Moritz meldet sich für BBQ an und erhält Bestätigungs-Link', async ({ page }) => {
  await page.goto('/');

  // Formular ausfüllen
  await page.fill('#name', '[TEST] Moritz');
  await page.fill('#mitbringsel', 'Würste und Brot');
  await page.fill('#hinweise', 'Bitte vegetarische Option für Tine');
  await page.fill('#email', 'test-moritz@quietschente.ch');

  // Warten: Mindest-Submit-Zeit > 2.5s (Spam-Filter)
  await page.waitForTimeout(3000);
  await page.click('#submit-btn');

  // Bestätigung sichtbar
  const confirmation = page.locator('.confirmation-card');
  await expect(confirmation).toBeVisible({ timeout: 15000 }); // GAS cold start
  await expect(confirmation).toContainText('Würste und Brot');

  // Persönlicher Edit-Link vorhanden
  const editLink = page.locator('.edit-link-url');
  await expect(editLink).toContainText('?token=');

  // Eintrag in Teilnehmerliste sichtbar
  await expect(page.locator('.entry-list')).toContainText('[TEST] Moritz');
});

test('Honeypot verhindert Bot-Anmeldung', async ({ page }) => {
  await page.goto('/');
  // Honeypot-Feld (versteckt) befüllen via JS
  await page.evaluate(() => {
    document.getElementById('website').value = 'http://spam.com';
  });
  await page.fill('#name', 'Bot');
  await page.fill('#mitbringsel', 'Spam');
  await page.waitForTimeout(3000);
  await page.click('#submit-btn');

  // Fake-Bestätigung (Honeypot zeigt Erfolg, speichert aber nicht)
  await expect(page.locator('.confirmation-card')).toBeVisible();

  // Eintrag NICHT in Teilnehmerliste
  await expect(page.locator('.entry-list')).not.toContainText('Bot');
});
```

### E2E Test: Edit/Delete via Token

```javascript
// tests/e2e/edit-delete.spec.js
test('Inge kann ihren Eintrag via Token bearbeiten', async ({ page, request }) => {
  // Anmeldung via API (nicht via UI — spart Zeit)
  const response = await request.post(process.env.GAS_STAGING_URL, {
    data: {
      action: 'anmelden',
      name: '[TEST] Inge',
      mitbringsel: 'Tiramisu',
      email: 'test-inge@quietschente.ch',
    }
  });
  const { token } = await response.json();

  // Edit-View via Token öffnen
  await page.goto(`/?token=${token}`);

  // Mitbringsel ändern
  await page.fill('#mitbringsel', 'Obstsalat');
  await page.click('#save-btn');

  await expect(page.locator('.success-toast')).toContainText('gespeichert');

  // Änderung in Teilnehmerliste sichtbar
  await page.goto('/');
  await expect(page.locator('.entry-list')).toContainText('Obstsalat');
  await expect(page.locator('.entry-list')).not.toContainText('Tiramisu');
});

test('Eintrag löschen entfernt ihn aus der Liste', async ({ page, request }) => {
  const response = await request.post(process.env.GAS_STAGING_URL, {
    data: { action: 'anmelden', name: '[TEST] ZuLöschen', mitbringsel: 'Salat' }
  });
  const { token } = await response.json();

  await page.goto(`/?token=${token}`);
  await page.click('#delete-btn');
  await page.click('#confirm-delete'); // Bestätigungsdialog

  await page.goto('/');
  await expect(page.locator('.entry-list')).not.toContainText('[TEST] ZuLöschen');
});
```

### E2E Test: Admin-Flow

```javascript
// tests/e2e/admin.spec.js
test('Admin moderiert verdächtigen Eintrag', async ({ page, request }) => {
  // URL-Spam-Eintrag erstellen (wird in_pruefung)
  await request.post(process.env.GAS_STAGING_URL, {
    data: { action: 'anmelden', name: '[TEST] Spammer', hinweise: 'Schau: http://spam.com', mitbringsel: 'nix' }
  });

  // Admin-Login
  await page.goto('/');
  await page.click('#admin-link');
  await page.fill('#admin-token-input', process.env.TEST_ADMIN_TOKEN);
  await page.click('#admin-login-btn');

  // Prüfungs-Tab
  await page.click('[data-tab="pruefung"]');
  const pendingEntry = page.locator('.pending-entry', { hasText: '[TEST] Spammer' });
  await expect(pendingEntry).toBeVisible();

  // Ablehnen
  await pendingEntry.locator('.reject-btn').click();
  await expect(pendingEntry).not.toBeVisible();

  // Nicht in öffentlicher Liste
  await page.goto('/');
  await expect(page.locator('.entry-list')).not.toContainText('[TEST] Spammer');
});
```

---

## Staging-Umgebung: Zwei parallele GAS-Deployments

```
┌───────────────────────────────────┐    ┌───────────────────────────────────┐
│  STAGING                          │    │  PRODUCTION                       │
│  Google Sheet: "Quietschente DEV" │    │  Google Sheet: "Quietschente PROD"│
│  GAS-Deployment: Staging-URL      │    │  GAS-Deployment: Prod-URL         │
│  Playwright-Tests laufen hier     │    │  Nie von Tests berührt            │
│  test_reset-Endpoint aktiv        │    │  test_reset-Endpoint deaktiviert  │
│  GAS_STAGING_URL=...              │    │  GAS_PROD_URL=...                 │
└───────────────────────────────────┘    └───────────────────────────────────┘
```

**Environment-Variablen lokal:**
```bash
# .env.test (nie committen!)
GAS_STAGING_URL=https://script.google.com/macros/s/STAGING_ID/exec
TEST_ADMIN_TOKEN=test-admin-secret-123
```

---

## CI/CD Pipeline (GitHub Actions)

```yaml
# .github/workflows/test.yml
name: Tests

on: [push, pull_request]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npm run test:unit
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: vitest-results
          path: test-results/

  e2e-tests:
    runs-on: ubuntu-latest
    needs: unit-tests          # Erst Unit, dann E2E
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: '20' }
      - run: npm ci
      - run: npx playwright install chromium
      - run: npm run build && clasp push --force  # Deploy zu Staging
        env:
          CLASP_TOKEN: ${{ secrets.CLASP_TOKEN }}
      - run: npm run test:e2e
        env:
          GAS_STAGING_URL: ${{ secrets.GAS_STAGING_URL }}
          TEST_ADMIN_TOKEN: ${{ secrets.TEST_ADMIN_TOKEN }}
      - uses: actions/upload-artifact@v4
        if: failure()
        with:
          name: playwright-report
          path: test-results/
```

---

## Was wird getestet, was nicht

### Immer Unit-testen (Vitest)

| Klasse | Kritische Tests |
|--------|----------------|
| `Email` | Validierung, Normalisierung, Gleichheit |
| `Token` | Länge, Zeichenraum, Eindeutigkeit |
| `EventPhase` | Erlaubte + verbotene Übergänge |
| `Registration` | Status-Übergänge (approve, flag, cancel) |
| `SpamCheckService` | Alle Regeltypen, Timing, Honeypot |
| `RegistrationService` | Happy path, Spam-Fall, E-Mail-Verhalten |

### Immer E2E-testen (Playwright)

| Journey | Warum E2E |
|---------|-----------|
| Anmeldung + Bestätigung | Vollständige User Journey inkl. GAS |
| Edit via Token | Token-URL-Verarbeitung im Browser |
| Delete + Listenaktualisierung | DOM-Reaktion auf API-Antwort |
| Spam-Filter sichtbar | Timing-abhängig, nur im Browser messbar |
| Admin-Moderation | Tab-Wechsel, Session-Logik, DOM-Updates |

### Bewusst nicht testen

- `GoogleSheetsRepository` Unit-Tests: zu aufwändig zu mocken, durch E2E abgedeckt
- `GasMailAppSender`: nicht testbar ohne GAS-Runtime (manuell prüfen oder E2E mit Mailbox-Check)
- Exakte E-Mail-Inhalte: reichen als Snapshot-Tests beim Template-Wechsel

---

## Zusammenfassung: TDD-Workflow für Quietschente

```
Neue Feature-Anforderung
        │
        ▼
1. Unit Test schreiben (Vitest)
   → Domain-Klasse oder Service-Methode
   → Test schlägt fehl (RED)
        │
        ▼
2. Minimale Implementierung in src/domain/ oder src/services/
   → Test besteht (GREEN)
        │
        ▼
3. Refactoring
   → Alle bestehenden Tests bleiben grün
        │
        ▼
4. E2E Test schreiben (Playwright)
   → User-sichtbares Verhalten gegen Staging
        │
        ▼
5. npm run build → clasp push (Staging)
   → E2E Test läuft gegen GAS
        │
        ▼
6. Alle Tests grün → clasp deploy (Production)
```
