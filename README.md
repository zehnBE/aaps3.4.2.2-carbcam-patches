# CarbCam Patches für DIY Closed-Loop Apps

Patchfiles für die Integration von [CarbCam.app](https://carbcam.app) in DIY Closed-Loop Insulin-Delivery Apps. Die Patches ergänzen jeweils einen schlanken, definierten Eingang (Android Intent oder URL-Scheme), über den CarbCam den geschätzten Kohlenhydrat-Wert – und wo die App das unterstützt auch Fett/Eiweiß/Ballaststoffe – in die native Carb-Entry-UI der App überträgt. **Die App zeigt danach ihren Standard-Bestätigungsdialog**; die Freigabe erfolgt immer manuell durch den User. Kein stiller Bolus.

Details/Anleitung/Blog: https://ns.10be.de/blog/2026/06/01/carbcam-kohlenhydrate-direkt-in-den-aaps-bolus-wizard-uebernehmen

---

## Was welche App annimmt

| App | Plattform | value (KH) | fett | eiweiss | fiber | notes | source | Kanal |
|---|---|:-:|:-:|:-:|:-:|:-:|:-:|---|
| AAPS 3.4.2.2 / 3.4.2.6 | Android / Kotlin | ✓ | – | – | – | ✓ | ✓ | Android Intent |
| AAPS V4 (dev) | Android / Kotlin (Compose) | ✓ | – | – | – | ✓ | ✓ | Android Intent |
| **iAPS main (8.3.4)** | iOS / Swift | ✓ | **✓** | **✓** | **✓** | ✓ | ✓ | `carbcam-iaps://` |
| **iAPS dev** | iOS / Swift | ✓ | **✓** | **✓** | **✓** | ✓ | ✓ | `carbcam-iaps://` |
| Trio | iOS / Swift | ✓ | ✓ | ✓ | – | ✓ | ✓ | `carbcam-trio://` |
| Loop | iOS / Swift | ✓ | – | – | – | ✓ | ✓ | `carbcam-loop://` |

Loop und AAPS haben in ihrer Carb-Entry-UI **keine** Fett/Eiweiß-Felder. Die Patches ignorieren die zusätzlichen URL-Parameter dort still. CarbCam kann die URL trotzdem einheitlich mit allen Feldern bauen – jede App nimmt nur was sie kennt.

---

## Aktuelle Patchfiles (Stand 29.09.2026)

### AAPS 3.4.2.2

- File: `0001-aaps-3.4.2.2-carbcam-integration.patch`
- Base: `nightscout/AndroidAPS` tag `3.4.2.2`
- zehnBE-Branch: `3.4.2.2-carbcam`
- Bolus-Wizard wird über `info.nightscout.androidaps.action.OPEN_BOLUS_WIZARD` mit `carbs`, `notes`, `source` Extras geöffnet
- Whitelist auf `de.be10.carbcam` (+ Debug-Varianten)

### AAPS 3.4.2.6 (aktuelle stable)

- File: `0001-aaps-3.4.2.6-carbcam-integration.patch`
- Base: `nightscout/AndroidAPS` tag `3.4.2.6`
- zehnBE-Branch: `3.4.2.6-carbcam`
- Struktur identisch zu 3.4.2.2

### AAPS V4 (dev, Compose)

- File: `0001-aaps-v4-dev-carbcam-integration.patch`
- Base: `nightscout/AndroidAPS` dev HEAD `cdc1c670f6`
- zehnBE-Branch: `v4-dev-carbcam`
- Nutzt existierende `AppRoute.WizardDialog.createRoute(carbs, notes)` — kein Patch am ViewModel/Screen nötig

### iAPS 8.3.4 (main)

- File: `0001-iaps-8.3.4-carbcam-integration.patch`
- Base: `Artificial-Pancreas/iAPS` main HEAD `005706209`
- URL: `carbcam-iaps://carbs?value=24&fat=20&protein=19&fiber=3&notes=Pizza&source=carbcam`

### iAPS dev

- File: `0001-iaps-dev-carbcam-integration.patch`
- Base: `Artificial-Pancreas/iAPS` dev HEAD `ac4e593cc`
- URL: identisch zu main

### Trio

- File: `0001-trio-carbcam-integration.patch`
- Base: `nightscout/Trio` main (Juni 2026 Stand)
- URL: `carbcam-trio://carbs?value=42&fat=12&protein=8&notes=Pizza&source=carbcam`

### Loop 3.14.8 (mit iOS 27 SDK Fixes)

- File: `0001-loop-3.14.8-carbcam-integration.patch`
- Base: `LoopKit/Loop` dev commit `c2fddb76` (v3.14.8, inkl. iOS 27 SDK Fixes)
- URL: `carbcam-loop://carbs?value=42&notes=Pizza&source=carbcam`
- Patch appliert auf den frischsten Loop-Stand — behebt gleichzeitig den iPhone Air / iOS 26.5.1 Crash aus früheren Builds (der war upstream-verursacht, ist jetzt raus)

---

## Historische Patchfiles (Juni 2026, deprecated)

Die folgenden Patches sind gegen ältere Upstream-Bases erstellt und deshalb nicht mehr aktuell — bleiben im Repo als Referenz:

- `0001-aaps-carbcam-3.4.2.2.patch` — Juni 2026, appliert weiterhin gegen 3.4.2.2 Tag
- `0001-aaps-v4-carbcam-integration.patch` — Juni 2026, Base `7734facd` (viel älter als aktueller dev)
- `0001-iaps-carbcam-integration.patch` — Juni 2026, veraltet nach Upstream-Umbauten (Micronutrient-Refactor)
- `0001-loop-carbcam-integration.patch` — Juni 2026, gegen Loop v3.14.0 (der Base mit iPhone Air Crash)

Für neue Installationen: die aktuellen Patchfiles oben verwenden.

---

## Anwendung

```bash
cd <app-repo>
git checkout <target-branch-oder-tag>
git am pfad/zur/datei/0001-xxxxxxxxxx.patch
```

Falls der Patch nicht sauber appliert (weil Upstream inzwischen weitergezogen ist):

```bash
git am --abort
git am --3way pfad/zur/datei/0001-xxxxxxxxxx.patch
```

Bei starken Divergenzen: Issue aufmachen oder Kontakt an support@ns.10be.de.

---

## Sicherheits-Design

Für alle Patches identisch angelegt:

1. **Range-Check** — `value` (Carbs) clamped auf 1..80 g. Fett/Eiweiß/Fiber auf 0..80 g. Werte außerhalb → still ignoriert, kein Crash.
2. **String-Limits** — `notes` auf 200 Zeichen, `source` auf 50 Zeichen.
3. **Keine automatische Insulinabgabe** — Patch öffnet nur die native Carb-Entry-UI; User muss im App-eigenen Dialog bestätigen.
4. **Whitelist (AAPS)** — Nur `de.be10.carbcam` und die Debug-Variante dürfen den Wizard extern öffnen.
5. **Source-Anzeige** — Der `source`-Parameter erscheint für den User sichtbar (Notes-Feld), wird aber nicht als Vertrauensbeweis behandelt.

---

## Session 29.09.2026 — was neu ist

- **AAPS 3.4.2.2**: bestätigt gegen originalen Tag, neuer Patchfile-Name
- **AAPS 3.4.2.6**: neu — neueste stable 3.4.x Version
- **AAPS V4 (dev)**: neu gebaut gegen aktuellen dev HEAD (10.266 Commits nach dem alten Juni-Stand), Compose Multiplatform Migration berücksichtigt
- **iAPS main + dev**: neu gebaut nach Upstream-Umbauten (67 dev-Commits, 12 main-Commits), Micronutrient-Refactor integriert, jetzt zusätzlich mit **Fett/Eiweiß/Ballaststoffe** Support
- **Loop**: neu gebaut auf v3.14.8 Base (LoopKit dev), enthält alle iOS 27 SDK Fixes + Xcode 27 Fixes — behebt gleichzeitig den iPhone Air / iOS 26.5.1 App-Start-Crash aus Juni-Builds
- **Trio**: unverändert

---

## Feedback und Support

- Blog: https://ns.10be.de/blog/2026/06/01/carbcam-kohlenhydrate-direkt-in-den-aaps-bolus-wizard-uebernehmen
- Kontakt: support@ns.10be.de
- CarbCam-App: https://carbcam.app
