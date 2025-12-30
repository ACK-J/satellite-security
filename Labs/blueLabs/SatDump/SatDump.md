![image](https://github.com/user-attachments/assets/068fae26-6e8f-402f-ad69-63a4e6a1f59e)

# Blue Lab 2 — SatDump Telemetry Replay: Decode, Compare, Detect

**Goal:** Use **SatDump** to decode two IQ recordings (clean vs replay) and **prove replay**

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

## Setup — Install SatDump + Dependencies

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
```bash
satdump-ui
```

---

## Setup

```bash
cd ~/Desktop/SatDumpLab
mkdir -p output_clean output_replay results
```

Register pipelines:

```bash
mkdir -p ~/.config/satdump ~/.satdump
cp -r pipelines ~/.config/satdump/ 2>/dev/null || true
cp -r pipelines ~/.satdump/ 2>/dev/null || true
```

---

# Part A — Decode CLEAN IQ

1. Launch SatDump
2. Input → IQ File → `pass_clean.iq`
3. Sample rate: `48000`
4. Center frequency: `437.5e6`
5. Pipeline: `ODYSSEY-1 → BFSK Telemetry (Lab)`
6. Output: `output_clean`
7. Start

Verify:

```bash
ls -lah output_clean
```

---

# Part B — Decode REPLAYED IQ

Repeat Part A using:

- IQ file: `pass_replay.iq`
- Output: `output_replay`

Verify:

```bash
ls -lah output_replay
```

---

# Part C — Replay Detection

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

# Part D — Evidence Collection

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
