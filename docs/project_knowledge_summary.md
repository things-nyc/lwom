# Project Knowledge Summary: LoRaWAN-over-Meshtastic (LWoM)

## Project Goal
Enable **LoRaWAN Class A devices** (using LMIC) to communicate when out of range of a LoRaWAN gateway by tunneling LoRaWAN traffic over **Meshtastic**. Keep LMIC’s high-level API unchanged.

## Architecture Overview
- **LMIC Meshtastic regions** (`MESHTASTIC_US915`, `MESHTASTIC_EU868`, …) define caps and soft RX timing; ADR disabled.
- **Device transport adapter** sends/receives **PHYPayload bytes** via Meshtastic `Data.payload` (private portnum, dedicated channel/key).
- **Dedicated Meshtastic receiver** on USB → **Linux bridge** → **TTS** (UDP first; Basics Station later).

## Wire Format
- Meshtastic `Data.payload` carries LoRaWAN **PHYPayload** (opaque).
- **Max PHYPayload**: **233 B** → **Max FRMPayload**: **220 B** (no FOpts).

## Timing Strategy
- **Soft RX windows**: RX1 delay **5–10 s**; RX2 fallback; configure via MAC (`RXTimingSetupReq`).

## US915 Details
- LoRaWAN payload tables per TTN docs; we do **not** emulate PHY data rates/timing—Meshtastic transport drives constraints.

## Risks & Limits
- ADR unreliable without RF metrics; disable.
- No gateway diversity/geolocation; focus on E2E delivery.

## Testing Plan
- Unit (payload caps, serialization), Integration (OTAA, ACKs via TTS), HIL (real Meshtastic node), latency injection.

## Deliverables
- LMIC regions, transport adapter, Linux bridge (UDP→TTS), CI harness.

## Next Steps
- Implement and test end-to-end with TTN/TTS; consider Basics Station migration.
