## What
Turns on the demo scenario panel (US31) on the Beta server `dev.colbits.com.co`, as Paola (PO) asked on 15 Nov for the Beta demo on 18 Nov. `deploy/docker-compose.yml` gives the backend `DEMO_SCENARIOS_ENABLED: 'true'` and `DEMO_SCENARIOS_DATABASE: colbits`, which are the two settings `docs/demo-scenarios.md` (SCRUM-253) lists for this server.

It is safe because this server only holds generator data: Colbits answered on 29 Oct that it would hold "data from your generator", never their real data (NF-03). Render production stays off. Nothing else changes.

Jira: https://colbits-success.atlassian.net/browse/SCRUM-252 (US31; needed by SCRUM-254 for the demo)

## How to test
1. `docker compose -f deploy/docker-compose.yml config` shows both settings in the backend environment.
2. After the deploy, `GET /api/scenarios` on dev.colbits.com.co (logged in) returns `"enabled": true`. With SCRUM-253 merged, the backend first needs one data refresh so the synthetic-dataset marker exists, as `docs/demo-scenarios.md` says.

## AI use
- [ ] **No AI**
- [x] **AI-assisted** (suggestions, review or parts of the code)
- [ ] **AI-generated** (most of the code)

Tool: Claude Code

## Checklist
- [x] CI passes (lint, tests, build)
- [x] I reviewed everything the AI produced before pushing it

🤖 Generated with [Claude Code](https://claude.com/claude-code)

https://claude.ai/code/session_012Za94ATKRpuhbxLyQmG5TG
