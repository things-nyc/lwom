
# LWoM Chat Constraints

## Formatting & Output Quality
- Produce **GitHub-ready Markdown** with **Mermaid diagrams** that render on GitHub.
- **No trailing whitespace** at the end of lines in `.md` files.
- When I ask for a “Markdown version” of content you just produced, make the **Markdown text substantially identical** to the prose—no omissions or rephrasing that change meaning.
- Use **quoted labels** in Mermaid nodes, avoid HTML entities within labels.
- When requested, **attach downloadable files** (and ZIP bundles) with the exact content shown; keep file contents consistent with the displayed text.

## Project Purpose & Architecture (must be retained in README)
- **Purpose (retain as-is)**:
  > The goal of this project is to tunnel **LoRaWAN Class A** traffic from devices over **Meshtastic**, allowing devices out of gateway range to reach a LoRaWAN Network Server (**LNS**) **without changing the higher level of the device implementation**. (For initial development, we assume use of using the [**Arduino LMIC**](https://github.com/mcci-catena/arduino-lmic).) To bridge traffic back to the LNS, a dedicated Meshtastic node forwards frames to a Linux process that emulates a gateway (**Semtech UDP** first; **LoRa Basics Station** later).
  > We map **LoRaWAN semantics** (registration, OTAA/ABP, end‑to‑end security, and **Class A** ordering) onto **Meshtastic** transport **without emulating the LoRa PHY** at the LMIC `radio()` layer. The LMIC high‑level API remains unchanged. We introduce new runtime regions like **`CFG_Meshtastic_us915`** and **`CFG_Meshtastic_eu868`** whose constraints are defined by Meshtastic.
- **High-level architecture** must be explained and kept:
  - Device (LoRaWAN MAC, e.g., LMIC) → Meshtastic transport adapter (PHYPayload bytes)
  - Meshtastic node (dedicated, no relay) → USB/Serial → Linux bridge
  - Linux bridge → LNS (**TTS** preferred; **ChirpStack** OK for testing)
  - Downlink returns via Meshtastic with **soft RX timing**

## Scope & Generalization
- **LMIC is the first implementation target**, but the architecture is **independent of the LoRaWAN MAC** and should generalize to other stacks once the first implementation is complete.

## Classes Supported
- Do **not** imply that **Class A** is the only class supported; **Class C** semantics can be supported with minimal additional complexity in Meshtastic.

## Meshtastic-Region Model (no PHY emulation)
- Do **not** emulate the LoRa PHY at the LMIC `radio()` layer.
- Preserve LoRaWAN **system semantics** (join, keys, MIC, end-to-end encryption, Class A/C ordering).
- **ADR disabled** in Meshtastic regions (no RF metrics).
- **Soft RX windows**: target **RX1 delay 5–10 s**, with RX2 as fallback (timing slack allowed).
- **Payload caps** driven by Meshtastic:
  - Meshtastic `Data.payload` **≤ 233 B**
  - LoRaWAN **FRMPayload ≤ 220 B** when **no FOpts** (to keep **PHYPayload ≤ 233 B**)
  - If **FOpts** present: `max FRMPayload = 220 − FOptsLen`

## Region Naming (LMIC convention)
- Use LMIC-style names: `CFG_Meshtastic_us915`, `CFG_Meshtastic_eu868`, etc.
- Keep **region details (caps, timing)** **only** in `docs/lwom-implementation.md`, not in the top-level README.

## LNS Preference
- **The Things Stack (TTN/TTS)** is preferred; **Semtech UDP** first for simplicity; plan **LoRa Basics Station** migration later.
- **ChirpStack** is acceptable for local testing.

## Meshtastic Channel & Behavior
- Use a **private portnum** and **dedicated Meshtastic channel key**.
- End devices **do not** act as relays; keep hop limit minimal.
