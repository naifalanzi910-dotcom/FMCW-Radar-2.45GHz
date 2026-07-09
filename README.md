FMCW Radar 2.45 GHz GNU Radio
FMCW Radar 2.45 GHz GNU Radio is a software-defined FMCW radar signal-processing project implemented using GNU Radio and Python. The project demonstrates a modular radar-processing pipeline for a 2.45 GHz FMCW radar prototype, including chirp generation, radar datacube construction, dechirping, range FFT, Doppler FFT, Range-Doppler Map visualization, 2D CFAR detection, and a MUSIC DOA framework for future multi-receiver expansion.
This repository is intended for academic, research, and portfolio demonstration purposes in radar signal processing, software-defined radio, and GNU Radio development.
---
Repository Name Recommendation
Recommended GitHub repository name:
```text
FMCW-Radar-2p45GHz-GNU-Radio
```
Alternative shorter name:
```text
FMCW-Radar-2p45GHz
```
GitHub short description:
```text
A 2.45 GHz SDR-based FMCW radar processing pipeline using GNU Radio, Python, Range-Doppler processing, CFAR detection, and a MUSIC DOA framework.
```
---
Important Clarification
This project is for a 2.45 GHz FMCW radar, not a 77 GHz automotive radar.
From the current code implementation, the radar receiver configuration is:
```text
Number of implemented RX channels = 1
```
This is determined from the `radar_cube_builder` block:
```python
in_sig=[np.complex64, np.complex64]   # tx_ref, rx0
```
The two input streams are:
```text
Input 0: TX reference signal
Input 1: RX0 received signal
```
The datacube is also explicitly created as:
```python
cube = np.zeros((1, chirps_per_frame, samples_per_chirp), dtype=np.complex64)
```
Therefore, the current implementation is:
```text
1 TX reference stream + 1 RX channel
```
It is not currently a 16-RX implementation. However, the code structure can be extended later to support multiple receive channels.
---
Project Overview
The current processing chain is designed around a standard FMCW radar architecture. A complex FMCW chirp is generated and transmitted using an SDR. The received signal is captured and processed together with the transmitted reference signal to form a radar datacube.
The main processing chain is:
```text
FMCW Chirp Source
        ↓
SDR / USRP Transmitter
        ↓
SDR / USRP Receiver
        ↓
Radar Cube Builder
        ↓
Range FFT
        ↓
Doppler FFT
        ↓
Range-Doppler Map Display
        ↓
2D CFAR Detection
        ↓
CFAR Detection Display
```
A MUSIC Direction-of-Arrival block is also included as a future framework, but meaningful MUSIC DOA estimation requires multiple coherent RX channels and complex per-RX phase data.
---
Main Features
2.45 GHz FMCW radar signal-processing pipeline
GNU Radio custom Python blocks
Complex baseband FMCW chirp generation
Radar datacube construction
FMCW dechirping using `RX × conj(TX)`
Range FFT processing
Doppler FFT processing
Range-Doppler Map generation
2D CA-CFAR target detection
Real-time Matplotlib visualization
Optional matched-filter processing block
MUSIC DOA framework for future multi-RX development
Message-based modular GNU Radio architecture
---
Default Radar Parameters
The current prototype uses the following default parameters:
```text
Center frequency:       2.45 GHz
Sample rate:            30 MS/s
FMCW bandwidth:         25 MHz
Samples per chirp:      900
Chirp duration:         30 µs
Chirps per frame:       1024
Range FFT length:       1024
Doppler FFT length:     2048
Implemented RX channels: 1
```
The chirp duration is:
```text
Tc = samples_per_chirp / sample_rate
Tc = 900 / 30e6
Tc = 30 µs
```
The FMCW chirp slope is:
```text
S = bandwidth / Tc
S = 25e6 / 30e-6
S = 8.33 × 10^11 Hz/s
```
The theoretical range resolution is:
```text
ΔR = c / (2B)
ΔR = 3e8 / (2 × 25e6)
ΔR = 6 m
```
The wavelength at 2.45 GHz is:
```text
λ = c / fc
λ = 3e8 / 2.45e9
λ ≈ 0.1224 m
```
If chirps are transmitted continuously with no idle time, the chirp repetition period is approximately:
```text
Tc = 30 µs
```
and the approximate maximum unambiguous radial velocity is:
```text
Vmax = λ / (4Tc)
Vmax ≈ 1020 m/s
```
---
GNU Radio Blocks
1. `fmcw_chirp_src`
Generates the transmitted FMCW chirp signal as a complex baseband waveform.
The basic chirp signal is:
```text
s(t) = A · exp(jπkt²)
```
where:
```text
A = amplitude
k = chirp slope
```
The block continuously repeats the chirp and outputs a `complex64` stream that can be connected to an SDR sink.
Important note: With a 30 MS/s sample rate and 25 MHz bandwidth, a centered chirp from `-12.5 MHz` to `+12.5 MHz` is safer than a chirp from `0 MHz` to `+25 MHz`, because the complex baseband Nyquist range is approximately `-15 MHz` to `+15 MHz`.
---
2. `radar_cube_builder`
Builds the radar datacube from the transmitted reference signal and the received signal.
Current inputs:
```text
Input 0: tx_ref
Input 1: rx0
```
Current output datacube shape:
```text
(1, chirps_per_frame, samples_per_chirp)
```
Axis meaning:
```text
Axis 0: RX channel
Axis 1: Chirp index / slow time
Axis 2: Sample index inside chirp / fast time
```
When dechirping is enabled, the block computes:
```text
beat signal = RX × conj(TX)
```
This produces the FMCW beat signal used for range estimation.
---
3. `range_fft_block`
Performs range processing by applying a Hanning window and an FFT along the fast-time dimension.
Input shape:
```text
(num_rx, chirps_per_frame, samples_per_chirp)
```
Output shape:
```text
(num_rx, chirps_per_frame, range_fft_len)
```
For the current implementation:
```text
Input shape:  (1, 1024, 900)
Output shape: (1, 1024, 1024)
```
This block converts the dechirped beat signal into range-frequency bins.
---
4. `doppler_fft_block`
Performs Doppler processing by applying a Hanning window and an FFT across the chirp dimension.
Input shape:
```text
(num_rx, chirps_per_frame, range_fft_len)
```
Output Range-Doppler Map shape:
```text
(range_fft_len, doppler_fft_len)
```
For the current implementation:
```text
RDM shape: (1024, 2048)
```
The Doppler FFT uses `fftshift`, so zero velocity appears at the center of the Doppler axis.
Important MUSIC note: The current `rdm_per_rx` output is magnitude-only. MUSIC DOA requires complex per-RX range-Doppler data, so this output must be modified in the future for real DOA estimation.
---
5. `cfar2d_block`
Applies a 2D Cell-Averaging Constant False Alarm Rate detector to the Range-Doppler Map.
The output is a binary detection map:
```text
0 = no detection
1 = target detection
```
The CFAR detector estimates the local noise level using training cells while excluding guard cells around the cell under test.
Default CFAR parameters:
```text
Training cells in range:    8
Training cells in Doppler:  8
Guard cells in range:       1
Guard cells in Doppler:     1
Probability of false alarm: 1e-6
```
With these values, the CFAR window size is:
```text
19 × 19 cells
```
and the number of training cells is:
```text
352 training cells
```
---
6. `rdm_display_block`
Displays the Range-Doppler Map using physical axes:
```text
x-axis: velocity in m/s
y-axis: range in meters
```
The block converts the normalized RDM to dB scale for visualization.
Recommended correction: If the input RDM is already power, use:
```python
arr_db = 10.0 * np.log10(np.maximum(arr, 1e-12))
```
Use `20log10` only for amplitude or magnitude data.
---
7. `cfar_display_block`
Displays the binary CFAR detection map using physical axes:
```text
x-axis: velocity in m/s
y-axis: range in meters
```
Detected targets appear as binary points in the Range-Doppler plane.
---
8. `music_doa_block`
Provides a MUSIC Direction-of-Arrival estimation framework.
The MUSIC algorithm estimates the target angle by using phase differences between multiple coherent receive antennas.
Current limitation: The present radar cube contains only one RX channel. Therefore, MUSIC DOA estimation is not physically meaningful in the current one-RX implementation.
For meaningful MUSIC DOA estimation, the system needs:
```text
1. At least two coherent RX channels
2. Known RX antenna spacing
3. Complex per-RX range-Doppler data
4. RX phase calibration
```
The current MUSIC block should be treated as a placeholder or future extension until multi-RX support is implemented.
---
9. `music_display_block`
Displays the MUSIC spatial spectrum as a function of angle.
The peak of the MUSIC spectrum corresponds to the estimated direction of arrival. In the current one-RX system, this plot is expected to be flat or non-meaningful because angle estimation requires multiple RX channels.
---
10. `matched_filter_block`
Applies matched-filter pulse compression along the fast-time dimension.
The matched filter is created as the time-reversed complex conjugate of the transmitted chirp:
```text
h(t) = s*(-t)
```
This block is useful when processing raw received chirps.
Important processing note: If `radar_cube_builder` is already using `dechirp=True`, the matched-filter block should normally not be inserted after it. Use either:
```text
Standard FMCW chain:
Dechirp → Range FFT → Doppler FFT
```
or:
```text
Matched-filter chain:
Raw RX chirps → Matched filter → further processing
```
Do not unintentionally combine both methods in a way that distorts the radar signal.
---
Recommended Repository Structure
```text
FMCW-Radar-2p45GHz-GNU-Radio/
│
├── README.md
├── LICENSE
├── .gitignore
│
├── grc/
│   └── fmcw_radar_2p45ghz.grc
│
├── blocks/
│   ├── fmcw_chirp_src.py
│   ├── radar_cube_builder.py
│   ├── range_fft_block.py
│   ├── doppler_fft_block.py
│   ├── cfar2d_block.py
│   ├── rdm_display_block.py
│   ├── cfar_display_block.py
│   ├── music_doa_block.py
│   ├── music_display_block.py
│   └── matched_filter_block.py
│
├── docs/
│   ├── architecture.md
│   └── processing_chain.md
│
├── images/
│   └── flowgraph.png
│
└── examples/
    └── example_parameters.md
```
---
Installation
Install the required Python packages:
```bash
pip install numpy matplotlib
```
GNU Radio must also be installed on the system.
For SDR operation, install the required hardware drivers for your SDR platform. For USRP devices, UHD must be installed and configured.
---
Usage
Clone the repository:
```bash
git clone https://github.com/your-username/FMCW-Radar-2p45GHz-GNU-Radio.git
cd FMCW-Radar-2p45GHz-GNU-Radio
```
Open the GNU Radio Companion flowgraph:
```bash
gnuradio-companion grc/fmcw_radar_2p45ghz.grc
```
Add or load the custom Python blocks.
Configure the radar parameters:
```text
sample_rate
center_freq
bandwidth
samples_per_chirp
chirps_per_frame
range_fft_len
doppler_fft_len
```
Run the flowgraph.
---
Processing Notes
Range Processing
The range FFT is applied along the fast-time samples of each chirp. In FMCW radar, the beat frequency is related to target range by:
```text
R = c fb / (2S)
```
where:
```text
R  = target range
c  = speed of light
fb = beat frequency
S  = FMCW chirp slope
```
---
Doppler Processing
The Doppler FFT is applied along the slow-time chirp dimension. Doppler information appears as phase variation from chirp to chirp.
The Doppler frequency is related to radial velocity by:
```text
v = fd λ / 2
```
where:
```text
v  = radial velocity
fd = Doppler frequency
λ  = wavelength
```
---
CFAR Detection
The CFAR detector applies an adaptive threshold to the Range-Doppler Map. It estimates the local noise level using surrounding training cells and excludes guard cells around the cell under test to avoid target-energy leakage into the noise estimate.
---
MUSIC DOA
MUSIC DOA is included as a future extension. In the current code, the system has only one RX channel, so DOA estimation is not physically valid.
To make MUSIC meaningful, future versions should modify the pipeline to output complex per-RX range-Doppler data with shape:
```text
(num_rx, range_bins, doppler_bins)
```
where `num_rx >= 2`.
---
Current Limitations
The current implementation has one RX channel.
MUSIC DOA is included only as a future framework.
MUSIC requires complex multi-RX data, but the current `rdm_per_rx` output is magnitude-only.
Python-based 2D CFAR may be slow for large real-time Range-Doppler Maps.
The range axis in display blocks should be calibrated carefully when zero-padding is used.
The velocity axis depends on the true chirp repetition period, including any idle time.
SDR transmission must comply with local radio-frequency regulations.
Accurate real-world radar operation requires hardware calibration and synchronization.
---
Safety and Compliance
This project is for academic and experimental radar signal-processing work. Users are responsible for ensuring that any RF transmission complies with local regulations, SDR hardware limits, bandwidth restrictions, and frequency allocation rules.
---
Author
Naif Abdulkarim Alanazi  
Electrical Engineering  
Project: FMCW Radar 2.45 GHz Processing System
---
---

