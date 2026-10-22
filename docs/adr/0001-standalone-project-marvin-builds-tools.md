# ADR 0001: Standalone project; MARVIN builds the tools

**Status:** Accepted (2026-10-21, Gil)

## Context

The idea started as a note in MARVIN: cool old computers with their own waste heat. Research showed the real problem is co-designing the cooling system and the computer.

## Decision

- thermal-forge is its own public repo and is not part of MARVIN.
- It co-designs the heat pump / cooling system and the computer build together.
- MARVIN scopes the work and builds reusable engineering tools (CAD, CFD, thermal analysis, schematics, data capture, real-world experiment design). Those tools live in MARVIN. The designs, models and data from using them live here.

## Consequences

- Research from 2026-10-21 moved here from MARVIN's docs. MARVIN keeps its toolchain survey.
- A tool needed here gets built generically in MARVIN, not as a one-off in this repo.
