# I 73.7 Control Analysis - BISSO Step5 Project

## Cross-Reference from QVL File

Based on the Quality Verification List (QVL) file analysis, here's what I found about **I 73.7**:

### Raw Cross-Reference Data:
```
:AN   I   73.7     2/AN    4/A     4/A
```

This means:
- **:AN** = Input is used with AND NOT (negated)
- **2/AN** = Used in 2 places with AND NOT operation
- **4/A** = Used in 4 places with AND operation  
- **4/A** = Used in 4 places with AND operation (appears again)

---

## What This Tells Us About I 73.7

### Usage Pattern:
```
Total References: 10 places in the code

1. 2 places where the code checks: IF I 73.7 = 0 (NOT active)
2. 4 places where the code checks: IF I 73.7 = 1 (active)
3. 4 additional places where I 73.7 = 1 is checked
```

### Context in the Program:

The line immediately following I 73.7 in the QVL shows:
```
:A    Q    6.0     1/A     5/A     7/R     7/O     9/O
```

This indicates that **Q 6.0** (Forward/Reverse Contactor) logic is directly related to I 73.7 processing.

---

## Interpretation: What I 73.7 Actually Controls

Based on the cross-reference pattern and your Elbo controller signal naming:

### **I 73.7 = Elbo V/S (Velocity/Speed Signal)**

This input is used to **GATE or ENABLE motion commands** based on whether the Elbo microcontroller is actively providing a velocity/speed signal.

### Most Likely Function:

**I 73.7 acts as a "velocity signal active" flag** that:

1. **Enables motion** when I 73.7 = 1 (Elbo is sending proportional speed)
2. **Inhibits motion** when I 73.7 = 0 (No speed signal from Elbo)

### Specific Logic Pattern:

```Step5 Logic (Reconstructed)

IF I 73.7 = 0 THEN
  ; No velocity signal from Elbo
  ; Stop or prevent motion
  CLEAR Q 6.0  ; Clear forward contactor
  CLEAR Q 6.1  ; Clear reverse contactor
  (etc.)
END

IF I 73.7 = 1 AND (other axis conditions) THEN
  ; Velocity signal is active from Elbo
  ; Allow motion based on other axis signals
  SET/CLEAR Q 6.x based on I 73.0-73.6 signals
END
```

---

## How I 73.7 Relates to Other Signals

### Signal Hierarchy:

```
Elbo Microcontroller sends:
  ├─ I 73.0 = Eixo Corte (Cutting axis - Z)
  ├─ I 73.1 = Eixo Translacao (Translation - Y)
  ├─ I 73.2 = Eixo Cala (Tilt)
  ├─ I 73.3 = Eixo Rotacao Mesa (Table rotation)
  ├─ I 73.4 = Eixo Rotacao Disco (Disk rotation)
  ├─ I 73.5 = Direcao "+" (Forward/Up direction)
  ├─ I 73.6 = Direcao "-" (Backward/Down direction)
  └─ I 73.7 = V/S (VELOCITY/SPEED - Master enable signal)
              └─ Acts as GATE to allow the above signals to take effect
```

### Why This Makes Sense:

The Elbo controller needs to signal:
- **WHICH axis** to move (I 73.0-73.4)
- **WHICH direction** (I 73.5-73.6)  
- **WHETHER THERE'S VALID SPEED DATA** (I 73.7)

If I 73.7 = 0, it could mean:
- Elbo is OFF or in standby
- Operator released the proportional control
- No valid speed reference
- Communication lost

---

## Impact on Output Contactors

The code uses I 73.7 to **gate** the following outputs:

### Primary Contactor Affected:
- **Q 6.0** = Forward/Reverse Contactor (cutting axis)

### Related Outputs (Likely Gated by Same Logic):
- **Q 6.1** = Left/Right Translation Contactor
- **Q 6.2** = Up/Down Vertical Contactor
- **Q 6.3** = Tilt Contactor
- **Q 6.4** = Table Rotation Contactor
- **Q 6.5** = Disk Rotation Contactor

---

## Summary: I 73.7 Controls

**Primary Function:** **VELOCITY/SPEED SIGNAL ACTIVE FLAG**

**Role in System:** Acts as a **master gate/enable** that allows axis movement signals to take effect

**When Active (I 73.7 = 1):**
- The PLC processes axis commands from I 73.0-73.6
- Contactors (Q 6.x) are allowed to energize
- Motors can move based on axis and direction signals

**When Inactive (I 73.7 = 0):**
- All axis signals from Elbo are ignored
- Contactors are prevented from energizing
- System is effectively in "stopped" state
- Emergency/safety condition

**Analogy:** I 73.7 is like the "enable" or "armed" signal for the entire proportional motion system. Without it being active, even if you send axis and direction signals, nothing will move.

