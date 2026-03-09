# Roadmap – Intelligent Heating Control

Hier sind alle geplanten Verbesserungen und Ideen für zukünftige Versionen dokumentiert.

---

## Version 1.1 – Intelligentere Regelung

### Adaptive Heizkurve (Auto-Learning)
- Die Heizkurve lernt automatisch aus vergangenen Daten
- Wenn ein Zimmer trotz Kurventemperatur zu kalt/warm ist, wird die Kurve angepasst
- Konfigurierbare Lernrate und Lerngrenze
- Protokoll der Kurvenanpassungen

### Predictive Pre-Heating (Vorausschauendes Vorheizen)
- Analysiert historische Aufheizzeiten pro Zimmer
- Berechnet, wann geheizt werden muss, damit die Zieltemperatur zum geplanten Zeitpunkt erreicht wird
- Beispiel: "Wohnzimmer braucht 45 min zum Aufheizen → Heizung um 16:15 starten für 17:00 Ziel"
- Integration mit Weather-Forecast (Wettervorhersage-Sensor für Vorausplanung)

### Temperaturverlauf-Analyse
- Speicherung der letzten 7 Tage Temperaturverläufe pro Zimmer
- Erkennung von Anomalien (plötzliche Abkühlung = Fenster offen?)
- Mini-Graphen im Debug-Tab

---

## Version 1.2 – Präsenzbasierte Steuerung

### Personenbasierte Präsenz
- Mehrere Personen pro Haushalt konfigurierbar
- Pro Person: Anwesenheit via device_tracker, GPS-Zone oder manuell
- "Alle abwesend" → System in Abwesend-Modus
- "Erste Person kommt nach Hause" → Vorheizbeginn X Minuten vor Ankunft (ETA-basiert)

### Zimmer-spezifische Anwesenheit
- z.B. Büro-Zimmer nur heizen wenn Person im Homeoffice
- Schlafzimmer tagsüber in Eco-Modus wenn niemand dort schläft
- Integration mit HA Presence Detection (bayesian sensor, etc.)

### Nachtabsenkung
- Automatische Nachtabsenkung basierend auf Sonnenuntergang/-aufgang
- Offset zur normalen Nacht-Zeitplan-Temperatur

---

## Version 1.3 – Energieoptimierung

### Energieverbrauchsübersicht
- Tracking der Heizlaufzeiten pro Zimmer und gesamt
- Geschätzter Energieverbrauch (konfigurierbare kW-Leistung des Kessels)
- Tages-/Wochen-/Monatsauswertungen
- Vergleich mit Vorperiode
- Export als HA-Sensor für externe Energieüberwachung (Energy Dashboard)

### Dynamische Tarif-Integration
- Integration mit Energiepreis-Sensoren (Tibber, Octopus, ENTSO-E)
- Heizung bevorzugt günstiger Stromstunden (für Wärmepumpen)
- Vorheizbetrieb wenn Strom günstig, Absenkbetrieb bei teuren Stunden
- Konfigurierbare Preisgrenze

### Solarüberschuss-Heizung
- Integration mit Solar-Überschuss-Sensoren
- Erhöhung der Zieltemperatur wenn Solarüberschuss vorhanden
- Puffer aufheizen wenn Sonne scheint

---

## Version 1.4 – Erweiterte Raumsteuerung

### Mehrzone-Heizkreis-Unterstützung
- Separate Heizkreise (z.B. Fußbodenheizung vs. Heizkörper)
- Pro Zimmer: Zuweisung zu einem Heizkreis
- Separate Pumpen/Ventile pro Heizkreis
- Klimabaustein pro Heizkreis

### Hydraulischer Abgleich (Hilfsfunktion)
- Assistent für den hydraulischen Abgleich von Heizkörpern
- Messung der Aufheizzeiten und Vergleich zwischen Zimmern
- Vorschläge für Ventilvoreinstellungen

### Vorlauftemperaturregelung
- Steuerung der Vorlauftemperatur des Kessels (Modbus, OpenTherm)
- Berechnung der benötigten Vorlauftemperatur aus Außentemperatur und Anforderung
- Witterungsgeführte Vorlauftemperatur (Heizkurve für Vorlauf)

### Raumfühler-Kalibrierung
- Offset-Kalibrierung für ungenaue Sensoren direkt in IHC
- Vergleich mit Referenzthermometer (manuell eingebbar)
- Automatische Drift-Korrektur über Zeit

---

## Version 1.5 – Erweiterte UI / UX

### Mobile-optimiertes Dashboard
- Optimierte Darstellung für Mobilgeräte
- Swipe-Gesten für Tab-Navigation
- Push-Benachrichtigungen für wichtige Events (Fenster offen vergessen, Frost-Alarm)

### Visualisierungen
- Temperaturverlauf-Graph pro Zimmer (letzte 24h)
- Heizkurven-Simulation: "Was wäre wenn die Außentemperatur X wäre?"
- Zeitplan-Kalenderansicht (Wochenüberblick aller Zimmer)
- Anforderungs-Heatmap (welche Zimmer wann heizen)

### Konfigurationsassistent
- Geführter Setup-Assistent für neue Benutzer
- Automatische Erkennung von Klimaentitäten im System
- Vorschläge für Heizkurven-Einstellungen basierend auf Gebäudetyp

### Backup & Restore
- Vollständiger Export der Konfiguration als JSON/YAML
- Import-Funktion für Konfigurationsübertragung auf neues System
- Automatische Konfigurations-Backups

---

## Version 2.0 – AI-Integration

### KI-basierte Temperaturvorhersage
- Machine Learning Modell (lokales TFLite oder externen API)
- Vorhersage der benötigten Heizenergie basierend auf:
  - Wettervorhersage (Temperatur, Wind, Sonne)
  - Belegungsmustern (Anwesenheit der letzten Wochen)
  - Historischen Verbrauchsdaten
- Proaktives Anpassen der Zeitpläne

### Anomalie-Erkennung
- Automatische Erkennung von ungewöhnlichem Heizverhalten
- Alarm bei dauerhaft hoher Anforderung (Wärmebrücke? Defektes Ventil?)
- Erkennung von defekten Temperatursensoren (stuck values, outliers)
- Benachrichtigungen via HA-Notifications

### Sprachsteuerung-Optimierung
- Verbesserte Expose-Integration für Google Assistant / Alexa / Siri
- Natürlichsprachliche Befehle: "Schlafzimmer auf 18 Grad für die nächsten 2 Stunden"

---

## Langfristige Ideen (Backlog)

### Integration mit externen Systemen
- **KNX**: Direkte KNX-Gruppenadressierung für Aktoren
- **Loxone**: Bidirektionale Sync mit Loxone Miniserver
- **OpenTherm**: Direkte Kesselkommunikation für Vorlauftemperaturregelung
- **Zigbee2MQTT**: Erweiterte TRV-Unterstützung mit Direktkopplung
- **MQTT Discovery**: Automatische MQTT-Geräteerkennung

### Gebäudespezifische Optimierungen
- **Thermische Masse**: Berücksichtigung der Gebäude-Wärmekapazität
- **Südausrichtung**: Solargewinn-Kompensation für südliche Zimmer
- **Nachbarschaftseffekte**: Wärmeübertragung zwischen Zimmern modellieren

### Smart Grid / Demand Response
- Integration mit Smart Grid Tarifsignalen
- Lastverschiebung für netzkonforme Steuerung
- Teilnahme an Demand-Response-Programmen

### Community-Features
- **Konfigurations-Templates**: Vorlagen für häufige Gebäudetypen teilen
- **Heizkurven-Community**: Bewährte Heizkurven für verschiedene Heizsysteme
- **Sensor-Bibliothek**: Getestete und empfohlene Sensoren

---

## Bekannte Einschränkungen (zu beheben)

- [ ] Zeitplan-Persistierung: Zeitpläne werden noch nicht vollständig über den HA-Config-Entry gespeichert – geplant für v1.1
- [ ] Fenster-Temperaturabfall-Erkennung benötigt Temperaturhistorie (geplant für v1.1)
- [ ] Config Flow Heizkurven-Editor: Beschränkt auf 7 Punkte (wird auf beliebig viele Punkte erweitert)
- [ ] Kühlmodus: Grundgerüst vorhanden, aber noch nicht vollständig getestet
- [ ] Frontend Panel: Zimmer-IDs müssen manuell für Services angegeben werden (verbesserte UI geplant)

---

*Zuletzt aktualisiert: 2026-03-09*

*Beiträge und Feature-Requests sind herzlich willkommen über [GitHub Issues](https://github.com/Jedrimos/intelligent-heatingcontroll/issues)*
