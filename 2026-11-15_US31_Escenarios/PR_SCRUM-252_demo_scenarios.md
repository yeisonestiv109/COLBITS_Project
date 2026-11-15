## What
Backend of the demo scenario panel (US31 · F-100, SCRUM-252): open or close a fuse, cut or restore voltage, drop or restore a transformer phase.

- `GET /scenarios` returns `{ enabled, actions }`. `enabled` is the demo-mode flag the frontend uses to show the panel, and `actions` lists each action with its target type, whether it needs a channel, and its expected effect.
- `POST /scenarios` takes `{ action, target, channel? }`, where `action` is one of `OPEN_FUSE`, `CLOSE_FUSE`, `CUT_VOLTAGE`, `RESTORE_VOLTAGE`, `DROP_PHASE` and `RESTORE_PHASE`. `target` is the fuse or transformer `custom_id`, as listed by `GET /nodes/{name}/detail`. `channel` is A, B or C and is only for phases. The request returns 201 with the written event code, its timestamp and the expected effect.
- Each action writes **what the device would send**: one telemetry reading (`falcon_values` or `smac_vc_values`, copied from the latest one with the state changed) and one `alarms` event with the Technical Annex §4.2 code (`13xx20`, `13xx21`, `05xx21y`), SET or CLEAR. No state or fault logic changes. The map, `GET /faults` and the node detail react through their normal rules on the next refresh.
- **Safety (NF-03):**
  - Off unless `DEMO_SCENARIOS_ENABLED=true` (403 otherwise).
  - Colbits administrators only, with the scope applied.
  - An action that repeats the current state (opening an open fuse) is refused with 409, so the SET/CLEAR pairs stay consistent.
  - A phase the transformer does not have is refused (CL-07).
- Settings: `.env.example` `false`, local `docker-compose.yml` `true`, Render staging (develop) `true`, Render production `false`.
- `scenario-actions.ts` holds the action registry, so SCRUM-253 (Javier: low energy, inactivity, overload, free fall, solar panel, and the demo-environment restriction) adds its actions there.

Jira: https://colbits-success.atlassian.net/browse/SCRUM-252 (US31; unblocks SCRUM-253 and SCRUM-254)

## How to test
1. `cd backend && npm test`: 729 unit tests pass (actions, event codes, the 409 rule, missing phase, demo mode off, admin only).
2. `npm run test:e2e`: the new `scenarios` suite (6 tests) runs over HTTP on PostgreSQL with Colbits' triggers. OPEN_FUSE gives a CRITICAL node and a FUSE_OPENING fault, CLOSE_FUSE gives NORMAL, CUT_VOLTAGE gives WARNING and NO_VOLTAGE, one dropped phase gives WARNING, two give CRITICAL, and the restores bring the node back to NORMAL. A consultation user gets 403. All e2e tests pass (176 + 3).
3. By hand: `docker compose up`, log in as admin, then `POST /scenarios {"action":"OPEN_FUSE","target":"0000100002-A-R"}` in Swagger. The node turns red on the map within one refresh.

## AI use
- [ ] **No AI**
- [ ] **AI-assisted** (suggestions, review or parts of the code)
- [x] **AI-generated** (most of the code)

Tool: Claude Code (details in `docs/bitacora-ia/2026-11-15-SCRUM-252.md`)

## Checklist
- [x] CI passes (lint, tests, build)
- [x] I reviewed everything the AI produced before pushing it

🤖 Generated with [Claude Code](https://claude.com/claude-code)

https://claude.ai/code/session_012Za94ATKRpuhbxLyQmG5TG
