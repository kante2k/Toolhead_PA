# Pressure Advance Management für Multi-Toolhead Klipper Drucker

## Überblick

Dieses Konfigurationssystem verwaltet automatisch die **Pressure Advance (PA)** Einstellungen für mehrere Extruder/Toolheads auf Klipper 3D-Druckern. Die PA-Werte werden dynamisch basierend auf dem Material angepasst, das sich derzeit in jedem Extruder befindet.

## Unterstützte Materialien

Das System beinhaltet vorkonfigurierte PA-Werte für folgende Materialien:

| Material | PA-Wert | Anwendung |
|----------|---------|-----------|
| **PLA** | 0.03 | Standard FDM Filament, universell einsetzbar |
| **PETG** | 0.042 | Engineering-Filament, beliebt für Funktionsteile |
| **ABS** | 0.055 | Hochtemperatur-Filament, erfordert Feinabstimmung |
| **TPU** | 0.020 | Flexibles Filament, benötigt niedrigere PA-Werte |

## Features

- ✅ Multi-Extruder Unterstützung (verfolgt Material pro Toolhead)
- ✅ Automatisches PA-Umschalten während des Drucks
- ✅ Material-Status-Verfolgung
- ✅ Einfaches Hinzufügen neuer Materialien
- ✅ Benutzer-Feedback über Nachrichten
- ✅ Klipper-native Macros

## Macros

### MATERIAL_STATE
Verfolgt den aktuellen Materialtyp für jeden Extruder.

```gcode
[gcode_macro MATERIAL_STATE]
variable_extruder: "PETG"       # Hauptextruder Material
variable_extruder1: "PETG"      # Extruder 1 Material
variable_extruder2: "PLA"       # Extruder 2 Material
variable_extruder3: "PLA"       # Extruder 3 Material
gcode:
    ; nichts
```

### SET_PA_FOR_EXTRUDER
Setzt den Pressure Advance Wert basierend auf dem angegebenen Material.

**Verwendung:**
```gcode
SET_PA_FOR_EXTRUDER MATERIAL=PLA
SET_PA_FOR_EXTRUDER MATERIAL=PETG
SET_PA_FOR_EXTRUDER MATERIAL=ABS
SET_PA_FOR_EXTRUDER MATERIAL=TPU
```

**Funktionsweise:**
```gcode
[gcode_macro SET_PA_FOR_EXTRUDER]
gcode:
    {% set mat = params.MATERIAL|upper %}

    {% if mat == "PLA" %}
        SET_PRESSURE_ADVANCE EXTRUDER=extruder ADVANCE=0.03
        RESPOND PREFIX=PA MSG="→ PLA → PA=0.03"

    {% elif mat == "PETG" %}
        SET_PRESSURE_ADVANCE EXTRUDER=extruder ADVANCE=0.042
        RESPOND PREFIX=PA MSG="→ PETG → PA=0.042"

    {% elif mat == "ABS" %}
        SET_PRESSURE_ADVANCE EXTRUDER=extruder ADVANCE=0.055
        RESPOND PREFIX=PA MSG="→ ABS → PA=0.055"

    {% elif mat == "TPU" %}
        SET_PRESSURE_ADVANCE EXTRUDER=extruder ADVANCE=0.020
        RESPOND PREFIX=PA MSG="→ TPU → PA=0.020"

    {% else %}
        RESPOND PREFIX=PA MSG="Unbekanntes Material: {mat}"
    {% endif %}
```

### SET_EXTRUDER_MATERIAL
Aktualisiert den Materialtyp für einen spezifischen Extruder und wendet den entsprechenden PA-Wert an.

**Verwendung:**
```gcode
SET_EXTRUDER_MATERIAL EXTRUDER=extruder MATERIAL=PLA
SET_EXTRUDER_MATERIAL EXTRUDER=extruder1 MATERIAL=PETG
SET_EXTRUDER_MATERIAL EXTRUDER=extruder2 MATERIAL=ABS
SET_EXTRUDER_MATERIAL EXTRUDER=extruder3 MATERIAL=TPU
```

**Funktionsweise:**
```gcode
[gcode_macro SET_EXTRUDER_MATERIAL]
gcode:
    {% set ext = params.EXTRUDER %}
    {% set mat = params.MATERIAL|upper %}

    SET_GCODE_VARIABLE MACRO=MATERIAL_STATE VARIABLE={ext} VALUE="'{mat}'"
    SET_PA_FOR_EXTRUDER EXTRUDER={ext} MATERIAL={mat}
```

## Konfiguration

1. Füge die `matmanage.cfg` Datei zu deinem Klipper-Konfigurationsverzeichnis hinzu
2. Binde sie in deine `printer.cfg` ein:
   ```ini
   [include matmanage.cfg]
   ```
3. Aktualisiere die Material-Variablen in `MATERIAL_STATE` entsprechend deiner Konfiguration
4. Passe PA-Werte basierend auf Kalibrierungsergebnissen an

## Workflow

### Vor dem Druck - Material setzen
```gcode
; Materialien vor dem Druck setzen
SET_EXTRUDER_MATERIAL EXTRUDER=extruder MATERIAL=PETG
SET_EXTRUDER_MATERIAL EXTRUDER=extruder1 MATERIAL=PLA
SET_EXTRUDER_MATERIAL EXTRUDER=extruder2 MATERIAL=ABS
SET_EXTRUDER_MATERIAL EXTRUDER=extruder3 MATERIAL=TPU
```

### Während des Drucks - Extruder wechseln
```gcode
; Zu Extruder mit PLA wechseln und PA anwenden
T0                                          ; Wähle Toolhead 0
SET_PA_FOR_EXTRUDER MATERIAL=PLA           ; Wende PLA PA-Wert an
```

## Anpassungen

### Neue Materialien hinzufügen
Bearbeite `matmanage.cfg` und füge eine neue Bedingung hinzu:

```gcode
{% elif mat == "NYLON" %}
    SET_PRESSURE_ADVANCE EXTRUDER=extruder ADVANCE=0.065
    RESPOND PREFIX=PA MSG="→ NYLON → PA=0.065"
```

### PA-Werte anpassen
Optimiere PA-Werte basierend auf deinen Kalibrierungsergebnissen:

```gcode
{% if mat == "PLA" %}
    SET_PRESSURE_ADVANCE EXTRUDER=extruder ADVANCE=0.035  ; Dein kalibrierter Wert
```

## Tipps zur PA-Kalibrierung

1. **PA-Tower verwenden**: Drucke bei verschiedenen PA-Werten, um die optimale Einstellung zu finden
2. **Multiple Geschwindigkeiten testen**: PA variiert mit der Druckgeschwindigkeit; bei typischer Geschwindigkeit kalibrieren
3. **Notizen machen**: Verfolge, welche Materialien bei welchen PA-Werten am besten funktionieren
4. **Pro-Extruder-Tuning**: Verschiedene Extruder benötigen möglicherweise unterschiedliche PA-Werte
5. **Temperatur-Einfluss**: PA kann durch die Düsentemperatur beeinflusst werden; neu kalibrieren wenn Temperatur geändert wird

## Fehlerbehebung

| Problem | Lösung |
|---------|--------|
| Unbekanntes Material Fehler | Überprüfe Schreibweise; Materialnamen sind case-insensitiv |
| PA ändert sich nicht | Verifiziere, dass Macro in printer.cfg eingebunden ist |
| Verschiedene Toolheads haben unterschiedliche PA | Modifiziere Macro, um `EXTRUDER=` Parameter zu übergeben |

## Referenzen

- [Klipper Pressure Advance Dokumentation](https://www.klipper3d.org/Pressure_Advance.html)
- [G-Code Macro Dokumentation](https://www.klipper3d.org/Command_Templates.html)

---

**Version:** 1.0  
**Letzte Aktualisierung:** 2026-02-17  
**Lizenz:** Open Source