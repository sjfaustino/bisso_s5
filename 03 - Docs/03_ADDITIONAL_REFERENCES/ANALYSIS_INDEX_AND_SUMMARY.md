# BISSO Step5 Project - Complete Analysis Index

## Documents Created

This analysis consists of 5 comprehensive documents explaining all Elbo microcontroller input signals:

### 1. **COMPLETE_ELBO_SIGNAL_ANALYSIS.md** ⭐ START HERE
**The most detailed reference - covers every signal in depth**
- All 16 Elbo signals analyzed individually
- Cross-reference counts from QVL file
- Logic patterns and functions
- Control signal hierarchy
- Safety interlocks and dependencies
- **Best for:** Understanding the complete system

### 2. **ELBO_SIGNALS_QUICK_REFERENCE.md** ⭐ FOR QUICK LOOKUP
**Summary table and practical examples**
- Quick reference table of all 16 signals
- Signal categories explained
- Typical Elbo control sequence
- Signal dependencies & interlocks
- Fault detection methods
- Integration examples
- **Best for:** Quick answers and troubleshooting

### 3. **ELBO_VISUAL_REFERENCE_GUIDE.md** ⭐ FOR UNDERSTANDING
**Diagrams and visual explanations**
- Input byte breakdowns (I 72.x and I 73.x)
- Speed control logic diagram
- Axis selection logic diagram
- Safety interlock diagrams
- Complete signal flow diagram
- Signal timing diagram
- Contactor activation matrix
- Troubleshooting checklist
- **Best for:** Visual learners and system designers

### 4. **STEP5_PROJECT_ANALYSIS.md**
**Overview of your entire Step5 system**
- Project overview and architecture
- Three axes explanation
- Current control interface
- Input/output mappings
- Timers and counters
- Flags and data storage
- Current program flow
- **Best for:** Understanding the complete BISSO system

### 5. **I73_7_ANALYSIS.md**
**Initial analysis of single signal (now superseded)**
- Detailed I 73.7 analysis from QVL
- Shows analysis methodology
- **Best for:** Understanding how the analysis was done

---

## Quick Answer Guide

### "What does each Elbo signal do?"

**Refer to: ELBO_SIGNALS_QUICK_REFERENCE.md - Summary Table**

### "How do signals work together?"

**Refer to: COMPLETE_ELBO_SIGNAL_ANALYSIS.md - Signal Hierarchy & Interlocks**

### "How do I troubleshoot a problem?"

**Refer to: ELBO_VISUAL_REFERENCE_GUIDE.md - Troubleshooting Checklist**

### "Show me in a diagram"

**Refer to: ELBO_VISUAL_REFERENCE_GUIDE.md - All diagrams**

### "What's the control sequence?"

**Refer to: ELBO_SIGNALS_QUICK_REFERENCE.md - Typical Sequence or Integration Examples**

### "What signals are critical?"

**Refer to: ELBO_SIGNALS_QUICK_REFERENCE.md - Signal Importance by Color**

### "How does safety work?"

**Refer to: COMPLETE_ELBO_SIGNAL_ANALYSIS.md - I 72.3 and I 72.7 sections**

---

## All 16 Elbo Signals Summary

### INPUT BYTE I 72.x (Speed, Mode, Status)

| Bit | Signal | Function | Type | Critical |
|-----|--------|----------|------|----------|
| 0 | **I 72.0** - Elbo Rapido | Fast speed selection | Speed Mode | No |
| 1 | **I 72.1** - Elbo Medio | Medium speed selection | Speed Mode | No |
| 2 | **I 72.2** - Cmd Rot | Rotation mode selector | Mode Select | No |
| 3 | **I 72.3** - Cmd LockTable | Lock table during cutting | Safety Lock | ⚠️ YES |
| 4 | **I 72.4** - Pressostato | Water pressure OK | Status | No |
| 5 | **I 72.5** - VFD1_IsRunning | Inverter 1 status | Status | No |
| 6 | **I 72.6** - VFD2_IsRunning | Inverter 2 status | Status | No |
| 7 | **I 72.7** - VFDDisk_IsRunning | Cutting disk running | Status | ⚠️ YES |

### INPUT BYTE I 73.x (Axis Selection & Direction)

| Bit | Signal | Function | Type | Critical |
|-----|--------|----------|------|----------|
| 0 | **I 73.0** - Eixo Corte | Vertical/Cutting (Z-axis) | Axis Select | No |
| 1 | **I 73.1** - Eixo Translacao | Horizontal/Translation (Y-axis) | Axis Select | No |
| 2 | **I 73.2** - Eixo Cala | Tilt/Angle control | Axis Select | No |
| 3 | **I 73.3** - Eixo Rotacao Mesa | Table rotation | Axis Select | No |
| 4 | **I 73.4** - Eixo Rotacao Disco | Disk rotation control | Axis Select | No |
| 5 | **I 73.5** - Direcao "+" | Positive direction (+) | Direction | No |
| 6 | **I 73.6** - Direcao "-" | Negative direction (-) | Direction | No |
| 7 | **I 73.7** - V/S | Velocity/Speed Enable | Master Gate | ⚠️ YES |

---

## Critical Safety Signals (3)

These are the most important for system safety:

### 1. **I 72.3 - Cmd LockTable** (Used 23 times)
**Purpose:** Lock table in place during cutting operations  
**When I 72.3 = 1:**
- Table is LOCKED
- Oil pump (Q 7.3) is de-energized
- Table CANNOT move
- Prevents accidental cutting hazard

**When I 72.3 = 0:**
- Table is FREE to move
- Oil pump can be energized
- Repositioning material allowed

### 2. **I 72.7 - VFDDisk_IsRunning** (Used 9 times)
**Purpose:** Indicate cutting disk is actively spinning  
**When I 72.7 = 1:**
- Disk is CUTTING
- Table should be LOCKED (via I 72.3)
- Prevents material movement during cut
- Safety interlock with I 72.3

**When I 72.7 = 0:**
- Disk is STOPPED
- Table can be repositioned
- Safe to move material

### 3. **I 73.7 - V/S (Velocity Enable)** (Used 10 times)
**Purpose:** Master gate for ALL motion  
**When I 73.7 = 1:**
- Valid speed signal from Elbo
- Motion is ALLOWED
- Process all axis and direction signals

**When I 73.7 = 0:**
- No speed signal
- ALL MOTION BLOCKED (like E-stop)
- Safe emergency shutdown

---

## Functional Signal Groups

### Speed Control Signals
- **I 72.0** - Fast speed (14 uses)
- **I 72.1** - Medium speed (5 uses)
- **Function:** Determine motion speed for all axes
- **Typical:** Both cannot be 1 at same time

### Axis Selection Signals
- **I 73.0** - Vertical/Cutting (7 uses)
- **I 73.1** - Translation/Horizontal (5 uses)
- **I 73.2** - Tilt (2 uses)
- **I 73.3** - Table Rotation (16 uses)
- **I 73.4** - Disk Rotation (indirect)
- **Function:** Select which physical axis to control
- **Typical:** Only one should be 1 at a time

### Direction Selection Signals
- **I 73.5** - Positive direction (17 uses)
- **I 73.6** - Negative direction (17 uses)
- **Function:** Select direction for selected axis
- **Typical:** One or neither, rarely both

### Status/Feedback Signals
- **I 72.4** - Pressure (1 use)
- **I 72.5** - VFD1 (11 uses)
- **I 72.6** - VFD2 (6 uses)
- **I 72.7** - Disk running (9 uses)
- **Function:** Monitor system health
- **Typical:** Read-only feedback from Elbo

---

## Key Findings from QVL Analysis

### Most Used Signals
1. **I 72.3** - Table Lock: 23 times ⚠️
2. **I 73.5** - Direction +: 17 times
3. **I 73.6** - Direction -: 17 times
4. **I 73.3** - Table Rotation: 16 times
5. **I 72.0** - Fast Speed: 14 times

### Less Used Signals
- **I 72.1** - Medium: 5 times
- **I 73.1** - Translation: 5 times
- **I 72.6** - VFD2: 6 times
- **I 73.0** - Vertical: 7 times
- **I 72.4** - Pressure: 1 time (!!)

### Signal Type Distribution
- **Speed Mode:** 2 signals (14 + 5 = 19 uses)
- **Mode/Safety:** 2 signals (11 + 23 = 34 uses)
- **Status:** 4 signals (1 + 11 + 6 + 9 = 27 uses)
- **Axis Select:** 5 signals (7 + 5 + 2 + 16 + ? = 30+ uses)
- **Direction:** 2 signals (17 + 17 = 34 uses)
- **Master Enable:** 1 signal (10 uses)

---

## How Signals Interact

### Motion Sequence (Normal Operation)
```
1. Operator selects speed via Elbo
   → I 72.0 = 1 (fast) or I 72.1 = 1 (medium)

2. Operator selects axis to move
   → One of I 73.0-73.4 = 1

3. Operator selects direction
   → I 73.5 = 1 (positive) or I 73.6 = 1 (negative)

4. Elbo provides velocity signal
   → I 73.7 = 1 (speed signal valid)

5. PLC evaluates all signals
   → Checks safety conditions (I 72.3, I 72.7)
   → Determines appropriate contactor

6. Contactor energizes
   → Motor moves in requested direction at requested speed

7. Operator releases control
   → I 73.5 and I 73.6 → 0 or I 73.7 → 0
   → PLC de-energizes contactor
   → Motor stops
```

### Safety Interlock (Table Lock + Disk Running)
```
Normal condition during cutting:
  I 72.3 = 0 (table unlocked - OK during cut)
  I 72.7 = 1 (disk running - cutting)
  ✓ Operation normal

Fault condition:
  I 72.3 = 1 (table locked)
  AND
  I 72.7 = 1 (disk running)
  
  Result: PLC prevents Q 7.3 (oil pump) from activating
          Table CANNOT move even if commanded
          ✓ Safety prevented accidental cut hazard
```

---

## System Control Matrix

```
TO MAKE THIS HAPPEN       → SET THESE SIGNALS

Blade moves UP           → I 73.0=1, I 73.5=1, I 73.7=1
Blade moves DOWN         → I 73.0=1, I 73.6=1, I 73.7=1
Material moves RIGHT     → I 73.1=1, I 73.5=1, I 73.7=1
Material moves LEFT      → I 73.1=1, I 73.6=1, I 73.7=1
Blade tilts UP          → I 73.2=1, I 73.5=1, I 73.7=1
Blade tilts DOWN        → I 73.2=1, I 73.6=1, I 73.7=1
Table rotates CW        → I 73.3=1, I 73.5=1, I 72.2=1, I 73.7=1
Table rotates CCW       → I 73.3=1, I 73.6=1, I 72.2=1, I 73.7=1
Control disk speed      → I 73.4=1, I 72.2=1, I 73.7=1

PREVENT ALL MOTION      → I 73.7=0 (Emergency Stop)
LOCK TABLE              → I 72.3=1 (Safety Lock)
PREVENT CUT DURING MOVE → I 72.7=0 AND I 72.3=0
FORCE SAFE STATE        → I 73.7=0 (master gate off)
```

---

## For Your Nextion HMI Integration

When you create positioning commands from the Nextion display, you'll essentially need to:

1. **Translate HMI commands** to Elbo signal patterns
   - Example: "Move X to 100mm" → Set I 73.1, I 73.5/73.6, I 73.7
   
2. **Monitor safety interlocks**
   - Respect I 72.3 (table lock during cutting)
   - Respect I 72.7 (disk running prevents repositioning)
   
3. **Respect speed settings**
   - Use I 72.0 or I 72.1 based on movement type
   
4. **Provide velocity signal**
   - Simulate I 73.7 = 1 when sending positioning command
   
5. **Monitor feedback**
   - Read I 72.5, I 72.6, I 72.7 for status
   - Verify motion occurred

---

## Testing Checklist

When testing your understanding:

- [ ] Can you identify which signals are critical (I 72.3, I 72.7, I 73.7)?
- [ ] Can you explain what happens if I 73.7 = 0?
- [ ] Can you describe the interlock between I 72.3 and I 72.7?
- [ ] Can you map all 5 axes to I 73.0-73.4?
- [ ] Can you explain why I 72.4 is used only once?
- [ ] Can you identify which signals prevent motion?
- [ ] Can you explain the difference between I 73.5 and I 73.6?
- [ ] Can you describe a normal cutting sequence?
- [ ] Can you list the 3 most critical signals?
- [ ] Can you explain how table repositioning works?

---

## Document Usage Recommendations

### For System Understanding
1. Start: **STEP5_PROJECT_ANALYSIS.md** - Understand the complete system
2. Then: **COMPLETE_ELBO_SIGNAL_ANALYSIS.md** - Deep dive into each signal
3. Reference: **ELBO_VISUAL_REFERENCE_GUIDE.md** - Visual reinforcement

### For Quick Answers
- Always use: **ELBO_SIGNALS_QUICK_REFERENCE.md**

### For Troubleshooting
- Check: **ELBO_VISUAL_REFERENCE_GUIDE.md - Troubleshooting Checklist**

### For Programming/Integration
- Reference: **COMPLETE_ELBO_SIGNAL_ANALYSIS.md - Control Signal Hierarchy**
- Use: **ELBO_SIGNALS_QUICK_REFERENCE.md - Contactor Activation Matrix**

---

## Analysis Methodology

These documents were created by:

1. **QVL File Analysis** - Examined cross-references for each input signal
   - Counted total uses of each signal
   - Identified code patterns (AND vs AND NOT)
   - Determined signal relationships

2. **Symbol File Review** - Examined signal definitions
   - Portuguese → English translations
   - Signal naming conventions
   - Functional groupings

3. **Logic Reconstruction** - Inferred behavior from cross-reference patterns
   - Mutual exclusions
   - Safety interlocks
   - Functional relationships

4. **Verification** - Cross-checked against:
   - Known hardware configuration
   - Typical industrial machine control patterns
   - Safety standards for cutting equipment

---

## Next Steps for Your Project

1. ✅ **Understand current system** - Review STEP5_PROJECT_ANALYSIS.md
2. ✅ **Learn all Elbo signals** - Study COMPLETE_ELBO_SIGNAL_ANALYSIS.md
3. ⏳ **Design Nextion-to-Elbo bridge** - Create logic to simulate Elbo signals from HMI
4. ⏳ **Add encoder feedback** - Install position sensors for absolute positioning
5. ⏳ **Integrate positioning logic** - Add FB20-23 to your Step5 program
6. ⏳ **Test safety interlocks** - Verify I 72.3 and I 72.7 work correctly
7. ⏳ **Validate communication** - Test UART between Nextion and PLC
8. ⏳ **Deploy and document** - Final system testing and handover

---

## Questions This Analysis Answers

- **What does I 73.7 control?** → Master motion enable gate
- **Why does I 72.3 appear 23 times?** → Critical safety interlock for table lock
- **How are the 5 axes controlled?** → I 73.0-73.4 select axis, I 73.5-73.6 select direction
- **What happens if I 73.7 = 0?** → All motion stops (emergency stop)
- **How does the system prevent cutting errors?** → I 72.3 + I 72.7 interlocks
- **What signals are just feedback?** → I 72.4-72.7 are mostly status
- **How do you stop the machine?** → Set I 73.7 = 0 or release I 73.5/73.6
- **Why are there two speed signals?** → I 72.0 (fast) and I 72.1 (medium) are selectable
- **How does Elbo control 5 different motions?** → Axis select (I 73.0-73.4) + direction (I 73.5-73.6)
- **What's the master enable signal?** → I 73.7 gates all motion

