# Elbo Signal Reference Guide - Visual & Wiring

## Quick Reference Card

### INPUT BYTE I 72.x - Speed & Mode Control
```
┌─────────────────────────────────────┐
│ INPUT BYTE I 72.x (Speed/Mode)     │
├─────────────────────────────────────┤
│ Bit 0: I 72.0 = Elbo Rapido (Fast) │
│ Bit 1: I 72.1 = Elbo Medio (Medium)│
│ Bit 2: I 72.2 = Cmd Rot (Rotation) │
│ Bit 3: I 72.3 = Cmd LockTable ⚠️ │
│ Bit 4: I 72.4 = Pressostato        │
│ Bit 5: I 72.5 = VFD1_IsRunning     │
│ Bit 6: I 72.6 = VFD2_IsRunning     │
│ Bit 7: I 72.7 = VFDDisk_IsRunning  │
└─────────────────────────────────────┘
```

### INPUT BYTE I 73.x - Axis & Direction Control
```
┌─────────────────────────────────────┐
│ INPUT BYTE I 73.x (Axis/Direction)  │
├─────────────────────────────────────┤
│ Bit 0: I 73.0 = Eixo Corte (Z)     │
│ Bit 1: I 73.1 = Eixo Translacao(Y) │
│ Bit 2: I 73.2 = Eixo Cala (Tilt)   │
│ Bit 3: I 73.3 = Rot Mesa (Table)   │
│ Bit 4: I 73.4 = Rot Disco (Disk)   │
│ Bit 5: I 73.5 = Direcao "+" (→)    │
│ Bit 6: I 73.6 = Direcao "-" (←)    │
│ Bit 7: I 73.7 = V/S Enable ⚠️     │
└─────────────────────────────────────┘
```

---

## Speed Control Logic Diagram

```
        ┌─────────────┐
        │ Elbo Speed  │
        │  Selector   │
        └──────┬──────┘
          ┌────┴────┐
          ▼         ▼
        I 72.0    I 72.1
       (Fast)   (Medium)
          │         │
          ▼         ▼
      ┌───────────────┐
      │  Speed Logic  │
      │  Selector     │
      └───────┬───────┘
              │
         ┌────┴────┬──────┬──────┬──────┬──────┐
         ▼         ▼      ▼      ▼      ▼      ▼
       Q 6.0     Q 6.1  Q 6.2  Q 6.3  Q 6.4  Q 6.5
       (Fwd)     (L/R)  (U/D)  (Tilt) (TRot) (DRot)
         │         │      │      │      │      │
         └─────────┴──────┴──────┴──────┴──────┘
              Speed Applied to All Axes
```

---

## Axis Selection Logic Diagram

```
Elbo Joystick Position → I 73.0-73.4 (Axis Select)
                        I 73.5-73.6 (Direction)
                            │
                        ┌───┼───┐
                        │   │   │
        ┌───────────────┼───┴───┴───────────────┐
        │               │                        │
    I 73.0 (Z)      I 73.1 (Y)              I 73.2 (Tilt)
    │                 │                        │
    ▼                 ▼                        ▼
   Q 6.2           Q 6.1                     Q 6.3
(Up/Down)       (Left/Right)              (Tilt Angle)
    │                 │                        │
    ├─────────────────┼────────────────────────┤
    │                 │                        │
Direction        Direction                 Direction
I 73.5 (+):       I 73.5 (+):              I 73.5 (+):
  UP               RIGHT                     UP-TILT
I 73.6 (-):       I 73.6 (-):              I 73.6 (-):
  DOWN             LEFT                      DOWN-TILT
```

---

## Table Lock Safety Interlock

```
                    I 72.3 (Cmd LockTable)
                            │
                ┌───────────┴───────────┐
                │                       │
            = 0 (Unlock)            = 1 (Lock)
                │                       │
                ▼                       ▼
        ┌──────────────┐        ┌──────────────┐
        │ ALLOW TABLE  │        │ LOCK TABLE   │
        │ MOVEMENT     │        │ PREVENT MOVE │
        └──────┬───────┘        └──────┬───────┘
               │                       │
          ┌────┴────┐            ┌────┴────┐
          ▼         ▼            ▼         ▼
       Q 7.3     Q 72.5-7    BLOCK Q 7.3  SET Q 8.7
      (Pump)   (Relays)      (Pump)     (Lock)
               (Up/Down)
               
        ▼▼▼ CRITICAL SAFETY ▼▼▼
    When I 72.3 = 1:
    - Oil Pump de-energizes (Table cannot move)
    - Cutting operations allowed (I 72.7 can = 1)
    - Prevents material movement during active cutting
```

---

## Disk Running Safety Interlock

```
                I 72.7 (VFDDisk_IsRunning)
                        │
                ┌───────┴───────┐
                │               │
            = 0 (OFF)       = 1 (Running)
            DISK IDLE       DISK CUTTING
                │               │
                ▼               ▼
        ┌──────────────┐  ┌──────────────┐
        │ CAN MOVE     │  │ CANNOT MOVE  │
        │ TABLE        │  │ TABLE        │
        │ SAFE         │  │ MATERIAL     │
        │              │  │ LOCKED       │
        └──────────────┘  └──────────────┘
        
    ▼▼▼ MUTUAL EXCLUSION ▼▼▼
    I 72.3 and I 72.7 together create:
    - When cutting (I 72.7=1): Table locked
    - When moving (I 72.7=0 & I 72.3=0): Disk off
```

---

## Velocity Signal Master Enable

```
                    I 73.7 (V/S Enable)
                            │
                ┌───────────┴───────────┐
                │                       │
            = 0 (No Signal)          = 1 (Valid Signal)
                │                       │
                ▼                       ▼
        ┌──────────────┐        ┌──────────────┐
        │ EMERGENCY    │        │ MOTION       │
        │ STOP / SAFE  │        │ ENABLED      │
        │              │        │              │
        │ ALL Q 6.x =0 │        │ Process:     │
        │ NO MOTION    │        │ Axis I 73.0-4│
        │              │        │ Direction 5-6│
        └──────────────┘        └──────────────┘
        
    ▼▼▼ MASTER GATE ▼▼▼
    I 73.7 must be = 1 for ANY motion to occur
    Loss of I 73.7 = Immediate safe stop
```

---

## Complete Signal Flow Diagram

```
╔═════════════════════════════════════════════════════════╗
║         ELBO MICROCONTROLLER OUTPUTS                   ║
║         (16 Digital Signals to PLC)                    ║
╚═════════════════════════════════════════════════════════╝
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
   ┌─────────┐      ┌─────────┐     ┌─────────┐
   │ I 72.x  │      │ I 73.x  │     │ Feedback│
   │ Byte    │      │ Byte    │     │ Inputs  │
   └────┬────┘      └────┬────┘     └────┬────┘
        │                │               │
   ┌────┴────────────────┴───────────────┴────┐
   │  STEP5 PLC LOGIC PROCESSING               │
   │  ─────────────────────────────────────    │
   │  1. Read all Elbo inputs                  │
   │  2. Check speed mode (I 72.0 or 72.1)    │
   │  3. Check master enable (I 73.7)         │
   │  4. Check safety locks (I 72.3, I 72.7)  │
   │  5. Read axis selection (I 73.0-73.4)    │
   │  6. Read direction (I 73.5-73.6)         │
   │  7. Apply conditional logic               │
   │  8. Energize appropriate contactors      │
   └────┬────────────────┬───────────────┬────┘
        │                │               │
        ▼                ▼               ▼
   ┌─────────┐      ┌─────────┐    ┌─────────┐
   │ Q 6.x   │      │ Q 7.x   │    │ Q 8.x   │
   │ Motors  │      │ VFDs &  │    │ Inverter│
   │         │      │ Relays  │    │ Commands│
   └────┬────┘      └────┬────┘    └────┬────┘
        │                │              │
        └────────────────┼──────────────┘
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
   ┌─────────┐      ┌─────────┐     ┌─────────┐
   │ Motors  │      │ Hydraulic   │ Proportional
   │ (via    │      │ Pump/Valves │ Drives
   │ Contac- │      │ (via Relays)│ (via VFD)
   │ tors)   │      │             │
   └────┬────┘      └────┬────┘     └────┬────┘
        │                │               │
        └────────────────┼───────────────┘
                         │
        ┌────────────────┴─────────────────┐
        │                                   │
        ▼                                   ▼
   CUTTING BLADE MOTION            TABLE POSITIONING
   - Vertical (Z-axis)             - Translation (Y-axis)
   - Speed regulation              - Rotation
   - Up/Down control               - Lock/Unlock
```

---

## Signal Timing Diagram

```
Time →

Operator Action    Signal Status                  Motor Response
─────────────────────────────────────────────────────────────────
Startup
I 72.0────────────  ═════════════ (Fast Selected)
I 72.3────────────  ═════════════ (Table Locked - Safe)
I 73.7────────────  ═════════════ (Speed Valid)
               │
Select Cut     │
I 73.0─────────┼───  ═════════════ (Vertical axis selected)
I 73.5─────────┼───  ═════════════ (Up direction)
               │                                Q 6.2─  ═════════════ (Blade up)
               │
Move Down      │
I 73.5─────────┼───  ════════════════ (Still up)
I 73.6─────────┼───  ════════════════ (Switch to down)
               │                                Q 6.2─  ════════ (Blade down)
               │
Release        │
I 73.5────────────  ══════════════════ (Down)
I 73.6────────────  ══════════════════ (Down)
I 73.7────────────  ══════════════════ (Signal valid)
               └────── Then to ─────
I 73.5────────────  ═══════════════════ (Release = Goes to 0)
I 73.6────────────  ═══════════════════ (Release = Goes to 0)
I 73.7────────────  ═══════════════════ (May go to 0)
                                        Q 6.2─  ════════ (Blade stops)
```

---

## Contactor Activation Matrix

```
╔════════════════════════════════════════════════════════════════════════════╗
║ WHICH CONTACTOR ENERGIZES BASED ON SIGNAL COMBINATION?                   ║
╠════════════════════════════════════════════════════════════════════════════╣
║                                                                            ║
║ I 73.7 = 0  → NO CONTACTORS (Emergency Stop / No Signal)                 ║
║              → ALL Q 6.x = 0 (Safe state)                                 ║
║                                                                            ║
║ I 73.7 = 1, I 73.0 = 1, I 73.5 = 1  → Q 6.2 = 1  (Blade UP)             ║
║ I 73.7 = 1, I 73.0 = 1, I 73.6 = 1  → Q 6.2 = 1  (Blade DOWN)           ║
║                                                                            ║
║ I 73.7 = 1, I 73.1 = 1, I 73.5 = 1  → Q 6.1 = 1  (Move RIGHT)           ║
║ I 73.7 = 1, I 73.1 = 1, I 73.6 = 1  → Q 6.1 = 0  (Move LEFT)            ║
║                                                                            ║
║ I 73.7 = 1, I 73.2 = 1, I 73.5 = 1  → Q 6.3 = 1  (Tilt UP)              ║
║ I 73.7 = 1, I 73.2 = 1, I 73.6 = 1  → Q 6.3 = 0  (Tilt DOWN)            ║
║                                                                            ║
║ I 73.7 = 1, I 73.3 = 1, I 72.2 = 1, I 73.5 = 1  → Q 6.4 = 1  (CW)       ║
║ I 73.7 = 1, I 73.3 = 1, I 72.2 = 1, I 73.6 = 1  → Q 6.4 = 0  (CCW)      ║
║                                                                            ║
║ I 73.7 = 1, I 73.4 = 1, I 72.2 = 1               → Q 6.5 = 1  (Disk)     ║
║                                                                            ║
║ I 72.3 = 1  → Q 7.3 = 0  (Oil Pump OFF - Table Locked)                   ║
║              → Cannot move tables regardless of other signals             ║
║                                                                            ║
║ I 72.7 = 1  → Table movement locked (material cannot move during cut)    ║
║              → Only cutting operations allowed                            ║
║                                                                            ║
╚════════════════════════════════════════════════════════════════════════════╝
```

---

## Electrical Wiring Reference

```
Elbo Microcontroller → PLC Input Terminals:

FROM ELBO:                           TO PLC:
─────────                           ───────
Input 1 (Fast/Slow)              → I 72.0 (bit 0)
Input 2 (Medium Speed)            → I 72.1 (bit 1)  
Input 3 (Rotation Cmd)            → I 72.2 (bit 2)
Input 4 (Lock Table)              → I 72.3 (bit 3) ⚠️ CRITICAL
Input 5 (Pressure Status)         → I 72.4 (bit 4)
Input 6 (VFD1 Running)            → I 72.5 (bit 5)
Input 7 (VFD2 Running)            → I 72.6 (bit 6)
Input 8 (Disk Running)            → I 72.7 (bit 7) ⚠️ SAFETY
Input 9 (Axis Cut)                → I 73.0 (bit 0)
Input 10 (Axis Translate)         → I 73.1 (bit 1)
Input 11 (Axis Tilt)              → I 73.2 (bit 2)
Input 12 (Axis Table Rotation)    → I 73.3 (bit 3)
Input 13 (Axis Disk Rotation)     → I 73.4 (bit 4)
Input 14 (Direction Forward/Up)   → I 73.5 (bit 5)
Input 15 (Direction Backward/Down)→ I 73.6 (bit 6)
Input 16 (Velocity Enable)        → I 73.7 (bit 7) ⚠️ CRITICAL

COMMON/GROUND                      → PGND (All inputs)
```

---

## Troubleshooting Checklist

```
Symptom: Motors won't move
Check: ├─ I 73.7 = 1? (Velocity signal active)
       ├─ I 73.0-73.4 = 1? (Axis selected)
       ├─ I 73.5 or 73.6 = 1? (Direction selected)
       ├─ I 72.0 or 72.1 = 1? (Speed selected)
       ├─ I 72.3 = 0? (Table not locked during motion)
       └─ Q 6.x outputting? (Contactor energizing)

Symptom: Table moves when shouldn't
Check: ├─ I 72.3 state when I 72.7 = 1? (Should prevent during cutting)
       ├─ I 72.7 = 0 for table movement? (Cutting disk should be off)
       └─ Q 7.3 controlled by I 72.3? (Oil pump gated correctly)

Symptom: No blade motion
Check: ├─ I 73.0 = 1? (Vertical axis selected)
       ├─ I 73.5 or 73.6 = 1? (Direction selected)
       ├─ Q 6.2 energizing? (Up/Down contactor)
       └─ End switches active? (Not at limit)

Symptom: Erratic or stuck motion
Check: ├─ I 73.7 signal intermittent? (Velocity signal noisy)
       ├─ Check Elbo microcontroller output stable?
       ├─ Any signals shorting/floating?
       └─ PLC input module working correctly?
```

