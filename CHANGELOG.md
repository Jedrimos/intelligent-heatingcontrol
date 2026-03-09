# Changelog

Alle nennenswerten Änderungen an diesem Projekt werden hier dokumentiert.

Das Format basiert auf [Keep a Changelog](https://keepachangelog.com/de/1.0.0/),
und dieses Projekt folgt [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [1.0.0] - 2026-03-09

### Erstveröffentlichung

#### Hinzugefügt

##### Kernfunktionen
- **Heizkurve (Außentemperaturgeführte Regelung)**
  - Lineare Interpolation zwischen konfigurierbaren Kurven-Punkten
  - Bis zu 7 Stützpunkte für die Kurve
  - Automatischer Rückfall auf Komforttemperatur wenn kein Außensensor verfügbar
- **Zimmer-Offset per Raum**
  - Jedes Zimmer hat einen individuellen ±5°C Offset zur Heizkurven-Basistemperatur
  - Laufzeit-anpassbar via `number.ihc_<zimmer>_offset` Entität
- **Zeitpläne mit Offset**
  - Wöchentliche Zeitpläne mit beliebig vielen Tagesgruppen und Zeiträumen
  - Pro Zeitraum: eigene Temperatur + eigener Offset
  - Formel: `Ziel = Zeitplan-Temp + Zeitplan-Offset + Zimmer-Offset`
  - Übernacht-Zeiträume unterstützt (z.B. 22:00–06:00)

##### Klimabaustein (Loxone-Style)
- Proportionale Anforderungsberechnung pro Zimmer (0–100 %)
- Gewichteter Durchschnitt aller Zimmeranforderungen
- Konfigurierbare Einschaltschwelle (Standard: 15 %)
- Hysterese für stabilen Betrieb (Standard: 5 %)
- Mindest-Einschaltzeit und Mindest-Ausschaltzeit (Kesselschutz)
- Mindestanzahl Zimmer mit Anforderung als Bedingung

##### Betriebsmodi (aus AHC v5)
- **Systemmodi**: Automatisch, Heizen, Kühlen, Aus, Abwesend, Urlaub
- **Zimmermodi**: Automatisch, Komfort, Eco, Schlafen, Abwesend, Aus, Manuell
- Temperatur-Presets pro Zimmer: Komfort, Eco, Schlaf, Abwesend
- Globale Abwesend- und Urlaubstemperatur

##### Zimmersteuerung
- Temperatursensor-Integration pro Zimmer
- Direkte Thermostat/TRV-Steuerung (climate-Entities)
- Fenster-offen-Erkennung via Binärsensor
- Min/Max Temperaturgrenzen pro Zimmer
- Zimmergewichtung für Klimabaustein

##### Home Assistant Integration
- Config Flow (Schritt-für-Schritt Einrichtung)
- Options Flow (Zimmer hinzufügen/bearbeiten/entfernen, Heizkurve, Einstellungen)
- 5 HA-Platforms: climate, sensor, switch, number, select
- Deutschsprachige Übersetzungen (de.json)
- Englischsprachige Übersetzungen (en.json)
- HA-Services: add_room, remove_room, update_room, set_room_mode, set_system_mode

##### Frontend Panel
- Eigenes Panel in der HA-Seitenleiste (IHC)
- **Tab "Übersicht & Debug"**: Echtzeit-Ansicht aller Zimmer, Klimabaustein-Status
- **Tab "Konfiguration"**: Zimmer verwalten, Systemmodus, Parameter
- **Tab "Zeitpläne"**: Wöchentlicher Zeitplan-Editor pro Zimmer
- **Tab "Heizkurve"**: Kurven-Editor mit Canvas-Vorschau und Außentemperatur-Marker
- Auto-Refresh alle 5 Sekunden im Debug-Tab
- Toast-Benachrichtigungen für Aktionen

---

## Geplante Versionen (Roadmap)

Siehe [ROADMAP.md](ROADMAP.md) für Details zu geplanten Funktionen.
