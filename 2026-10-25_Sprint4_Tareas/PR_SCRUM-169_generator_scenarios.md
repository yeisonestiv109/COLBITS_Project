## What
Adds US9 test scenarios to the generator. `--scenarios demo` (or a JSON file) induces every RF-GEN-04 scenario: single-, two- and three-phase openings, faults on branches A and B of one node (CU-07 2b), loss of voltage, transformer phase drop, overload, low device energy (Falcon and SMAC), inactivity, free fall and solar-panel alarm. It also induces openings separated by configurable gaps inside and outside the grouping window (F-94, `--grouping-window`, default 5 min). Each scenario changes its devices' telemetry and, where the Technical Annex §4.2 has an event code, writes `alarms` rows (`131420`, `131421`, `131401`, `051421A`…). The CLI prints what the platform should show for each scenario. Without `--scenarios`, the data is byte-identical to before.

Jira: https://colbits-success.atlassian.net/browse/SCRUM-169 and https://colbits-success.atlassian.net/browse/SCRUM-170 (US9 · SCRUM-107)

## How to test
1. `cd generator && npm ci && npm test`: 31 unit tests pass (38 with `TEST_DATABASE_URL`, which also checks the `alarms` codes and the node colours with the US6 / F-37 rules on PostgreSQL).
2. Load it and look at the map: `npm run generate -- --reset --scenarios demo --end "$(date -u +%Y-%m-%dT%H:%M:00Z)"`. Against the real backend, `GET /nodes` gave 6 CRITICAL, 1 WARNING, 1 INACTIVE and 1 `smac_inactive`, exactly the nodes the CLI lists. Every other node was NORMAL.

## AI use
- [ ] **No AI**
- [ ] **AI-assisted** (suggestions, review or parts of the code)
- [x] **AI-generated** (most of the code)

Tool: Claude Code (details in `docs/bitacora-ia/2026-10-25-SCRUM-169.md`)

## Checklist
- [x] CI passes (lint, tests, build)
- [x] I reviewed everything the AI produced before pushing it

🤖 Generated with [Claude Code](https://claude.com/claude-code)

https://claude.ai/code/session_012Za94ATKRpuhbxLyQmG5TG
