# Satellite Communications Basics


## view

A satellite communication link is a radio path between a ground station and a satellite (uplink when ground→satellite, downlink when satellite→ground). Designing and analyzing these links requires accounting for propagation loss, antenna performance, transmitter power, receiver sensitivity, noise, and dynamic effects from motion. //Doppler

---

## Link budget

A **link budget** is an accounting of gains and losses between transmitter and receiver that predicts received signal strength and margin. Instead of raw formulas, this section describes the components you must consider and why each matters for satellite links.

### Transmitter side

- **Transmit power (Tx)**: the RF power the transmitter puts into the antenna feed (often expressed as dBW or dBm). Higher transmit power increases the chance the receiver can detect and demodulate the signal, but is limited by regulatory and hardware constraints.
- **Antenna gain (Tx gain)**: how effectively the antenna focuses power into a particular direction. High-gain antennas (parabolic dishes) concentrate energy into a narrow beam, greatly improving link range in that direction but requiring accurate pointing.
- **EIRP (Effective Isotropic Radiated Power)**: the combination of transmitter power and antenna gain expressed as if power were radiated isotropically. EIRP is the real-world quantity that determines how much power is sent toward the receiver.

### Propagation losses

- **Free-space path loss (FSPL)**: the geometric spreading of energy with distance causes loss proportional to the square of distance and the square of frequency. FSPL grows quickly with higher frequency and longer range — an important reason higher-frequency links (S/X/Ku/Ka-band) require larger antennas or more power.
- **Atmospheric & weather losses**: rain, humidity, gaseous absorption and scintillation can add additional losses at certain frequencies (ex: rain fade at Ku/Ka bands). For LEO amateur work these are usually minor but can be important for geostationary or Ka-band links.

### Receive side

- **Antenna gain (Rx gain)**: as with transmit, the ground antenna's gain determines how much of the arriving signal is captured. Parabolic dishes and helical antennas provide higher gain than simple dipoles or small Yagis.
- **System noise temperature and G/T**: the receiver front-end contributes noise (from LNA noise figure, feed losses, and sky noise). The figure-of-merit G/T (antenna gain over system noise temperature) captures how well the ground station converts incoming energy to usable signal relative to noise. Higher G/T = better sensitivity.
- **Receiver sensitivity & required Eb/N0**: the minimum signal level at the receiver needed to decode a given modulation and coding reliably depends on the *energy per bit to noise density ratio* (Eb/N0). Robust modulations and strong FEC require lower Eb/N0 to work.

### Link margin

- **Link margin** is the difference between the predicted received signal level (given the link budget) and the minimum required signal for reliable communications (receiver sensitivity). Positive margin indicates expected reliable operation; negative margin predicts failure or high error rates. Designers include fade margin to cover variabilities. //pointing errors, atmospheric effects, hardware tolerances

>[!IMPORTANT] 
>
>link margin determines how easy it is to detect or jam a signal. Low-margin links are fragile and susceptible to interference; high-margin links are more resilient.

---

## Uplink vs Downlink — key differences

- **Directionality of impairments:** uplink and downlink paths can experience differing impairments. For example, a small ground transmitter may have limited EIRP for uplink, while a satellite downlink may have limited antenna gain or power budget.
- **Asymmetric constraints:** spacecraft often operate with tight power and antenna-size constraints (limited EIRP). Ground stations can often compensate with large, high-gain antennas and better G/T. This asymmetry changes what protection measures are practical on each side.
- **Licensing & regulatory differences:** uplinks may require stricter licensing (transmitting toward a satellite) than downlink monitoring; always verify legal requirements before transmitting.

>[!IMPORTANT]
>
>From a security perspective, uplinks are the high-risk channel for unauthorized commands — they must be tightly controlled and authenticated. Downlinks carry telemetry and may reveal sensitive state if unprotected.

---

## Doppler effect in satellite links

- **What happens:** relative motion between the transmitter and receiver (satellite-to-ground when the satellite moves across the sky) causes the received frequency to be shifted — this is the Doppler effect. For LEO satellites the Doppler shift can be large and varies quickly during a pass; for geostationary satellites it is negligible.
- **Implications for receivers:** tuners and demodulators must compensate for Doppler to keep the carrier within the demodulator's passband. Compensation can happen by precomputing a Doppler profile (using orbital predictions), by closed-loop tracking (PLL/Costas loops inside the demodulator), or by adaptive frequency correction in SDR software.
- **Security angle:** Doppler makes spoofing and replay attacks more complex — an attacker must mimic the Doppler profile to avoid detection. Conversely, attackers can exploit Doppler-ignorant receivers by injecting signals at shifted frequencies to cause desynchronization or denial of service.

---

## Ground station components

A functional ground station for satellite work typically includes these elements:

- **Antenna**: captures or radiates RF energy. Common types:
  - *Parabolic dish*: high gain, narrow beam — used for higher-frequency or low-EIRP links.
  - *Helical antenna*: often used for circular polarization and moderate gain (VHF/UHF satellite beacons and telemetry).
  - *Yagi / dipole arrays*: low-to-moderate gain for VHF/UHF with simpler physical setups.
- **Rotator / tracking mount**: mechanical system that allows pointing the antenna in azimuth and elevation (or full polar mounts) to follow LEO passes. Accurate pointing is critical for high-gain antennas.
- **Low-noise amplifier (LNA) / preamplifier**: placed close to the antenna feed to raise weak signals above downstream cable losses and receiver noise.
- **Filters (bandpass, notch)**: prevent strong out-of-band signals from desensitizing the receiver. For example, bandpass filters block broadcast FM or mobile signals that could overload the front-end.
- **SDR / receiver front-end**: digitizes signals and produces IQ streams for software processing. In a receive-only stack, the SDR plus host PC performs demodulation, decoding, and logging.
- **Control & processing software**: software coordinates pointing schedules (GPredict, SatNOGS components), handles Doppler correction for the SDR, demodulates data, and stores telemetry.
- **Cabling, lightning protection, grounding**: practical but important — poor grounding or lack of surge protection risks equipment and can add noise.

>[!IMPORTANT]
>
> physical security of ground-station equipment (antenna access, control systems) is as important as RF security. An attacker with access to a rotator or transmitter can disrupt operations.

---

## Polarization (linear vs circular)

- **Polarization** is the orientation of the electric field vector in the electromagnetic wave. Matching polarization between transmitter and receiver maximizes received power.
- **Linear polarization**: ex: horizontal or vertical; easy to implement physically but susceptible to polarization mismatch due to antenna orientation changes or Faraday rotation in the ionosphere.
- **Circular polarization**: right-hand or left-hand circular polarization (RHCP/LHCP) rotates the electric field vector and is commonly used for satellite links because it is less sensitive to orientation mismatch between moving satellite and ground antenna.

>[!WARNING]
>
>**Cross-polarization loss:** mismatched polarizations cause additional loss (up to many dB), so correct polarization selection and alignment are critical for link margin. For security, attackers who use the wrong polarization will suffer reduced effectiveness unless they explicitly match it.

---

## Beacon signals and housekeeping data

- **Beacons** are simple, continuous or scheduled transmissions that satellites use to announce presence, health, or telemetry. They are often unencrypted and intended for easy reception by many ground stations.
- **Housekeeping (HK) data** contains satellite health metrics (battery voltages, temperatures, status flags). HK is essential for operations but can be sensitive — an unprotected beacon revealing system state can aid adversaries (ex: telemetry revealing a vulnerable subsystem or when a satellite is in a critical low-power state).

**recommendations:**
- Treat unprotected beacons as observable information — adversaries can use them for reconnaissance and timing attacks.
- Where possible in operational missions, restrict critical state information and implement authentication and encryption for telecommands and sensitive telemetry channels.

---

## checks & diagnostic

- **If received power is low:** check antenna pointing, polarization, cable losses, LNA biasing, and whether a wrong center frequency or Doppler correction is applied.
- **If the pass shows frequency drift:** verify orbital prediction source and ensure SDR is applying Doppler compensation; verify LO stability of SDR and any external frequency references.
- **If demodulation fails despite adequate power:** inspect modulation format, symbol rate mismatch, sample rate misconfiguration, or incorrect clock recovery. Also check cross-polarization and interference.

---


### Glossary //quick terms

- **EIRP:** Effective Isotropic Radiated Power.
- **FSPL:** Free-space path loss.
- **Eb/N0:** Energy per bit to noise density ratio (measure of link quality for digital links).
- **G/T:** Antenna gain over system noise temperature — a ground station sensitivity metric.
- **LNA:** Low-noise amplifier.
- **LEO / GEO:** Low Earth Orbit / Geostationary Earth Orbit.



// also check out https://www.mathworks.com/help/satcom/gs/satellite-link-budget.html for further reading!
