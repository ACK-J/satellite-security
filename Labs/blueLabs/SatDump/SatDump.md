![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# Blue Lab 2 - SatDump Telemetry Replay: Decode, Compare, Detect

**Goal:** Use **SatDump** to decode two IQ recordings (clean vs replay) and **prove replay**

Replay in this lab is demonstrated by showing that two passes:

- Both look valid
- Both demodulate cleanly
- Both would be trusted by an operator

But when you compare their derived artifacts:

- The files are not identical (different hashes)

- Yet they show unnatural similarity:
    - Same structure
    - Same spectral fingerprint
    - Same repetition patterns

A real satellite pass changes naturally.
A replayed pass changes just enough to look different, but not enough to be real.

---

## What you have (provided)

Under `~/Desktop/SatDumpLab/`:

```
SatDumpLab/
├── pass_clean.iq
├── pass_replay.iq
├── pipelines/
    └── odyssey_bfsk_lab/
        └── pipeline.json
```

---

## Setup - Install SatDump + Dependencies

### System packages

```bash
sudo apt update
sudo apt install -y git curl unzip p7zip-full   libfftw3-dev libusb-1.0-0-dev   qtbase5-dev qtchooser qt5-qmake qtbase5-dev-tools
```

### Install SatDump

```bash
cd ~/Downloads
```
```bash
git clone https://github.com/SatDump/SatDump.git
```
```bash
cd SatDump
```
```bash
mkdir build && cd build
```
```bash
cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/usr ..
```
```bash
make -j`nproc`
```
```bash
sudo make install
```


---

## Setup

```bash
cd ~/Desktop/SatDumpLab
mkdir -p output_clean output_replay results
```

---

# Part A - Decode CLEAN IQ

- Launch SatDump

```bash
satdump-ui
```

<img width="999" height="635" alt="image" src="https://github.com/user-attachments/assets/fa04cdf8-0004-4adb-9faf-a7bcf8818a52" />


1. Pipepline: `Analog Demodulation`
2. Input File: `pass_clean.iq`
3. Output Directory: `output_clean`
4. Baseband Format: `cs8`
5. Samplerate: `48000`

<img width="999" height="635" alt="image" src="https://github.com/user-attachments/assets/34cdc2da-1d4b-41d4-912a-957f7d2778bd" />

- Start

Verify:

```bash
ls -lah output_clean
```

<img width="877" height="84" alt="image" src="https://github.com/user-attachments/assets/d6fd28c5-56c5-4446-b14c-353ac635fa28" />

---

# Part B - Decode REPLAYED IQ

Repeat **Part A** using:

- IQ file: `pass_replay.iq`
- Output: `output_replay`

Verify:

```bash
ls -lah output_replay
```

<img width="877" height="84" alt="image" src="https://github.com/user-attachments/assets/afc5f05b-0f1a-4945-bb7a-08e55a675be7" />


---

# Part C - Replay Detection

## Hash comparison

```bash
sha256sum output_clean/* | sort > clean.hashes.txt
sha256sum output_replay/* | sort > replay.hashes.txt
diff -u clean.hashes.txt replay.hashes.txt || true
```

## Telemetry diff

```bash
diff -u output_clean/telemetry.json output_replay/telemetry.json | head -n 80 || true
```

## Repeated records (CSV or JSONL)

```bash
tail -n +2 output_replay/*.csv 2>/dev/null | sort | uniq -c | sort -nr | head -n 50
cat output_replay/*.jsonl 2>/dev/null | sort | uniq -c | sort -nr | head -n 50
```

---

# Part D - Evidence Collection

```bash
diff -u output_clean/telemetry.json output_replay/telemetry.json > results/telemetry_diff.txt || true
ls -lah results
```

---

# Cleanup

Destroy the lab VM or remove the workspace:

```bash
rm -rf ~/Desktop/SatDumpLab
```

---

***                                                                 

<b><i>Want to go back? </br>[Previous Lab](/Labs/blueLabs/BlueLab/BlueLab.md)</i></b>

<b><i>Looking for a different lab? </br>[Lab Directory](/navigation.md)</i></b>

***Finished with the Labs?***

Please be sure to destroy the lab environment!
