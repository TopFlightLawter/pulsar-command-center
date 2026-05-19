# IR LED Wiring Guide for ESP32 (Pulsar Rebirth)

Covers 2, 4, 5, 6, 7, 8, 9, and 10 IR LEDs wired for constant illumination from an ESP32.

> **Recommended build:** ULN2003AN driver + 5 V external supply. See the
> [ULN2003AN section](#uln2003an-driver--5-v-external-supply) for the configurations
> to actually build. The MOSFET/12 V content below is retained as a reference alternative.

---

## Baseline Assumptions

| Parameter | Value | Notes |
|---|---|---|
| IR LED forward voltage (Vf) | 1.2 V | Typical 940 nm IR LED (TSAL6100, etc.) — check your datasheet |
| Target continuous current (If) | 50 mA | Good brightness, thermally safe for continuous on |
| ESP32 GPIO output | 3.3 V logic | Never sink/source >10 mA continuously from a GPIO |
| "IRLL LEDs" | Treated as standard IR LEDs | If Vf differs, recalculate resistor using R = (Vsupply − n×Vf) / If |

---

## Universal Rule: Always Use a Driver

ESP32 GPIOs are not rated for continuous LED current. Route GPIO through a **transistor or
logic-level MOSFET** so the GPIO only switches the driver, not the LEDs directly.

### Option A — NPN Transistor (BC547, 2N2222, PN2222A)

```
Vsupply ──── R_LED ──── LED string(s) ──┐
                                        Collector
ESP32 GPIO ──── 1kΩ ──── Base        (NPN transistor)
                                        Emitter ──── GND
```

- Simple, cheap, widely available
- Suitable up to ~200 mA collector current (2N2222)

### Option B — Logic-Level N-Channel MOSFET (IRLZ44N) ← Recommended

```
Vsupply ──── R_LED ──── LED string(s) ──┐
                                        Drain
ESP32 GPIO ──── 100Ω ──── Gate       (IRLZ44N)
                                        Source ──── GND
```

- IRLZ44N: Vgs(th) ~1–2 V → **fully on at 3.3 V GPIO** (critical for ESP32)
- Handles up to 47 A, so no heat sink needed for these loads
- 100 Ω gate resistor dampens ringing; not strictly required for DC use but good practice
- Add a 10 kΩ resistor from Gate to GND to keep the MOSFET off during ESP32 boot

---

## Supply Voltage Strategy

| Supply | Max LEDs per series string | Resistor formula |
|---|---|---|
| 5 V | 2 | R = (5 − 2×1.2) / 0.05 = **52 Ω → use 47 Ω** |
| 12 V | 7 | R = (12 − n×1.2) / 0.05 |

> **Rule of thumb:** Use 5 V for 2–4 LEDs. Use 12 V for 4–10 LEDs.
> Multiple parallel strings all share one MOSFET/transistor unless you need independent control.

---

## Configurations by LED Count

---

### 2 IR LEDs

**Recommended: 5 V supply, 1 series string**

```
5V ──── 47Ω ──── [LED1] ──── [LED2] ──── DRAIN
                                              |
GPIO ── 100Ω ── GATE (IRLZ44N)           SOURCE ──── GND
```

| Item | Value |
|---|---|
| Supply | 5 V |
| String layout | 1 × (2 in series) |
| Resistor | 47 Ω |
| Current per string | ~55 mA |
| Total current | ~55 mA |
| GPIO pins needed | 1 |

---

### 4 IR LEDs ("IRLL LEDs")

**Recommended: 12 V supply, 1 series string of 4**

```
12V ──── 150Ω ──── [LED1]──[LED2]──[LED3]──[LED4] ──── DRAIN
                                                             |
GPIO ── 100Ω ── GATE (IRLZ44N)                          SOURCE ──── GND
```

| Item | Value |
|---|---|
| Supply | 12 V |
| String layout | 1 × (4 in series) |
| Resistor | 150 Ω |
| Current | ~48 mA |
| Total current | ~48 mA |
| GPIO pins needed | 1 |

> **5 V alternative:** 2 parallel strings of 2, each with a 47 Ω resistor, one MOSFET → ~110 mA total.

---

### 5 IR LEDs

**Recommended: 12 V supply, 1 series string of 5**

```
12V ──── 120Ω ──── [LED1]──[LED2]──[LED3]──[LED4]──[LED5] ──── DRAIN
                                                                      |
GPIO ── 100Ω ── GATE (IRLZ44N)                                   SOURCE ──── GND
```

| Item | Value |
|---|---|
| Supply | 12 V |
| String layout | 1 × (5 in series) |
| Resistor | 120 Ω |
| Current | 50 mA |
| Total current | 50 mA |
| GPIO pins needed | 1 |

> **5 V alternative:** 2 strings of 2 (47 Ω each) + 1 string of 1 (82 Ω) → 3 resistors, 1 MOSFET, ~150 mA.

---

### 6 IR LEDs

**Recommended: 12 V supply, 1 series string of 6**

```
12V ──── 100Ω ──── [LED1]──[LED2]──[LED3]──[LED4]──[LED5]──[LED6] ──── DRAIN
                                                                              |
GPIO ── 100Ω ── GATE (IRLZ44N)                                           SOURCE ──── GND
```

| Item | Value |
|---|---|
| Supply | 12 V |
| String layout | 1 × (6 in series) |
| Resistor | 100 Ω |
| Current | ~48 mA |
| Total current | ~48 mA |
| GPIO pins needed | 1 |

> **If you want more brightness:** Split into 2 strings of 3 on 12 V, each with 180 Ω → ~93 mA total, same 1 MOSFET.

---

### 7 IR LEDs

**Recommended: 12 V supply, 2 parallel strings (4 + 3)**

```
12V ──┬── 150Ω ──── [LED1]──[LED2]──[LED3]──[LED4] ──┐
      └── 180Ω ──── [LED5]──[LED6]──[LED7]            ├──── DRAIN
                                                            |
GPIO ── 100Ω ── GATE (IRLZ44N)                         SOURCE ──── GND
```

| Item | Value |
|---|---|
| Supply | 12 V |
| String layout | 1 × (4 in series) + 1 × (3 in series) |
| Resistors | 150 Ω (4-LED string) + 180 Ω (3-LED string) |
| Current per string | ~48 mA each |
| Total current | ~96 mA |
| GPIO pins needed | 1 |

> **5 V alternative:** 3 strings of 2 (47 Ω each) + 1 string of 1 (82 Ω) → 4 resistors, 1 MOSFET, ~220 mA.

---

### 8 IR LEDs

**Recommended: 12 V supply, 2 parallel strings of 4**

```
12V ──┬── 150Ω ──── [LED1]──[LED2]──[LED3]──[LED4] ──┐
      └── 150Ω ──── [LED5]──[LED6]──[LED7]──[LED8] ──┼──── DRAIN
                                                            |
GPIO ── 100Ω ── GATE (IRLZ44N)                         SOURCE ──── GND
```

| Item | Value |
|---|---|
| Supply | 12 V |
| String layout | 2 × (4 in series) |
| Resistors | 2 × 150 Ω |
| Current per string | ~48 mA |
| Total current | ~96 mA |
| GPIO pins needed | 1 |

> This is one of the cleanest configurations — perfectly balanced strings, single driver.

---

### 9 IR LEDs

**Recommended: 12 V supply, 2 parallel strings (5 + 4)**

```
12V ──┬── 120Ω ──── [LED1]──[LED2]──[LED3]──[LED4]──[LED5] ──┐
      └── 150Ω ──── [LED6]──[LED7]──[LED8]──[LED9]            ├──── DRAIN
                                                                    |
GPIO ── 100Ω ── GATE (IRLZ44N)                               SOURCE ──── GND
```

| Item | Value |
|---|---|
| Supply | 12 V |
| String layout | 1 × (5 in series) + 1 × (4 in series) |
| Resistors | 120 Ω (5-LED string) + 150 Ω (4-LED string) |
| Current per string | ~50 mA each |
| Total current | ~98 mA |
| GPIO pins needed | 1 |

> **Balanced alternative:** 3 strings of 3 (each 180 Ω on 12 V) → ~140 mA, same 1 MOSFET.

---

### 10 IR LEDs

**Recommended: 12 V supply, 2 parallel strings of 5**

```
12V ──┬── 120Ω ──── [LED1]──[LED2]──[LED3]──[LED4]──[LED5]  ──┐
      └── 120Ω ──── [LED6]──[LED7]──[LED8]──[LED9]──[LED10] ──┼──── DRAIN
                                                                     |
GPIO ── 100Ω ── GATE (IRLZ44N)                                SOURCE ──── GND
```

| Item | Value |
|---|---|
| Supply | 12 V |
| String layout | 2 × (5 in series) |
| Resistors | 2 × 120 Ω |
| Current per string | 50 mA |
| Total current | 100 mA |
| GPIO pins needed | 1 |

> Perfectly balanced, lowest part count for 10 LEDs. This is the ideal maximum for a
> single 12 V / IRLZ44N channel at 50 mA/LED.

---

## Quick Reference Table

| LED Count | Supply | String Layout | Resistor(s) | Total Current | GPIO Pins |
|---|---|---|---|---|---|
| 2 | 5 V | 1 × 2 | 47 Ω | ~55 mA | 1 |
| 4 | 12 V | 1 × 4 | 150 Ω | ~48 mA | 1 |
| 5 | 12 V | 1 × 5 | 120 Ω | 50 mA | 1 |
| 6 | 12 V | 1 × 6 | 100 Ω | ~48 mA | 1 |
| 7 | 12 V | 1×4 + 1×3 | 150 Ω + 180 Ω | ~96 mA | 1 |
| 8 | 12 V | 2 × 4 | 2 × 150 Ω | ~96 mA | 1 |
| 9 | 12 V | 1×5 + 1×4 | 120 Ω + 150 Ω | ~98 mA | 1 |
| 10 | 12 V | 2 × 5 | 2 × 120 Ω | 100 mA | 1 |

All values assume Vf = 1.2 V and If = 50 mA. MOSFET driver: IRLZ44N.

---

## Resistor Calculation Formula

```
R (Ω) = (Vsupply − n × Vf) / If

Where:
  Vsupply = supply voltage (V)
  n       = number of LEDs in series
  Vf      = LED forward voltage (V) — from datasheet
  If      = desired current (A)
```

**Example:** 3 LEDs in series on 12 V at 50 mA:
`R = (12 − 3 × 1.2) / 0.05 = (12 − 3.6) / 0.05 = 168 Ω → use 180 Ω standard value`

Use the **next higher standard resistor value** (E24 or E12 series) to keep current at or
slightly below your target. Running at 45–50 mA continuous is fine for rated LEDs; 
going over will shorten LED life.

---

## Power Dissipation Check

The resistor dissipates power: **P = I² × R**

Example for 10 LED setup (2 × 120 Ω at 50 mA each):
- P per resistor = 0.05² × 120 = **0.30 W**
- Use ½ W resistors minimum, 1 W for margin

The IRLZ44N at 100 mA total with Rds(on) < 0.022 Ω dissipates < 0.001 W — negligible.

---

## ESP32 Firmware Note

For "constantly illuminated," the simplest firmware approach is to set the GPIO high in `setup()`
and leave it. No PWM or timer needed:

```cpp
#define IR_LED_PIN 25

void setup() {
  pinMode(IR_LED_PIN, OUTPUT);
  digitalWrite(IR_LED_PIN, HIGH);  // LEDs on continuously
}

void loop() {
  // nothing needed for IR illumination
}
```

If you want software control (e.g., turn off IR when device sleeps), you can call
`digitalWrite(IR_LED_PIN, LOW)` to cut power via the MOSFET.

---

## Parts List Summary

| Component | Purpose | Suggested Part |
|---|---|---|
| N-ch MOSFET | LED driver | IRLZ44N (through-hole), AO3400A (SMD) |
| Gate resistor | Ringing suppression | 100 Ω, ¼ W |
| Gate pull-down | Boot-state safety | 10 kΩ, ¼ W |
| Current resistors | Set LED current | See table above, ½ W minimum |
| Decoupling cap | Supply noise | 100 µF electrolytic near supply |

---

---

## ULN2003AN Driver + 5 V External Supply

This section covers the recommended build: standalone IR LEDs, a **ULN2003AN**
Darlington array IC, and a **5 V supply isolated from the ESP32 circuit**.

### Why the ULN2003AN Works Well Here

- 7 Darlington channels in one DIP-16 package — no discrete transistors or MOSFETs needed
- Each channel sinks up to **500 mA** (continuous) — more than enough for multiple LED strings
- Inputs switch on at ~1 V and are fully compatible with **ESP32 3.3 V GPIO** — no level shifter required
- Built-in 2.7 kΩ base resistors — connect GPIO directly to input pin, nothing in between
- **Active HIGH:** GPIO HIGH → output sinks current → LEDs on

### Voltage Budget (Why Resistors Change)

The ULN2003AN's Darlington pair drops approximately **1.0 V** (Vce_sat) at 50–100 mA.
That voltage is gone before it reaches the LEDs, so:

```
V_available = 5.0 V − 1.0 V (ULN2003AN drop) = 4.0 V for LEDs + resistor
```

Updated resistor formula:

```
R = (4.0 − n × Vf) / If
  = (4.0 − n × 1.2) / 0.05
```

### Maximum Series Length: 2 LEDs per String

With 5 V and a ULN2003AN, 3 LEDs in series leaves only ~0.4 V across the resistor —
too little headroom. Any Vf variation (1.2–1.4 V is typical) would cause uncontrolled
or zero current. **Keep every series string at 2 LEDs maximum.**

### Circuit Topology

```
                   ┌─ 33Ω ─── [LED] ─── [LED] ─┐
5V (external) ─────┤                             ├──── OUT1 (ULN2003AN pin 11)
                   └─ 33Ω ─── [LED] ─── [LED] ─┘          │
                                                        (sinks to GND internally)
ESP32 GPIO ──────────────────────────────────────── IN1  (ULN2003AN pin 1)

COM pin (pin 9) ── GND (shared with ESP32 GND)
```

- **Each parallel string must have its own resistor** — never share one resistor across
  parallel strings (current will be uneven)
- All strings on the same channel share one GPIO pin
- The ULN2003AN COM pin (pin 9) ties to GND and must be common with the ESP32 GND

### Per-Count Configurations (5 V + ULN2003AN)

All configurations below use **one ULN2003AN channel** and **one ESP32 GPIO pin**,
since all LEDs are constantly on together.

---

#### 2 IR LEDs

```
5V ──── 33Ω ──── [LED1] ──── [LED2] ──── OUT1
ESP32 GPIO ──────────────────────────── IN1
```

| Item | Value |
|---|---|
| String layout | 1 × (2 in series) |
| Resistor | 33 Ω |
| Current | ~48 mA |
| ULN2003AN channels used | 1 |
| GPIO pins | 1 |

---

#### 4 IR LEDs

```
5V ──┬── 33Ω ──── [LED1] ──── [LED2] ──┐
     └── 33Ω ──── [LED3] ──── [LED4] ──┴──── OUT1
ESP32 GPIO ──────────────────────────────── IN1
```

| Item | Value |
|---|---|
| String layout | 2 × (2 in series), parallel |
| Resistors | 2 × 33 Ω |
| Current per string | ~48 mA |
| Total current | ~96 mA |
| ULN2003AN channels used | 1 |
| GPIO pins | 1 |

---

#### 5 IR LEDs

```
5V ──┬── 33Ω ──── [LED1] ──── [LED2] ──┐
     ├── 33Ω ──── [LED3] ──── [LED4] ──┤
     └── 56Ω ──── [LED5] ─────────────┴──── OUT1
ESP32 GPIO ──────────────────────────────── IN1
```

| Item | Value |
|---|---|
| String layout | 2 × (2 in series) + 1 × (1) |
| Resistors | 2 × 33 Ω + 1 × 56 Ω |
| Total current | ~144 mA |
| ULN2003AN channels used | 1 |
| GPIO pins | 1 |

---

#### 6 IR LEDs

```
5V ──┬── 33Ω ──── [LED1] ──── [LED2] ──┐
     ├── 33Ω ──── [LED3] ──── [LED4] ──┤
     └── 33Ω ──── [LED5] ──── [LED6] ──┴──── OUT1
ESP32 GPIO ──────────────────────────────── IN1
```

| Item | Value |
|---|---|
| String layout | 3 × (2 in series), parallel |
| Resistors | 3 × 33 Ω |
| Total current | ~144 mA |
| ULN2003AN channels used | 1 |
| GPIO pins | 1 |

---

#### 7 IR LEDs

| Item | Value |
|---|---|
| String layout | 3 × (2 in series) + 1 × (1) |
| Resistors | 3 × 33 Ω + 1 × 56 Ω |
| Total current | ~192 mA |
| ULN2003AN channels used | 1 |
| GPIO pins | 1 |

---

#### 8 IR LEDs

| Item | Value |
|---|---|
| String layout | 4 × (2 in series), parallel |
| Resistors | 4 × 33 Ω |
| Total current | ~192 mA |
| ULN2003AN channels used | 1 |
| GPIO pins | 1 |

---

#### 9 IR LEDs

| Item | Value |
|---|---|
| String layout | 4 × (2 in series) + 1 × (1) |
| Resistors | 4 × 33 Ω + 1 × 56 Ω |
| Total current | ~240 mA |
| ULN2003AN channels used | 1 |
| GPIO pins | 1 |

---

#### 10 IR LEDs

| Item | Value |
|---|---|
| String layout | 5 × (2 in series), parallel |
| Resistors | 5 × 33 Ω |
| Total current | ~240 mA |
| ULN2003AN channels used | 1 |
| GPIO pins | 1 |

---

### ULN2003AN Quick Reference Table

| LED Count | String Layout | Resistors | Total Current | Channels Used |
|---|---|---|---|---|
| 2 | 1 × 2 | 1 × 33 Ω | ~48 mA | 1 |
| 4 | 2 × 2 | 2 × 33 Ω | ~96 mA | 1 |
| 5 | 2×2 + 1×1 | 2 × 33 Ω, 1 × 56 Ω | ~144 mA | 1 |
| 6 | 3 × 2 | 3 × 33 Ω | ~144 mA | 1 |
| 7 | 3×2 + 1×1 | 3 × 33 Ω, 1 × 56 Ω | ~192 mA | 1 |
| 8 | 4 × 2 | 4 × 33 Ω | ~192 mA | 1 |
| 9 | 4×2 + 1×1 | 4 × 33 Ω, 1 × 56 Ω | ~240 mA | 1 |
| 10 | 5 × 2 | 5 × 33 Ω | ~240 mA | 1 |

All values: Vf = 1.2 V, If = 50 mA target, ULN2003AN Vce_sat = 1.0 V, 5 V supply.

### ULN2003AN Pinout (DIP-16)

```
         ┌──────────┐
  IN1  1 │          │ 16  IN2
  IN3  2 │          │ 15  IN4
  IN5  3 │          │ 14  IN6
  IN7  4 │          │ 13  GND (not used for LEDs)
 OUT7  5 │ULN2003AN │ 12  OUT6
 OUT5  6 │          │ 11  OUT4 (wait — standard pinout below)
```

Standard pinout (verify against your datasheet — the above is illustrative):

| Pin | Function |
|---|---|
| 1–7 | Inputs (IN1–IN7) — connect to ESP32 GPIO |
| 8 | GND — connect to common ground |
| 9 | COM — connect to GND (for LED use; used for inductive flyback on motor loads) |
| 10–16 | Outputs (OUT7–OUT1) — connect to LED string low-side ends |

> For LED loads, tie COM (pin 9) to GND. The built-in flyback diodes on COM are for
> inductive loads (motors, relays) and have no effect on LEDs.

### Firmware (Constant On)

```cpp
#define IR_LED_PIN 25  // connects to ULN2003AN IN1

void setup() {
  pinMode(IR_LED_PIN, OUTPUT);
  digitalWrite(IR_LED_PIN, HIGH);  // HIGH = Darlington on = LEDs lit
}

void loop() {}
```

### Parts List (ULN2003AN Build)

| Component | Value | Notes |
|---|---|---|
| IC | ULN2003AN | DIP-16, 7-channel Darlington array |
| Current resistors | 33 Ω (pairs), 56 Ω (singles) | ½ W minimum; see table |
| 5 V supply | External, isolated from ESP32 | Sized for total current + 20% margin |
| Decoupling cap | 100 µF electrolytic | Across 5 V supply, close to IC |
| Wire | — | Keep LED supply wiring short and direct |
