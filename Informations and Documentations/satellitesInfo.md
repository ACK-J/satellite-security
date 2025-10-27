## Why satellites matter
Satellites are no longer exotic toys of space agencies—they underpin navigation, weather forecasting, banking transactions, military reconnaissance, emergency communications, and even agriculture. With thousands in orbit, they form a critical backbone of modern life. Like any complex system, they are vulnerable. Unlike data centers, satellites cannot be patched with a reboot or a truck roll. Once in orbit, they are frozen in hardware and painfully slow to update in software. This permanence makes security not an afterthought, but the central challenge

## How Satellites Work
A satellite is essentially a specialized computer system bolted to a bus and hurled into space, it follows predictable physics and is kept alive by careful engineering

### The Satellite Bus (the chassis)
This is the platform that keeps the payload alive and functional:

- **Power subsystem:** solar panels, batteries, power distribution, manages energy harvesting, storage, and delivery
- **Thermal control:** heaters, radiators, insulation to survive extremes of +120°C in sunlight and -150°C in shadow
- **Attitude determination and control:** star trackers, gyros, reaction wheels, magnetorquers to keep antennas or cameras pointing the right way
- **Command and data handling (C&DH):** the onboard computer and storage. Executes commands, logs telemetry
- **Propulsion (sometimes):** thrusters for orbit correction, collision avoidance, or de-orbit at end-of-life

### The Payload
This is the “mission”: the camera, the sensor, the transponder, the scientific instrument, or the communications relay hardware, the payload defines why the satellite exists

### The Communications System
- **Uplink:** signals from ground → satellite, carrying commands.
- **Downlink:** signals from satellite → ground, carrying telemetry or mission data.
- **Transponders:** shift frequencies, amplify, and relay signals.
- **Antennas:** often high-gain for data, low-gain for housekeeping.

### The Ground Segment
Ground stations provide tracking, telemetry, command, and data reception, they often network together globally so operators can reach their satellite during each orbit


## The Attack Surface
A satellite isn’t just the spacecraft. It’s an ecosystem:

- **Space Segment** – the satellite itself, its onboard computers, radios, sensors, and actuators.
- **Ground Segment** – mission control, ground stations, operators, the uplink/downlink infrastructure.
- **Link Segment** – the RF channels: command uplinks, telemetry downlinks, inter-satellite links.
- **User Segment** – terminals, GPS receivers, satellite phones, IoT devices using the space service.

Each segment exposes unique vulnerabilities

### Space Segment Risks
- **Firmware flaws:** once launched, software bugs are forever exploitable
- **Hardware backdoors:** malicious silicon or supply chain compromise
- **Unsafe protocols:** many smallsats use CubeSat Space Protocol (CSP), often without authentication
- **Resource exhaustion:** limited CPU, RAM, and power make DoS attacks catastrophic

### Ground Segment Risks
- **Weak perimeter:** ground stations often run on commodity IT with internet connections
- **Credential theft:** stealing operator logins grants mission control
- **Unpatched servers:** legacy systems lingering for decades

### Link Segment Risks
- **Signal interception:** unencrypted telemetry reveals mission data
- **Command injection:** forging uplink commands without cryptographic checks
- **Replay attacks:** resending valid past commands to induce state changes
- **Jamming:** overpowering a legitimate signal to deny service

### User Segment Risks
- **Spoofed services:** fake GPS signals causing navigation errors
- **Cloned terminals:** pirate satellite TV or bandwidth theft
- **Supply chain attacks:** compromised receiver firmware

## Penetration Testing Satellites
Penetration testing satellites isn’t like pentesting a web app, you can’t just “scan” a satellite, instead:

- **No direct access:** unless you own the satellite, transmitting is illegal
- **One shot hardware:** crashing the system means permanent loss
- **Distance and delay:** round-trip latency and limited bandwidth
- **Safety and law:** regulators treat unauthorized signals as severe crimes

## Typical Attack Scenarios
1. **Signal Analysis & Eavesdropping** – capturing downlink IQ files, identifying modulation, decoding telemetry.
2. **Replay & Command Injection** – in a lab sim, replaying captured beacons or forging commands.
3. **Protocol Fuzzing** – targeting CSP or other satellite buses with malformed frames.
4. **GNSS Spoofing** – broadcasting fake GPS signals in a shielded chamber or simulated SDR flow.
5. **Jamming & Denial** – modeled virtually, showing how interference disrupts operations.
6. **Ground Station Breach** – treating mission control as a standard IT environment with exploitable services.


## Defense in Depth
- **Encryption everywhere** – uplinks, downlinks, inter-sat links, ground comms
- **Authentication** – every command cryptographically signed
- **Anomaly detection** – watch for unusual Doppler, power levels, or traffic patterns
- **Segmentation** – ground IT separated from mission-critical systems
- **Resilience** – redundant satellites, error correction, safe-mode fallbacks





