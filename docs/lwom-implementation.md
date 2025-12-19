# LoRaWAN‑over‑Meshtastic (LWoM) — Implementation Details

This document captures concrete interfaces, formats, and test plans for building the LWoM stack.

---

## 1) Wire Format: LWoM Frame (inside Meshtastic Data)

A private protobuf carried on a dedicated Meshtastic channel (own AES key), transporting the LoRaWAN **PHYPayload** and minimal radio meta needed for pseudo‑gateway emulation.

```text
LwomFrame
  dir           : UPLINK | DOWNLINK
  phy_payload   : bytes      // opaque LoRaWAN PHYPayload
  dev_eui       : uint64     // optional correlation
  fcnt          : uint32     // optional correlation
  tmst_hint     : uint32     // end-of-TX device-local tick (optional)
  freq_mhz      : float      // e.g., 904.1
  datr          : string     // "SF7BW125", "SF8BW500", etc.
  codr          : string     // "4/5"
  txpwr_dbm     : int32
  rssi_dbm      : int32      // optional synthetic metrics
  lsnr_db       : float      // optional synthetic metrics
```

**Notes**
- Keep payload sizes within Meshtastic `Data` limits (~233 B app bytes).
- PHYPayload stays E2E encrypted by LMIC; the bridge never decrypts it.

---

## 2) Device-side: LMIC Radio Shim

**TX path**
- LMIC calls `radio_tx(freq, dr, pwr, payload, toa)`.
- Shim wraps `{phy_payload, freq_mhz, datr, codr, txpwr_dbm, tmst_hint}` into `LwomFrame(dir=UPLINK)` and sends via Meshtastic `Data` on `PORTNUM_LWOM` with `want_ack=true`.

**RX path**
- Shim sets timers for **RX1/RX2** based on LMIC’s timing.
- On downlink arrival from Meshtastic, the shim injects bytes to LMIC with synthetic `rssi/lsnr` if needed.

**Power**
- Device Meshtastic radio: **non‑relay**, minimal hop limit, wake to TX, sleep otherwise.

---

## 3) Linux Bridge: Semtech UDP Backend

**Uplink translation (Meshtastic → LNS)**
- Convert `LwomFrame` to an `rxpk` JSON object:
  - `time`: UTC timestamp (RFC3339).
  - `tmst`: monotonic tick aligned to bridge clock.
  - `freq`: from frame; `datr`: DR string; `codr`: coding rate.
  - `rssi/lsnr`: synthetic (configurable).
  - `data`: base64 of `phy_payload`.

**Downlink scheduling (LNS → Meshtastic)**
- Listen for `PULL_RESP` → `txpk` with target `tmst`.
- Compute deadline for **RX1/RX2** based on uplink `tmst` and configured delays.
- Send `LwomFrame(dir=DOWNLINK)` carrying downlink PHYPayload in time for the device’s window.

**Networking loops**
- Maintain `PUSH_DATA` (uplink) and `PULL_DATA` (downlink polling) sockets to LNS.

---

## 4) Optional Backend: LoRa Basics Station

- Add an alternative path using **LNS** (data) and **CUPS** (config/updates).
- Benefits: TLS, centralized channel plans, better fleet management.

---

## 5) US915 DR/Channel Mapping Tables

Keep consistent tables in **LMIC shim** and **bridge**:

```text
Uplink DRs:
  DR0: SF10 @ 125 kHz
  DR1: SF9  @ 125 kHz
  DR2: SF8  @ 125 kHz
  DR3: SF7  @ 125 kHz
  DR4: SF8  @ 500 kHz

Downlink DRs:
  DR8:  SF12 @ 500 kHz
  DR9:  SF11 @ 500 kHz
  DR10: SF10 @ 500 kHz
  DR11: SF9  @ 500 kHz
  DR12: SF8  @ 500 kHz
  DR13: SF7  @ 500 kHz

Downlink frequencies:
  923.3–927.5 MHz (8 channels, 600 kHz step)
Default RX2:
  923.3 MHz @ DR8 (SF12@500 kHz)
```

---

## 6) Timing & Latency

**Default LoRaWAN Class A**
- RX1: +1 s; RX2: +2 s (after uplink end).
- OTAA Join‑Accept defaults: 5 s / 6 s.

**Plan**
- Configure **RX1 delay = 5 s** via MAC (RXTimingSetupReq or Join‑Accept parameters).
- Treat **RX2** as fallback.
- Log and alert on deadline misses; auto‑adjust RX delays if supported.

---

## 7) ADR Policy

- **Disable ADR initially** due to synthetic RF metrics.
- If enabling later:
  - Provide bounded synthetic `rssi/lsnr`.
  - Cap DR changes and require stable history windows.
  - Prefer manual DR pinning for critical devices.

---

## 8) Observability

- Bridge metrics:
  - `rxpk_sent_total`, `txpk_received_total`, `downlink_deadline_met_total`, `downlink_deadline_miss_total`.
  - Average path latency (Meshtastic→bridge→LNS and back).
- Logs per uplink/downlink:
  - DR/freq, size, tmst, selected RX window, result (met/miss).

---

## 9) Testing & TDD

### Unit Tests
- **LMIC shim**: DR/freq mapping golden vectors; `LwomFrame` serialization (nanopb).
- **Bridge**: `rxpk` JSON generation correctness; RX deadline math.

### Integration Tests
- Local **ChirpStack** or **The Things Stack** with UDP gateway registration.
- **OTAA join**; uplink→downlink echo.
- Latency injection (200–1500 ms) → verify with **RX1 delay=5 s**.

### Hardware‑in‑the‑Loop
- Real Meshtastic node over USB; LMIC device.
- Validate confirmed uplinks (ACK), MAC commands delivery, RX timing.

---

## 10) Security

- LoRaWAN **PHYPayload** is opaque (end‑to‑end encrypted).
- Use **dedicated Meshtastic channel + private port**; disable relaying.

---

## 11) Deliverables Checklist

- `LwomFrame.proto` + nanopb generation in both device and bridge repos.
- LMIC radio shim (TX/RX windows; Meshtastic send/receive).
- Linux UDP bridge (uplink/downlink paths; scheduler; metrics).
- Dockerized LNS for local tests (ChirpStack/TTS).
- CI: unit + integration tests (with latency knobs).

---

## 12) Roadmap

1. MVP: Semtech UDP bridge + LMIC shim + OTAA success.
2. Reliability: RX1 delay configuration, deadline accounting, metrics.
3. Scale: Basics Station backend (TLS, CUPS), centralized channel plans.
4. Optional: guarded ADR; operational dashboards; HIL regression suite.
