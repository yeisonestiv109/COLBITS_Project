# Ponerse al día · 15 nov 2026 (último día del Sprint 5)

| Jira | Qué | Estado |
|---|---|---|
| **SCRUM-252** · F-100 | Escenarios de demo: abrir/cerrar fusible, cortar/restaurar tensión, caer/restaurar fase | ✅ Hecho y probado → subir PR (paso 1) |
| SCRUM-242, 243, 244 | Energía baja y umbrales | ✅ Ya mergeado (PR #79) → pasar a **Finalizado** en Jira |
| SCRUM-263 | Formulario 05/11 | Si ya lo pegaste → **Finalizado** |
| **SCRUM-274** | Diapositivas Beta (secciones 3, 5, 9 y tus filas en 10 y 11) | ✅ Textos listos (paso 4) → pegarlos en el doc de Mahmoud |
| **SCRUM-277** | Presentar la Beta (18 nov) con Mahmoud | Guion listo (paso 5) → ensayar |

**Pruebas de SCRUM-252:**
- 729 pruebas unitarias y 179 pruebas e2e contra PostgreSQL con los triggers de Colbits (6 de ellas nuevas, por HTTP real).
- Lint, build y Prettier en verde, y 0 bloques duplicados para que Sonar no se queje.
- En la prueba e2e, cada escenario hace reaccionar el mapa y la lista de alarmas como se espera: fusible abierto da rojo y una falla, tensión cortada da amarillo, 1 fase caída da amarillo y 2 dan rojo, y al restaurar todo vuelve a verde.

---

## 1. Subir el PR (como siempre)
```bash
cd COLBITS---colbits
git fetch origin
git checkout -B SCRUM-252-demo-scenarios origin/develop
curl -sL https://raw.githubusercontent.com/yeisonestiv109/COLBITS_Project/claude/modest-planck-nvemui/2026-11-15_US31_Escenarios/SCRUM-252-0001-demo-scenarios.patch | git am
git -c credential.helper= push -u fork SCRUM-252-demo-scenarios
```
- **Base:** `develop`
- **Título:** `SCRUM-252 | feat(scenarios): demo scenarios for fuses, voltage and phases (US31) [AI-GENERATED]`
- **Descripción:** copia **todo el texto** de este link: https://raw.githubusercontent.com/yeisonestiv109/COLBITS_Project/claude/modest-planck-nvemui/2026-11-15_US31_Escenarios/PR_SCRUM-252_demo_scenarios.md
- Pide revisión a **Javier** (su SCRUM-253 se construye encima) y avísale a Paola.

## 2. Jira
- [SCRUM-242](https://colbits-success.atlassian.net/browse/SCRUM-242), [243](https://colbits-success.atlassian.net/browse/SCRUM-243) y [244](https://colbits-success.atlassian.net/browse/SCRUM-244) → **Finalizado** (el PR #79 ya está mergeado).
- [SCRUM-252](https://colbits-success.atlassian.net/browse/SCRUM-252) → **En curso** ahora, y **Finalizado** cuando mergeen.
- [SCRUM-263](https://colbits-success.atlassian.net/browse/SCRUM-263) → **Finalizado**, si ya pegaste el formulario.
- [SCRUM-274](https://colbits-success.atlassian.net/browse/SCRUM-274) → **Finalizado** cuando pegues lo del paso 4.

## 3. Mensajes

### a) #colbits, respondiendo a **Paola** (el contrato que pidió)
> Hi Paola, sorry for the delay! The backend of the scenario panel is ready (PR SCRUM-252, tested end to end). Here is the contract:
>
> **Is demo mode on?** `GET /scenarios` (any logged-in user) returns
> `{ "enabled": true, "actions": [ { "action": "OPEN_FUSE", "target": "FUSE", "channel_required": false, "effect": "..." }, ... ] }`
> Show the panel only when `enabled` is `true`. It is true in local docker-compose and Render staging, and false in production.
>
> **Run a scenario:** `POST /scenarios` (admin only) with body
> `{ "action": "OPEN_FUSE", "target": "0000100002-A-R" }`, or for a phase
> `{ "action": "DROP_PHASE", "target": "0000100004-T1-3", "channel": "A" }`
> - Actions: `OPEN_FUSE`, `CLOSE_FUSE`, `CUT_VOLTAGE`, `RESTORE_VOLTAGE` (target = a fuse `id` from `GET /nodes/{name}/detail` → `fuse_sets[].fuses[].id`), and `DROP_PHASE`, `RESTORE_PHASE` (target = a transformer `id` from `transformers[].id`, plus `channel` A, B or C).
> - **201** → `{ "action", "node", "target", "channel", "hw_id", "event_code": "131420", "event_type": "SET", "recorded_at", "effect" }`
> - **409** = it's already in that state (e.g. the fuse is already open), so show "already open". **403** = demo mode off or not an admin. **404** = target not found. **400** = bad body or the transformer has no such phase.
>
> After a 201, the map, the alarm list and the node detail change on the next refresh, with no extra calls. Javier's actions (low energy, inactivity, overload, free fall, solar) will be added to the same `actions` list with the same body shape. It's all in Swagger under **scenarios**.

### b) DM a **Javier**
> Hola Javier! Ya subí SCRUM-252 (PR "demo scenarios"), puedes arrancar tu SCRUM-253 encima de esa rama o de develop cuando se mergee. Te dejé la estructura lista:
> • `backend/src/scenarios/scenario-actions.ts` tiene el registro `SCENARIO_ACTIONS`. Ahí agregas tus acciones (LOW_ENERGY, INACTIVITY, OVERLOAD, FREE_FALL, SOLAR_ALARM…) y salen solas en `GET /scenarios`.
> • `scenarios.service.ts` escribe lo que mandaría el dispositivo: copia la última lectura (`falcon_values` / `smac_vc_values`) con el cambio, más el evento en `alarms` con código del Anexo. Para las tuyas, la mayoría son solo telemetría (`supercaps_per`/`battery_per` bajo el 30 %, `is_free_fall_alarm` + `13xx01`, `is_solar_panel_alarm`, `load_percent` > 90 %). Inactividad sería no escribir lecturas o mover el `recorded_at` hacia atrás.
> • La restricción al ambiente demo ya tiene una base: `DEMO_SCENARIOS_ENABLED` (403 si no es `true`), solo administradores, y 409 si la acción repite el estado. Tu NF-03 puede reforzarla (por ejemplo, negar si la BD no es de generador).
> • El test e2e `test/scenarios.e2e-spec.ts` te sirve de plantilla.
> Cualquier cosa me escribes 🙌

### c) DM a **Mahmoud** (las diapositivas)
> Hi Mahmoud! Sorry for the late reply. My parts of the Beta doc are ready: sections 3 and 5, the test numbers for section 9, and my rows in 10 and 11. I'm pasting them in the document now. For the rehearsal, could we do it tomorrow (Monday)? Tell me a time that works for you 🙏

---

## 4. Diapositivas Beta (SCRUM-274) · para pegar en el doc de Mahmoud
Doc: https://studentmdh.sharepoint.com/:w:/r/sites/SUCCESSDistributedSoftwareDevelopmentCourse-Colbits26_27/Delade%20dokument/06_Presentations/2026-11-18_Beta_Prototype/COLBITS_Beta_Prototype_Presentation.docx?d=w8c47f5a487244226bddb5e22c36e36c3&csf=1&web=1&e=Bk4sF8
Busca las líneas amarillas que dicen **"TO FILL · Yeison"**, reemplázalas por lo de abajo y quita el resaltado.

### Section 3 · Architecture and technology changes since the Alpha
- **Same stack:** React + TypeScript, NestJS + TypeScript (Node 24), and PostgreSQL with Colbits' schema (read-only). The team changed what the backend does, not the tools.
- **New backend modules:**
  - **faults:** active and resolved alarms, fault grouping and severity order.
  - **users:** user and scope administration.
  - **device:** inactive devices and low-energy alerts.
  - **scenarios:** the demo panel.
- **Alarm events are the source of truth.** After Colbits' answers (Nov 3), fuse, voltage and phase states come from the `alarms` table (SET/CLEAR events). Telemetry is used only when a device has no events. The transformer phases come from its voltage readings.
- **Everything configurable:** inactivity (9 h / 3 h), low energy (30 %), overload (90 %), the fault window (5 min) and the severity weights are all settings, not code.
- **Deployment:** Render production (main) and staging (develop), plus Colbits' VPS with Docker containers that run as non-root.
- **Quality:** SonarCloud quality gate on every PR (duplication ≤ 3 %), plus e2e tests on an isolated PostgreSQL with Colbits' triggers.

### Section 5 · Backend changes (one line)
> Backend: alarms that open and resolve from the device events, transformer colours, inactive devices and low-energy alerts, user administration, and a demo scenario API.

### Section 9 · Backend and generator tests, CI, integration problems
- **Backend:** 729 unit tests (35 suites), plus 179 end-to-end tests on PostgreSQL with Colbits' triggers. Statement coverage is **93.6 %** (branches 84.2 %), and the target is 80 %.
- **Generator:** 57 tests. They cover structure, counts, periodicity, every scenario and the grouping window, and some run on a real database.
- **CI (GitHub Actions on every PR):** lint, build and tests with coverage for the backend and the frontend, the `pr-standards` check (title, AI declaration, description), and the SonarCloud quality gate.
- **Integration problems and fixes:**
  - The frontend waited for endpoints that did not exist yet. Fix: we publish the API contract first, so the frontend can start with mocks.
  - Parallel PRs edited the same config lines and caused merge conflicts. Fix: we keep each setting in its own place and merge `develop` before pushing.
  - SonarCloud rejected one PR for duplicated test setup. Fix: we moved it into one shared e2e helper.
  - The client's answers changed some rules (alarms over telemetry). Fix: one decisions document, and code and tests updated to match it.

### Row 10 · Main challenge (your row)
| Member | Challenge | How we dealt with it |
|---|---|---|
| Yeison | My backend tasks blocked other people. The frontend panels waited for my endpoints, and Javier's scenarios build on mine. | I published the API contract before the code, built the scenario service as a registry that others can extend, and tested each endpoint end to end on the real database before handing it over. |

### Row 11 · (adapt it to the column of that table)
> Lesson learned: agree on the API contract before writing the code, and keep PRs small, so frontend and backend can work in parallel instead of waiting for each other.

---

## 5. Guion para presentar (SCRUM-277) · tu parte, unos 2 minutos, inglés B2
> "I will talk about how the system changed since the Alpha, and how we test it.
> The stack is the same: React, NestJS and PostgreSQL. What changed is what the backend does. We added modules for faults, users, inactive devices and demo scenarios.
> The biggest change came from Colbits' answers. Now the state of a fuse, the voltage or a transformer phase comes from the alarm events the devices send, SET and CLEAR. So an alarm opens and closes exactly when the device says so. All the thresholds, like nine hours of inactivity or thirty percent of battery, are settings, not code.
> For quality, every pull request runs lint, build, tests and SonarCloud. The backend has more than seven hundred unit tests and almost two hundred end-to-end tests on a real database with Colbits' triggers, with ninety-three percent coverage.
> Now the demo. In the scenario panel, I choose a node and press 'Open fuse'. The backend writes exactly what a Falcon would send. On the next refresh, the node turns red on the map, and the fault appears in the list. Then I press 'Close fuse', and it goes back to green, and the fault moves to 'Resolved'."

**Para el ensayo:** corre la demo una vez completa con Mahmoud y toma el tiempo. Necesitas el panel de Paola y el PR SCRUM-252 mergeado en staging (o `docker compose up` local).
