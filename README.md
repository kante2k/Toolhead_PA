# Pressure Advance Management for Multi-Toolhead Klipper Printers

## Overview

This configuration system automatically manages **Pressure Advance (PA)** settings for multiple extruders/toolheads on Klipper 3D printers. PA values are dynamically adjusted based on the material type currently loaded in each extruder.

## Supported Materials examples

The system includes pre-configured PA values for the following materials:

| Material | PA Value | 
|----------|----------|
| **PLA** | 0.03 |
| **PETG** | 0.042 |
| **ABS** | 0.055 |
| **TPU** | 0.020 |

## Features

- ✅ Multi-extruder support (tracks material per toolhead)
- ✅ Automatic PA switching during print
- ✅ Material state tracking
- ✅ Easy addition of new materials


## Macros

### MATERIAL_STATE
Tracks the current material type for each extruder.

```gcode
[gcode_macro MATERIAL_STATE]
variable_extruder: "PETG"       # Main extruder material
variable_extruder1: "PETG"      # Extruder 1 material
variable_extruder2: "PLA"       # Extruder 2 material
variable_extruder3: "PLA"       # Extruder 3 material
gcode:
    ; nothing
```

### SET_PA_FOR_EXTRUDER
Sets the Pressure Advance value based on the specified material.

**Usage:**
```gcode
SET_PA_FOR_EXTRUDER MATERIAL=PLA
SET_PA_FOR_EXTRUDER MATERIAL=PETG
SET_PA_FOR_EXTRUDER MATERIAL=ABS
SET_PA_FOR_EXTRUDER MATERIAL=TPU
```

**How it works:**
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
        RESPOND PREFIX=PA MSG="Unknown material: {mat}"
    {% endif %}
```

### SET_EXTRUDER_MATERIAL
Updates the material type for a specific extruder and applies the corresponding PA value.

**Usage:**
```gcode
SET_EXTRUDER_MATERIAL EXTRUDER=extruder MATERIAL=PLA
SET_EXTRUDER_MATERIAL EXTRUDER=extruder1 MATERIAL=PETG
SET_EXTRUDER_MATERIAL EXTRUDER=extruder2 MATERIAL=ABS
SET_EXTRUDER_MATERIAL EXTRUDER=extruder3 MATERIAL=TPU
```

**How it works:**
```gcode
[gcode_macro SET_EXTRUDER_MATERIAL]
gcode:
    {% set ext = params.EXTRUDER %}
    {% set mat = params.MATERIAL|upper %}

    SET_GCODE_VARIABLE MACRO=MATERIAL_STATE VARIABLE={ext} VALUE="'{mat}'"
    SET_PA_FOR_EXTRUDER EXTRUDER={ext} MATERIAL={mat}
```

## Configuration

1. Add the `matmanage.cfg` file to your Klipper configuration directory
2. Include it in your `printer.cfg`:
   ```ini
   [include matmanage.cfg]
   ```
3. Update the material variables in `MATERIAL_STATE` according to your configuration
4. Adjust PA values based on calibration results

5. Add this to your Toolchange macro

    ```gcode
       {% set mat = printer["gcode_macro MATERIAL_STATE"].extruder %}
        SET_PA_FOR_EXTRUDER EXTRUDER=extruder MATERIAL={mat}
    ```
    Change EXTRUDER=extruder1 or your extruder name for other Extruder/Hotend

   Example :
   ```gcode
               [gcode_macro T0]
				gcode:

			# --- Pressure Advance ---
    	{% set mat = printer["gcode_macro MATERIAL_STATE"].extruder %}
    	SET_PA_FOR_EXTRUDER EXTRUDER=extruder MATERIAL={mat}

   				[gcode_macro T1]
				gcode:
   			{% set mat = printer["gcode_macro MATERIAL_STATE"].extruder1 %}
  		  SET_PA_FOR_EXTRUDER EXTRUDER=extruder1 MATERIAL={mat}
    ```

## Workflow


### Before Print - Set Material
```gcode
; Set materials before starting print
SET_EXTRUDER_MATERIAL EXTRUDER=extruder MATERIAL=PETG
SET_EXTRUDER_MATERIAL EXTRUDER=extruder1 MATERIAL=PLA
SET_EXTRUDER_MATERIAL EXTRUDER=extruder2 MATERIAL=ABS
SET_EXTRUDER_MATERIAL EXTRUDER=extruder3 MATERIAL=TPU
```

### During Print - Switch Extruder
```gcode
; Switch to extruder with PLA and apply PA
T0                                          ; Select toolhead 0
SET_PA_FOR_EXTRUDER MATERIAL=PLA           ; Apply PLA PA value
```

## Customization

### Adding New Materials
Edit `matmanage.cfg` and add a new condition:

```gcode
{% elif mat == "NYLON" %}
    SET_PRESSURE_ADVANCE EXTRUDER=extruder ADVANCE=0.065
    RESPOND PREFIX=PA MSG="→ NYLON → PA=0.065"
```

### Adjusting PA Values
Fine-tune PA values based on your calibration results:

```gcode
{% if mat == "PLA" %}
    SET_PRESSURE_ADVANCE EXTRUDER=extruder ADVANCE=0.035  ; Your calibrated value
```

## PA Calibration Tips

1. **Use a PA Tower**: Print at different PA values to find the optimal setting
2. **Test Multiple Speeds**: PA varies with print speed; calibrate at your typical speed
3. **Keep Notes**: Track which materials work best at which PA values
4. **Per-Extruder Tuning**: Different extruders may need different PA values
5. **Temperature Impact**: PA can be affected by nozzle temperature; recalibrate if changing temps

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Unknown material error | Check spelling; material names are case-insensitive |
| PA not changing | Verify macro is included in printer.cfg |
| Different toolheads have different PA | Modify macro to pass `EXTRUDER=` parameter |

## References

- [Klipper Pressure Advance Documentation](https://www.klipper3d.org/Pressure_Advance.html)
- [G-Code Macro Documentation](https://www.klipper3d.org/Command_Templates.html)

---

**Version:** 1.0  
**Last Updated:** 2026-02-17 16:45:33  
**License:** Open Source
