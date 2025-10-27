# Advanced Topics in Space Dynamics




## Attitude and attitude-control models

Attitude is the spacecraft’s instantaneous orientation in 3D space and is central for pointing antennas, orienting sensors, managing solar power, and controlling aerodynamic/photonic torques. Accurate attitude modeling and estimation are required to translate nominal orbital predictions into expected RF/gain observables and to diagnose many security-relevant anomalies.

Representation and trade-offs

- Euler angles (yaw, pitch, roll) are the most intuitive: three sequential rotations that describe orientation relative to a reference. They are excellent for human-readable displays and for describing commanded slews. Their downside is gimbal lock — configurations where two rotation axes align and the representation loses a degree of freedom — and numerical instability for continuous integration.

- Quaternions are four-component unit vectors that represent rotations without singularities. They compose robustly (quaternion multiplication) and are the de-facto standard for onboard attitude representation and for filters used in estimation and control. Quaternions require maintaining unit norm; numerical integration schemes should renormalize periodically.

- Rotation matrices (3×3 orthonormal matrices) are straightforward linear transforms between reference frames. They are redundant and can drift numerically, so orthonormalization is sometimes applied.

Kinematics vs dynamics

- Kinematics relates angular velocity (measured by gyros) to the time rate of change of the attitude representation (quaternion derivative or Euler-angle rates). It determines how to propagate orientation given measured or commanded angular rates.

- Dynamics governs how torques change angular velocity through the spacecraft’s inertia tensor (Euler’s rotational equations). External torques (gravity-gradient, magnetic, aerodynamic, SRP-induced) and internal actuators (reaction wheels, thrusters, control-moment gyros, magnetorquers) drive the dynamics.

Attitude determination and filtering

- Sensors: star trackers provide high-precision absolute attitude fixes; sun sensors and horizon sensors give coarse absolute pointing; magnetometers give orientation relative to Earth’s field; gyroscopes provide high-rate relative motion but suffer drift.

- Estimation: filters (ex: Extended Kalman Filter variants, Unscented KF) fuse gyro and absolute sensor data to estimate attitude and rate. Quaternion-aware filtering avoids singularities and preserves consistency. Filter tuning (noise covariances) directly affects responsiveness and false-alarm rates.

Control strategies

- Simple PD controllers stabilize pointing for many missions. For precision pointing (ex: optical payloads), advanced control (model-predictive, LQR) and active disturbance rejection are common.

Observable signatures and security relevance

- Attitude changes are directly observable in RF: antenna boresight mispoints produce abrupt or gradual RSSI and SNR changes, altered polarization coupling, and possible modulation impairments if antennas move relative to polarization alignment.

- Attack or anomaly scenarios: unauthorized or anomalous attitude commands (malicious or accidental) can hide a downlink by pointing away, reduce power generation, or expose sensitive sensors. Correlate attitude telemetry, wheel-speed telemetry, sun-load payload data, and RF-level observations when investigating unexplained signal changes.

>[!TIP]
>
>- Always preserve raw attitude sensor telemetry with the same chain-of-custody as RF captures; do not rely solely on processed attitude quaternions because post-processing can hide tampering.
>- Implement sanity checks and cross-validation: verify that commanded attitude changes match expected mission plans and that corresponding power/thermal telemetry is consistent with the new attitude.

---

## Perturbation models 

High-fidelity orbit prediction requires modeling a hierarchy of non-central forces. Each perturbation class contributes with characteristic magnitudes and timescales; understanding them helps distinguish natural evolution from potential malicious actions.

Gravity field and geopotential

- Earth’s gravity departs from point-mass behavior; it is modeled by spherical harmonics. The leading zonal term (J2) captures Earth’s oblateness and is the dominant perturbation beyond central gravity for many LEO and MEO missions. J2 causes secular precession of the RAAN and rotation of the argument of perigee.

- Higher-order zonal (J3, J4...), tesseral and sectoral terms encode longitudinal and local mass anomalies. Those terms matter for precision orbit determination, long-term propagation, and resonant-orbit behavior.

Atmospheric drag

- Drag force \(F_D\) ≈ 0.5 × ρ × C_D × A × v² where ρ is atmospheric density, C_D is drag coefficient, A the effective cross-sectional area, and v the relative velocity. In LEO, drag causes semi-major axis decay and increasing mean motion.

- Atmospheric density models are empirical and incorporate solar activity proxies (F10.7 flux), geomagnetic indices (Ap/Kp), local solar heating, and diurnal variations. During geomagnetic storms the thermosphere expands, increasing drag unpredictably — TLEs and simple analytic propagators lose accuracy rapidly under such conditions.

Solar radiation pressure (SRP)

- SRP is a continuous tiny force due to solar photons. For many small satellites SRP is more important than higher-order gravity terms. SRP depends on the spacecraft optical properties (specular/diffuse reflectivity), attitude (orientation of panels), and projected area.

- Simplified 'cannonball' models approximate the craft as a sphere with effective area-to-mass ratio. More accurate models treat individual surface facets (box-wing models) and include shadowing during eclipse passages.

Third-body gravity and tides

- The gravitational pull of the Moon and Sun introduces periodic perturbations. For high-altitude or eccentric orbits, third-body effects dominate certain secular variations.

- Solid Earth tides and ocean tide loading modulate the geopotential; for the highest-precision applications these are included.

Model and uncertainty

- Choose models according to mission needs: low-order geopotential and simple atmospheric models may suffice for casual tracking; precision mission analysis requires high-degree gravity models, realistic SRP with per-surface optical properties, and state-of-the-art atmospheric models.

- Uncertainty sources: model inadequacy (ex: wrong C_D), environmental uncertainty (solar flux forecasts), and unmodeled spacecraft attitude changes. For security analysis, quantifying uncertainty is essential: large residuals outside predicted uncertainty bounds indicate potential anomalies or maneuvers.


>[!IMPORTANT]
>- Maintain ensembles of propagation runs using different plausible environmental parameters (ex: high vs low solar flux) to detect when observed deviations fall outside modeled uncertainty.
>- Use residual signatures (range/Doppler/time) to separate natural perturbations (which often have predictable frequency content) from abrupt maneuver-like signatures.

---

## Spacecraft maneuver modeling //impulsive vs continuous burns

Maneuvers alter orbital energy and geometry. Modeling them correctly is key for orbit determination, collision avoidance, and forensic attribution.

Impulsive approximations

- Useful when thrust is applied briefly relative to orbital period (short pulse firings). An impulsive Δv vector instantaneously changes the velocity state and, hence, orbital elements.

- Impulsive burns conveniently decompose into radial, along-track (tangential), and normal components; each component affects different orbital elements (ex: tangential Δv mostly changes semi-major axis and energy).

Finite-duration / continuous thrusts

- Electric propulsion and low-thrust systems apply continuous low-magnitude thrust over long periods. Modeling requires integrating the thrust acceleration vector over time, accounting for pointing, mass loss (propellant consumption), and changing orbital geometry.

- Continuous burns create ramp-like residuals in tracking data rather than the step-like signatures from impulsive burns.

Maneuver detection and parameter estimation

- Observationally, unmodeled maneuvers produce discontinuities or systematic trends in residuals (range and Doppler) when compared to model predictions. Orbit determination filters that include maneuver epochs or stochastic Δv parameters can estimate the timing and vector magnitude of maneuvers.

- Joint estimation with telemetry (ex: propellant usage, thruster on/off logs) increases confidence. In absence of on-board telemetry, repeated ground observations and filtering techniques (batch least-squares, Kalman-based estimation) can detect and estimate maneuvers.


>[!IMPORTANT]
>
>- Treat unexplained small Δv events as potentially high-risk: they may indicate clandestine reconfiguration, de-orbit attempts, or collision avoidance without operator notification. Prioritize cross-checks with multiple observers and telemetry sources.

---

## Constellations & formations //relative motion, Clohessy–Wiltshire intuition


Relative coordinate systems and intuition

- Use a local-vertical-local-horizontal (LVLH) frame centered on a reference satellite. Relative motion decomposes into radial (toward/away from Earth), along-track (velocity direction), and cross-track (out-of-plane) coordinates.

Clohessy–Wiltshire (CW) linearized equations

- CW provides a linear approximation for relative motion around a circular reference orbit. Solutions reveal oscillatory behavior with natural frequencies tied to mean motion: relative in-plane motion exhibits coupled radial and along-track oscillations; cross-track motion decouples into simple harmonic oscillation.

- CW is highly useful for quick design and analysis (estimating phasing maneuvers, rendezvous windows), but it fails for large separations, eccentric reference orbits, or when perturbations (differential drag, SRP) dominate.

Differential perturbations and formation control

- Even small differences in area-to-mass ratio, attitude, or surface properties cause differential drag and SRP effects that slowly change relative phasing. Formation-keeping requires regular stationkeeping maneuvers or clever use of differential drag via attitude control.

- For precise formations (ex: synthetic aperture interferometry), very accurate relative-state estimation (meters to sub-meter) and tightly coordinated control is necessary.

Security considerations

- Formation deviations can be signs of unauthorized maneuvers or failures; they can also create collision risk. Cross-validate relative telemetry and inter-satellite ranging (if available) to detect stealthy deviations early.

---

## Covariance analysis & orbit determination

Orbit determination (OD) is the process of estimating a satellite’s orbit given observations (range, Doppler, angles, GNSS fixes). Covariance analysis quantifies uncertainty in estimated states and supports decision-making (collision probability, maneuver planning).

Observation models and measurement types

- Common observables: range (two-way or one-way), Doppler (range-rate), angles (azimuth/elevation or RA/Dec), and GNSS-derived position/velocity. Measurement noise characteristics (variance, biases) feed estimation filters.

Filtering & batch estimation

- **Batch least-squares** fits a state to a batch of observations, minimizing residuals. It is robust for offline processing and provides covariance estimates from the inverse normal matrix.

- **Sequential/Kalman filtering** (Extended or Unscented variants for nonlinear dynamics) processes observations in time order and updates state and covariance estimates recursively. Filters can include process noise models to account for unmodeled accelerations (ex: maneuvers treated as process noise or explicit estimated parameters).

Covariance propagation and interpretation

- The state covariance matrix evolves under linearized dynamics and process noise; propagated covariance gives expected position/velocity uncertainty at future epochs. Comparing observed residuals to predicted covariances indicates model adequacy and helps flag anomalies.

- Covariance-based conjunction assessment: use propagated covariances of two objects to compute collision probability (assuming Gaussian uncertainties) 


>[!TIP]
>
>- Keep measurement-origin provenance with OD inputs. Bad or maliciously forged observations can dramatically skew OD results. Cross-validate with independent data sources before taking high-impact actions //ex: collision avoidance burns

---

## Ephemerides & SPICE kernels

Ephemerides provide accurate position/velocity data for solar system bodies and spacecraft; SPICE kernels (NAIF/SPICE) are a rich format used extensively in planetary and mission analysis to store spacecraft trajectory, instrument geometry, and frame definitions.

Kernel types and uses

- SPK: space-object ephemerides (position/velocity over time segments).
- CK: attitude/instrument pointing kernels for spacecraft.
- PCK: planetary constants (rotation models, reference ellipsoid parameters).
- FK, IK: frame and instrument kernels describing reference frames and instrument geometry.

SPICE and high-fidelity analysis

- SPICE provides a consistent framework to compute geometric quantities (look angles, illumination, solar incidence) with high fidelity and well-documented frame/time handling. For many high-precision missions, SPICE kernels (or other precise ephemeris formats) are preferred over TLEs for mission-critical computations.

>[!TIP]
>
>- Secure your ephemeris sources and metadata: corrupted or forged ephemeris data can lead to mispointing, mis-scheduling, or incorrect anomaly attribution. Store kernel provenance and signatures where possible.

---

## Mission analysis and Orekit-style capabilities //eclipse prediction, ground visibility windows

Mission analysis synthesizes dynamics, geometry, and constraints to support operations: scheduling ground contacts, planning maneuvers, and predicting eclipses that affect power and thermal conditions.

Eclipse prediction and illumination

- Eclipses occur when the Sun is occulted by Earth (or another body). Predicting eclipse entry and exit times requires precise ephemerides, occultation geometry, and attitude (for instrument thermal/illumination modeling).

- Eclipse windows matter for power planning (battery depth-of-discharge), thermal control, and payload scheduling. For security incidents, unexpected eclipses can explain telemetry gaps and must not be mistaken for interference.

Ground visibility and access windows

- Ground visibility computations convert satellite ephemerides into AZ/EL tracks for a given station, apply elevation masks and local constraints, and return AOS/LOS and maximum-elevation times. Incorporate topography and local obstructions for realistic visibility.

Integrated mission analysis

- Modern libraries provide features to combine force models, event detectors, attitude dependence, and visibility analysis. Orekit-style systems support event detection (sunrise/sunset, eclipses, rise/set), maneuver modeling, and batch propagation with selectable force models.

>[!TIP]
>
>- Use mission-analysis event logs to cross-check reported outages: if an RF blackout coincides with an eclipse and the attitude/thermal data match, the cause is likely natural. If not, escalate investigation.

---

### Glossary // quick terms

- Attitude: spacecraft orientation.
- Quaternion: 4-parameter rotation representation.
- J2: Earth's primary oblateness gravity coefficient.
- SRP: Solar Radiation Pressure.
- CW: Clohessy–Wiltshire equations for relative motion.
- OD: Orbit Determination.
- SPICE: NAIF kernel framework for ephemerides and geometry.
