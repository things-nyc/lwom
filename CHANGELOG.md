# Changelog

## 2025-12-18 — Meshtastic Region Model
- Switch from PHY emulation at LMIC `radio()` to **concept-level mapping**.
- Add **payload cap**: FRMPayload ≤ **220 B** (no FOpts) so PHYPayload ≤ **233 B**.
- Introduce **soft RX windows** (RX1 delay 5–10 s, grace 3–5 s); ADR disabled.
- Prefer **TTS** (TTN) with UDP initially; plan **Basics Station** migration.
- Add documentation: `docs/lwom-overview.md`, `docs/lwom-implementation.md`, `docs/conversation_summary.md`, `docs/project_knowledge_summary.md`.
