# Pressure Advance Documentation

## Overview
The Pressure Advance feature optimizes the extrusion during the print process, allowing for better control over filament flow and reducing issues like oozing and stringing. This document provides an in-depth look at the settings, macros, and examples of workflows for utilizing pressure advance effectively.

## Material Table
| Material      | Recommended PA Setting | Printer Settings       |
|---------------|------------------------|------------------------|
| PLA           | 0.1                    | Temperature: 210°C     |
| PETG          | 0.2                    | Temperature: 240°C     |
| ABS           | 0.3                    | Temperature: 250°C     |

## Macro Descriptions
### Macro: Set Pressure Advance
```gcode
M572 D0 S{value}
```
Use this macro to set the pressure advance value for the current filament. Adjust the `{value}` parameter according to the material being used.

### Macro: Disable Pressure Advance
```gcode
M572 D0 S0
```
This macro will disable pressure advance for the current print.

## Workflow Examples
### Example 1: Standard Print with Pressure Advance
1. Start with the following pre-print macro:
    ```gcode
    M572 D0 S0.1 ; Set PA for PLA
    G28 ; Home all axes
    G1 Z15.0 F9000 ; Move the platform down 15 mm
    G92 E0 ; Reset the extrusion distance
    ```
2. Run the print using your standard G-code.

### Example 2: Pressure Advance Adjustment
1. If you notice stringing, increase the PA value:
    ```gcode
    M572 D0 S0.15 ; Increase PA for better control
    ```

## Customization Guide
1. **Identify Material Type**: Always begin by knowing the type of filament you are printing with.
2. **Set the Pressure Advance**: Use the provided macros to adjust the PA settings based on the material you are using.
3. **Test Prints**: Conduct test prints to evaluate the effectiveness of the PA settings.

## Troubleshooting
- **Issue**: Excessive stringing during prints.
  - **Solution**: Increase the pressure advance setting gradually.
- **Issue**: Under-extrusion or gaps in prints.
  - **Solution**: Decrease the pressure advance setting to allow for better material flow.

For further assistance, consult the community forums or the printer manual.