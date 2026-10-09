# Project 4: FPGA FMCW Sonar with Triangular Array

**Author:** Akhona Hlanjwa (HLNAKH003) 
**Platform:** Terasic DE10-Lite, Intel MAX 10 FPGA  
**Software:** Quartus Prime Lite 25.1std

## Project status

This submission demonstrates FPGA ADC acquisition, fixed-point mixing and FIR filtering, and an integrated ADC-to-DSP hardware data path.

The acoustic transmitter, working triangular microphone array, target range measurement, and direction estimation were not demonstrated. Microphone availability and an unresolved microphone bias problem limited acoustic testing.

## Implemented designs

### 1. SONAR_ADC_TEST

- Interfaces with the Sonar v1.0 board's U2 MAX11059 ADC.
- Captures 256 sample pairs from U2 channels 0 and 1.
- Uses a nominal sampling rate of 10 ksample/s per channel.
- Stores raw 14-bit ADC data for JTAG export.
- Includes conversion timeout and sampling-period overrun detection.

### 2. SONAR_DSP_TEST

- Uses a stored reference with a triangular frequency sweep from 1 to 3 kHz.
- Generates a synthetic echo using a circular delay of 16 samples.
- Implements fixed-point multiplication and a 63-tap FIR low-pass filter with a nominal 600 Hz cutoff.
- Exports 256 mixed and filtered sample pairs.

All 512 exported hardware values matched the independent integer reference model exactly.

The synthetic delay is a digital test stimulus; it is not a measured acoustic target delay.

### 3. SONAR_ADC_DSP_INTEGRATED

The integrated design processes U2 channel 0 through:

ADC acquisition → offset-binary conversion → reference mixing → FIR filtering → stored results → JTAG CSV export.

It captures 256 raw ADC samples and 256 corresponding filtered samples. All 256 exported filtered values matched the integer reference model calculated from the captured raw ADC data, with zero mismatches.

This verifies the implemented digital processing path. It does not establish microphone performance or acoustic ranging accuracy.

## Hardware results

- Synthetic DSP compilation: 0 errors.
- Integrated ADC/DSP compilation: 0 errors.
- FPGA programming completed successfully.
- Integrated capture completed with 256 samples and no reported capture error.
- Integrated raw ADC values ranged from 6238 to 9193.
- Integrated filtered values ranged from −312 to 219.

CSV timestamps are nominal values calculated from the 50 MHz clock and 5000-cycle sampling period.

## Repository contents

- ADC acquisition project archive.
- Verified synthetic DSP project archive.
- Integrated ADC/DSP project archive.
- Exported integrated hardware capture CSV.
- Source code, project settings, and verification scripts within the project archives.

The integrated archive contains a `project` folder and a `verification` folder. Open `project/LED_BLINK.qpf` in Quartus. The project retains the name `LED_BLINK` from the initial board test.

## Reproducing the integrated test

1. Extract the integrated project archive.
2. Open `project/LED_BLINK.qpf`.
3. Compile and program the resulting SOF.
4. Open the In-System Sources and Probes Editor and select the CTRL instance.
5. Write source value 0, then 1, to trigger a capture.
6. Read the probe and confirm count 256, idle state, done asserted, and no error.
7. Run `export_sonar_integrated.tcl` using `quartus_stp` to export the stored results.
8. Run `verify_integrated.py` against the exported CSV using the supplied verification files.

The source width is 10 bits and the probe width is 32 bits. Source bit 0 triggers capture on a rising edge; bit 9 selects the result bank; bits 8:1 select the sample index.

## Limitations

- The specified microphones could not be obtained.
- One substitute microphone was attached at MK1, but its measured positive-terminal voltage was 0 V. The cause remained unresolved.
- No repeatable acoustic microphone response was verified.
- No acoustic chirp transmission, physical target range, or direction-of-arrival result is claimed.
- ADC results are raw and uncalibrated.
- Instrument calibration uncertainty limits interpretation of analog measurements.
- Positive reported internal timing slack does not establish complete timing verification of the external ADC interface.
- Sample storage used FPGA logic resources rather than external SDRAM.

## Future work

Resolve microphone bias and compatibility, complete the triangular array and acoustic transmitter, synchronize transmission with acquisition, and implement beat-frequency and direction estimation. Validate results against known target distances and angles, then optimize memory use and complete external interface timing constraints.
