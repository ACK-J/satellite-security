# Lab — The Takeover

**Mission:** Using the downlink, reverse the auth scheme and forge a valid uplink to change the satellite mode in our simulator.
Everything is local & synthetic. No RF transmissions.

## Requirements
- GNU Radio Companion (≥ 3.9)
- Python 3.9+ (pip install numpy)
- Docker + Docker Compose
- curl (and optionally jq)

Download the zip for this main folder from [Here](./Lab_Takeover_Kit_Full.zip)

- Click the Download button

<img width="330" height="177" alt="image" src="https://github.com/user-attachments/assets/df15f9ee-985a-4f6a-af65-32698e1aa337" />


- Extract it

## Provided
- ``assets/takeover_pass.iq`` — synthetic downlink (48 kS/s cf32)
- ``groundstation/`` — control app (verifies your forged uplink)
- ``tools/craft_uplink_template.py`` — OPTIONAL helper

Protocol recap (from Lab 1):
- SYNC=0x1ACFFC1D
- Frame: [SYNC(4)][VER(1)][SEQ(2)][TYPE(1)][LEN(2)][PAY(JSON)][CRC16(2)]
- CRC16-CCITT over [VER..PAY]

Auth rule (discover/confirm from pass): auth = sha1(str(epoch) + sat + "-BLUE")[:8]

## Part A — Decode
1. GRC chain: File Source (Complex, 48k, Repeat=Yes) → Throttle(48k) → Quadrature Demod (gain≈3.82) → (LPF) → Clock Recovery (omega=40.0) → Binary Slicer → File Sink (takeover.bits)
2. Parse frames (reuse your Lab 1 script). Grab:
   - epoch & sat from telemetry
   - the example auth from the ACK (to check your math)

## Part B — Forge
Payload JSON (TYPE=3):
{"cmd":"SET_MODE","mode":"CAL","epoch":<from tlm>,"sat":"ODYSSEY-1","auth":"<derived>"}

Assemble full packet with header+CRC. Use your own script or fill:
```bash
python3 tools/craft_uplink_template.py
# creates uplink.bin
```

## Part C — Submit
Start app:
```bash
cd groundstation
docker compose up --build
# open http://localhost:5000
```

Send packet (binary upload):
```bash
curl -F "packet=@uplink.bin" http://localhost:5000/uplink | jq .
```
Or send hex JSON:
```bash
HEX=$(xxd -p uplink.bin | tr -d '\n')
curl -H "Content-Type: application/json" -d "{"hex":"$HEX"}" http://localhost:5000/uplink | jq .
```

Success returns: ok=true, ack for SET_MODE CAL, and flag: FLAG_TAKEOVER{mode_set_CAL}

## Troubleshooting
- bad sync/crc → verify header order and CRC-16 scope (no SYNC in CRC)
- wrong type → ptype must be 0x03
- bad auth → recompute sha1 using telemetry epoch+sat; confirm with example tag in capture
- no response → ensure docker compose shows port 5000 published, visit http://localhost:5000

## Safety
This is a closed simulation for training purposes.
