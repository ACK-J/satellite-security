# Software-Defined Radio (SDR) Basics



## Software-Defined Radio 

A Software-Defined Radio (SDR) is a radio communication system where signal processing tasks traditionally implemented in dedicated hardware (mixers, filters, modulators/demodulators) are instead done in software. The physical radio front-end converts wireless RF energy to digital samples; the heavy lifting — demodulation, decoding, transmission processing — happens on a programmable device (FPGA/CPU/GPU) or the host PC.

In the context of satellite security, SDRs are invaluable because they provide flexibility (reconfigure for new bands and modulations), reproducibility (you can record raw IQ for analysis), and accessibility. //many low-cost SDRs exist

---

## SDR architecture (building blocks)

### RF front-end

- **Antenna**: collects the electromagnetic waves.
- **Bandpass filtering**: early filtering prevents strong out-of-band signals from saturating the receive chain.
- **Low-Noise Amplifier (LNA)**: amplifies weak signals at the front-end while adding as little noise as possible.
- **Switches and duplexers** (when supporting transmit/receive) separate TX and RX paths.
- **Mixers / local oscillators (LOs)**: translate RF to an intermediate or baseband frequency. Many SDRs use direct-conversion (zero-IF) where RF is translated directly to complex baseband.
- **Automatic gain control (AGC) or manual gain stages**: set the signal level ahead of the ADC.

>[!TIP]
>
>satellite signals are often weak and narrowband; a good RF front-end (antenna + LNA + filters) and accurate LO frequency control are critical.

### ADC (Analog-to-Digital Converter)

- The ADC digitizes the analog baseband or intermediate-frequency signal into discrete samples.
- Key properties: **sample rate**, **resolution (bits)**, and **dynamic range**.
- ADC resolution (bits) determines quantization noise and the effective dynamic range that can be encoded digitally.

### FPGA / DSP block

- Many SDRs use an FPGA or dedicated DSP to implement high-speed tasks that cannot be done in software on a host PC in real time.
- Typical FPGA tasks: digital downconversion (DDC), decimation/interpolation, resampling, filtering, and sometimes fast arithmetic for real-time demodulation.

### Host PC / Control software

- The host computer runs higher-level signal processing tasks (demodulators, decoders, visualization, logging) and user interfaces.
- Host software can be GNU Radio flowgraphs, SDR applications (GQRX, SDR#), or custom code that consumes IQ streams.

---

## IQ (I/Q) signals and complex baseband

- SDRs commonly work with **complex baseband samples** composed of an In-phase (I) and Quadrature (Q) pair. These represent the amplitude and phase of the received signal at baseband.
- IQ samples are typically interleaved in the recording (I0, Q0, I1, Q1, ...).
- Representations: signed integers (ex: 8-bit `int8`, 16-bit `int16`) or floating point (`float32` complex pairs). The representation affects storage size and dynamic range.


>[!IMPORTANT]
>
>**Why complex/IQ?** Complex representation allows a single-sideband representation of the positive and negative frequency contents, makes frequency translation trivial (multiply by complex phasor), and supports preserving phase information needed by many modulations.

---

## IQ recording & playback

- Recording raw IQ is essential for reproducible analysis: you can replay the exact radio environment later, try different demodulators, and share captures with others.
- Typical recording considerations:
  - **Sample rate:** choose a rate that covers the whole signal of interest (many satellite downlinks are narrowband; others require larger bandwidths).
  - **Format / bit depth:** common choices are signed 16-bit integer interleaved IQ (`int16`) or 32-bit floating-point (`float32`) complex.
  - **File size:** IQ files grow quickly. Calculate size = sample_rate × bytes_per_sample × capture_duration.
  - **Metadata:** include capture frequency, sample rate, center frequency, and any frequency offsets — otherwise replay may be ambiguous.


---

## Data formats — common on SDR workflows

- **Raw IQ (binary interleaved)**: simple raw files with interleaved I/Q samples and no header. Many tools support this format but metadata must be tracked separately.

- **WAV IQ**: standard WAV files can be used to store IQ by using two channels (left = I, right = Q) and a known sample format (ex: 16-bit). This makes playback easy with audio tools compatible with WAV.

- **SigMF (Signal Metadata Format)**: an open metadata standard to describe IQ captures in a JSON sidecar (describes sample rate, center frequency, annotations). SigMF helps preserve context for captures and is recommended for reproducibility and sharing.

- **Tool-specific wrappers**: some SDR software uses its own capture format (with headers) — always check the tool's documentation when exchanging files.

---

## Sample rate, decimation and the role of resampling

- **Sample rate** determines the instantaneous bandwidth captured around the SDR's center frequency.
- **Decimation** in the FPGA or DSP reduces sample rate while preserving the signal of interest (through filtering + down-sampling) to reduce CPU load on the host.
- **Resampling** on the host can convert recorded data to the rate required by a demodulator or decoder.

>[!TIP]
>
> start with a higher sample rate to capture the whole signal (allowing margin for Doppler), then decimate or extract narrow channels for decoding.

---

## Dynamic range, gain control, and AGC

- **Dynamic range**: range between the smallest detectable signal above noise and the largest signal the ADC can represent without clipping. Higher ADC bit depth usually yields larger dynamic range.
- **Noise floor**: set by the front-end and ADC quantization; weaker satellite signals may be close to the noise floor.
- **Gain staging**: the combination of LNA gain, variable gain amplifiers, and digital gain controls determines whether the ADC input is near optimal amplitude.
- **Clipping and saturation**: if gain is too high, strong signals will clip and corrupt IQ; if too low, quantization noise dominates.
- **Automatic Gain Control (AGC)**: AGC adapts gain in real time to maintain desired amplitude at the ADC or in digital processing. AGC algorithms vary in speed and strategy and may be unsuitable for signals with rapid level changes unless tuned.

>[!TIP]
>
>avoid strong local signals (broadcast stations, nearby transmitters) that can desensitize the receiver. Use filters and proper gain staging; sometimes a small amount of attenuation prevents front-end compression when a strong nearby signal is present.

---

## Imperfections & calibration 

- **DC offset**: direct-conversion SDRs often show a DC spike at zero frequency. Software routines typically remove it in post-processing.
- **IQ imbalance**: amplitude and phase mismatch between I and Q channels can create mirror images or degrade demodulation performance. Many SDR toolchains include IQ imbalance compensation blocks.
- **Frequency offset and LO drift**: the SDR's LO may be offset from its nominal frequency. For satellite work, frequency accuracy matters: correct for offsets and Doppler.
- **Phase noise**: LO instability causes phase noise which can reduce demodulator performance, especially for higher-order modulations.

---

## SDR platform notes 

- **RTL-SDR**: very low-cost USB dongles originally designed as DVB-T receivers. Typically limited in dynamic range and sample depth. //popular for learning and quick scans

- **HackRF**: low-cost transceiver capable of wideband operation. //hundreds of kHz to several MHz depending on configuration

- **BladeRF**: mid-range FPGA-based SDR offering higher performance and flexibility; useful when you need better dynamic range and lower latencies than entry-level devices.

- **ADALM-PLUTO (PlutoSDR)**: compact transceiver with decent performance and an integrated FPGA. //often used for labs and portable operations

- **Other platforms**: LimeSDR, USRP (Ettus) — these provide more channels, wider tuning ranges, and professional-grade performance but at a higher cost.

---

## SDR software ecosystem

- **GNU Radio**: flowgraph-oriented toolkit for building signal processing chains. It exposes blocks for filters, demodulators, and IO, and is widely used in research and education.
- **GQRX**: a simple graphical receiver useful for quick tuning, spectrum/waterfall visualization, and listening to audio-range signals.
- **SDR# (SDRSharp)**: Windows-oriented GUI-based SDR receiver popular in certain communities for its plugin ecosystem.
- **CubicSDR**: cross-platform GUI receiver with a simple user experience.
- **gr-osmosdr / SoapySDR / RTL-SDR drivers**: driver layers that connect hardware to software (GNU Radio, SDRangel, etc.).
- **SDRangel, Pothos, URH**: other tools for specialized functions (transmit chains, modulation analysis, protocol reverse-engineering).

>[!IMPORTANT]
>
>understanding the role of each layer (hardware driver → stream of IQ → signal processing blocks → demodulator/decoder) will help you map tool behavior to theoretical concepts.

---

## File formats

- **Raw interleaved IQ**: binary files written by many capture tools. Must be opened with correct endianness, sample type, and rate.
- **WAV IQ**: easier for audio tools; two-channel WAV where left/right = I/Q.
- **SigMF**: JSON sidecar plus raw IQ; recommended because it preserves essential metadata and annotations.



---

## Special considerations for satellite signals

- **Doppler shift**: moving satellites induce Doppler; this shifts received frequency during pass. Compensation can be done in software by shifting frequency in real time or during post-processing.
- **Narrowband vs wideband downlinks**: many telemetry and beacon signals are narrow; others (ex: some telemetry carriers or experimental payloads) may require broader capture bandwidths.
- **Frequency coordination & legality**: when using transmit-capable SDRs, be mindful of regulations and do not transmit on satellite uplink/downlink frequencies unless authorized.

---

## Troubleshooting 

- If the desired signal is absent: verify antenna alignment, check passband filters, ensure the SDR is tuned to the correct center frequency and sample rate.
- If the signal is corrupted or contains strong spurious tones: inspect for DC spike, look for local strong interferers, check ADC clipping or front-end compression.
- If demodulation fails despite visible carrier: check frequency offset and Doppler compensation, IQ imbalance, and baseband sample rate alignment required by the demodulator.

---



### Glossary // quick terms

- **IQ**: In-phase / Quadrature components of a complex baseband signal.
- **ADC**: Analog-to-digital converter.
- **AGC**: Automatic gain control.
- **LO**: Local oscillator.
- **Sample rate**: number of complex samples per second (Hz).
- **SigMF**: Signal Metadata Format (JSON sidecar describing captures).




