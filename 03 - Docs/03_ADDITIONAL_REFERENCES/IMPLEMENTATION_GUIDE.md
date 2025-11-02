# Stone Bridge Saw Positioning Controller - Implementation Guide

## Overview
This HMI (Human-Machine Interface) is designed for a Nextion 7.0" TFT display (800x480) that controls a 3-axis stone bridge saw via GCode commands sent to a PLC.

## Hardware Setup

### Nextion Display Connection
- **Model**: Nextion 7.0 Inch TFT 800x480 UART
- **Communication**: UART/Serial at 115200 baud
- **Power**: 5V DC
- **Pin Connections**:
  - VCC → 5V
  - GND → GND
  - RX → MCU TX
  - TX → MCU RX

## HMI Features

### Main Page (Page 0)
- **Status Display**: Real-time system status indicator
- **Axis Selection**: Quick buttons for X, Y, Z axis control pages
- **Quick Commands**:
  - Home All: Send all axes to home position
  - STOP: Emergency stop command
  - Reset: System reset
- **Position Display**: Real-time current position for all axes
- **Feed Rate**: Current feed rate display (mm/min)

### Axis Control Pages (Pages 1-3)

Each axis has dedicated controls:

#### X-Axis Page (Page 1)
- Current position display
- Target position input (-500 to 5000 mm range)
- Move To button (generates G00 command)
- Jog Controls:
  - Left/Right buttons
  - Adjustable jog distance
- Home button
- Stop button
- Live GCode output display

#### Y-Axis Page (Page 2)
- Same structure as X-Axis
- Up/Down jog buttons
- Generates G00 Y commands

#### Z-Axis Page (Page 3)
- Same structure as X/Y
- Up/Down jog buttons
- Generates G00 Z commands

## GCode Output Format

The HMI generates GCode commands in the following format:

### Absolute Positioning
```
G00 X100.000 F1000
; Move to X position 100mm at 1000 mm/min feed rate
```

### Home Commands
```
G28 X0
; Home X axis
```

### Stop Command
```
!
; Emergency stop (soft stop via GCode)
```

## Communication Protocol

### Data Format
- **Baud Rate**: 115200
- **Data Bits**: 8
- **Stop Bits**: 1
- **Parity**: None
- **Flow Control**: None

### Command Structure
Commands from Nextion to PLC:

```
[COMMAND]|[AXIS]|[VALUE]\n
```

Example commands:
- `MOVE|X|100.5` - Move X to 100.5 mm
- `JOG|Y|10` - Jog Y by 10 mm
- `HOME|Z|0` - Home Z axis
- `STOP|ALL|0` - Emergency stop

### Response Structure
Expected responses from PLC back to Nextion:

```
POS:[AXIS],[VALUE]\n
```

Example:
- `POS:X,50.25` - X position is now 50.25 mm
- `STATUS:MOVING` - System is moving
- `STATUS:READY` - System is idle

## Implementation Steps

### 1. Import into Nextion Editor
- Download the Nextion Editor from https://nextion.tech/
- Open Editor → Import → Select `stone_saw_controller.hmi`
- Review all pages to ensure proper layout

### 2. Customize Parameters
In Nextion Editor:
- Modify min/max values for target position inputs (currently -500 to 5000)
- Adjust jog distance range if needed
- Update axis names if using different nomenclature

### 3. Compile and Upload
- In Editor: File → Compile
- Connect Nextion to PC via USB adapter
- File → Download to Device
- Wait for completion (do not power off)

### 4. PLC-Side Implementation

Your PLC firmware should:

1. **Read UART Input** at 115200 baud
2. **Parse Commands** using the format above
3. **Generate GCode** based on HMI input
4. **Send GCode** to motion control system
5. **Monitor Position** from encoders/sensors
6. **Send Position Updates** back to Nextion
7. **Handle Emergency Stops** immediately

### Sample PLC Pseudocode:

```
MAIN LOOP:
  1. Read UART buffer
  2. IF data available:
     a. Parse command string
     b. Extract axis and value
     c. Generate GCode string
     d. Send GCode to motion controller
     e. Wait for acknowledgment
     f. Read current position
     g. Send position back to Nextion
  3. Update status display
  4. Check for timeout/errors
  5. Loop
```

## GCode Commands Reference

### G00 - Rapid Positioning
```
G00 X100.0 F1000
```
- Move to absolute position at feed rate

### G28 - Home
```
G28 X0
G28 Y0
G28 Z0
G28 X0 Y0 Z0  ; Home all axes
```

### G92 - Set Position
```
G92 X0 Y0 Z0
; Set current position as origin
```

### M00 - Program Stop
```
M00
; Pause program
```

### Soft Stop
```
!
; Immediate stop
```

## Variable Mapping

The HMI uses Nextion variables that your PLC should monitor:

| Variable | Page | Object | Type | Purpose |
|----------|------|--------|------|---------|
| status_display | 0 | text | Display | System status |
| pos_x_value | 0 | text | Display | X position |
| pos_y_value | 0 | text | Display | Y position |
| pos_z_value | 0 | text | Display | Z position |
| feed_rate_value | 0 | text | Display | Feed rate |
| current_pos_x | 1 | text | Display | X axis current |
| target_input | 1 | number | Input | X target value |
| jog_distance | 1 | number | Input | X jog amount |
| gcode_display | 1 | text | Display | GCode output |
| (Similar for Y and Z pages) | 2,3 | ... | ... | ... |

## Troubleshooting

### Display Not Responding
1. Check UART connections
2. Verify baud rate: 115200
3. Test with serial monitor at 115200
4. Check power supply (5V stable)

### Position Not Updating
1. Verify PLC is sending position updates
2. Check Nextion variable names match your code
3. Use serial monitor to view data stream

### GCode Not Sending
1. Verify commands are properly formatted
2. Check PLC is parsing UART input
3. Monitor UART debug output

### Button Presses Not Registering
1. Check object IDs match page definitions
2. Re-compile and re-upload HMI
3. Check touch screen calibration

## Customization Tips

### Add More Axes
1. Duplicate Page 3 (Z-Axis)
2. Change title and axis labels (X→A, etc.)
3. Update all button press commands
4. Increment page ID numbers

### Modify Display Colors
Color codes (RGB565 format):
- Black: 0x0000
- White: 0xFFFF
- Red: 0xF800
- Green: 0x07E0
- Blue: 0x001F
- Yellow: 0xFF80
- Cyan: 0x07FF

Update `bco` (background color) and `fontcolor` values

### Add Preset Positions
In any axis page, add buttons like:
- "Cut Position 1"
- "Cut Position 2"
- etc.

Example button event: `target_input.val=2500` sets input to 2500

## Performance Considerations

- **UART Buffer**: Keep command messages short (<100 chars)
- **Update Rate**: Send position updates at ~100ms intervals for smooth display
- **Response Time**: HMI button press to motion should be <500ms
- **Status Messages**: Keep status text under 50 characters

## Safety Notes

⚠️ **IMPORTANT**
1. Always implement hardware emergency stop independent of this interface
2. Validate all position limits in PLC before motion
3. Test jog movements at lowest speeds first
4. Implement position error checking
5. Add watchdog timer for UART communication loss

## Support & Resources

- Nextion Wiki: https://nextion.tech/
- GCode Reference: https://www.reprap.org/wiki/G-code
- Serial Communication: Standard UART protocol

---

**Version**: 1.0  
**Last Updated**: 2025  
**Compatible Display**: Nextion 7.0" NX8048P070-011R-1  
