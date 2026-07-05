# Plan: Automatisiertes Testing mit Playwright

## Context

Die InnoProved-Website ist eine statische HTML/CSS/JS-Seite (kein Framework, kein npm) mit 5 Seiten und einem Admin-Bereich. Das Backend läuft über Google Apps Script. Bisher existiert nur eine manuelle Testdokumentation (`tests/responsive-design-testfaelle.md`). Ziel ist es, Regression nach Änderungen effizient und zuverlässig zu prüfen — ohne jede Seite manuell durchzuklicken.

**Wichtig:** Playwright läuft vollständig lokal auf dem Entwicklungsrechner. Auf dem Hostpoint-Server wird nichts installiert oder verändert.

## Empfehlung: Playwright E2E-Tests

**Playwright** ist die beste Wahl für dieses Projekt:
- Funktioniert mit statischen HTML-Seiten ohne Framework
- Kann die Seite lokal über einen eingebauten Webserver starten
- Unterstützt Viewport-Tests für Responsive Design (passt zu den vorhandenen Testfällen)
- API-Aufrufe zum GAS-Backend können gemockt werden
- Gute VS Code Integration (Playwright Extension)
- Einzige Anforderung: Node.js installieren

## Umsetzungsschritte

### 1. Node.js + npm initialisieren
```bash
npm init -y
npm install --save-dev @playwright/test
npx playwright install chromium  # reicht für Basics
```

### 2. Playwright konfigurieren (`playwright.config.js`)
- Webserver: `npx serve .` (oder `http-server`) auf Port 3000 starten
- Viewports: 375px (Mobile), 768px (Tablet), 1280px (Desktop) — aus bestehender Testdokumentation
- Browser: Chromium (+ optional Firefox/Safari)
- Testordner: `tests/e2e/`

### 3. Teststruktur aufbauen (bestehende Testfälle automatisieren)

```
tests/
├── e2e/
│   ├── navigation.spec.js      # Hamburger-Menü, Links
│   ├── workshops.spec.js       # Tabelle, Accordion/Modal, Buchung
│   ├── events.spec.js          # Events-Liste
│   ├── kontakt.spec.js         # Formular-Validierung
│   ├── profil.spec.js          # Profil-Seite
│   └── admin.spec.js           # Login, CRUD
└── responsive-design-testfaelle.md  # (bestehendes Dokument, bleibt als Referenz)
```

### 4. GAS-Backend mocken
Da `api.js` Fetch-Aufrufe an Google Apps Script macht, werden diese in Tests per `page.route()` gemockt — so laufen Tests ohne live Backend:
```js
await page.route('**/macros/**', route => route.fulfill({ json: mockData }));
```

### 5. npm-Scripts definieren
```json
"scripts": {
  "test": "playwright test",
  "test:ui": "playwright test --ui",  // visueller Debugger
  "test:headed": "playwright test --headed"
}
```

## Kritische Dateien

| Datei | Aktion |
|-------|--------|
| `playwright.config.js` | neu erstellen |
| `package.json` | neu erstellen |
| `tests/e2e/*.spec.js` | neu erstellen (eine Datei pro Seite) |
| `tests/responsive-design-testfaelle.md` | als Vorlage für Testfälle verwenden |
| `js/api.js` | lesen, um Mock-URLs korrekt zu setzen |

## Optionaler nächster Schritt: GitHub Actions

Nach lokaler Einrichtung kann ein `.github/workflows/test.yml` hinzugefügt werden, der bei jedem Push automatisch die Tests ausführt — sodass Fehler vor dem Deployment auf Hostpoint erkannt werden.

## Verifikation

1. `npm test` ausführen → alle Tests grün
2. Eine CSS-Änderung machen, die Navigation kaputt bricht → Test schlägt an
3. Mit `npm run test:ui` einzelne Tests visuell debuggen
