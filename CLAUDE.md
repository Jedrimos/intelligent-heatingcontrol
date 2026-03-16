# CLAUDE.md – Entwicklungsregeln für Intelligent Heating Control

## Analyse: Woher kamen die Bugs der letzten Stunden?

Alle Bugs hatten **ein einziges Muster**: Eine Änderung wurde an Stelle A gemacht,
aber die abhängige Stelle B wurde vergessen. Die Bugs waren keine Denkfehler –
sie waren fehlende "zweite Schritte".

| Bug | Ursache |
|-----|---------|
| `CONF_WINDOW_OPEN_TEMP` Import-Crash in HA | In `const.py` gelöscht, aber `coordinator.py` importierte es noch |
| `enable-heating-switch` JS-Fehler | HTML-Element entfernt, aber `querySelector` im save-handler blieb |
| Zeitpläne/Kalender Switch-Case blieb | Tab aus HTML entfernt, aber `case` in `_renderTabContent()` blieb |
| Add-Room fehlte CO₂ + Presence-Feld | Felder im Edit-Modal ergänzt, aber Add-Modal nie synchronisiert |
| `<select>` hatte Klasse `form-input` | Aus `<input>`-Template kopiert ohne Klasse anzupassen |
| Float statt Int für Reaktionszeiten | Frontend: `parseFloat()`, Backend: `int()` – nie abgeglichen |

**Lösung**: Vor jedem Commit die Checklisten unten durchgehen.

---

## Checklisten nach Änderungstyp

### Wenn eine Konstante aus `const.py` gelöscht oder umbenannt wird

```bash
# PFLICHT vor dem Löschen: alle Referenzen suchen
grep -rn "CONF_XYZ\|DEFAULT_XYZ" custom_components/ --include="*.py"
```

Alle gefundenen Stellen müssen **im gleichen Commit** bereinigt werden:
- [ ] `const.py` – Definition entfernt
- [ ] `coordinator.py` – Import + alle Verwendungen entfernt
- [ ] `config_flow.py` – Import + Formular + save-handler bereinigt
- [ ] `climate.py` – Import + `extra_state_attributes` bereinigt

### Wenn eine Konstante in `const.py` neu hinzugefügt wird

- [ ] `const.py` – Definition mit `CONF_` und `DEFAULT_` Wert
- [ ] `config_flow.py` – Import, im Formular anzeigen, im save-handler `int()`/`float()`/`str()` speichern
- [ ] `climate.py` – Import, in `extra_state_attributes` zurückgeben mit korrektem Fallback-Typ
- [ ] `coordinator.py` – Import, in der Logik verwenden

### Wenn ein HTML-Element im Frontend entfernt oder umbenannt wird

Für jede entfernte `id="foo"`:
- [ ] Suche alle `querySelector("#foo")` und `#foo` im JS → entfernen oder anpassen
- [ ] Prüfe ob Event-Listener (`addEventListener`) für dieses Element existieren → entfernen

```bash
grep -n "foo" custom_components/intelligent_heating_control/frontend/ihc-panel.js
```

### Wenn ein Tab aus der Tab-Bar entfernt wird

Beide Stellen sind immer gemeinsam zu ändern:
- [ ] HTML: `<div class="tab" data-tab="xyz">` entfernen
- [ ] JS: `case "xyz": this._renderXyz(content); break;` in `_renderTabContent()` entfernen
- [ ] JS: Auto-Refresh-Timer prüfen (suche nach `this._activeTab === "xyz"`)

### Wenn ein Feld zu einem Modal hinzugefügt wird

Add-Room und Edit-Room Modal müssen **immer synchron** bleiben:
- [ ] Feld im `_showAddRoomModal()` HTML ergänzt
- [ ] Feld im `_showAddRoomModal()` save-handler ergänzt (mit `?.value` für neue Felder)
- [ ] Feld im `_showEditRoomModal()` HTML ergänzt (mit Vorbelgung aus `room.xyz`)
- [ ] Feld im `_showEditRoomModal()` save-handler ergänzt

### Wenn Typen zwischen Frontend und Backend ausgetauscht werden

| Python-Typ | Frontend → Backend | Backend → Frontend |
|------------|--------------------|--------------------|
| `int` | `parseInt(val, 10)` | `?? 0` |
| `float` | `parseFloat(val)` | `?? 0.0` |
| `bool` | `val === "true"` | `!== false` |
| `str` | `.value.trim()` | `\|\| ""` |
| `list` | `.split(",").map(s=>s.trim()).filter(Boolean)` | `\|\| []` |

Niemals `parseFloat()` für Werte verwenden die der Backend als `int` erwartet (z.B. Sekunden, Minuten, ppm).

---

## Pflicht-Checks vor jedem Commit

```bash
cd /home/user/intelligent-heatingcontroll

# 1. Python-Syntax aller geänderten Dateien prüfen
python3 -m py_compile custom_components/intelligent_heating_control/const.py
python3 -m py_compile custom_components/intelligent_heating_control/coordinator.py
python3 -m py_compile custom_components/intelligent_heating_control/config_flow.py
python3 -m py_compile custom_components/intelligent_heating_control/climate.py

# 2. Keine verwaisten Referenzen auf gelöschte Konstanten
grep -rn "CONF_WINDOW_OPEN_TEMP" custom_components/ --include="*.py"
# → muss leer sein

# 3. Alle querySelector ohne ?. auf tatsächlich vorhandene Elemente prüfen
# (bei Zweifeln: ?.value statt .value verwenden)

# 4. git diff lesen und für jede Löschung prüfen: "Wo wird das noch verwendet?"
git diff --stat
```

---

## Architektur

### Datenfluss
```
config_flow.py  →  HA options  →  coordinator.py  →  climate.py  →  ihc-panel.js
   (UI-Config)                    (Logik/Berechnung)  (Attribute)    (Frontend)
                                        ↑
                              callService() vom Frontend
```

### Welche Datei ist wofür zuständig

| Datei | Zuständigkeit |
|-------|--------------|
| `const.py` | Alle Konstanten-Definitionen. Keine Logik. |
| `coordinator.py` | Heizlogik, Berechnung, Service-Handler |
| `config_flow.py` | HA-Konfigurationsflow (Setup + Options) |
| `climate.py` | Climate-Entität, `extra_state_attributes` (Frontend-Daten) |
| `ihc-panel.js` | Komplettes Frontend, 1 Datei, Web Components |

### Tab-Struktur (Frontend, Stand nach letztem Refactor)
- `🏠 Dashboard` → `_renderOverview()`
- `🚪 Zimmer` → `_renderRooms()` → bei Zimmer-Auswahl → `_renderRoomDetail()`
  - Sub-Tab `📅 Zeitplan` → `_renderRoomScheduleInline(room, container)`
  - Sub-Tab `🗓️ Wochenansicht` → `_renderRoomCalendarInline(room, container)`
- `📊 Übersicht` → `_renderDiagnose()`
- `⚙️ Einstellungen` → `_renderSettings()`
- `📈 Heizkurve` → `_renderCurve()`

### State-Variablen im Frontend
```javascript
this._activeTab         // aktueller Haupt-Tab
this._selectedRoom      // entity_id des ausgewählten Zimmers (null = Listenansicht)
this._selectedRoomTab   // "schedule" | "calendar"
this._editingSchedules  // { [entityId]: schedules[] } – gepuffert bis Speichern
this._modalOpen         // true wenn ein Modal offen ist
```

### Service-Calls (Frontend → Backend)
```javascript
_callService("add_room",             { name, temp_sensor, valve_entities, ... })
_callService("update_room",          { id, schedules, ha_schedules, ... })
_callService("remove_room",          { id })
_callService("set_room_mode",        { id, mode })
_callService("boost_room",           { id, duration_minutes, temp?, cancel? })
_callService("update_global_settings", { outdoor_temp_sensor, controller_mode, ... })
_callService("set_system_mode",      { mode })
_callService("reset_stats",          {})
_callService("deactivate_guest_mode",{})
```

### Branch & Push
```bash
git push -u origin claude/hacs-heating-control-plugin-NXmK3
```
Niemals auf `main` pushen.
