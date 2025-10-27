# Digital Communication Protocols

## Why protocols matter?

Protocols define how bits on the wire (or in the air) are grouped, identified, routed, and validated. For satellite security, understanding protocols helps you assess authenticity, integrity, and resilience of telemetry, telecommands, and payload data. Protocol weaknesses — like predictable headers, weak CRCs, or unprotected control fields — can be exploited or can cause accidental misinterpretation.

---

## Framing & packet structure

A **frame** or **packet** is a structured block of bits/bytes that packages payload data with control information so that the receiver can reliably detect, extract, and process the payload. Typical components include:

- **Preamble / sync word**: a known pattern sent before a packet to allow the receiver to detect the start of a frame and perform bit/byte synchronization.
- **Header**: contains addressing, packet length, type fields, sequence numbers, quality-of-service / priority flags, and sometimes routing information.
- **Payload**: the actual application data — telemetry, command, or user data.
- **Trailer / FCS (Frame Check Sequence)**: error-detection field (commonly a CRC) appended to the packet to detect transmission errors.
- **Optional redundancy**: FEC parity or error-correction fields may be included either as separate trailer fields or applied across the payload.

### Common framing issues and protections

- **Bit/byte stuffing & escape sequences**: ensures that special marker patterns (like frame delimiters) do not appear in the payload by escaping or inserting extra bits.
- **Length vs sentinel termination**: some protocols use an explicit length field; others use sentinel markers. Length fields are concise but require accurate parsing, while sentinel markers must be carefully escaped.
- **Sequence numbers & ACKs**: used for in-order delivery and retransmission strategies in connection-oriented transports.

>[!TIP]
>
>Understanding the exact byte layout and endianness is essential when decoding captured data — a single swapped field can render payloads unreadable.

---

## Error detection & correction

Transmission errors are inevitable over radio links (noise, fading, interference). Protocols use detectors to **detect** errors (so corrupted frames are discarded) and correctors to **repair** errors without retransmission when possible.

### CRC (Cyclic Redundancy Check)

- CRCs are widely used error-detection codes. A CRC appends a short checksum computed from the packet bits using polynomial arithmetic; at the receiver the CRC is recomputed and compared to detect altered data.
- Common CRC widths include CRC-8, CRC-16, CRC-32 — the wider the CRC, the lower the chance that random errors will go undetected.

**Role:** CRCs are used in most space and groundlink protocols as a first line of defense against bit errors. They are not error-correcting — only detecting — and must be combined with higher-layer strategies (retransmission, FEC) if reliability is required.

### Forward Error Correction (FEC)

FEC adds redundant symbols to a message so that the receiver can correct certain patterns of bit errors without retransmitting. FEC is essential for long-delay or one-way links (ex: deep-space, many satellite downlinks) where retransmission is costly or impossible.

**Families of FEC used in satellite and broadcast systems:**

- **Block codes (ex: Reed–Solomon):** operate on symbols (bytes or multi-bit symbols). Reed–Solomon codes are especially effective against burst errors and are used in many satellite broadcasting and storage systems.

- **Convolutional codes and Turbo codes:** convolutional codes were common for earlier satellite and deep-space links; turbo codes (iterative decoders) provided large gains and were used in some space standards.

- **LDPC (Low-Density Parity-Check) codes:** modern high-throughput standards (ex: DVB-S2) use LDPC combined with BCH or outer codes to approach channel capacity. LDPC codes are computationally heavier but provide excellent performance at low SNR.

>[!IMPORTANT]
>
>**Interleaving:** Interleavers are frequently used with FEC to convert burst errors into pseudorandom single-bit errors that the FEC can better correct.

---

## AX.25 (Amateur packet radio protocol)

- **What it is:** AX.25 is a data link layer protocol historically used in amateur packet radio and by many older satellite/TNC setups. It derives from HDLC and provides framing, addresses (call signs + SSID), control fields, and a Frame Check Sequence.
- **Common use cases:** APRS (Automatic Packet Reporting System), telemetry beacons, and amateur digipeater networks. Several cubesats and amateur satellites still use AX.25 for basic telemetry and command links due to its simplicity and wide support.
- **Key characteristics:** HDLC-style frames, NRZI/NRZ physical-level encodings in many implementations, support for UI (unconnected) frames for broadcast-style messaging.

>[!WARNING]
>
> AX.25 was not designed with modern security in mind — there is no built-in authentication or encryption at the link layer; payload protection must be handled at higher layers if confidentiality or integrity beyond CRC is required.

---

## CubeSat Space Protocol (CSP)

- **CSP** is a lightweight network-layer protocol suite designed for small satellites and embedded networks. It exposes a simple socket-style API and supports connection-oriented and connectionless messaging, routing, and simple ports for services.
- **Why CSP for cubesats:** small code footprint, built for resource-constrained on-board computers, supports features useful for onboard bus architectures (routing, prioritization, optional CRCs, and fragmentation).
- **Typical stack placement:** CSP commonly runs on top of a reliable link layer (ex: a serial link to a radio or an AX.25-like link) and provides addressing and multiplexing between onboard services and groundlink applications.

>[!TIP]
>
>CSP provides convenient routing and port features but does not enforce encryption or authentication by default — mission designers must add security at appropriate layers if needed.

---

## CCSDS Standards — Telemetry, Telecommand, Space Packet Protocol

- **CCSDS (Consultative Committee for Space Data Systems)** produces widely adopted standards for space data handling. The **Space Packet Protocol** (CCSDS 133.x) defines a common packet format used to carry telemetry (TM) and telecommands (TC) in many professional missions.
- **Space Packet structure:** primary header (with packet ID and sequence control), secondary header (optional, contains time tags or application data descriptors), packet data field (payload), and optional error-control fields.
- **Why CCSDS matters:** CCSDS provides interoperability and well-documented packet semantics for large and professional missions. Even small-sat projects often use a subset of CCSDS for compatibility with ground segment tools.

>[!IMPORTANT]
>
>CCSDS standards define how data is packaged and time-tagged; higher-level security (encryption, access control) is handled by mission-specific suites (ex: using PUS services and secure telecommand procedures).

---

## Encoding schemes — NRZ, Manchester, differential encoding

Physical or line encoding maps digital bits into electrical or modulated waveforms. Line encoding affects clock recovery, DC balance, and spectral content — all important in RF link budgets and receiver design.

- **NRZ (Non-Return-to-Zero):** simple level encoding where a logical 1 or 0 maps to a constant voltage (or phase) during the bit interval. NRZ is efficient but can produce long runs of identical bits that make clock recovery harder.

- **NRZI (Non-Return-to-Zero Inverted):** encodes data by toggling the signal on a `1` (or `0`, depending on convention). This helps reduce baseline wander and can ease certain receiver designs.

- **Manchester encoding (phase encoding):** each bit interval has a mid-bit transition; ex: low-to-high = `1`, high-to-low = `0` (or vice versa). Manchester guarantees transitions for clock recovery and is self-clocking, at the expense of doubling the signal bandwidth compared to NRZ.

- **Differential Manchester:** combines differential and mid-bit transition properties so that information is encoded in the presence/absence of transitions rather than the absolute voltage polarity — helpful when polarity inversion may occur.

**Choosing an encoding:** tradeoffs are bandwidth, DC balance, ease of clock recovery, and compatibility with modulation schemes. Satellite physical layers often use baseband NRZ/NRZI feeding modems (FSK/PSK/QPSK) or use symbol-level mappings (ex: QPSK/8PSK) where line encoding is implicit in symbol timing.

---

## Putting it together for satellite links

- **Telemetry downlink:** often uses a well-defined packet format (CCSDS or simpler) with CRC and FEC. Packets may be time-tagged and multiplexed; payloads carry housekeeping and science data.
- **Telecommand uplink:** commands must be authenticated and integrity-protected; classical link-layer CRCs detect transmission errors, but mission-critical commands often require higher assurance (sequence numbers, replay protection, authenticated command verification).
- **Beacon & beacons-as-broadcast:** simple unconnected frames (AX.25 UI-frames or plain CCSDS packets) broadcast basic telemetry or status — these are easy to decode but typically unauthenticated.

---


### Glossary //quick terms

- **FEC:** Forward Error Correction.
- **CRC / FCS:** Cyclic Redundancy Check / Frame Check Sequence.
- **APID:** Application Process ID (CCSDS packet field).
- **UI-frame:** Unnumbered Information frame (AX.25 broadcast-style frame).
- **Interleaver:** rearranges symbols to convert burst errors into random errors for FEC.





