## What
Adds the backend part of US7 (F-19 to F-23), which was blocking the transformer layer of the map. `GET /nodes` now sends `transformer_state` per node:
- `CRITICAL` when two or more SMAC phases are down.
- `WARNING` when exactly one phase is down.
- `INACTIVE` when no phase is down and the SMAC is inactive (F-37, 3 h).
- `NORMAL` otherwise.
- `null` when the node has no transformer.

A phase is down when its last `05xx21y` event (Annex §4.2) is a SET. A CLEAR at the same instant wins, and `05xx20y` (out of range) does not count. Phases are counted against the transformer's own phase count, read from its id (`…-T1-2` / `…-T1-3`, Annex §3.3), so a two-phase transformer with both phases down is red (F-23 / RF-MAP-05). A node with several transformers takes the most severe state. It reuses the `PHASE_LOSS` pattern of the fault list, and `smac_inactive` stays as it was. The frontend already reads `transformer_state`, so the colours appear with no frontend change.

Jira: https://colbits-success.atlassian.net/browse/SCRUM-105 (US7, backend part; unblocks F-19 to F-23 and F-24 SCRUM-168)

## How to test
1. `cd backend && npm test`: 396 unit tests pass. They cover every RF-MAP-04 case, precedence, the two-phase rule, several transformers, unknown channels and the config threshold.
2. Optional, real DB: with the isolated `colbits_f10_test` database (01/02/03 SQL loaded), run `DB_NAME=colbits_f10_test npm run test:e2e -- transformer-state`. 11 tests pass.
3. On the map: load `generator` data with `--scenarios` containing `phase-drop` and `inactivity` (`device: smac`), log in, and turn on only the "Transformers · SMAC" layer. Locally, 1 phase down showed yellow, 2 of 2 and 3 of 3 showed red, a recovered phase showed green, and a silent SMAC showed gray.

## AI use
- [ ] **No AI**
- [ ] **AI-assisted** (suggestions, review or parts of the code)
- [x] **AI-generated** (most of the code)

Tool: Claude Code (details in `docs/bitacora-ia/2026-10-30-SCRUM-105.md`)

## Checklist
- [x] CI passes (lint, tests, build)
- [x] I reviewed everything the AI produced before pushing it

🤖 Generated with [Claude Code](https://claude.com/claude-code)

https://claude.ai/code/session_012Za94ATKRpuhbxLyQmG5TG
