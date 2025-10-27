# Security Considerations



## Signal spoofing and replay attacks

Signal spoofing occurs when an adversary transmits radio signals that mimic legitimate satellite signals, with the goal of confusing receivers, inducing incorrect state changes, or hiding real signals. Replay attacks are a special case where previously-captured legitimate signals are re-transmitted later to cause incorrect behavior.

### How spoofing and replay work?

- **Direct spoofing:** an attacker generates signals with similar modulation, timing, and framing to the target satellite. If the spoofed signal is stronger at the victim receiver (or properly timed), the receiver may accept it as genuine.
- **Relay-and-delay:** an attacker can capture a live signal and re-transmit it with a controlled delay or frequency offset to spoof geographic or timing information.
- **Replay of recorded telemetry or command frames:** previously recorded frames (including telecommands or telemetry) can be replayed to produce the same operational effects as the original transmission unless freshness protections exist.

### Observable indicators of spoofing/replay

- **Doppler inconsistency:** spoofed transmissions that do not follow the expected Doppler curve for the satellite path will show frequency evolution mismatches when compared to predicted profiles.
- **Timing & sequence anomalies:** repeated sequence numbers, out-of-order timestamps, or duplicate packets where none are expected.
- **Signal strength and angle-of-arrival discrepancies:** if the signal’s amplitude or apparent direction differs from the satellite’s predicted geometry, suspect spoofing.
- **Payload inconsistencies:** telemetry that contains impossible or contradictory values (ex: impossible attitude or energy state) or mismatched cryptographic MACs.

### High-level mitigations and resilience patterns

- **Cryptographic authentication & freshness:** authenticate messages (telemetry and commands) using message authentication codes or digital signatures combined with nonces, sequence numbers, or timestamps to detect replays.
- **Cross-check Doppler & ephemeris:** make it routine to test whether received carrier frequency evolution matches predicted Doppler from trusted ephemerides.
- **Multi-sensor/antenna correlation:** compare signals across spatially separated ground stations or from multiple antennas (angle-of-arrival) to confirm signal origin.

>[!TIP]
>
>treat Doppler and timing as inexpensive and effective anti-spoofing checks — even unsophisticated systems can detect many spoofing attempts by validating frequency evolution against a predicted profile.

---

## GPS/GNSS spoofing //GPS-SDR-SIM

GNSS spoofing targets navigation and timing receivers by broadcasting counterfeit GNSS signals or by manipulating GNSS-like simulators. In satellites and ground stations, GNSS provides time and position; corrupting it can desynchronize systems or cause mis-pointing.

### Attack vectors and impacts

- **Local receiver spoofing:** an attacker near a ground station can overpower genuine GNSS signals with local transmissions that cause the station’s clock or position solution to shift.
- **GNSS time spoofing for coordinated attacks:** time offsets can break authentication schemes that use timestamps or make Doppler/ephemeris checks inconsistent across stations.
- **Onboard receiver spoofing:** if a satellite depends on GNSS for orbit determination or timing, onboard spoofing can cause erroneous attitude control, navigation errors, or mis-scheduled actions.

### Defenses & best practices

- **Multi-source time & position validation:** cross-check GNSS-derived time against secondary sources (network time, atomic references, inter-satellite links) and reject large unexplained offsets.
- **Signal-level GNSS authentication:** modern GNSS systems (ex: Galileo OSNMA, future GPS upgrades) include cryptographic authentication features — adopt authenticated GNSS where mission-critical.
- **Antenna and RF protections:** use choke rings or antenna siting to reduce vulnerability to local spoofers and consider directional nulling and RF monitoring for unexpected strong local GNSS carriers.

>[!WARNING]
>
> never accept GNSS time as the sole authoritative time source for critical security decisions — include independent cross-checks and sanity checks on clock behavior.

---

## Jamming and denial of service

Jamming is the intentional emission of RF energy to raise the noise floor, saturate receivers, or otherwise prevent correct demodulation. Denial-of-service can also be indirect (flooding ground networks with telemetry data, saturating processing pipelines, or physically blocking access to ground assets).

### Types of jamming

- **Broadband noise jamming:** raises the noise floor over a wide band, reducing SNR and causing packet loss.
- **Narrowband or tone jamming:** targets the carrier or a control channel specifically; can be much more power-efficient and hard to detect if intermittent.
- **Protocol-aware jamming:** selectively targets preambles, synchronization symbols, or control messages to disrupt framing while using less average power.

### Observable signs of jamming

- **Sudden widespread CRC failures or increased BER.**
- **Elevation-independent signal loss:** if a satellite with good link margin suddenly becomes unreadable at many elevations, suspect strong localized jamming.
- **Correlated blackouts across services** using the same band.

###  mitigations

- **Diversity:** frequency hopping, multiple ground stations, alternative bands, and spatial diversity reduce single-point jamming effects.
- **Robust modulations & FEC:** choose modulations and coding schemes that can maintain link under higher noise; stronger FEC and interleaving can sustain operation when SNR drops.
- **Directional nulling & monitoring:** deploy RF monitors to detect interferers and use adaptive antenna nulling to suppress localized jammers.

>[!TIP]
>
>implement rapid detection and fallback modes — when jamming is detected, switch to a pre-planned degraded mode (low-rate telemetry, prioritized health channels, or alternate ground stations) to preserve critical situational awareness.

---

## Unauthorized command injection

Unauthorized command injection is one of the most severe threats: if an adversary can send a valid uplink command accepted by the satellite, they may change modes, disable safeguards, or cause harmful maneuvers.

### Why uplinks are high-risk

- **Asymmetric resource constraints:** satellites often have limited compute/power and may use lightweight uplink authentication or none at all.
- **Delayed detection:** commands can take effect quickly while detection and human-in-the-loop response may be slow.

### Attack paths

- **Impersonation:** an attacker crafts uplink frames that match the expected framing and addresses — without authentication, these may be accepted.
- **Replay:** previously captured valid commands replayed to cause repeated actions.
- **Insider-assisted injection:** an operator or compromised ground asset issues unauthorized commands via legitimate channels.

### Mitigations and defense-in-depth

- **Strong uplink authentication:** use cryptographic message authentication and replay protection (nonces or sequence numbers). Prefer authenticated, integrity-protected command channels.
- **Command whitelisting and guarded modes:** enforce that certain high-risk commands require multi-party authorization or are only executable when the satellite is in defined modes.
- **Anomaly-based gating:** commands that would cause out-of-nominal physical actions (ex: large delta-v, detach payload) should trigger additional verification steps before execution.

>[!TIP]
>
>design a failsafe mode that limits command surface for legacy systems: if authentication is not possible immediately, default to a conservative mode that rejects high-risk commands until operator approval is confirmed through authenticated channels.

---

## Telemetry poisoning and data integrity

Telemetry poisoning refers to intentional modification or injection of false telemetry values to hide malicious activity, cause incorrect operator decisions, or trigger unsafe automated responses.

### Common poisoning strategies

- **Value manipulation:** selectively flipping or scaling HK values to make a subsystem appear nominal when it is not.
- **Sequence tampering:** causing sequence numbers to roll over or reordering frames to create confusion in state estimation.
- **Selective suppression:** jamming or filtering critical telemetry channels while leaving innocuous channels intact to conceal an ongoing attack.

### Detection signals and forensic traces

- **Cross-sensor inconsistency:** compare related telemetry (ex: battery voltage vs solar panel current vs temperature); logical contradictions often reveal tampering.
- **Statistical anomalies:** deviations from learned baselines or sudden changes in noise properties of telemetry fields.
- **Meta-data discrepancies:** mismatches in timestamps, capture metadata, or unexpected decoding artifacts.

### Protective measures

- **End-to-end integrity checks:** authenticate telemetry payloads with MACs or signatures so that ground-side systems can detect tampering.
- **Redundancy & cross-validation:** duplicate critical sensors and cross-compare values; use physics-based models to validate telemetry against expected behavior.
- **Tamper-evident logging:** immutably log raw captures, signed hashes, and processing steps to enable later forensic verification.

>[!IMPORTANT]
>
>always preserve raw RF captures and apply integrity checks before trusting decoded telemetry — corrupted or poisoned telemetry should be treated as untrusted until proven otherwise.

---

## Trust assumptions in crowdsourced ground stations

Crowdsourced ground networks (volunteer setups, distributed receivers) provide valuable coverage but introduce trust and integrity challenges.

### Risk profile

- **Variable operator competence and security posture:** volunteers may run unpatched systems, weak credential management, or misconfigured time sources.
- **Potential for malicious contributors:** a rogue ground station could feed forged data, incorrect timestamps, or manipulated demodulations into shared repositories.
- **Data provenance complexity:** establishing a reliable chain-of-custody is harder when data originates from many untrusted sources.

### Recommended trust & validation approaches

- **Reputation & certification:** score contributors by historical reliability, cryptographic attestation of their software versions, or periodic audits.
- **Cross-validation & redundancy:** do not accept single-station observations as authoritative — require corroboration from multiple independent stations before drawing conclusions.
- **Signed submissions:** require stations to sign their submissions with keys provisioned during a vetting process; verify signatures before ingesting data into critical workflows.

---

## Incident detection & response

- **Detection pipelines:** combine simple rule-based checks (CRC explosion, sequence anomalies) with statistical detectors (residuals, outlier detection) and geometry-based validators. //Doppler and AOAs matching ephemeris
- **Triage:** categorize events by confidence and potential impact, isolate affected data streams, and preserve raw captures for forensic analysis.
- **Containment & remediation:** switch to safe modes, re-key compromised links, revoke or quarantine ground assets, and escalate to legal/operational channels as appropriate.

---

## Forensics & evidence preservation

- **Preserve the RF raw evidence** (IQ files) with secure hashing and write-once backups.
- **Record system state and operator actions** around the time of the incident (rotator logs, operator consoles, command uplinks). These contextual records are often essential for attribution.
- **Correlate multi-station data** to triangulate interferers/spoofers and to verify whether an observed anomaly is localized or global.

---

## Governance, policy and secure design principles

- **Least privilege & separation of duties:** operational roles (command authorization, key management, ground-station control) should be separated and audited.
- **Secure supply chain & firmware management:** ensure satellites and ground assets receive authenticated firmware updates; track provenance of hardware and software.
- **Design for graceful degradation:** plans for reduced capability operation under attack preserve mission-critical telemetry and command paths.

---

### Glossary // quick terms

- **Spoofing:** forging signals to deceive receivers.
- **Replay attack:** re-transmission of previously captured valid signals to cause unintended behavior.
- **KISS:** lightweight serial framing (mentioned earlier) often used in volunteer ground station pipelines.
- **MAC:** Message Authentication Code.
- **AOA:** Angle of Arrival.

