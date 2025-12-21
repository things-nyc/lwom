# Conversation Summary: LoRaWAN-over-Meshtastic Architecture

## Initial Goal
Design an architecture to tunnel **LoRaWAN Class A** traffic over **Meshtastic**, keeping the **LMIC API** unchanged. Use a dedicated Meshtastic node + Linux bridge to reach an LNS.

## Steps Taken
1. Drafted an initial architecture (PHY emulation at LMIC `radio()` layer).
2. Produced GitHub-ready docs with Mermaid diagrams.
3. Split into overview and implementation files; generated downloadable .md files.
4. Pivoted after reflection to a **Meshtastic-region model** (concept-level mapping, no PHY emulation).
5. Regenerated docs with **payload cap 220 B**, **soft RX windows**, **ADR disabled**, **TTS preferred**.

## Current State
- Overview and Implementation docs completed for the **Meshtastic-region model**.
- Supplemental summaries created to capture decisions and context.

## Next Actions
- Integrate with **The Things Stack (TTN/TTS)** via **UDP** first; plan migration to **Basics Station**.
- Implement LMIC Meshtastic runtime regions and the device transport adapter.
- Build the Linux bridge and run OTAA/ACK tests with soft RX timing.
