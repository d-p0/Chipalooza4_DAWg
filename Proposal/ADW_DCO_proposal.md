# DCO


### Type of ip block: DCO

### Team name: ADW

## Summary

An on-chip clock generator (can digitally control clock speed). Starts as plain ring oscillator and frequency divider, then progresses to a current-starved oscillator (frequency is set by a digital tuning word over the harness's SPI/digital inputs). 

No crystal or external clock is needed. It uses only basic CMOS parts (inverters, switches, a counter), so there is no op-amp and the layout is repetitive and manageable.

---

Planning to do it as a progression, so each step is still acceptable to turn into a final result. (see progression subheading)

---
### Type of IP block

Digitally-tuned current-starved ring oscillator (DCO) with output divider, on Sky130.

(set final feature after simulation)

### Functional description

- **Ring of inverters:** an odd number of inverters in a loop. The signal keeps chasing itself around, so the output switches continuously.
- **Current-starved inverters:** each inverter gets its current through extra transistors. Less current makes it switch slower, and more current makes it faster.
- **Tuning bank:** a set of transistors of different sizes (1, 2, 4, 8, ...) that the TUNE word turns on or off. The total current (therefore the frequency) follows the digital number.
- **Divider:** a chain of divide-by-2 stages. It slows the clock to a speed the output pin and test equipment can handle. The last divide-by-2 also makes the output spend equal time high and low (50% duty cycle).
- **Enable (EN):** switches the oscillator on and off. It is designed so the first and last pulses are whole, with no runt pulses.
- **Output buffer:** drives the harness pin.

### List of I/O (including test ports)

| Signal | Type | Harness resource used | Notes |
|---|---|---|---|
| VDD / VSS | Power | Per-slot power switch | The harness can turn our block on and off. |
| IBIAS | Bias in | 1 of up to 2 mirrored bias currents (about 1 µA) | Only used if we do temperature compensation (step 4). |
| TUNE[7:0] | Digital in | 8 of 24 digital control inputs | Frequency tuning word. Width to be set after simulation. |
| DIV[1:0] | Digital in | 2 of 24 digital control inputs | Divider ratio select (optional, see below*). |
| RANGE[1:0] | Digital in | 2 of 24 digital control inputs | Coarse frequency band select (optional). |
| EN | Digital in | 1 of 24 digital control inputs | Enable / power-down. |
| CLK_OUT | Digital out | 1 of 12 digital status outputs | The divided clock. |
| CLK_RAW (optional) | Analogue out | 1 of 4 shared analogue mux lines | Undivided oscillator output, for debugging only. |

**Total**: up to 13 of 24 digital inputs, 1 of 12 digital outputs, and 1 of 4 analogue lines (only for the debug output). 

**(Confirm against FAQ):**

No dedicated pads needed (main output is a digital clock that goes through a normal digital status output). Digital pins stay well inside the harness limit (24 in, 12 out).

*Choose if keeping divider ratio as fixed (saves pins and means less to test). Can have selectable ratio if one fixed ratio can't cover the clock speeds wanted, or if we find out the harness output can't pass the clock fast enough. We decide after the research below. RANGE overlaps with DIV in what it does, so we likely keep only one.

**To be checked in FAQ: maximum frequency the harness digital outputs can pass and mux bandwidth.**

---

### Target specification

To be updated after simulation. All frequency rows are placeholders until band research is done. Hand calculations documented separately - but need to peer check.

| Parameter | Min | Typ | Max |
|---|---|---|---|
| Supply | 1.62 V | 1.8 V | 1.98 V |
| Oscillator frequency range (over TUNE) | 10 MHz | n/a | 200 MHz |
| Output frequency after divider | n/a | about 1 to 10 MHz | n/a |
| Tuning step | n/a | about 1 to 3% per LSB | n/a |
| Frequency drift over -20 to 120 °C | n/a | to be measured | n/a |
| Duty cycle (after divide-by-2) | 45% | 50% | 55% |
| Supply current | n/a | to be measured | 500 µA |
| Start-up time | n/a | few cycles | 10 µs |
| Settling time after a TUNE/RANGE change | n/a | to be measured | to be set |
| Enable/disable glitches | none | none | none (no runt pulses) |
| Temperature | -20 °C | | 120 °C |

Absolute limits: supply must not exceed the 1.8 V device rating (to confirm in the PDK documentation).
---

### Test-plan outline

- Measure supply current with EN high and low.
- Set TUNE to a few codes (min, mid, max) and measure CLK_OUT frequency with a frequency counter or scope.
- Step through all TUNE codes to check frequency rises steadily with code and to measure the step size.
- Measure frequency against supply voltage (1.62 to 1.98 V).
- Repeat at several temperatures between -20 and 120 °C to measure drift.
- Measure duty cycle and cycle-to-cycle jitter on a scope.
- Toggle EN and look at the first and last cycles, to confirm there are no runt pulses.
- Change TUNE/RANGE while running and measure how long the frequency takes to settle.


## Progression 
(each step is ok to submit on its own)

| Step | What we build | Description | Why it's useful |
|---|---|---|---|
| 1 | Ring oscillator and divider | A loop of inverters that switches on its own, followed by a counter that slows the output down. | A cheap clock on the chip. It is also the core of many bigger blocks. |
| 2 (main target) | Tunable oscillator (DCO) | The same loop, but a digital number sets how fast it runs. | A digital designer can choose or calibrate the clock speed in software. |
| 3 | Clean output | A start/stop input with no half-pulses, an equal high/low time, and a buffer to drive the output pin. | The clock is safe to connect to real digital logic. |
| 3b (extra) | Use an existing DAC | A ready-made digital-to-analogue converter feeds the oscillator instead of switching taps directly. | Only worth it if a good DAC is freely available. It adds extra circuitry, so the baseline stays as in step 2. |
