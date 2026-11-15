# to do

## Risks to check

- Maximum switching speed of the harness digital outputs (sets how much division is needed).
- Ring oscillator frequency varies a lot with the manufacturing process and temperature, so we report the measured spread honestly rather than a tight number.
- Jitter will be reported as a result, not a target.
- Confirm with the organisers that a DCO counts as an accepted analogue block.

## Research to do before fixing the spec

- Find the harness's maximum digital output frequency (template repo and FAQ). This sets the divider ratios.
- Find what clock frequencies digital designers actually want (SPI, microcontroller and peripheral clocks).
- Simulate a current-starved ring in Sky130 to see the achievable frequency per stage count, and set the bands from that.
- Check whether a pre-existing DAC is available and allowed.
- Ask the engineer what "time-based filter at start and end" and "different ranges" meant.

- ## Potential extensions
| Stage | Block | What it is and what it is used for |
|---|---|---|
| 4 (maaaybe?) | **Temperature-robust DCO** | Use the harness bias current to reduce how much frequency drifts with temperature, or add a counter-friendly mode so the digital side can calibrate the frequency. |
| 5 (stretch) | **TRNG or time-to-digital mode** | Ring oscillators have natural timing noise (jitter). Sampling one ring with another gives random bits (a true random number generator, TRNG). The same ring can also measure short time intervals (a time-to-digital converter).|


Both are Hard-column blocks in the ideas table... we shall see
