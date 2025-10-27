# Tools & Ecosystem




## How to think about this?

A healthy satellite-security toolchain resembles a data pipeline with clear layers and responsibilities. Conceptually separate these layers when designing labs or operational systems:

- **Sensing & acquisition** — antennas, SDRs, front-ends that capture RF energy and convert it into IQ samples.
- **Signal processing & demodulation** — software blocks that convert IQ into symbol streams and extract frames (GNU Radio flowgraphs, OOT modules).
- **Decoding & protocol analysis** — demultiplexing, deframing and interpreting telemetry or network protocols (gr-satellites, custom decoders).
- **Orbit & timing services** — prediction, Doppler profiles, and time synchronization used to guide receivers and sanity-check observations (Gpredict, Orekit references).
- **Networking & distributed reception** — systems that aggregate remote stations, share captures, and coordinate observations (SatNOGS-like networks).
- **Reverse-engineering & research tools** — utilities to analyze unknown protocols, intercept ISM traffic, or experiment with modulation (RFQuack-style tools).
- **Test & simulation** — signal generators and simulators (ex: GNSS simulators) used for testing and research in controlled environments.
- **Data storage, provenance, & analysis** — raw IQ archives, decoded frames, processed telemetry databases, and chain-of-custody artifacts.

---

## GNU Radio 

GNU Radio is a modular signal-processing framework whose primary role is to implement flowgraphs: chains of signal-processing blocks that perform filtering, mixing, resampling, demodulation, and higher-level protocol parsing.

- **Typical inputs:** IQ streams (RTL, HackRF, BladeRF, files), clocked sample flows.
- **Typical outputs:** demodulated audio/bitstreams, packetized frames, visual diagnostics (spectrum, constellation). Outputs are often piped into decoders (ex: gr-satellites) or recorded as WAV/IQ files.
- **Why it matters:** GNU Radio lets you experiment with alternate demodulators quickly, insert instrumentation, or prototype novel detection algorithms.

>[!TIP]
>
>instrument GNU Radio flowgraphs to log intermediate stages (ex: constellation error, SNR over time). These signals can give early indications of protocol-aware jamming or tampering.

---

## gr-satellites and telemetry decoders 

gr-satellites is an ecosystem of GNU Radio OOT modules and decoders tailored for many satellite beacons and telemetry formats. Conceptually it is part of the decoding/protocol layer.

- **Role:** take demodulated bitstreams and extract telemetry frames (AX.25, CCSDS, vendor formats). It maps known modulation/baud formats to parsers and outputs decoded fields.
- **Inputs/outputs:** expects synchronized bitstreams (often from GNU Radio) and emits decoded packets, log-friendly JSON, or CSV for ingestion.

>[!WARNING]
>
>always preserve the raw demodulated bitstream and the time/metadata alongside decoded outputs to enable later forensic re-analysis if a decoder behaves unexpectedly.

---

## SatNOGS

SatNOGS is an example of a distributed ground-station network that coordinates observations, schedules passes, and aggregates captures from volunteer stations.

- **Conceptual value:** expands geographic coverage, provides redundancy, and enables collaborative verification of observations (useful for cross-validation and anomaly confirmation).
- **Data flows:** scheduler → station executes capture (SDR + rotator) → uploads IQ/demods/decoded frames to central repository → indexing and public sharing.

>[!TIP]
>
> when using crowdsourced data for security conclusions, require corroboration and provenance metadata; do not base high-impact decisions on single-station observations.

---

## Gpredict & automated antenna control

Gpredict and similar prediction tools provide the real-time pass schedules and Doppler offsets needed by rotator controllers and SDR clients.

- **Function in pipeline:** compute AOS/LOS, Az/El tracks, and Doppler profiles from ephemerides, then feed setpoints to rotator controllers and frequency-control daemons.
- **Integration concerns:** ensure that command paths to rotators and SDRs are authenticated and that automated scripts enforce safe limits to prevent mispointing or commands that could expose hardware to risk.

>[!IMPORTANT]
>
>log every automated pointing or frequency command with operator identity and provide a secure manual override; automate sanity checks that verify predicted and actual telemetry match expected profiles after actions.

---

## RFQuack & reverse-engineering tools — role and ethical boundaries

RFQuack-style toolkits are invaluable for exploring proprietary or undocumented ISM/UHF protocols, capturing unknown beacons, and prototyping modulation/demodulation strategies.

- **Role:** rapid capture, replay (in controlled lab conditions), fuzzing, and protocol introspection for security analysis.
- these tools can also be misused for interference or unauthorized access. Always perform active experiments (replay, transmit) in closed, authorized test environments and follow local laws.

>[!TIP]
>
> when analyzing unknown protocols, refrain from on-air replays unless explicit permission is granted; instead prefer offline lab-based emulation with RF shielding. //Faraday cage

---

## GPS‑SDR‑SIM and GNSS simulation

GNSS simulators like GPS‑SDR‑SIM are critical test tools for receiver validation, Doppler/time synchronization testing, and algorithm development. They can also be misused for spoofing experiments if misapplied on open air.

- **Test roles:** produce reproducible GNSS time/position scenarios for ground-station and satellite-onboard tests (when legal and contained).
- **Safety note:** never connect simulators to real-world antennas without proper containment; local transmissions may disrupt receivers and violate regulations.


---

## Orekit & orbit/motion services

Orekit (mentioned as a reference in other chapters) provides accurate propagation, frame transforms, and event detection (rise/set, eclipses). In the tools ecosystem its role is to provide authoritative ephemeris-based services that feed prediction, scheduling, and validation pipelines.

- **Functional interface:** ephemeris ingestion → prediction/Doppler outputs → scheduling and validation layers.
- **Integration note:** keep ephemeris sources and Orekit configurations in version-controlled artifacts so you can reproduce predictions and validate historical analyses.

>[!TIP]
>
>verify ephemeris sources and sign ephemeris files where possible; do not accept untrusted ephemeris updates without validation and cross-checks.

---

## File formats, metadata conventions and interoperability

Interoperability is often the harder part of a toolchain. Standardize how components exchange data:

- **IQ capture formats:** use SigMF or a stable documented sidecar to preserve sample rate, center frequency, sample format, capture time, and device configuration.
- **Decoded telemetry:** prefer a structured, versioned JSON schema (or protobuf) that includes frame-level metadata, decoding parameters, and provenance fields.
- **Ephemeris & TLEs:** source and timestamp ephemeris files; if you transform or re-fit elements, store the transformation parameters.
- **Log & audit trails:** standardize log formats (timestamps in ISO 8601 UTC, operator IDs, tool versions) and store hashes for forensic traceability.

---



## Logging, telemetry, and provenance across the stack

Collect structured telemetry from each tool (CPU, Uptime, flowgraph version, sample rates, RTL/driver versions, rotator firmware). Correlate tool telemetry with captured signal metadata to build a full picture for forensic analysis.

- **Minimum provenance model:** capture-id, capture-timestamp (UTC), device-id + firmware, operator-id, flowgraph-id/version, decoder-id/version, ephemeris-id/version, and processed-output-hash.
- **Auditability:** ensure logs are tamper-evident (append-only stores, signed manifests), and keep a chain-of-custody ledger linking operators to actions and artifacts.

---

## Community resources, collaboration and contribution

The satellite-security ecosystem is community-driven. Contribute reproducible examples, sample captures, and well-documented decoders. When sharing captures publicly, sanitize any sensitive metadata and comply with contributor agreements.

Suggested collaboration patterns:
- Maintain a canonical repository of annotated captures (SigMF + JSON metadata) for benchmarking.
- Share decoder test suites and sample flowgraphs as container images.
- Use ephemeral keys and signed submissions for crowd-sourced networks to enable trust without exposing long-term secrets.


---

### Glossary// quick terms

- **IQ:** in-phase/quadrature sample pairs from SDRs.
- **OOT:** Out-of-tree module (GNU Radio extension).
- **SigMF:** Signal Metadata Format.
- **Flowgraph:** GNU Radio pipeline of connected blocks.
- **EIRP / Doppler / AOS/LOS:** referenced elsewhere; crucial to mapping tools to operations.
