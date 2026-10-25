## What
Adds F-29: `GET /nodes/:name/fuse-sets` returns every fuse set of a node with its number of clients (`fuses_sets.client_count`; `null` when Colbits has not recorded it, which is not the same as 0). The single set comes first, then sets by branch letter. It applies the user scope like `GET /nodes`; a node outside the scope gives the same 404 as one that does not exist. It is documented in Swagger. It is small on purpose, so the node detail of F-27 (SCRUM-179) can call it or reuse `NodesService.fuseSetsOf()`.

Jira: https://colbits-success.atlassian.net/browse/SCRUM-180 (US11 · SCRUM-109)

## How to test
1. `cd backend && npm test`: 87 unit tests pass (mapping, null clients, empty node, scope 404, invalid user).
2. Optional, real DB: with the isolated `colbits_f10_test` database (01/02/03 SQL loaded), run `DB_NAME=colbits_f10_test npm run test:e2e -- fuse-sets`. 6 tests pass: `0000100002` gives A = 18 and B = 9, `0000100001` gives 42, a node outside circuit 1 gives 404, no token gives 401. Or open Swagger → `GET /nodes/{name}/fuse-sets`.

## AI use
- [ ] **No AI**
- [ ] **AI-assisted** (suggestions, review or parts of the code)
- [x] **AI-generated** (most of the code)

Tool: Claude Code (details in `docs/bitacora-ia/2026-10-25-SCRUM-180.md`)

## Checklist
- [x] CI passes (lint, tests, build)
- [x] I reviewed everything the AI produced before pushing it

🤖 Generated with [Claude Code](https://claude.com/claude-code)

https://claude.ai/code/session_012Za94ATKRpuhbxLyQmG5TG
