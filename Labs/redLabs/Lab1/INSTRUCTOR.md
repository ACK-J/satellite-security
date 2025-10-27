# Instructor Guide — Lab 1 The Intercept (Solutions & Rubric)

## Quick Answers
- Modulation: 2‑FSK (baseband BFSK), ~1200 bps, captures at 48000 S/s complex float32
- Sync word: 0x1ACFFC1D
- Header: VER:1, SEQ:2 (BE), TYPE:1, LEN:2 (BE), PAYLOAD:JSON(UTF‑8), CRC16‑CCITT over [VER..PAY]
- Auth rule: `sha1(str(epoch)+sat+"-BLUE")[:8]`
- Sample epochs in telemetry: 1713371337 and 1713371430 (1713371337+93)

## Expected Flags
- FLAG1: in `pass_01` telemetry status field
- FLAG2: in `pass_02` ACK string
- FLAG3: returned by `tools/sat_gateway.py` upon valid TYPE=3 command

## Reference Uplink Craft (Python)
```python
import json, struct, binascii, hashlib
SYNCWORD=0x1ACFFC1D
def crc16_ccitt(b, init=0xFFFF):
    import binascii; return binascii.crc_hqx(b, init)
def pkt(ver, seq, ptype, payload:dict):
    pay=json.dumps(payload,separators=(',',':')).encode()
    hdr=struct.pack('>IBHBH', SYNCWORD, ver, seq, ptype, len(pay))
    crc_in=struct.pack('>BHBH', ver, seq, ptype, len(pay))+pay
    crc=crc16_ccitt(crc_in)
    return hdr+pay+struct.pack('>H',crc)
epoch=1713371337; sat="ODYSSEY-1"
auth=hashlib.sha1(f"{epoch}{sat}-BLUE".encode()).hexdigest()[:8]
uplink=pkt(1, 4242, 0x03, {"cmd":"SET_MODE","mode":"CAL","epoch":epoch,"sat":sat,"auth":auth})
open('uplink.bin','wb').write(uplink)
```
Run `python3 tools/sat_gateway.py uplink.bin` to get FLAG3
