<!-- markdownlint-disable MD013 -->
<!-- MD013: allow long lines; URLs and code examples exceed 80 chars -->

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project Overview

LoRaWAN-over-Meshtastic (LWoM) tunnels LoRaWAN Class A/C traffic over Meshtastic transport, allowing devices out of gateway range to reach a LoRaWAN Network Server (LNS) without modifying the higher-level device implementation. The first implementation targets [Arduino LMIC](https://github.com/mcci-catena/arduino-lmic).

## Architecture

```text
Device (LoRaWAN MAC, e.g., LMIC)
    ↓ Meshtastic transport adapter (PHYPayload bytes)
Meshtastic node (dedicated, no relay)
    ↓ USB/Serial
Linux bridge
    ↓ Semtech UDP / Basics Station
The Things Stack (TTN/TTS) or ChirpStack
```

## Key Design Constraints

- **No PHY emulation**: Does not emulate LoRa PHY at LMIC `radio()` layer; preserves LoRaWAN semantics (join, keys, MIC, encryption, Class A/C ordering)
- **Payload cap**: Meshtastic `Data.payload` ≤ 233 B → LoRaWAN FRMPayload ≤ 220 B (no FOpts)
- **Soft RX windows**: RX1 delay 5-10 s with RX2 fallback (not strict 1 s/2 s)
- **ADR disabled**: No RF metrics available through Meshtastic
- **Region naming**: LMIC convention - `CFG_Meshtastic_us915`, `CFG_Meshtastic_eu868`

## Transport Adapter API (planned)

```c
bool meshtastic_transport_send(const uint8_t* phy, size_t phy_len);
void meshtastic_transport_on_downlink(lmic_downlink_cb_t cb);
```

## LNS Preference

- **The Things Stack (TTN/TTS)** preferred
- **Semtech UDP** protocol first, **LoRa Basics Station** later
- **ChirpStack** acceptable for local testing

## Meshtastic Configuration

- Use private portnum and dedicated channel key
- End devices do not act as relays; keep hop limit minimal

## Documentation Conventions

- GitHub-ready Markdown with Mermaid diagrams
- Use quoted labels in Mermaid nodes, avoid HTML entities
- Keep region details (caps, timing) only in `docs/lwom-implementation.md`, not README

### Markdown Linting

All markdown files must pass `markdownlint`:

- Run `npx markdownlint <file.md>` before considering any markdown file complete
- No trailing whitespace in `.md` files
- Ask for user approval before adding markdownlint disable annotations
- When using markdownlint-disable, add separate comment lines explaining each rule:

```markdown
<!-- markdownlint-disable MD013 MD041 -->
<!-- MD013: allow long lines; URLs exceed 80 chars -->
<!-- MD041: allow non-heading first line; resume reminder needed -->
```

### Linter Suppression Annotations

When suppressing any linter warning, always include an explanatory comment:

- **Format**: `# noqa: CODE -- what is allowed; why it's needed`
- **Purpose**: Future maintainers should understand both what and why

## File Headers

All source files must have a standard header block:

### C/Arduino Files

```c
/*******************************************************************************
 *
 * Name: filename.cpp
 *
 * Function:
 *      Brief description of the module's purpose
 *
 * Copyright notice and license:
 *      See LICENSE
 *
 * Author:
 *      Terry Moore
 *
 ******************************************************************************/
```

### Python Files (for Linux bridge)

```python
##############################################################################
#
# Name: filename.py
#
# Function:
#       Brief description of the module's purpose
#
# Copyright notice and license:
#       See LICENSE
#
# Author:
#       Terry Moore
#
##############################################################################
```

## README Meta Section

The README.md should end with a `## Meta` section containing:

- **Contributors**: List of contributors
- **Support**: Link to [thethings.nyc](https://thethings.nyc) with message: "If you find this helpful, please support The Things Network New York by joining, participating, or donating."

## Git Hooks

After cloning, configure git hooks:

```bash
git config core.hooksPath .githooks
```
