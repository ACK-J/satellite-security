# Reference Frames & Time Systems

Accurate frames and timekeeping are the backbone of satellite operations and RF analysis. Mistakes or mismatches in reference frames or time scales produce position errors, Doppler miscalculations, and misaligned event correlations — all of which can make benign anomalies look like security incidents //or conceal real ones

---

## Earth‑Centered Inertial (ECI) vs Earth‑Centered Earth‑Fixed (ECEF)

These two global frames are the most frequent starting points for orbital and ground-station work. They represent two different ways of pinning coordinates to space and Earth.

**ECI (Earth‑Centered Inertial)**

- ECI frames are effectively non-rotating with respect to the distant stars (inertial). They are used for describing the satellite’s motion in space without including the Earth’s rotation.
- Orbits are most naturally expressed in an inertial frame: the laws of motion (Newton/Kepler) are simpler there because the coordinate axes do not spin with the Earth.
- Different ECI flavors exist (ex: J2000), each defining a precise orientation and epoch.

**ECEF (Earth‑Centered Earth‑Fixed)**

- The ECEF frame rotates with the Earth. Coordinates in ECEF are fixed to points on or near the Earth’s surface (longitude/latitude map naturally into ECEF coordinates at a given time).
- Ground-station locations, antenna boresight vectors, and terrain features are usually expressed in ECEF for planning and pointing.

>[!IMPORTANT]
>
>**Why both matter?**
>
> to compute azimuth/elevation for a ground station looking at a satellite, you typically compute the satellite position in an inertial frame (ECI), transform it to ECEF at the same instant (accounting for Earth rotation), and then convert to the local topocentric coordinates of the ground site.

---

## Topocentric horizon coordinates: Azimuth & Elevation

Topocentric coordinates are local coordinates centered at a ground observer. The two primary angles used for pointing an antenna are:

- **Azimuth (AZ):** the compass direction from the observer to the satellite, usually measured from true north toward the east.
- **Elevation (EL):** the angle above the horizon to the satellite. An elevation of 0° is on the horizon; positive values are above the horizon; many ground stations use elevation masks (ex: 5° or 10°) to avoid signals obscured by local terrain or low-elevation multipath.

>[!TIP]
>
>Antenna pointing and rotator control rely on accurate AZ/EL computations derived from the satellite position in an Earth-fixed frame and the ground-site coordinates (latitude, longitude, height). For security analysis, small AZ/EL errors can cause large link-margin losses for high-gain antennas and may be misinterpreted as transmitter power changes.

---

## Transformations between frames 

Transformations are more than simple rotations; precise conversions must account for Earth rotation, polar motion, precession, nutation, and light-time effects if high accuracy is needed.

**Key conceptual steps for a typical workflow:**

1. **Compute satellite position in an inertial frame** (ECI) at a given time from orbital elements or a propagator.
2. **Apply Earth orientation at that time** to rotate the ECI coordinates into the ECEF frame (this rotation depends on Earth rotation angle and smaller corrections).
3. **Transform ECEF coordinates to local topocentric coordinates** using the ground station’s geodetic latitude, longitude, and altitude.
4. **From local topocentric vectors, compute azimuth and elevation.**

**Common pitfalls:**

- Using epoch-mismatched coordinates (position at one time but Earth rotation for another) creates large errors.
- Ignoring polar motion and UT1−UTC differences causes small but sometimes mission-relevant pointing errors.
- Confusing geodetic vs geocentric latitude (the Earth is flattened at the poles, so the two are not identical) when converting ground coordinates.


---

## Time scales: UTC, TAI, GPS Time, and Julian Dates

Timekeeping in space systems is subtle because multiple time standards coexist and differ by seconds (or leaps) depending on historical decisions.

- **UTC (Coordinated Universal Time):** the de facto civil time standard. UTC tracks atomic time but inserts *leap seconds* occasionally to stay close to UT1 (astronomical time based on Earth rotation). Leap seconds make UTC discontinuous at insertion times.

- **TAI (International Atomic Time):** a continuous atomic timescale without leap seconds. TAI runs ahead of UTC by an integer number of seconds (the cumulative leap seconds).

- **GPS Time:** a continuous time scale used by GPS satellites. GPS time was aligned with UTC in 1980 but does not include leap seconds; therefore GPS time is offset from UTC by a fixed number of seconds that increases when new leap seconds are added to UTC.

- **Julian Date / Modified Julian Date:** continuous day-count formats used in astronomy and orbit propagation. They provide nicely ordered continuous time values useful for interpolation and logging.

**Why this matters:** when correlating RF captures across systems or comparing SDR timestamps to predicted ephemerides, you must ensure all timestamps are expressed in the same time scale. A mismatch of even one second (ex: UTC vs GPS without correction) can produce gross azimuth/elevation and Doppler errors.

---

## Leap seconds & time synchronization

Because UTC occasionally inserts leap seconds, systems that rely on continuous time (ex: numerical propagators or time-difference measurements) must either use a continuous scale (TAI, GPS) internally or explicitly account for UTC leap seconds.

**Practical synchronization points:**

- **NTP (Network Time Protocol):** sufficient for many tasks but can be affected by network jitter and may not always account for leap-second announcements cleanly.
- **PTP (Precision Time Protocol) and GPS-disciplined clocks:** provide sub-millisecond to microsecond synchronization when needed for high-precision correlation or timestamping of SDR captures.

>[!IMPORTANT]
>
>if an attacker can manipulate a ground station’s time source (NTP spoofing, GPS time spoofing), they can desynchronize datasets, cause misalignment with predicted passes, or induce erroneous Doppler corrections — potentially facilitating stealthier attacks or hiding traces of malicious actions.

---

## Orekit’s handling of frames & time  

High-quality space libraries (for example Orekit) explicitly separate frames and time scales and provide robust transformations between them. They typically:

- Offer a wide set of standard inertial and Earth-fixed frames (with clear naming conventions).
- Provide Earth orientation parameters and handle precession, nutation, and polar motion when converting between ECI and ECEF.
- Provide time-scale conversion utilities (UTC↔TAI↔TT↔GPS) and manage leap seconds.


---

## checks & best practices

- **Always record the time scale used in your capture metadata** (UTC, GPS, etc.) and, when possible, include the UTC timestamp and GPS time offset.
- **Verify the epoch of orbital elements** and use consistent epochs for propagation and observation correlation.
- **Use GPS-disciplined or PTP-synchronized clocks for multi-station correlation** and for experiments where microsecond accuracy matters (Doppler/time-of-arrival studies).
- **Be wary of leap-second events** when planning long-term automated operations: schedule tests across upcoming leap-second dates if your system must remain robust through those transitions.

---


### Glossary // quick terms

- **ECI:** Earth‑Centered Inertial frame.
- **ECEF:** Earth‑Centered Earth‑Fixed frame.
- **AZ/EL:** Azimuth and Elevation (topocentric pointing angles).
- **UTC / TAI / GPS time:** common time scales used in space systems.
- **UT1:** an astronomical time scale tied to Earth rotation (relevant when extreme pointing precision is required).
