# 2.45 GHz Software-Defined FMCW Radar Using GNU Radio and USRP B210

A modular **Electrical Engineering graduation project** for a **2.45 GHz software-defined FMCW radar**, developed using **GNU Radio Companion**, embedded Python signal-processing blocks, and the **Ettus USRP B210**.

The repository demonstrates a radar-processing workflow for chirp generation, SDR transmission and reception, radar-datacube construction, range and Doppler processing, Range–Doppler visualization, two-dimensional CFAR detection, and a MUSIC direction-of-arrival framework intended for future multi-receiver development.

> **Author:** Naif Abdulkarim Alanazi  
> **Discipline:** Electrical Engineering  
> **Project type:** Graduation Project  
> **Main GNU Radio file:** [`grc/Radar1.grc`](grc/Radar1.grc)

---

## Important Scope Statement

This repository contains two distinct technical parts:

1. **My GNU Radio implementation**  
   The flowgraph parameters, connections, embedded Python blocks, processing sequence, limitations, and project documentation are based on the supplied `Radar1.grc` file.

2. **A published RF-hardware reference architecture**  
   The antenna type and selected RF-front-end values discussed in this README are summarized from the following IEEE Access article:

   **P. Janpangngern et al., “High-Resolution FMCW Radar for Small UAV Detection Using GNU Software-Defined Radio,” IEEE Access, vol. 13, 2025.**  
   DOI: [10.1109/ACCESS.2025.3570635](https://doi.org/10.1109/ACCESS.2025.3570635)

The published hardware system is used only as a **technical reference for the proposed or considered hardware configuration**. Unless explicitly supported by evidence in this repository:

- the paper’s physical prototype is not claimed as my design;
- the paper’s antenna measurements are not claimed as my measurements;
- the paper’s transmit power is not claimed as hardware that I personally operated;
- the paper’s detection ranges and field-test results are not claimed as results reproduced by my graduation project;
- no sponsorship, endorsement, collaboration, or institutional relationship with the paper’s authors is implied.

---

## Third-Party Attribution and License Notice

The referenced IEEE Access article states that it is distributed under the **Creative Commons Attribution 4.0 International License (CC BY 4.0)**.

License information:

- [Creative Commons Attribution 4.0](https://creativecommons.org/licenses/by/4.0/)
- [CC BY 4.0 legal code](https://creativecommons.org/licenses/by/4.0/legalcode)

Hardware specifications and system concepts taken from the article are **summarized and adapted** in this README for academic and technical discussion. The original authors and publication are credited above.

The repository does not reproduce the full paper or claim ownership of its figures, photographs, antenna design, experimental data, or reported results. Any future use of a figure, table, photograph, or substantial adapted material from the article should include:

- the full source citation;
- an indication that the material was adapted;
- a link to the CC BY 4.0 license;
- a statement that no endorsement by the original authors is implied.

The GNU Radio flowgraph image included in this repository is a screenshot of this project’s own flowgraph, not a figure copied from the IEEE article.

---

## Project Objectives

The project was developed to strengthen practical understanding of:

- FMCW and LFM radar principles;
- software-defined radio using the USRP B210;
- GNU Radio Companion development;
- custom Python signal-processing blocks;
- radar-datacube organization;
- range estimation using fast-time processing;
- radial-velocity estimation using slow-time processing;
- Range–Doppler Map generation;
- two-dimensional CFAR detection;
- MUSIC direction-of-arrival requirements;
- limitations caused by a single receiving antenna;
- RF-front-end and antenna-system design considerations.

---

## GNU Radio Flowgraph

![GNU Radio flowgraph](images/gnu_radio_flowgraph.png)

The current flowgraph follows this software path:

```text
FMCW Chirp Source
    ├──> UHD: USRP Sink
    │       └──> Reference TX RF Chain
    │               └──> Reference TX Antenna
    │
    └──> TX Reference Signal
                  │
                  v
UHD: USRP Source <── Reference RX RF Chain <── Reference RX Antenna
        │
        v
Radar Cube Builder
        │
        v
Matched Filter
        │
        v
Range FFT
        │
        v
Doppler FFT
        ├──> Range–Doppler Display
        ├──> 2D CFAR ──> CFAR Display
        └──> MUSIC Framework ──> MUSIC Display
```

The radar-cube builder is configured with:

```text
dechirp = True
```

Therefore, it forms a beat signal using:

```text
beat = RX × conj(TX reference)
```

---

## Exact GNU Radio Parameters

The values below are taken from `Radar1.grc`.

| Parameter | Configured value |
|---|---:|
| Center frequency | 2.45 GHz |
| Complex sample rate | 30 MS/s |
| Chirp bandwidth | 25 MHz |
| Samples per chirp | 900 |
| Chirp-source amplitude | 0.7 |
| Chirps per frame | 1024 |
| Declared RX count | 1 |
| Range FFT length | 1024 |
| Doppler FFT length | 1024 |
| USRP TX channels | 1 |
| USRP RX channels | 1 |
| USRP TX gain setting | 76 |
| USRP RX gain setting | 85 |
| USRP TX antenna port | `TX/RX` |
| USRP RX antenna port | `RX2` |
| USRP TX/RX bandwidth | 25 MHz |
| Radar-cube dechirping | Enabled |
| Matched-filter amplitude | 1.0 |
| MUSIC source count | 1 |
| MUSIC element spacing | 0.0612 m |
| CFAR range training cells | 8 |
| CFAR Doppler training cells | 8 |
| CFAR range guard cells | 1 |
| CFAR Doppler guard cells | 1 |
| CFAR false-alarm probability | 1 × 10⁻⁶ |

The implemented radar cube contains one receiving channel:

```text
cube shape = (1, chirps_per_frame, samples_per_chirp)
```

The two input signals to the radar-cube builder are:

```text
Input 0 = TX reference
Input 1 = RX0
```

The TX reference is not a second receiving antenna.

---

## Values Derived From the GRC Configuration

### Chirp duration

```text
Tc = samples_per_chirp / sample_rate
Tc = 900 / 30,000,000
Tc = 30 µs
```

### Chirp slope

```text
S = bandwidth / Tc
S = 25 × 10^6 / 30 × 10^-6
S ≈ 8.333 × 10^11 Hz/s
```

### Wavelength at 2.45 GHz

```text
λ = c / fc
λ = 3 × 10^8 / 2.45 × 10^9
λ ≈ 0.12236 m
λ ≈ 122.36 mm
```

### Half-wavelength spacing

```text
d = λ / 2
d ≈ 0.06118 m
d ≈ 61.18 mm
```

The configured MUSIC spacing of `0.0612 m` is approximately half a wavelength at 2.45 GHz.

### Theoretical range resolution

```text
ΔR = c / (2B)
ΔR = 3 × 10^8 / (2 × 25 × 10^6)
ΔR ≈ 6 m
```

This is the physical range resolution set by the waveform bandwidth. Increasing the FFT length through zero-padding can create more plotted bins, but it does not improve the actual 6 m range resolution.

### Nominal chirp repetition frequency

The flowgraph continuously repeats 900-sample chirps without a separately configured idle period:

```text
PRF ≈ 1 / 30 µs
PRF ≈ 33.33 kHz
```

### Frame duration

```text
Tframe = 1024 × 30 µs
Tframe = 30.72 ms
```

### Nominal velocity values

Assuming the actual chirp repetition interval is exactly 30 µs:

```text
Maximum unambiguous radial velocity ≈ ±1019.7 m/s
Velocity-bin spacing ≈ 1.99 m/s
```

These are mathematical values based on the software configuration. Experimental velocity calibration must use the actual measured chirp repetition interval, including UHD buffering, scheduling, USB transfer behavior, and any additional delay.

### Mathematical beat-frequency range limit

Using a positive complex-baseband beat-frequency limit of approximately `Fs/2`:

```text
Mathematical beat-frequency-limited range ≈ 2.70 km
```

This value is not an experimental detection-range claim. Practical detection range depends on many factors, including:

- transmitted power;
- antenna gain;
- receiver noise figure;
- target radar cross section;
- TX-to-RX leakage;
- environmental clutter;
- filtering;
- calibration;
- legal transmit-power limits.

---

## Hardware Reference Architecture

The following hardware section summarizes the architecture reported in the cited IEEE Access article. It is included as the **reference hardware configuration considered for this project**.

### Software-defined radio

- Ettus **USRP B210**
- full-duplex operation;
- USB 3.0 connection;
- integrated ADC and DAC;
- FPGA-based data handling;
- RF mixers and local oscillator;
- GNU Radio control through UHD.

### Reference transmitting path

```text
USRP B210 TX
    -> Driver / Two-Stage Power-Amplifier Chain
    -> Vivaldi-Fed Parabolic Reflector TX Antenna
```

Values reported by the paper:

- final transmit power: approximately **50 dBm**;
- equivalent power: approximately **100 W**;
- separate directional transmitting antenna;
- Vivaldi antenna used as the feed of a parabolic reflector.

These values describe the published reference system. They are not automatically evidence that the same high-power chain was constructed or operated in this repository.

### Reference receiving path

```text
Vivaldi-Fed Parabolic Reflector RX Antenna
    -> Band-Pass Filter
    -> Low-Noise Amplifier
    -> USRP B210 RX
```

Components and values reported by the paper:

- LNA model: **Mini-Circuits ZX60-83LN-S+**;
- LNA gain: approximately **20 dB**;
- LNA noise figure: approximately **1.4 dB**;
- BPF model: **Mini-Circuits VBFZ-2575-S+**;
- separate directional receiving antenna.

### Reference antenna type

The reference radar uses two separate directional antennas:

```text
TX antenna = Vivaldi feed + parabolic reflector
RX antenna = Vivaldi feed + parabolic reflector
```

The Vivaldi element acts as the broadband feed. The parabolic reflector focuses the radiated or received energy into a narrower beam.

Antenna values reported in the paper at 2.45 GHz:

| Characteristic | Paper-reported value |
|---|---:|
| Simulated gain | 17.6 dBi |
| Measured gain | 17.3 dBi |
| Measured H-plane HPBW | 13.0° |
| Measured E-plane HPBW | 12.8° |
| Evaluated operating range | 2–6 GHz |
| Return loss | approximately `S11 ≤ -10 dB` |

These are the paper’s reported antenna results and are cited as such. They are not presented as measurements performed by the author of this repository.

### Reference processing platform

The paper reports the following processing system:

- LattePanda Sigma;
- Intel Core i5-1340P;
- 32 GB RAM;
- NVIDIA Quadro K6000;
- 12 GB GPU memory;
- Ubuntu 22.04;
- GNU Radio with `gr-plasma`;
- ArrayFire using CUDA, OpenCL, or CPU execution.

This repository does not require that exact processing computer unless the implementation is intentionally reproduced on equivalent hardware.

---

## Difference Between the GRC Timing and the Paper

The reference paper reports:

```text
LFM pulse width = 30 µs
Pulse repetition interval = 50 µs
Listening period = 20 µs
```

The supplied GNU Radio file continuously repeats a 30 µs chirp and uses a 30 µs chirp period in its software calculations.

Therefore:

- the paper’s waveform timing and the current GRC timing are not identical;
- velocity and range calculations must use the timing of the actual implementation;
- the paper’s experimental performance should not be assigned directly to this flowgraph without reproduction and calibration.

---

## Embedded Python Processing Blocks

### `fmcw_chirp_src`

Generates a repeated complex-baseband chirp:

```text
s(t) = A exp(jπSt²)
```

Configured values:

```text
A = 0.7
S ≈ 8.333 × 10^11 Hz/s
Tc = 30 µs
```

#### Sampling limitation

The current waveform increases approximately from 0 to 25 MHz in complex baseband.

At a 30 MS/s complex sampling rate, the Nyquist interval is approximately:

```text
-15 MHz to +15 MHz
```

The section of the chirp above +15 MHz aliases.

Recommended future corrections:

```text
Option 1: center the chirp from approximately -12.5 MHz to +12.5 MHz
Option 2: increase the sample rate beyond 50 MS/s, subject to hardware limits
```

---

### `radar_cube_builder`

Inputs:

```text
Input 0 = TX reference
Input 1 = RX0
```

Current cube shape:

```text
(1, 1024, 900)
```

Axis definitions:

```text
Axis 0 = receive channel
Axis 1 = chirp index / slow time
Axis 2 = sample index / fast time
```

With dechirping enabled:

```text
cube[0] = RX0 × conj(TX reference)
```

---

### `matched_filter_block`

The matched filter uses a time-reversed complex-conjugated reference chirp:

```text
h(t) = s*(-t)
```

The supplied flowgraph applies the matched filter after dechirping.

Two standard processing choices are:

#### Conventional FMCW processing

```text
Dechirp -> Range FFT -> Doppler FFT
```

#### Pulsed-LFM processing

```text
Raw received pulses -> Matched filter -> Range/Doppler processing
```

The current flowgraph includes both dechirping and matched filtering. This is an experimental configuration and should be reviewed carefully. A future revision should select and validate one consistent processing method rather than combining both unintentionally.

---

### `range_fft_block`

The block:

- applies a Hanning window in fast time;
- performs a 1024-point FFT;
- converts the current cube from:

```text
(1, 1024, 900)
```

to:

```text
(1, 1024, 1024)
```

For a conventional FMCW beat signal:

```text
R = c fb / (2S)
```

where:

- `R` is target range;
- `fb` is beat frequency;
- `S` is chirp slope;
- `c` is the speed of light.

---

### `doppler_fft_block`

The block:

- applies a Hanning window across 1024 chirps;
- performs a 1024-point Doppler FFT;
- applies `fftshift`;
- generates a normalized-power Range–Doppler Map.

Current output shape:

```text
(1024 range bins, 1024 Doppler bins)
```

The block also publishes per-RX data for the MUSIC stage. However, the current per-RX output is converted to magnitude-only real data before publication.

---

### `cfar2d_block`

The two-dimensional cell-averaging CFAR configuration is:

```text
Range training cells = 8
Doppler training cells = 8
Range guard cells = 1
Doppler guard cells = 1
Pfa = 1 × 10^-6
```

This produces:

```text
Complete CFAR window = 19 × 19 cells
Guard region = 3 × 3 cells
Training cells = 352
Approximate threshold multiplier α ≈ 14.09
```

The block uses nested Python loops over a 1024 × 1024 map. Optimization or compiled acceleration may be required for sustained real-time performance.

---

## MUSIC Direction-of-Arrival Limitation

### Current configuration

```text
Physical receiving antennas = 1
Implemented RX channels = 1
Radar-cube receive dimension = 1
```

MUSIC estimates direction of arrival from relative phase differences across multiple coherent receiving antenna elements.

With only one receiving antenna:

- no antenna-to-antenna phase difference is available;
- a valid spatial covariance matrix cannot be constructed;
- angular separation cannot be performed;
- any apparent MUSIC angle cannot be treated as a reliable physical direction.

The current MUSIC block is therefore included as a **future-development framework**, not as evidence of valid one-antenna angle estimation.

### Complex-data limitation

The current Doppler block publishes magnitude-only per-RX data:

```text
float32 magnitude
```

MUSIC requires complex per-channel samples so that phase differences are preserved.

A future valid MUSIC input should have a structure such as:

```text
complex[num_rx, range_bins, Doppler_bins]
```

### Required future hardware

At 2.45 GHz, a possible uniform linear receiving array could use approximately half-wavelength spacing:

```text
d ≈ 61.2 mm
```

A future implementation should include:

- at least two coherent receive channels;
- preferably four or more receive elements;
- known array geometry;
- synchronized sampling;
- complex per-channel Range–Doppler data;
- phase and gain calibration;
- cable-delay calibration;
- adequate snapshots for covariance estimation;
- mutual-coupling evaluation.

### What remains valid with one RX antenna

The one-RX limitation does not prevent the system from supporting:

- beat-signal formation;
- range FFT processing;
- Doppler FFT processing;
- Range–Doppler visualization;
- CFAR-based target detection;
- single-channel radar experimentation.

It specifically prevents reliable MUSIC direction-of-arrival estimation.

---

## Current Capability Status

| Function | Status |
|---|---|
| 2.45 GHz chirp generation | Implemented |
| USRP TX/RX configuration | Implemented in GRC |
| TX reference signal | Implemented |
| One-RX radar cube | Implemented |
| FMCW dechirping | Implemented |
| Matched-filter experiment | Implemented |
| Range FFT | Implemented |
| Doppler FFT | Implemented |
| Range–Doppler display | Implemented |
| 2D CA-CFAR | Implemented |
| MUSIC software framework | Included |
| Valid MUSIC angle with one RX | Not possible |
| Calibrated multi-channel DOA | Future work |
| Reproduction of paper field-test results | Not claimed |

---

## Known Technical Limitations

1. **Single receiving antenna** prevents valid MUSIC direction estimation.
2. **Magnitude-only per-RX output** removes the phase information required by MUSIC.
3. **Chirp aliasing** occurs because the current 0-to-25 MHz baseband chirp exceeds the ±15 MHz Nyquist interval at 30 MS/s.
4. **Dechirping and matched filtering are both active**, although they normally represent different range-processing approaches.
5. **Range-axis mapping requires correction** using FFT frequency bins, chirp slope, positive-beat selection, and calibrated delay.
6. **Velocity-axis accuracy depends on the true chirp repetition interval**, not only the nominal software value.
7. **USRP TX/RX delay and buffering require calibration**.
8. **Direct TX-to-RX leakage may dominate weak target echoes**.
9. **Python CFAR loops may be computationally expensive** for 1024 × 1024 maps.
10. **TX and RX gains must be tuned experimentally** to prevent clipping, compression, or unnecessary noise.
11. **Reference-paper hardware values must not be treated as installed hardware unless physically verified**.
12. **Experimental detection claims require controlled measurements and documented evidence**.

---

## Installation

Install GNU Radio, UHD, NumPy, and Matplotlib using the method appropriate for the operating system.

Example Python dependencies:

```bash
python -m pip install numpy matplotlib pyyaml
```

Check the connected USRP:

```bash
uhd_find_devices
uhd_usrp_probe
```

Open the flowgraph:

```bash
gnuradio-companion grc/Radar1.grc
```

---

## Safe Initial Testing

Start with a conducted loopback configuration:

```text
USRP TX -> suitable attenuation -> USRP RX
```

Do not connect a power amplifier directly to the USRP receiver.

Before enabling RF transmission:

- verify the receiver’s maximum safe input power;
- calculate the required attenuation;
- confirm 50-ohm impedance matching;
- begin at the lowest practical TX gain;
- verify spectral occupancy;
- monitor clipping and compression;
- confirm legal frequency and power limits.

For over-the-air testing:

- begin at low power;
- maintain adequate TX/RX antenna isolation;
- verify antenna alignment;
- protect the RX input from direct leakage;
- calibrate the zero-range offset;
- compare results with targets at known positions and speeds.

---

## Safety and Regulatory Notice

The reference article reports a transmit chain reaching approximately 50 dBm or 100 W. Such power can:

- damage SDR equipment;
- exceed receiver input ratings;
- create hazardous RF exposure;
- interfere with other systems;
- violate spectrum regulations.

This repository does not authorize operation at that power level.

Users are responsible for compliance with:

- national frequency-allocation rules;
- applicable 2.45 GHz ISM-band limits;
- permitted bandwidth and spectral masks;
- RF exposure requirements;
- amplifier, cable, filter, and antenna ratings;
- laboratory and outdoor test-site procedures.

---

## Future Development

- center the chirp within the complex Nyquist interval;
- select and validate one range-processing method;
- correct and calibrate range and velocity axes;
- characterize TX/RX latency;
- add direct-leakage cancellation;
- add clutter suppression;
- optimize CFAR processing;
- add detection clustering and target tracking;
- add coherent multi-RX hardware;
- construct a uniform linear receiving array;
- preserve complex per-channel data;
- calibrate receive-channel phase and gain;
- validate MUSIC using controlled target angles;
- document real hardware and experimental results separately from literature values;
- compare measured results with the cited paper without presenting them as identical or reproduced unless verified.

---

## Repository and Content Rights

This README distinguishes between:

### Original project material

- `Radar1.grc`;
- project-specific embedded Python blocks;
- the project flowgraph screenshot;
- original documentation written for this repository;
- future measurements generated by the project author.

### Third-party reference material

- the IEEE Access article;
- the published antenna design and measurements;
- the paper’s RF-component values;
- the paper’s experimental setup and reported detection results;
- any figures, tables, photographs, or data originating from the article.

Third-party material remains attributed to its original source. No transfer of authorship or ownership is claimed.

A separate repository `LICENSE` file should clearly state the license selected for the author’s original code and documentation. That license should not be interpreted as relicensing third-party material beyond the permissions granted by its original license.

---

## Recommended Citation

When referring to the hardware reference, cite:

```text
P. Janpangngern et al., “High-Resolution FMCW Radar for Small UAV
Detection Using GNU Software-Defined Radio,” IEEE Access, vol. 13,
2025, doi: 10.1109/ACCESS.2025.3570635.
```

When referring to this repository, cite the repository author and URL after the repository is published.

---

## Disclaimer

This repository is provided for academic, educational, and engineering-development purposes.

It does not claim:

- endorsement by IEEE;
- endorsement by the referenced authors or institutions;
- ownership of the paper’s antenna design or experimental data;
- reproduction of the paper’s detection performance;
- regulatory approval for RF transmission;
- validated MUSIC direction-of-arrival results using the current single-RX hardware.

---

## Author

**Naif Abdulkarim Alanazi**  
Electrical Engineering Graduate  
Graduation Project: **2.45 GHz Software-Defined FMCW Radar Using GNU Radio and USRP B210**
