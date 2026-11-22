# DCO

### Team name: DAWg

## Summary

An on-chip digitally controlled oscillator (DCO): a clock generator. Speed is set by a digital tuning word, written over the harness's digital inputs. No crystal is needed to make the clock.

The output is a buffered clock with low jitter as a design goal, meant to drive a digital system directly.

### Typical applications

1. **Master clock:** the system clock for a small processor and its peripherals, with no external crystal.
2. **Sampling clock:** the clock for a data converter or signal generator, where the frequency can be set in software.
3. **Frequency-shift keying (FSK):** switching between two tuning codes to send data on a clock-derived signal.

**Main application - master clock for a digital system:** a digital designer needs a system clock for a small processor and its peripherals, and doesn't want an external crystal. They write a tuning code to set the DCO to about 50 MHz for the processor, and use the divider setting to get lower clocks (for example 25 MHz or 12.5 MHz) for peripherals. They can re-tune the code in software to calibrate the frequency.

Because everything on the chip is timed from this clock, **low jitter is an important requirement**. Timing noise on a master clock directly reduces the timing margin of every circuit it drives.

---

### Type of IP block

Digitally controlled oscillator (DCO) with output divider and buffered output.

### Functional description

- **Oscillator core:** produces the raw clock. Its frequency is set by the tuning control.
- **Tuning control:** a digital word that sets the oscillator's frequency. Word is split into **coarse** bits and **fine** bits. Extra bits are included beyond what we expect to need, to leave tolerance after tapeout.
- **Divider:** a chain of divide-by-2 stages with a selectable ratio. It gives lower master-clock frequencies from the same oscillator range, and the last stage gives a 50% duty cycle.
- **Enable (EN):** switches the clock on and off. It is designed so the clock never starts or stops with a partial (cut-off) pulse.
- **Output filter and buffer:** cleans up the clock and drives the harness output pad, which is expected to be a large load (to be checked).
- **Test access:** selected internal nodes (for example the raw oscillator output and a bias node) can be sent to the harness's shared analogue lines, for debugging after tapeout.
- **Reference input (optional):** a spare input reserved for an external reference clock, in case we add a mode that locks the output to it.

### List of I/O (including test ports)

| Signal | Type | Harness resource used | Notes |
|---|---|---|---|
| VDD / VSS | Power | Per-slot power switch | The harness can turn our block on and off. |
| TUNE_COARSE[3:0] | Digital in | 4 of 24 digital control inputs | Coarse frequency tuning. |
| TUNE_FINE[5:0] | Digital in | 6 of 24 digital control inputs | Fine frequency tuning. |
| DIV[1:0] | Digital in | 2 of 24 digital control inputs | Divider ratio select (see note below). |
| EN | Digital in | 1 of 24 digital control inputs | Enable / power-down. |
| REF_IN (optional) | Digital in | 1 of 24 digital control inputs | Reserved reference clock input. |
| SPARE[1:0] | Digital in | 2 of 24 digital control inputs | Reserved for additional tuning bits or test modes, to leave tolerance after tapeout. |
| CLK_OUT | Digital out | 1 of 12 digital status outputs | The buffered output clock. |
| ANA_OSC | Analogue out | 1 of 4 shared analogue mux lines | Raw oscillator output, for debugging. |
| ANA_BIAS | Analogue out | 1 of 4 shared analogue mux lines | Internal bias/control node, for debugging. |
| IBIAS (optional) | Bias in | 1 of up to 2 mirrored bias currents (about 1 µA) | Only if used for temperature compensation. |

**Total (max asked for):** up to 16 of the 24 digital inputs, 1 of the 12 digital outputs, 2 of the 4 analogue lines, and 1 bias current. These are maximum, so we're happy with fewer if resources are tight. Any spare inputs we don't use will just be tied off.

**Dedicated pads:** none needed. Our main output is a digital clock, so it can go through a normal digital status output.

**Still to check:** the max frequency the harness digital outputs can pass, the analogue mux bandwidth, the pad load capacitance, and the bias current values. We'll size the divider and output buffer once we know these.

---

### Target specification

Values to be updated after simulation. The frequency rows are placeholders until the band research is done.

| Parameter | Min | Typ | Max |
|---|---|---|---|
| Supply | 1.62 V | 1.8 V | 1.95 V |
| Oscillator frequency range (over tuning word) | 25 MHz | - | 200 MHz |
| Output frequency after divider | - | 50 MHz (selectable by DIV) | set by the harness output speed (to be checked) |
| Coarse tuning step | - | to be measured | -|
| Fine tuning step | - | to be measured | about 3% per LSB |
| Cycle-to-cycle jitter (rms) | - | about 1% of the clock period or better | to be set after simulation |
| Frequency drift over -20 to 120 °C | - | to be measured | - |
| Duty cycle (after divide-by-2) | 45% | 50% | 55% |
| Supply current (including the output buffer driving its pad load) | - | to be measured | 2 mA |
| Start-up time | - | few cycles | 10 µs |
| Settling time after a tuning change | - | to be measured | to be set |

| Temperature | -20 °C | | 120 °C |

Absolute limits: supply must not exceed the 1.8 V device rating (to be confirmed in the PDK documentation).

### Test-plan outline

**Simulation checks (before tapeout):**
- Run the design across process, voltage and temperature corners and check every spec in the table above.
- Extract the parasitics from the layout and re-run, to make sure it still meets spec.

**Measurements (after we get the chip back):**
- **Tuning:** set different tuning codes and check the output frequency. Does it keep going up as the code goes up /are the steps small enough?
- **Divider and duty cycle:** try each divider setting. Is the output frequency right and is the clock about 50% high and 50% low?
- **Supply and temperature:** how far does the frequency move with supply (1.62 to 1.95 V) and temperature (-20 to 120 °C)? How much current does it draw with the output driving its load?
- **Jitter:** how steady are the clock edges? We measure at full speed and again with extra division, so we can tell the oscillator's jitter apart from the output path's.
- **Start, stop and settling:** toggle EN and check there are no half-pulses. Change the tuning code while it's running and see how long the frequency takes to settle.
- **Debugging:** look at the internal nodes (ANA_OSC and ANA_BIAS) through the analogue lines, to find problems before the output stage.
- **Reference input (if we include it):** does the output follow a reference clock on REF_IN?
- **Real use:** can the clock drive a simple digital circuit, like a counter or a microcontroller, without any extra help?

