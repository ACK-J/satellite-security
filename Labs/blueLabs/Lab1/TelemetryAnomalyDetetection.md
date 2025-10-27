# Blue Team Lab 1 — Telemetry Anomaly Detection
**Goal:** Detect suspicious events in CubeSat telemetry without touching RF. Produce a brief incident report and a JSON of findings

**What you get**
- `telemetry.csv` — 4 hours of beacons at 5s cadence with a few surprises
- `report_template.md` — fill this in
- `report.json` — machine-readable findings for grading
- `grade.py` — offline grader (compares your JSON to the hidden truth)
- `anomaly_baseline.py` — helper functions and thresholds just in case you need it
- `telemetry_labels.json` — used to grade your response, try not to use it unless you really need to see the answers

Download the zip for this main folder from [Here](./Lab1_Telemetry_Anomaly_Detection.zip)

- Click the Download button

<img width="330" height="177" alt="image" src="https://github.com/user-attachments/assets/df15f9ee-985a-4f6a-af65-32698e1aa337" />


- Extract it


> Everything is offline and synthetic. No transmissions, no live satellites

---

## Tasks

1) **Load & sanity-check**
- Parse CSV
- Verify time monotonicity; note any jumps backwards
- Check for missing beacons (gaps > 30s)

2) **Sequence & replay checks**
- `seq` should increase by +1. Flag duplicates or resets that weren’t preceded by a reboot event

3) **Range & rate checks**
- Voltage in [6.8 V, 9.0 V]
- Battery temperature change >= 8°C / min jump in 5 seconds
- CPU load ∈ [0, 100]
- Flag out-of-bounds or impossible rate-of-change

4) **Context checks**
- If `sunlight == False`, the satellite should not be in `CHARGE` with positive `battery_i`
- Flag contradictions between `op_mode`, `sunlight`, and charge current

5) **Stuck sensor**
- Detect any signal flatlines > 10 minutes, e.g., `radio_rssi_dbm` constant

6) **Integrity signals**
- Burst of `packet_crc_ok == False` above 20% within a 2-minute sliding window → flag as link integrity issue

7) **Deliverables**
- `report.json` — list of anomalies with start/end timestamps and type
- `report.md` — 1–2 pages explaining what you found and why it matters
- Optional: one PNG plot (voltage or Rssi over time) to illustrate a finding

---

## Suggested JSON schema

```json
{
  "student": "NAME",
  "findings": [
    {"type": "time_skew", "start": "2025-01-06T12:..Z", "end": "2025-01-06T12:..Z"},
    {"type": "replay_seq", "start": "...", "end": "..."},
    {"type": "oob_voltage", "start": "...", "end": "..."}
  ]
}
```

---

## How to run
- If you need help or want to see how to parse them also check the script's code
```bash
python3 anomaly_baseline.py telemetry.csv --plot out.png
```

- To check your responses

```bash
python3 grade.py report.json
```

Tips:
- Keep thresholds simple and explainable.
- It’s fine if your windows don’t align exactly; the grader allows slack.

