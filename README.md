# Abfuhrkalender Detmold

Eine iOS-App (SwiftUI), die für eine ausgewählte Straße in Detmold den
**nächsten Abfuhrtermin** sowie eine Liste der **kommenden Termine** anzeigt.
Beim Öffnen der App wird automatisch der nächste Termin angezeigt – inklusive
Abfallart, Farbe und Symbol.

## Funktionen

- **Straßenauswahl** mit Suche – die Straßenliste wird direkt von der Detmolder
  Webseite geladen.
- **Manueller Fallback**: Falls die Straßenliste nicht geladen werden kann, lässt
  sich die Straßen-ID (`strid`) manuell eingeben.
- **Nächster Termin groß**: „Heute“ / „Morgen“ / „in N Tagen“ mit Abfallart(en).
- **Liste „Demnächst“** für die weiteren kommenden Termine.
- **Automatische Aktualisierung** beim App-Start (`scenePhase == .active`),
  aber nur, wenn der lokale Cache älter als ~6 Stunden ist.
- **Offline-fähig**: Termine werden lokal gecached und sofort beim Start angezeigt.
- **Jahreswechsel-sicher**: Es werden das aktuelle und das nächste Jahr geladen,
  damit auch um den Jahreswechsel der nächste Termin korrekt ist.

## Abfallarten

| Art            | Farbe  | Symbol                  |
|----------------|--------|-------------------------|
| Restmüll       | grau   | `trash.fill`            |
| Bioabfall      | braun  | `leaf.fill`             |
| Papier/Pappe   | blau   | `newspaper.fill`        |
| Gelber Sack    | gelb   | `arrow.3.trianglepath`  |
| Weihnachtsbaum | grün   | `tree.fill`             |
| Sonstiges      | grau   | `trash.fill`            |

Die Klassifizierung erfolgt anhand des Roh-Labels aus dem Kalender
(siehe `WasteType.classify(_:)` in `Models.swift`).

## Datenquelle

Die Stadt Detmold bietet pro Straße einen iCal-Export:

```
https://abfuhrkalender.detmold.de/icsmaker.php?strid=<ID>&year=<JAHR>
```

- `strid` = numerische Straßen-ID, `year` = Jahr (z.B. 2026)
- Rückgabe: Standard-iCalendar (`.ics`), ein `VEVENT` pro Abfuhr
- Jedes `VEVENT` enthält `DTSTART` (Datum, meist `VALUE=DATE` `YYYYMMDD`) und
  `SUMMARY` der Form `Müllabfuhr: Restmüll` (das Präfix `Müllabfuhr:` wird
  abgeschnitten).

Referenz: Home-Assistant-Integration
[`mampfes/hacs_waste_collection_schedule`](https://github.com/mampfes/hacs_waste_collection_schedule),
Dokumentation `doc/ics/detmold_de.md` (generische ICS-Source, Beispiel `strid=146`).

> **Hinweis:** `abfuhrkalender.detmold.de` blockt Rechenzentrums-IPs/Bots mit
> HTTP 403. Auf einem echten iPhone (Mobilfunk/WLAN) funktionieren die Requests.
> Die App sendet einen Browser-User-Agent mit, um 403 auf echten Geräten zu
> vermeiden. **Aus einer Cloud-/CI-Umgebung lässt sich die Seite nicht abrufen
> oder testen.**

## Straßen-ID (strid) selbst herausfinden

Falls die automatische Straßenliste nicht lädt:

1. [abfuhrkalender.detmold.de](https://abfuhrkalender.detmold.de) öffnen
2. Straße auswählen
3. „Download ics-Datei (iCal)“ wählen
4. In der URL die Zahl hinter `strid=` übernehmen, z.B. `…?strid=146&…` → `146`
5. In der App oben rechts in der Straßenauswahl auf das `#`-Symbol tippen und
   die Zahl eingeben.

## Projekt bauen

Das Xcode-Projekt wird mit [XcodeGen](https://github.com/yonyon/XcodeGen) aus
`project.yml` erzeugt. Die generierte `.xcodeproj` wird **nicht** eingecheckt.

```bash
# XcodeGen installieren (falls noch nicht vorhanden)
brew install xcodegen

# Projekt generieren
cd Abfuhrkalender   # Repo-Wurzel mit project.yml
xcodegen generate

# Projekt öffnen
open Abfuhrkalender.xcodeproj
```

Anschließend in Xcode ein Gerät/einen Simulator wählen und mit ⌘R starten.

### Projekt-Eckdaten

- **Deployment Target:** iOS 16.0
- **Ausrichtung:** Portrait
- **Bundle-ID:** `de.OliverBeine.Abfuhrkalender`
- **Product Name:** `Abfuhrkalender`
- **Development Team:** `B96YX3A33W`
- **Code Signing:** Automatic
- **Architektur:** SwiftUI, `async/await`, eigener ICS-Parser ohne Abhängigkeiten

## Projektstruktur

```
project.yml                              XcodeGen-Projektdefinition
.gitignore                               ignoriert .xcodeproj, Info.plist, Xcode-Output
README.md                                diese Datei
Abfuhrkalender/
  AbfuhrkalenderApp.swift                App-Entry, Auto-Refresh bei .active
  Models.swift                           Street, WasteType, CollectionEvent
  ICSParser.swift                        DTSTART/SUMMARY-Parser, RFC-5545 Unfolding
  DetmoldService.swift                   Straßenliste + ICS laden (Browser-UA)
  ScheduleStore.swift                    @MainActor Store, Cache, nextDate/upcoming
  ContentView.swift                      nächster Termin groß + Liste (refreshable)
  SetupView.swift                        Straßensuche + manueller strid-Fallback
  Assets.xcassets/                       AccentColor (grünlich), AppIcon-Platzhalter
```

## Hinweis zur Entwicklung

Diese App wurde so geschrieben, dass sie sauber kompiliert. In Umgebungen ohne
macOS/Xcode lässt sich das Projekt nicht bauen; der Code ist jedoch
plattformkonform (iOS 16, SwiftUI) und ohne externe Abhängigkeiten gehalten.
