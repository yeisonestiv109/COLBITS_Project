## What
Backend of US17, so the "Inactive devices" tab and its alarms can use real data: F-38, F-39 and F-40 (SCRUM-232, 233, 234), NF-08 (SCRUM-244) and the low-energy alerts F-44 and F-45 (SCRUM-242, 243).

New endpoint `GET /device/inactive?hours=24`. It lists the devices of the user's scope that were inactive at some moment of the window, each with its **inactivity periods** `{ start, end }`:
- A device is inactive when its next record comes more than its threshold after the previous one (9 h Falcon, 3 h SMAC, configurable), or when it has not reported for longer than that.
- Each silence is the device's **inactivity alarm**. It opens at "previous record + threshold" (F-38) and resolves by itself at the next stored record (F-39); `end` is `null` while it is open. The F-42 example holds: records at 00:00 and 20:00 give one period from 09:00 to 20:00. Every time comes from stored timestamps (F-46). A SMAC record is a record in any of its three tables (TD-07).
- Order (F-40): devices inactive now come first, oldest last record first, and a device that never reported comes first of all. Then come the devices that reported again, most recently resolved first.
- Each device carries `node`, `zone`, `substation` and `circuit`, plus `threshold_hours`, `last_record_at`, `inactive_now`, `energy_percent` and `low_energy`. That is the shape of the frontend model in `inactiveDevices.types.ts` (SCRUM-235).
- NF-08: new settings `FALCON_LOW_ENERGY_PERCENT` and `SMAC_LOW_ENERGY_PERCENT`, 30 % by default (client answer CL-11). They are added to `.env.example`, docker-compose, deploy and `render.yaml`. `low-energy.ts` is ready for the F-44 / F-45 alerts too.

Second endpoint `GET /device/low-energy` (F-44, F-45). It returns one `LOW_ENERGY` alert per device of the scope whose latest stored `supercaps_per` (Falcon) or `battery_per` (SMAC) is below its threshold. A reading at or above the threshold clears it. Each alert carries `energy_percent`, `threshold_percent`, `recorded_at` and `since` (the first reading of the current low streak), lowest level first. It lives in the device module and not in `GET /faults`, because the fault list and its frontend do not handle `LOW_ENERGY` yet; it can be moved there later.

The user scope is applied in SQL, as in the other endpoints (F-10). Nothing is written to the database. Swagger documents the endpoint.

Jira: https://colbits-success.atlassian.net/browse/SCRUM-115 (US17; SCRUM-232, SCRUM-233, SCRUM-234, SCRUM-244, SCRUM-242, SCRUM-243)

## How to test
1. `cd backend && npm test`: 515 unit tests pass. They cover the F-42 example, open and resolved alarms, the threshold boundary (exactly 9 h is not inactive), the F-40 order (30 h, 12 h, 4 h), scope, a device that never reported, and NF-08 (a reading between the old and the new threshold becomes low).
2. `npm run test:e2e`: the new `inactive-devices` (7 tests) and `low-energy-alerts` (3 tests: lowest first, exactly 30 % is not low, a reading above clears the alert, start of the streak) suites run on PostgreSQL with Colbits' triggers. It covers the order, a consultation user who does not see another circuit, the open alarm starting at last record + 9 h, the alarm closing at the next record, a SMAC after 3 h, and low energy. All e2e tests pass (98 + 3).
3. With data: `cd generator && npm run generate -- --reset --scenarios demo --silent-devices 3 --end "$(date -u +%Y-%m-%dT%H:%M:00Z)"`, log in, then call `GET /device/inactive` (Swagger). Locally it listed the 3 silent devices and the inactivity scenarios in F-40 order. A device that reported again had its alarm closed at that record.

## AI use
- [ ] **No AI**
- [ ] **AI-assisted** (suggestions, review or parts of the code)
- [x] **AI-generated** (most of the code)

Tool: Claude Code (details in `docs/bitacora-ia/2026-11-12-SCRUM-115.md`)

## Checklist
- [x] CI passes (lint, tests, build)
- [x] I reviewed everything the AI produced before pushing it

🤖 Generated with [Claude Code](https://claude.com/claude-code)

https://claude.ai/code/session_012Za94ATKRpuhbxLyQmG5TG
