# Intelligent Heating Control (IHC)

[![HACS Custom](https://img.shields.io/badge/HACS-Custom-orange.svg)](https://hacs.xyz/)
[![HA Version](https://img.shields.io/badge/HA-2023.6%2B-blue.svg)](https://home-assistant.io)

Eine fortschrittliche Heizungssteuerung für Home Assistant mit allen Funktionen von Advanced Heating Control v5, erweitert um:

1. **Außentemperaturgeführte Raumtemperatur** (Heizkurve) mit einstellbarem Offset je Zimmer
2. **Tag/Nacht-Zeitpläne** mit eigenem Offset je Zeitraum
3. **Loxone-artiger Klimabaustein** – aggregiert alle Zimmeranforderungen und entscheidet zentral ob die Heizung läuft

---

## Funktionen

### Klimabaustein (Loxone-Style)
Das Herzstück des Plugins. Jedes Zimmer berechnet seine Heizanforderung (0–100 %) basierend auf der Temperaturdifferenz zwischen Ist und Soll. Alle Anforderungen werden **gewichtet aggregiert**. Erst wenn die Gesamtanforderung einen konfigurierbaren Schwellenwert überschreitet, schaltet die Heizung ein.

```
Gesamtanforderung = Σ(Zimmeranforderung × Gewichtung) / Σ(Gewichtung)
Heizung EIN:  wenn Gesamtanforderung ≥ Einschaltschwelle
Heizung AUS:  wenn Gesamtanforderung < (Einschaltschwelle - Hysterese)
```

**Vorteile:**
- Verhindert Kurzläufe (Mindest-Ein/Aus-Zeit)
- Hysterese verhindert schnelles Takten
- Mindestanzahl Zimmer als zusätzliche Bedingung
- Gewichtung ermöglicht priorisierte Zimmer (z.B. Wohnzimmer × 2)

### Außentemperaturgeführte Regelung (Heizkurve)
Die Heizkurve mappt die Außentemperatur auf eine **Basis-Solltemperatur**. Je kälter draußen, desto höher die Heizzieltemperatur:

```
-20°C Außen → 24°C Basis-Ziel
  0°C Außen → 22°C Basis-Ziel
 15°C Außen → 19.5°C Basis-Ziel
 25°C Außen → 16°C Basis-Ziel
```

Jedes Zimmer hat einen **individuellen Offset** (z.B. +1.5°C für Wohnzimmer, -0.5°C für Schlafzimmer).

```
Zimmer-Zieltemperatur = Heizkurven-Basis + Zimmer-Offset
```

### Zeitpläne mit Offset
Zeitpläne überschreiben die Heizkurven-Temperatur für definierte Zeiträume:

```
Zeitplan-Zieltemperatur = Zeitplan-Temperatur + Zeitplan-Offset + Zimmer-Offset
```

**Beispiel:**
- Montag–Freitag 06:30–08:00: 22°C + 0°C Offset = 22°C (morgendliche Aufheizphase)
- Montag–Freitag 17:00–22:30: 21.5°C + 0.5°C Offset = 22°C (Abendkomfort)
- Wochenende 08:00–23:00: 21°C + 0°C Offset = 21°C

Außerhalb der Zeitpläne wird die **Heizkurve** verwendet.

### Betriebsmodi

| Systemmodus | Beschreibung |
|-------------|-------------|
| Automatisch | Zeitplan + Heizkurve aktiv |
| Heizen | Erzwingt Heizmodus |
| Kühlen | Erzwingt Kühlmodus |
| Aus | Heizung deaktiviert |
| Abwesend | Feste Abwesend-Temperatur für alle Zimmer |
| Urlaub | Reduzierte Frostschutz-Temperatur |

| Zimmermodus | Beschreibung |
|-------------|-------------|
| Automatisch | Zeitplan + Heizkurve |
| Komfort | Feste Komfort-Temperatur |
| Eco | Reduzierte Eco-Temperatur |
| Schlafen | Schlafzimmer-Temperatur |
| Abwesend | Zimmer-Abwesenheitstemperatur |
| Manuell | Manuell eingestellte Temperatur |
| Aus | Zimmer ausgeschaltet (keine Anforderung) |

### Weitere Funktionen (aus Advanced Heating Control v5)
- ✅ Fenster-offen-Erkennung (Binärsensor oder Temperatursturz-Detektion)
- ✅ Direkte Thermostat/TRV-Steuerung (climate-Entities)
- ✅ Min/Max Temperaturbegrenzungen pro Zimmer
- ✅ Zimmergewichtung für Klimabaustein
- ✅ Komfort / Eco / Schlaf / Abwesend Presets
- ✅ Mindest-Ein/Aus-Zeiten (Kesselschutz)
- ✅ Kühlungsunterstützung (optional)

---

## Installation

### HACS (empfohlen)
1. HACS → Integrationen → ⋮ → Benutzerdefinierte Repositories
2. URL: `https://github.com/Jedrimos/intelligent-heatingcontroll`
3. Kategorie: Integration
4. `Intelligent Heating Control` installieren
5. Home Assistant neu starten

### Manuell
1. Ordner `custom_components/intelligent_heating_control` in dein HA-Konfigurationsverzeichnis kopieren
2. HA neu starten

---

## Konfiguration

### Initiale Einrichtung
1. **Einstellungen → Integrationen → + Integration hinzufügen** → "Intelligent Heating Control"
2. Außentemperatursensor auswählen
3. Heizungsschalter auswählen (Switch/Input Boolean der den Kessel/die Pumpe steuert)
4. Klimabaustein-Parameter konfigurieren
5. Globale Temperaturen (Abwesend, Urlaub) einstellen

### Zimmer hinzufügen
- Über **Einstellungen → Integrationen → IHC → Konfigurieren → Zimmer hinzufügen**
- Oder im **IHC Panel** (Seitenleiste) → Konfiguration → Zimmer hinzufügen
- Oder per HA-Service: `intelligent_heating_control.add_room`

### Heizkurve anpassen
Im IHC Panel → Tab "Heizkurve": Punkte anpassen und graphisch prüfen.

### Zeitpläne erstellen
Im IHC Panel → Tab "Zeitpläne": Wochentage und Zeiträume pro Zimmer konfigurieren.

---

## Erstellte Entitäten

### Pro Zimmer
| Entität | Typ | Beschreibung |
|---------|-----|-------------|
| `climate.ihc_<zimmer>` | Climate | Hauptentität mit Ist/Soll-Temp, Presets |
| `sensor.ihc_<zimmer>_demand` | Sensor | Heizanforderung 0–100 % |
| `sensor.ihc_<zimmer>_target` | Sensor | Berechnete Zieltemperatur |
| `number.ihc_<zimmer>_offset` | Number | Zimmer-Offset (laufzeit-anpassbar) |
| `select.ihc_<zimmer>_mode` | Select | Zimmermodus-Auswahl |

### Global
| Entität | Typ | Beschreibung |
|---------|-----|-------------|
| `sensor.ihc_gesamtanforderung` | Sensor | Gewichtete Gesamtanforderung % |
| `sensor.ihc_aussentemperatur` | Sensor | Außentemperatur (Spiegel) |
| `sensor.ihc_heizkurven_zieltemperatur` | Sensor | Aktueller Heizkurven-Basiswert |
| `switch.ihc_heizung_aktiv` | Switch | Heizungsstatus (schalt-/lesbar) |
| `select.ihc_systemmodus` | Select | Globaler Systemmodus |

---

## Services

```yaml
# Zimmer hinzufügen
service: intelligent_heating_control.add_room
data:
  name: "Wohnzimmer"
  temp_sensor: sensor.wohnzimmer_temp
  valve_entity: climate.wohnzimmer_thermostat
  room_offset: 1.5
  comfort_temp: 22.0
  eco_temp: 18.0

# Systemmodus setzen
service: intelligent_heating_control.set_system_mode
data:
  mode: away  # auto, heat, cool, off, away, vacation

# Zimmermodus setzen
service: intelligent_heating_control.set_room_mode
data:
  id: "abc12345"
  mode: eco  # auto, comfort, eco, sleep, away, off, manual
```

---

## IHC Panel (Frontend)

Das Plugin erstellt ein eigenes Panel in der HA-Seitenleiste:

- **📊 Übersicht & Debug**: Echtzeit-Ansicht aller Zimmer mit Anforderungen, Temperaturen, Klimabaustein-Status
- **⚙️ Konfiguration**: Zimmer verwalten, Systemmodus, Klimabaustein-Parameter
- **📅 Zeitpläne**: Wöchentliche Zeitpläne per Zimmer mit grafischem Wochenplaner
- **📈 Heizkurve**: Heizkurven-Editor mit Live-Vorschau und aktuellem Außentemperatur-Marker

---

## Architektur

```
intelligent_heating_control/
├── __init__.py          # Integration setup, Services, Panel-Registrierung
├── manifest.json        # HA-Manifest
├── const.py             # Konstanten
├── config_flow.py       # Einrichtungs- und Options-Flow
├── coordinator.py       # Haupt-Datenkoordinator (Update-Zyklus)
├── heating_curve.py     # Heizkurven-Logik (lineare Interpolation)
├── schedule_manager.py  # Zeitplan-Verwaltung
├── heating_controller.py # Klimabaustein (Anforderungs-Aggregation)
├── climate.py           # Climate-Platform
├── sensor.py            # Sensor-Platform
├── switch.py            # Switch-Platform
├── number.py            # Number-Platform (Offsets)
├── select.py            # Select-Platform (Modi)
├── services.yaml        # Service-Definitionen
├── translations/
│   ├── de.json          # Deutsch
│   └── en.json          # Englisch
└── frontend/
    └── ihc-panel.js     # Custom Panel (Vanilla JS)
```

---

## Lizenz

MIT License – siehe [LICENSE](LICENSE)

## Mitwirkende

- [@Jedrimos](https://github.com/Jedrimos) – Projektinitiator und Hauptentwickler

---

> **Hinweis:** Dieses Plugin befindet sich in aktiver Entwicklung. Bitte Bugs und Feature-Wünsche als [GitHub Issue](https://github.com/Jedrimos/intelligent-heatingcontroll/issues) melden.
