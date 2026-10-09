# Project 4: FPGA FMCW Sonar

University of Cape Town — Masters FPGA Course  
Author: Akhona Hlanjwa (HLNAKH003)

## Project status

This submission demonstrates two separately tested FPGA designs:

1. Two-channel ADC acquisition using the Sonar v1.0 board.
2. A digital synthetic-echo test of an FMCW reference, mixer and FIR filter.

The ADC and DSP designs have not been integrated into a complete acoustic sonar system. Acoustic range and direction-of-arrival measurements were not demonstrated. The microphones provided were not the correct ones for this project.

## Hardware and software

- Terasic DE10-Lite FPGA board
- MAX 10 device: 10M50DAF484C7G
- Sonar v1.0 board with MAX11059 ADCs
- Quartus Prime Lite 25.1
- 50 MHz FPGA clock
- USB-Blaster JTAG interface

## Repository packages

- SONAR_ADC_TEST.zip: two-channel ADC acquisition project.
- The DSP ZIP: verified synthetic-echo DSP project.

Extract each package into a separate folder before opening its Quartus project. Both projects retain the project filename LED_BLINK.qpf.

## ADC acquisition

The FPGA captures 256 sample pairs from U2 channels 0 and 1 at a nominal 10 ksample/s per channel.

The controller starts conversions, monitors EOC and reads all eight U2 channel words, retaining the first two. U3 is not read. Samples are exported through JTAG using export_sonar_capture.tcl.

These captures demonstrate acquisition and export. They do not establish calibrated analogue accuracy or a verified microphone response.

## DSP implementation

The digital self-test uses:

- A 256-sample reference with frequency swept from 1 to 3 kHz and back.
- A synthetic echo formed by a circular delay of 16 samples.
- A signed fixed-point multiplier for mixing.
- A 63-tap FIR low-pass filter with a 600 Hz design cutoff.
- Storage of 256 mixer and 256 filtered output samples.
- JTAG export using export_sonar_dsp.tcl.

The synthetic delay is a test input, not a measured acoustic delay.

## Verified DSP results

The FPGA-exported CSV was compared with the expected integer model:

| Check | Result |
|---|---|
| Mixer samples | 256 of 256 match exactly |
| FIR samples | 256 of 256 match exactly |
| Maximum numerical error | 0 |
| CLOCK_50 setup slack | +3.899 ns |
| CLOCK_50 hold slack | +0.150 ns |
| Total negative slack | 0 ns |
| Logic elements | 12,282 / 49,760 (25%) |
| Registers | 9,135 / 51,509 (18%) |
| Embedded 9-bit multiplier elements | 4 / 288 |
| M9K blocks | 0 / 182 |

Timing figures apply to the analysed paths in the DSP design. Sample times in CSV files assume the nominal 50 MHz clock.

## Running the DSP test

1. Extract the DSP package and open LED_BLINK/LED_BLINK.qpf.
2. Confirm that the top-level entity is SONAR_DSP_TEST.
3. Compile and program output_files/LED_BLINK.sof.
4. Open In-System Sources and Probes Editor and scan the chain.
5. Select CTRL, with source width 10 and probe width 32.
6. Write source 0, then source 1 to start a capture.
7. Wait one second and read the probe.
8. Run export_sonar_dsp.tcl through Tools → Tcl Scripts.

The exported CSV contains sample_index, time_us, mixed_signed and filtered_signed.

testbench.v is for simulation only; do not add it to the Quartus synthesis project.

## Hardware limitations

The intended microphones were unavailable. A substitute microphone was installed at MK1, but a working acoustic response was not established. Its measured bias was approximately 0 V during troubleshooting; the cause remains unresolved.

Consequently, the physical triangular microphone array, acoustic transmitter operation, range estimation and angle extraction remain incomplete.

Future work includes resolving microphone compatibility and bias, integrating ADC acquisition with DSP, implementing the transmitter and array processing, and testing against known acoustic targets.
