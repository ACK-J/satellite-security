# Blue Team Lab 3 — GNSS Deception & Defense

**Scenario:** 
Your ground receiver’s GNSS data looks suspicious between **12:10 and 12:18 UTC**

You need to determine whether this is a spoofing attempt, using only offline telemetry and signal quality metrics

No RF gear, no live sky testing — everything is synthetic

---

## What you’re given
- `gnss_epochs_normal_spoof.csv` — 30 min of GNSS receiver metrics (1 Hz)
- `report_template.md` — write-up form
- `report.json` — machine-readable findings
- `grade.py` — offline grader (compares your JSON to the hidden truth)
- `gnss_detect.py` — starter detector, use it if you are stuck (already flags obvious anomalies)
- `labels.json` — all answers used for grade.py to compare with, check only if you really need to


Download the zip for this main folder from [Here](./Lab3_GNSS_Deception_And_Defense.zip)

- Click the Download button

<img width="330" height="177" alt="image" src="https://github.com/user-attachments/assets/df15f9ee-985a-4f6a-af65-32698e1aa337" />


- Extract it



---

## What you should be looking for and why
### 1. Position jump
**What it is:** A sudden change in reported latitude/longitude > 50 m within 1 second
**Why it matters:** A spoofer often “captures” the receiver by dragging the position toward a fake one. That causes an abrupt jump or drift at onset
**Where to look:** `lat`, `lon` columns

### 2. Clock bias step
**What it is:** The GNSS receiver’s internal clock bias suddenly shifts by ≥ 100 ns
**Why it matters:** Spoofers rarely match the real constellation’s precise timing. When the spoof takes over, the receiver’s clock solution “steps”
**Where to look:** `clock_bias_ns`

### 3. AGC step
**What it is:** A sudden change in Automatic Gain Control (AGC) level ≥ 6 dB
**Why it matters:** If a single strong transmitter (the spoofer) replaces multiple satellites, the receiver adjusts gain drastically
**Where to look:** `agc_db`

### 4. C/N0 uniformity
**What it is:** All satellites show nearly identical signal power (low standard deviation ≤ 2 dB)
**Why it matters:** In real space signals, PRNs have different path losses and elevation angles. Spoofers often rebroadcast all PRNs at one power, producing uniform C/N0
**Where to look:** `cn0_mean_dbhz`, `cn0_std_db`

### 5. Doppler inconsistency
**What it is:** The RMS Doppler residual inflates (≥ 60 Hz for ≥ 60 s)
**Why it matters:** Real satellites move at 4 km/s, each with unique Doppler. A spoofer at a fixed location cannot replicate this precisely, causing inconsistent residuals
**Where to look:** `doppler_rms_hz`

### 6. Return to truth
**What it is:** When spoofing ends, AGC drops, C/N0 standard deviation increases again, and the position solution jitters
**Why it matters:** This “reacquisition jitter” is the tell-tale sign that the receiver has let go of the fake signals and locked back onto real ones
**Where to look:** `agc_db`, `cn0_std_db`, `lat`/`lon`

## Tasks resumed

1. **Sanity checks**
   - Verify timestamps monotonic (no clock issues)
   - Plot `cn0_mean_dbhz`, `cn0_std_db`, `agc_db`, `doppler_rms_hz`

2. **Detect spoofing indicators**
   - **Position jump**: ≥50 m in 1 second
   - **Clock bias step**: ≥100 ns sudden change
   - **AGC step**: ≥6 dB sudden change
   - **C/N0 uniformity**: standard deviation ≤2.0 dB for ≥30 s, mean ≥36 dB-Hz
   - **Doppler inconsistency**: RMS ≥60 Hz for ≥60 s

3. **Optional per‑PRN forensics**
   - If provided a per-PRN dataset, show how PRNs line up with uniform C/N0 and noisy Doppler residuals during the spoofing window

4. **Deliverables**
   - `report.json` — anomalies with type + start/end timestamps
   - `report.md` — 1–2 pages describing evidence, plots, and recommendations

See how a report should be filled [here](./report_template.md)

---

## JSON Schema Example

```json
{
  "student": "YOUR_NAME",
  "findings": [
    {"type":"spoof_onset_jump","start":"2025-01-07T12:10:00Z","end":"2025-01-07T12:10:01Z"},
    {"type":"clock_bias_step","start":"...","end":"..."},
    {"type":"agc_step","start":"...","end":"..."},
    {"type":"cn0_uniformity_spoof","start":"...","end":"..."},
    {"type":"doppler_inconsistency","start":"...","end":"..."}
  ]
}
```

---

## Run Example

```bash
# quick scan
python3 gnss_detect.py gnss_epochs_normal_spoof.csv --plot charts.png > findings.json

# edit report.json with your final spans
python3 grade.py report.json
```

---

**Everything is offline, synthetic, and safe**
