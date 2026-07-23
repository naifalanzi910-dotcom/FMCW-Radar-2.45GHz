# 2.45 GHz Software-Defined Radar Using GNU Radio and USRP B210

A modular **2.45 GHz FMCW radar graduation project** developed with **GNU Radio Companion**, embedded Python blocks, and the **Ettus USRP B210**.

The repository focuses on waveform generation, SDR transmission and reception, radar-cube construction, range–Doppler processing, two-dimensional CFAR detection, and a MUSIC direction-of-arrival framework for future multi-antenna development.

> **Author:** Naif Abdulkarim Alanazi  
> **Discipline:** Electrical Engineering  
> **Project type:** Graduation Project  
> **Main flowgraph:** `grc/Radar1.grc`

---

## Repository Description

**Suggested repository name**

```text
FMCW-Radar-2p45GHz-GNU-Radio
```

**Suggested GitHub description**

```text
A 2.45 GHz GNU Radio and USRP B210 graduation project implementing chirp generation, radar-cube processing, Range–Doppler FFTs, 2D CFAR, and a future MUSIC DOA framework.
```

---

## Scope and Technical Basis

This repository combines two clearly separated parts:

1. **Software parameters and processing blocks** are taken directly from the supplied `Radar1.grc` GNU Radio Companion file.
2. **The reference RF hardware architecture** follows the IEEE Access paper:

> P. Janpangngern et al., “High-Resolution FMCW Radar for Small UAV Detection Using GNU Software-Defined Radio,” *IEEE Access*, vol. 13, 2025.  
> DOI: [10.1109/ACCESS.2025.3570635](https://doi.org/10.1109/ACCESS.2025.3570635)

The hardware values reported by that paper are included as a **reference architecture**. The paper’s reported detection results are not automatically claimed as independently reproduced by this repository.

---

## GNU Radio Flowgraph

![GNU Radio flowgraph](images/gnu_radio_flowgraph.png)

The active flowgraph in `Radar1.grc` is:

```text
FMCW Chirp Source
    ├──> UHD: USRP Sink ──> TX RF Hardware ──> TX Antenna
    │
    └──> TX Reference ───────────────┐
                                     v
UHD: USRP Source <── RX RF Hardware <── RX Antenna
            │                        │
            └──────────────> Radar Cube Builder
                                      │
                                      v
                              Matched Filter
                                      │
                                      v
                                  Range FFT
                                      │
                                      v
                                 Doppler FFT
                         ┌────────────┼─────────────┐
                         v            v             v
                   RDM Display     2D CFAR      MUSIC Block
                                      │             │
                                      v             v
                                CFAR Display   MUSIC Display
```

The radar-cube builder is configured with `dechirp=True`, so it forms the FMCW beat signal as:

```text
beat = RX × conj(TX reference)
```

---

## Exact GNU Radio Configuration

The following values were read directly from `Radar1.grc`.

| Parameter | Value |
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
| Display update interval | Every message |

> The variable `rx = 1` is present in the flowgraph. More importantly, the `radar_cube_builder` code explicitly creates a cube with one receive dimension, so the implemented system is physically and computationally single-RX.

---

## Derived Values From the GRC Parameters

Using the exact values in `Radar1.grc`:

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
λ ≈ 0.12236 m
λ ≈ 122.36 mm
```

### Half-wavelength spacing

```text
d = λ / 2
d ≈ 0.06118 m
d ≈ 61.18 mm
```

The configured MUSIC spacing of `0.0612 m` is therefore approximately half a wavelength at 2.45 GHz.

### Theoretical range resolution

```text
ΔR = c / (2B)
ΔR ≈ 5.996 m
```

This is the physical range resolution determined by the 25 MHz bandwidth. Increasing the FFT length by zero-padding can interpolate the display but does not improve the physical resolution.

### Nominal chirp repetition frequency

The present flowgraph continuously repeats 900-sample chirps and specifies no idle interval:

```text
PRF ≈ 1 / Tc
PRF ≈ 33.333 kHz
```

### Frame or coherent-processing duration

```text
Tframe = 1024 × 30 µs
Tframe = 30.72 ms
```

### Nominal velocity quantities

Assuming the actual chirp repetition interval is exactly 30 µs:

```text
Maximum unambiguous radial velocity ≈ ±1019.7 m/s
Velocity-bin spacing ≈ 1.99 m/s
```

These are software-model values. Real velocity calibration must use the measured chirp repetition interval, including UHD buffering, USB transfer behavior, scheduling, and any idle time.

### Beat-frequency range limit

Using a positive complex-baseband beat-frequency limit of `Fs/2`:

```text
Theoretical beat-frequency-limited range ≈ 2.70 km
```

This is not a claimed detection range. Practical range is limited by antenna gain, transmit power, receiver noise, target RCS, leakage, clutter, calibration, and regulatory constraints.

---

## Reference Hardware Architecture From the IEEE Paper

The paper’s RF architecture is used here as the project’s hardware reference.

### Core SDR

- **Ettus USRP B210**
- Full-duplex transmit and receive operation
- Integrated ADC, DAC, FPGA, mixers, and local oscillator
- USB 3.0 connection
- UHD interface to GNU Radio

### Transmitting RF chain

```text
USRP B210 TX
    -> Driver / Two-Stage Power Amplifier
    -> Vivaldi-Fed Parabolic Reflector TX Antenna
```

Reference values reported in the paper:

- Final transmit power: approximately **50 dBm**
- Equivalent transmit power: approximately **100 W**
- Separate high-gain directional transmitting antenna
- Antenna type: **Vivaldi feed combined with a parabolic reflector**

### Receiving RF chain

```text
Vivaldi-Fed Parabolic Reflector RX Antenna
    -> Low-Noise Amplifier and Band-Pass Filter
    -> USRP B210 RX
```

Reference receiver components:

- LNA: **Mini-Circuits ZX60-83LN-S+**
- LNA gain: approximately **20 dB**
- LNA noise figure: approximately **1.4 dB**
- BPF: **Mini-Circuits VBFZ-2575-S+**
- Separate high-gain directional receiving antenna

### Vivaldi-fed parabolic antenna

At 2.45 GHz, the paper reports:

| Antenna characteristic | Reported value |
|---|---:|
| Simulated gain | 17.6 dBi |
| Measured gain | 17.3 dBi |
| Measured H-plane HPBW | 13.0° |
| Measured E-plane HPBW | 12.8° |
| Operating range used for antenna evaluation | 2–6 GHz |
| Return loss | approximately `S11 ≤ -10 dB` |

The Vivaldi element acts as the feed, while the parabolic reflector focuses the transmitted or received electromagnetic energy into a narrow directional beam.

### Reference processing computer

The paper reports:

- **LattePanda Sigma**
- Intel **Core i5-1340P**
- **32 GB RAM**
- NVIDIA **Quadro K6000**
- **12 GB GPU memory**
- Ubuntu 22.04
- GNU Radio and `gr-plasma`
- ArrayFire with CUDA, OpenCL, or CPU execution

### Important timing difference

The IEEE paper describes a 30 µs LFM pulse with a 50 µs pulse-repetition interval, leaving a 20 µs listening period. The current `Radar1.grc` file continuously repeats a 30 µs chirp and sets the display chirp period to 30 µs.

Therefore, the paper’s hardware timing and the present GNU Radio timing are not identical. The current repository documents the supplied GRC values exactly and treats the paper as the RF-hardware reference.

---

## Embedded Python Blocks

### `fmcw_chirp_src`

Generates a repeated complex-baseband chirp:

```text
s(t) = A exp(jπSt²)
```

Current values:

```text
A = 0.7
S = 8.333 × 10^11 Hz/s
Tc = 30 µs
```

#### Sampling note

The current code generates an instantaneous baseband frequency that increases approximately from 0 to 25 MHz. At a 30 MS/s complex sample rate, the Nyquist interval is approximately −15 to +15 MHz.

The portion above +15 MHz aliases. A future revision should either:

- center the chirp approximately from −12.5 to +12.5 MHz, or
- increase the sample rate to more than 50 MS/s, subject to hardware limits.

---

### `radar_cube_builder`

Inputs:

```text
Input 0: TX reference
Input 1: RX0 signal
```

Current cube shape:

```text
(1, 1024, 900)
```

Axis interpretation:

```text
Axis 0: RX channel
Axis 1: chirp index / slow time
Axis 2: sample index / fast time
```

With dechirping enabled:

```text
cube[0] = RX0 × conj(TX reference)
```

---

### `matched_filter_block`

The block generates the reference chirp internally and applies the time-reversed complex conjugate:

```text
h(t) = s*(-t)
```

The current flowgraph applies this matched filter after the radar cube has already been dechirped.

#### Processing-method warning

Two common valid approaches are:

**Conventional FMCW**

```text
Dechirp -> Range FFT -> Doppler FFT
```

**Pulsed-LFM**

```text
Raw RX pulses -> Matched filter -> Range/Doppler processing
```

The current experimental flowgraph contains both dechirping and matched filtering. This should be evaluated carefully and normally refactored so that one consistent range-processing method is selected.

---

### `range_fft_block`

- Applies a Hanning window along fast time
- Performs a 1024-point FFT
- Converts `(1, 1024, 900)` into `(1, 1024, 1024)`

For conventional FMCW processing:

```text
R = c fb / (2S)
```

---

### `doppler_fft_block`

- Applies a Hanning window across 1024 chirps
- Performs a 1024-point Doppler FFT
- Applies `fftshift`
- Produces a normalized power Range–Doppler Map with shape:

```text
(1024 range bins, 1024 Doppler bins)
```

The block also publishes `rdm_per_rx`, but it converts each channel to magnitude-only `float32` data before publishing.

---

### `cfar2d_block`

Implements two-dimensional cell-averaging CFAR.

Current window:

```text
Total window: 19 × 19 cells
Guard region: 3 × 3 cells
Training cells: 352
Pfa: 1 × 10^-6
Threshold multiplier α ≈ 14.09
```

The block is implemented with nested Python loops over a 1024 × 1024 map, so optimization may be required for sustained real-time operation.

---

### Display blocks

The Range–Doppler display plots:

- horizontal axis: radial velocity
- vertical axis: range

The CFAR display plots the binary detection map in the same range–velocity plane.

#### Range-axis note

The current display code assigns `c/(2B)` to every FFT bin. This treats each bin as approximately 6 m even though the 1024-point FFT zero-pads 900 samples and includes both positive and negative frequency bins.

A corrected physical range axis should use the chirp slope, sample rate, FFT frequency bins, positive-beat-frequency selection, and measured range-offset calibration.

#### dB conversion note

The Doppler block publishes normalized power. A power map should normally be converted using:

```python
10.0 * np.log10(power)
```

The current display uses `20log10`, which is appropriate for amplitude rather than power.

---

## MUSIC Direction-of-Arrival Limitation

### Current hardware and software condition

```text
Physical receiving antennas: 1
Implemented RX channels: 1
Radar-cube receive dimension: 1
```

The two streams entering the radar-cube builder are not two receiving antennas:

```text
Stream 0 = TX reference
Stream 1 = RX0
```

### Effect on MUSIC results

MUSIC requires spatial phase differences measured by multiple coherent receiving antennas. With one receiving antenna:

- no inter-element phase difference exists;
- a spatial covariance matrix cannot be formed correctly;
- a physical direction of arrival cannot be estimated;
- any displayed MUSIC angle must not be interpreted as a valid target direction.

The current code recognizes this condition. When `num_rx < 2`, it publishes a flat zero spectrum. Therefore, the MUSIC blocks are included as a **future multi-RX framework**, not as a validated one-antenna DOA solution.

### Additional MUSIC limitation

The Doppler block currently publishes `rdm_per_rx` as magnitude-only real data:

```text
float32 magnitude per RX
```

MUSIC requires complex per-channel data to preserve phase. Even after adding multiple receiving antennas, the pipeline must be changed to retain:

```text
complex[num_rx, range_bins, Doppler_bins]
```

### Future array configuration

At 2.45 GHz, a possible uniform linear array can use approximately:

```text
61.2 mm element spacing
```

A valid future MUSIC implementation should include:

- at least two coherent RX channels;
- preferably four or more receiving elements;
- known array geometry;
- complex per-RX data;
- phase and gain calibration;
- multiple snapshots for covariance estimation;
- mutual-coupling and cable-delay compensation.

---

## Current Capabilities

| Function | Status |
|---|---|
| 2.45 GHz chirp generation | Implemented |
| USRP TX and RX integration | Configured |
| TX-reference capture | Implemented |
| Single-RX radar-cube formation | Implemented |
| FMCW dechirping | Implemented |
| Matched-filter experiment | Implemented |
| Range FFT | Implemented |
| Doppler FFT | Implemented |
| Range–Doppler visualization | Implemented |
| 2D CA-CFAR | Implemented |
| MUSIC software framework | Included |
| Valid one-antenna MUSIC angle | Not possible |
| Multi-channel calibrated DOA | Future work |

---

## Known Limitations and Recommended Improvements

1. **One RX antenna:** prevents valid MUSIC direction-of-arrival estimation.
2. **Magnitude-only MUSIC input:** removes the inter-channel phase required by MUSIC.
3. **Chirp aliasing:** the current 0-to-25 MHz baseband chirp exceeds the ±15 MHz Nyquist interval of a 30 MS/s complex stream.
4. **Mixed processing methods:** dechirping and matched filtering are both active.
5. **Range-axis calibration:** the display axis needs frequency-bin-based mapping and range-offset correction.
6. **Velocity-axis calibration:** the actual chirp repetition interval must be measured.
7. **USRP timing:** TX and RX latency, buffering, and sample alignment require calibration.
8. **Direct-path leakage:** separate high-gain TX and RX antennas still require adequate isolation.
9. **CFAR performance:** nested Python loops may not sustain the intended frame rate.
10. **Gain settings:** TX gain 76 and RX gain 85 must be tuned to avoid compression, clipping, or excessive noise.
11. **Hardware validation:** reference-paper component values should be verified against the actual installed RF chain.
12. **Regulatory compliance:** 2.45 GHz transmission power and occupied bandwidth must comply with local rules.

---

## Installation

Install GNU Radio, UHD, NumPy, and Matplotlib.

Example Python dependencies:

```bash
python -m pip install numpy matplotlib pyyaml
```

Check the USRP connection:

```bash
uhd_find_devices
uhd_usrp_probe
```

Open the flowgraph:

```bash
gnuradio-companion grc/Radar1.grc
```

---

## Safe Testing Procedure

Begin with a conducted loopback test:

```text
USRP TX -> suitable attenuator chain -> USRP RX
```

Never connect a power amplifier directly to a USRP receiver. Confirm attenuation, maximum input level, impedance matching, and isolation before enabling transmission.

For over-the-air testing:

- begin at low power;
- use a legal test frequency and power;
- verify antenna alignment;
- monitor PA temperature;
- protect the RX input from TX leakage;
- calibrate the zero-range offset;
- validate results with targets at known distances and speeds.

---

## Future Work

- Center the transmitted chirp within the complex Nyquist interval
- Select and validate one range-processing method
- Correct the physical range and velocity axes
- Add fixed-delay and leakage calibration
- Optimize CFAR computation
- Add target clustering and tracking
- Add two or more coherent receiving channels
- Build a calibrated uniform linear array
- Preserve complex per-RX Range–Doppler data
- Validate MUSIC with controlled-angle measurements
- Compare experimental results with the IEEE reference architecture

---

## Safety and Compliance

The reference paper includes a 50 dBm, 100 W transmit chain. This power level can damage equipment, create hazardous RF exposure, and violate radio regulations when used without authorization.

Users are responsible for compliance with:

- national spectrum regulations;
- permitted ISM-band power and bandwidth;
- RF exposure limits;
- amplifier and antenna ratings;
- filtering and spectral-mask requirements;
- safe laboratory procedures.

---

## Academic Purpose

This repository documents my Electrical Engineering graduation project and demonstrates practical work in:

- GNU Radio Companion
- embedded Python signal-processing blocks
- software-defined radio
- FMCW/LFM radar waveforms
- USRP hardware integration
- radar datacubes
- range and Doppler FFT processing
- two-dimensional CFAR
- MUSIC DOA requirements and limitations
- RF front-end architecture
- engineering validation and calibration

---

## Author

**Naif Abdulkarim Alanazi**  
Electrical Engineering Graduate  
Graduation Project: **2.45 GHz Software-Defined Radar Using GNU Radio and USRP B210**
