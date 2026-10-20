## What
Adds F-37: `GET /nodes` now returns `smac_inactive` per node — `true` when a SMAC on the node has no record or its newest record (across `last_smac_vc/pw/pf_values`) is older than `SMAC_INACTIVITY_HOURS` (3 h by default), `false` otherwise, and `null` when the node has no transformer. Same approach as the Falcon rule from US6; both thresholds now share one config reader. F-36 needs no change: it is already covered by `FUSE_SUMMARY_SQL` (US6).

Jira: https://colbits-success.atlassian.net/browse/SCRUM-80 (US5 · SCRUM-64; F-36 = SCRUM-79 reviewed)

## How to test
1. `cd backend && npm test` → 68 unit tests pass (mapping, threshold from env, fallback to 3 h, Falcon/SMAC independent).
2. Optional, real DB: with the isolated `colbits_f10_test` database (01/02/03 SQL loaded), `DB_NAME=colbits_f10_test npm run test:e2e -- smac-inactivity` → 7 pass (2 h active, 4 h inactive, no record inactive, newest of the three tables wins, mixed transformers, no device, node without transformer = null).

## AI use
- [ ] **No AI**
- [ ] **AI-assisted** (suggestions, review or parts of the code)
- [x] **AI-generated** (most of the code)

Tool: Claude Code (details in `docs/bitacora-ia/2026-10-20-SCRUM-80.md`)

## Checklist
- [x] CI passes (lint, tests, build)
- [x] I reviewed everything the AI produced before pushing it
