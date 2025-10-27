# gr-satellites

> **Tool:** gr-satellites ([daniestevez/gr-satellites](https://github.com/daniestevez/gr-satellites))
>
> A GNU Radio out-of-tree (OOT) module that provides a large collection of telemetry decoders, deframers, and satellite-specific helpers for many amateur LEO CubeSats and smallsats. It is commonly used via the `gr_satellites` command-line tool or as blocks in GNU Radio Companion flowgraphs.

---

## 1. Practical scope & intent

This file focuses on: how to use gr-satellites as a workhorse for parsing archived or live satellite IQ, extracting telemetry frames, building fingerprints, and integrating decoder outputs into monitoring and forensic pipelines. It assumes you already use SDR toolchains ([GNU Radio](./GNU_radio.md), [SatNOGS](./satNOGS.md)) and skips generic SDR theory.

---

## 2. What gr-satellites gives you

- **A wide decoder catalogue:** ready-made decoders and transport-layer deframers for dozens to hundreds of amateur satellites. //AX.25, AX100, CSP, AFSK, GMSK, BPSK variants, custom framing
- **Command-line decoding (gr_satellites):** decode live SDR streams or recorded files without writing custom flowgraphs — ideal for fast analysis of archive data or integrating into scripts.
- **GRC block / flowgraph integration:** use the Satellite Decoder block to embed supported-decoder options into GNU Radio Companion flowgraphs for more complex processing or pre/post-processing.
- **SatYAML metadata-driven support:** satellite support is described in SatYAML files that list transmitters, center frequencies, modulation and framing parameters — useful for automated workflows.
- **Auxiliary tools:** doppler helpers, converters (SigMF support), and utilities to convert between capture formats or apply time-synchronisation tags.

---

## 3.  use-cases 

- **Automated archive processing:** batch-decode SatNOGS or local IQ archives with `gr_satellites` to extract telemetry frames and metadata for indexing into a forensic DB.
- **Signature & fingerprint extraction:** use decoded frames and raw IQ captures to build signatures (preamble shapes, header patterns, timing distributions) to detect unauthorized transmitters or protocol deviations.
- **Protocol validation & anomaly detection:** check for unexpected changes in framing, CRC failures, or novel packet types indicating potential injection, replay, or firmware anomalies on the satellite.
- **Rapid reconnaissance:** when monitoring a new or unknown satellite, try supported decoders first — gr-satellites often identifies format parameters quickly, saving time over building a custom decoder.
- **Poisoning detection:** compare outputs from gr-satellites decoders run on different stations/epochs to find inconsistent decodes that could indicate poisoning or spoofed telemetry.

---

## 4. How to use it?

1. **Install matching version:** align gr-satellites release with your GNU Radio version. //see releases and compatibility notes  
2. **Decode a recording:** use `gr_satellites` to run a supported decoder on a WAV/IQ file or directly from an SDR input. // CLI options let you pick satellite by name or SatYAML 
3. **Ingest outputs:** capture the decoder’s KISS/TCP/JSON outputs, save raw frames, and index them into your evidence store with timestamps and provenance metadata.  
4. **Automate:** wrap `gr_satellites` calls in scripts to process SatNOGS downloads or new local captures; tag each decode with station ID/TLE/pass info.  

>[!IMPORTANT]
>
>*Note: the `gr_satellites` tool supports both live SDR inputs and offline files — consult the tool’s docs for exact CLI flags.*

---

## 5. Integration points 

- **SatNOGS:** process archived IQs from SatNOGS observations to extract telemetry at scale. There are community add-ons (ex: `satnogs_gr-satellites`) that extend station-side decoding support.  
- **GNU Radio flowgraphs:** embed the Satellite Decoder block in GRC when you need prefiltering, spectral analysis, or chaining decoders.  
- **SIEM / analytics:** export decoded packets and metadata as JSON/KISS for downstream enrichment, correlation with TLEs, and anomaly alerting.  
- **SigMF & reproducibility:** use SigMF-compatible captures and the converter support to maintain reproducible datasets for audit and testing.

---

## 6. Operational considerations & security hardening

- **Version compatibility:** ensure decoder versions match the GNU Radio API; mismatches can break decoders or create false negatives.  
- **Provenance tags:** always tag decoder outputs with capture source, station ID, timestamp, and software version to maintain chain-of-custody.  
- **Trust assumptions:** treat results from third-party-sourced decoders or community-run stations as unverified until corroborated.  
- **Automation safety:** run bulk decoding in isolated environments; malformed frames occasionally trigger parser bugs — keep tools and OS up to date.

---

## 7. Threats & dual-use considerations

- **Facilitates reconnaissance:** easy decoding lowers barrier for adversaries to understand protocols and payload fields.  
- **Decoder fingerprinting:** known decoders may produce stereotyped outputs — watch for attackers using identical tools to craft deceptive telemetry.  
- **Denial-of-service on decoders:** malformed or intentionally crafted captures might trigger crashes or lockups in certain decoder paths — sandbox batch jobs.

---

## 8. Mitigations &  recommendations

- **Corroboration before conclusions:** require multi-source confirmation for critical alerts; use cross-station comparison.  
- **Harden decode pipelines:** run decoders in containers, limit resource access, and monitor for crashes or abnormal behavior during bulk processing.  
- **Sanitize and validate inputs:** check checksums/CRCs and apply sanity filters before trusting decoded fields; flag unexpected rate-of-change in telemetry.  
- **Contribute fixes upstream:** when you find protocol edge-cases or parser bugs, open issues or PRs — community maintenance reduces long-term risk.

---

## 9. Practical resources & links

- gr-satellites docs (Read the Docs!!): https://gr-satellites.readthedocs.io/en/latest/  
- GitHub repository: https://github.com/daniestevez/gr-satellites  
- Supported satellites index: https://gr-satellites.readthedocs.io/en/latest/supported_satellites.html  
- Command-line tool docs: https://gr-satellites.readthedocs.io/en/latest/command_line.html  
- Releases & compatibility notes: https://github.com/daniestevez/gr-satellites/releases  
- Community discussion (Libre Space / SatNOGS integration): https://community.libre.space/t/install-the-satnogs-gr-satellites-addon-and-use-your-station-more/8027  
- Debian packaging / distro info: https://tracker.debian.org/gr-satellites

---


