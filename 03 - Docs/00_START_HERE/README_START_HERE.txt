================================================================================
BISSO STONE BRIDGE SAW - STEP5 PROJECT ANALYSIS
Complete Elbo Signal Documentation
================================================================================

OVERVIEW
--------
This analysis completely disassembles and documents your BISSO stone bridge saw
Step5 PLC project, specifically focusing on all 16 Elbo microcontroller input
signals (I 72.0-72.7 and I 73.0-73.7).

The analysis was performed by examining the QVL (Quality Verification List) file
from your compiled Step5 project, extracting cross-references, and reconstructing
the control logic.

================================================================================
DOCUMENTS CREATED (5 FILES)
================================================================================

1. ⭐ ANALYSIS_INDEX_AND_SUMMARY.md
   START HERE for complete overview
   - Master index of all documents
   - Quick answer guide
   - All 16 signals summary
   - Critical safety signals
   - System control matrix
   - Document usage recommendations

2. ⭐ COMPLETE_ELBO_SIGNAL_ANALYSIS.md
   MOST DETAILED REFERENCE
   - Each signal analyzed in depth
   - Cross-reference counts from QVL file
   - Logic patterns and functions
   - Signal dependencies and interlocks
   - Complete control hierarchy
   - Safety mechanisms
   - Typical operation sequences

3. ⭐ ELBO_SIGNALS_QUICK_REFERENCE.md
   FOR QUICK LOOKUPS AND TROUBLESHOOTING
   - Summary table of all 16 signals
   - Signal categories explained
   - Typical control sequences
   - Integration examples
   - Fault detection methods
   - Troubleshooting checklist

4. ⭐ ELBO_VISUAL_REFERENCE_GUIDE.md
   FOR VISUAL LEARNERS
   - Input byte diagrams
   - Speed control logic diagram
   - Axis selection logic diagram
   - Safety interlock diagrams
   - Signal flow diagram
   - Timing diagram
   - Contactor activation matrix
   - Troubleshooting checklist

5. STEP5_PROJECT_ANALYSIS.md
   SYSTEM OVERVIEW
   - Your complete BISSO system architecture
   - Three axes explanation
   - I/O mappings and functions
   - Timers, counters, flags
   - Current program flow

6. I73_7_ANALYSIS.md
   SINGLE SIGNAL ANALYSIS (EXAMPLE)
   - Shows analysis methodology
   - I 73.7 (master enable) detailed

================================================================================
QUICK FACTS
================================================================================

SYSTEM: Stone Bridge Saw (Bisso - Portuguese: "serra para bisso")
PLC: Siemens S5 (Step5/AWLEP)
CONTROL: Elbo Microcontroller (proportional joystick-based)
INTERFACE: 16 digital inputs (I 72.x and I 73.x)

TOTAL SIGNALS ANALYZED: 16
CRITICAL SIGNALS: 3 (I 72.3, I 72.7, I 73.7)
MOST USED SIGNAL: I 72.3 (Table Lock) - 23 references
MASTER ENABLE: I 73.7 (Velocity Signal)

FIVE CONTROLLED AXES:
  1. Z-Axis (Vertical) - Blade Up/Down
  2. Y-Axis (Translation) - Left/Right 
  3. Tilt - Blade Angle Adjustment
  4. Table Rotation - Clockwise/Counter-clockwise
  5. Disk Rotation - Cutting Speed Control

================================================================================
THE THREE CRITICAL SIGNALS
================================================================================

⚠️ I 72.3 (Cmd LockTable) - Used 23 times
   Purpose: LOCK TABLE during cutting operations
   Effect: When active, prevents table movement (safety)
   When I 72.3 = 1: Oil pump de-energizes, table cannot move
   When I 72.3 = 0: Table free to move

⚠️ I 72.7 (VFDDisk_IsRunning) - Used 9 times
   Purpose: Indicate cutting disk is actively spinning
   Effect: Safety interlock with table lock
   When I 72.7 = 1: Disk is cutting, table should stay locked
   When I 72.7 = 0: Disk stopped, table can be repositioned

⚠️ I 73.7 (V/S Velocity Signal) - Used 10 times
   Purpose: MASTER MOTION ENABLE - gates all movement
   Effect: If I 73.7 = 0, NO MOTION POSSIBLE (like E-stop)
   When I 73.7 = 1: Valid speed signal from Elbo, motion allowed
   When I 73.7 = 0: No speed signal, emergency/safe stop

================================================================================
SIGNAL CATEGORIES
================================================================================

SPEED SELECTION (2 signals):
  I 72.0 - Elbo Rapido (Fast speed)
  I 72.1 - Elbo Medio (Medium speed)

AXIS SELECTION (5 signals):
  I 73.0 - Eixo Corte (Z-Axis: Vertical/Blade)
  I 73.1 - Eixo Translacao (Y-Axis: Horizontal/Translation)
  I 73.2 - Eixo Cala (Tilt axis)
  I 73.3 - Eixo Rotacao Mesa (Table rotation)
  I 73.4 - Eixo Rotacao Disco (Disk rotation)

DIRECTION CONTROL (2 signals):
  I 73.5 - Direcao "+" (Positive: Up, Right, Forward, CW)
  I 73.6 - Direcao "-" (Negative: Down, Left, Backward, CCW)

MODE & SAFETY (2 signals):
  I 72.2 - Cmd Rot (Rotation mode selector)
  I 72.3 - Cmd LockTable (CRITICAL: Table lock during cutting)

STATUS FEEDBACK (4 signals):
  I 72.4 - Pressostato (Water pressure OK)
  I 72.5 - VFD1_IsRunning (Inverter 1 running)
  I 72.6 - VFD2_IsRunning (Inverter 2 running)
  I 72.7 - VFDDisk_IsRunning (CRITICAL: Disk running status)

MASTER ENABLE (1 signal):
  I 73.7 - V/S (CRITICAL: Velocity/Speed enable - gates all motion)

================================================================================
HOW THE SYSTEM WORKS - TYPICAL MOTION SEQUENCE
================================================================================

1. OPERATOR SELECTS SPEED via Elbo controller
   → I 72.0 = 1 (fast) or I 72.1 = 1 (medium)

2. OPERATOR SELECTS AXIS to move
   → One of I 73.0-73.4 = 1 (only one at a time)

3. OPERATOR SELECTS DIRECTION
   → I 73.5 = 1 (forward/up/right/CW) OR I 73.6 = 1 (backward/down/left/CCW)

4. ELBO PROVIDES VELOCITY SIGNAL
   → I 73.7 = 1 (valid speed signal)

5. PLC LOGIC PROCESSES:
   • Check I 73.7 = 1? (Master gate - if no, stop everything)
   • Check safety interlocks (I 72.3 and I 72.7)?
   • Determine appropriate contactor to energize

6. RESULT:
   Motor moves in selected direction at selected speed
   
7. OPERATOR RELEASES CONTROL:
   I 73.5 and I 73.6 both go to 0, or I 73.7 goes to 0
   → PLC de-energizes contactor → Motor stops

================================================================================
SAFETY INTERLOCKS
================================================================================

PREVENT CUTTING HAZARD:
  IF I 72.3 = 1 (table locked) THEN Q 7.3 = 0 (oil pump off)
  → Table cannot move during cutting operations

PREVENT MATERIAL SHIFT:
  IF I 72.7 = 1 (disk running) THEN prevent table movement
  → Material stays locked in place while blade is active

EMERGENCY STOP:
  IF I 73.7 = 0 THEN all Q 6.x = 0 (all motors stop)
  → Loss of velocity signal = immediate safe stop

================================================================================
MOST IMPORTANT FINDINGS
================================================================================

1. I 72.3 (Table Lock) is used 23 times in the code
   → Critical for preventing cutting hazards
   → Prevents table movement when I 72.3 = 1

2. I 73.7 (Velocity Enable) GATES ALL MOTION
   → Must be = 1 for any movement to occur
   → If I 73.7 = 0, system goes into safe stop

3. I 72.7 (Disk Running) works with I 72.3
   → Prevents table move when disk is cutting
   → Prevents accidental material shift

4. Axis signals I 73.0-73.4 select which of 5 motions
   → Only one should be active at a time
   → Typically mutually exclusive

5. Direction signals I 73.5 and I 73.6 control direction
   → Usually mutually exclusive
   → Both = 0 means no motion in that axis

6. Speed signals I 72.0 and I 72.1 determine velocity
   → Applied to all axes
   → Fast vs Medium operation modes

================================================================================
FOR YOUR NEXTION HMI PROJECT
================================================================================

When integrating the Nextion HMI for positioning control:

1. You'll need to SIMULATE the Elbo signals
   → Nextion commands → Convert to I 72.x and I 73.x pattern
   → PLC processes as if from Elbo controller

2. RESPECT safety interlocks
   → Don't allow table move if disk running
   → Don't allow motion if master enable off

3. PROVIDE position feedback
   → Read current position from encoders
   → Send back to Nextion for display

4. MAINTAIN emergency stop capability
   → Hardware emergency stop independent of HMI
   → Software E-stop via I 73.7

5. HONOR speed settings
   → Use I 72.0 or I 72.1 based on movement type
   → Fast for jog, appropriate speed for precise moves

6. VALIDATE position limits
   → Enforce in PLC, not just display
   → Add encoder feedback for safety

================================================================================
DOCUMENT RECOMMENDATIONS BY NEED
================================================================================

IF YOU NEED:                           LOOK AT:
─────────────────────────────────────────────────────────────────────────────
Quick answer about one signal          ELBO_SIGNALS_QUICK_REFERENCE.md
                                       (Summary Table)

Understanding the complete system      STEP5_PROJECT_ANALYSIS.md +
                                       COMPLETE_ELBO_SIGNAL_ANALYSIS.md

Visual explanation of signals          ELBO_VISUAL_REFERENCE_GUIDE.md

Troubleshooting a problem              ELBO_VISUAL_REFERENCE_GUIDE.md
                                       (Troubleshooting Checklist)

How signals work together              COMPLETE_ELBO_SIGNAL_ANALYSIS.md
                                       (Signal Hierarchy & Interlocks)

Integration example for HMI            ELBO_SIGNALS_QUICK_REFERENCE.md
                                       (Integration Examples section)

Complete master index                  ANALYSIS_INDEX_AND_SUMMARY.md

Understanding analysis methodology     I73_7_ANALYSIS.md
                                       (Shows how each signal was analyzed)

================================================================================
VERIFICATION CHECKLIST
================================================================================

After reading this analysis, you should be able to answer:

☐ What are the 3 critical signals?
☐ What does I 73.7 control?
☐ Why is I 72.3 used 23 times?
☐ How many axes can be controlled? Name them.
☐ What happens if I 73.7 = 0?
☐ How does the table lock system work?
☐ What's the relationship between I 72.3 and I 72.7?
☐ Describe a normal cutting sequence.
☐ Describe a table repositioning sequence.
☐ How would you integrate Nextion control?
☐ What signal prevents all motion?
☐ What signals provide feedback vs control?
☐ Why are there two speed signals?
☐ How do direction signals work?
☐ Explain the safety interlock logic.

================================================================================
GETTING HELP
================================================================================

1. Start with: ANALYSIS_INDEX_AND_SUMMARY.md
   → Gives you complete overview and quick answers

2. Need details?: COMPLETE_ELBO_SIGNAL_ANALYSIS.md
   → Every signal explained in depth

3. Visual learner?: ELBO_VISUAL_REFERENCE_GUIDE.md
   → Diagrams and visual explanations

4. Quick reference?: ELBO_SIGNALS_QUICK_REFERENCE.md
   → Tables and summary

5. Troubleshooting?: ELBO_VISUAL_REFERENCE_GUIDE.md
   → Troubleshooting Checklist section

================================================================================
CONTACT & INTEGRATION NEXT STEPS
================================================================================

This analysis document package provides:
✓ Complete understanding of all 16 Elbo signals
✓ Control logic for your BISSO system
✓ Safety interlock documentation
✓ Troubleshooting guides
✓ Integration examples for Nextion HMI

For your positioning controller project, you'll also need:
• Position encoder feedback implementation
• Step5 code blocks for UART communication (provided separately)
• Nextion HMI project (provided separately)
• Motor contactor wiring diagram (not provided)
• Encoder wiring specification (not provided)

================================================================================
END OF README
================================================================================

Next: Open ANALYSIS_INDEX_AND_SUMMARY.md for complete overview

Version: 1.0
Created: November 2025
Project: BISSO Stone Bridge Saw - Step5 Analysis
