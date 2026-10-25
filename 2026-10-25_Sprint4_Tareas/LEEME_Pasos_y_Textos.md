# Sprint 4 · puesta al día del 25 oct 2026

No se ha enviado nada a Slack ni a GitHub. Todo está listo para que lo pegues tú.

| Jira | Qué es | Estado |
|---|---|---|
| SCRUM-169 · F-93 | Generador: escenarios de fallas y alarmas | Código listo y probado → falta subir el PR (paso 1) |
| SCRUM-170 · F-94 | Generador: aperturas dentro y fuera de la ventana | Mismo PR que SCRUM-169 |
| SCRUM-180 · F-29 | Clientes de cada conjunto de fusibles en el detalle del nodo | Código listo y probado → falta subir el PR (paso 1) |
| SCRUM-195 | Formulario de supervisión del 22/10 | Texto listo (sección 3) → pégalo en SharePoint |
| SCRUM-211 | Información para la presentación Alpha | Texto listo (sección 4) → pégalo en el documento de Paola **hoy** |
| — | Retrospectiva del Sprint 3 (Sofía) | Texto listo (sección 5) → respóndele en el hilo |
| — | Horas reales del Sprint 3 (Sofía, Sprint Backlog) | **Solo tú las sabes**: llénalas en `COLBITS_Sprint_Backlog.xlsx` |

---

## 1. Subir los dos PRs (5 minutos)

En tu carpeta del repo del equipo (la misma de SCRUM-80):

```bash
cd COLBITS---colbits
git fetch origin

# PR 1: generador (SCRUM-169 + SCRUM-170)
git checkout -B SCRUM-169-generator-scenarios origin/develop
curl -sL https://raw.githubusercontent.com/yeisonestiv109/COLBITS_Project/claude/modest-planck-nvemui/2026-10-25_Sprint4_Tareas/SCRUM-169-0001-generator-scenarios.patch | git am
git -c credential.helper= push -u fork SCRUM-169-generator-scenarios

# PR 2: clientes por conjunto de fusibles (SCRUM-180)
git checkout -B SCRUM-180-fuse-set-clients origin/develop
curl -sL https://raw.githubusercontent.com/yeisonestiv109/COLBITS_Project/claude/modest-planck-nvemui/2026-10-25_Sprint4_Tareas/SCRUM-180-0001-fuse-set-clients.patch | git am
git -c credential.helper= push -u fork SCRUM-180-fuse-set-clients
```

Luego, en GitHub, abre cada PR **hacia `develop`** (no `main`):

| Rama | Título (cópialo exacto) | Descripción |
|---|---|---|
| `SCRUM-169-generator-scenarios` | `SCRUM-169 \| feat(generator): induce fault and alarm scenarios (F-93, F-94) [AI-GENERATED]` | `PR_SCRUM-169_generator_scenarios.md` |
| `SCRUM-180-fuse-set-clients` | `SCRUM-180 \| feat(nodes): clients of each fuse set for the node detail (F-29) [AI-GENERATED]` | `PR_SCRUM-180_fuse_set_clients.md` |

(Sin la barra invertida `\` del título: en la tabla solo escapa el `|`.) Los dos títulos y descripciones
ya pasan el chequeo `pr-standards` del repo. Cuando los aprueben y hagas merge, me dices y paso
SCRUM-169, 170 y 180 a **Finalizado** (la DoD del equipo dice: después del merge).

---

## 2. Mensajes para Slack

### a) Canal **Backend Team** (o #colbits)
> Hi team! Two PRs to `develop` from my side:
> 1. **SCRUM-169/170 (US9 · F-93, F-94): generator test scenarios.** `npm run generate -- --reset --scenarios demo --end "$(date -u +%Y-%m-%dT%H:%M:00Z)"` loads 16 scenarios: 1/2/3-phase openings, two branches of one node, openings 4 min and 6 min apart (inside and outside the 5-min window), voltage loss, SMAC phase drop, overload, low energy, inactivity, free fall and solar alarm. It also fills `alarms` with the Technical Annex codes. The CLI prints what the map should show for each node. I tested it with our real backend: `GET /nodes` gave 6 red, 1 yellow, 1 gray and 1 inactive SMAC, exactly as expected. Without `--scenarios` nothing changes.
> 2. **SCRUM-180 (US11 · F-29): `GET /nodes/:name/fuse-sets`.** It returns each fuse set with its `client_count`, with the user scope applied (404 outside it). Unit and e2e tests are included.
> Could someone review them? 🙏

### b) DM a **Sofía** (porque F-27 es suyo y depende de esto)
> Holaa Sofi 🙌 Ya subí el PR de F-29 (SCRUM-180): `GET /nodes/:name/fuse-sets` devuelve los conjuntos de fusibles del nodo con su número de clientes (`client_count`, `null` si Colbits no lo tiene), con el scope del usuario. Lo hice pequeño a propósito para que tu detalle del nodo (F-27) lo llame o reutilice `NodesService.fuseSetsOf()` dentro de tu respuesta, como prefieras. Si quieres que lo meta yo dentro de tu endpoint cuando lo tengas, me dices.
> También quedó el generador con escenarios (US9). Con `--scenarios demo` ya tienes nodos rojos, amarillos y grises para probar el mapa y el panel de fallas.

### c) DM a **Javier** (F-95 es suyo y es muy parecido)
> Hola Javier, te cuento que en el PR de US9 (SCRUM-169) el escenario `inactivity` con `durationMinutes` ya genera un hueco de datos y el dispositivo vuelve a reportar después. Puede servirte para F-95 (SCRUM-171): si te sirve, solo agrégale los casos que te falten encima de eso, para no duplicar código. Está explicado en el README del generador.

### d) Hilo de **Sofía** (retrospectiva). Ver sección 5. Después reacciona con ✅ a su mensaje.

---

## 3. Formulario de supervisión (SCRUM-195) · reunión del 22/10/2026
SharePoint → `COLBITS_Supervision_Meetings.docx` → tu fila en **"Individual contribution this sprint"**.

- **Main tasks this week:** My synthetic data generator (US1: F-90, F-91, F-92) was reviewed and merged into `develop`. I implemented F-37 (SCRUM-80): `GET /nodes` now marks a node when its SMAC has sent no data for more than 3 hours, using the newest record of the three SMAC tables. It has unit and end-to-end tests and was merged. I checked that F-36 (Falcon, 9 h) was already covered by the US6 code. I also fixed an accidental merge to `main` and restored the branch.
- **Plan for next week:** Add test scenarios to the generator (US9: F-93, F-94): fuse openings, voltage loss, phase drop, overload, low energy, inactivity, free fall, solar-panel alarm and openings inside and outside the 5-minute window. Add the clients of each fuse set to the node detail (F-29). Prepare my part of the Alpha prototype presentation.

Después de pegarlo, pasa SCRUM-195 a Finalizado (o me dices y lo paso yo).

---

## 4. Información para la presentación Alpha (SCRUM-211)
Documento de Paola: `06_Presentations/2026-10-28_Alpha_Prototype/Alpha_Prototype_Presentation.docx`.
Pega esto en tu parte (o donde ella pida "info per member"). Está en inglés B2, lista para diapositivas.

**Yeison Delgado — Synthetic data, test scenarios and device inactivity**

*What I built for the Alpha*
- **Synthetic data generator (US1):** a command-line tool that creates Colbits' exact database structure and fills it with a realistic network: zones, substations, circuits, poles, fuse sets and transformers, with Falcon and SMAC readings every few minutes. The same seed always gives the same data, so tests are repeatable. 50 poles and 7 days of data (about 140,000 rows) load in about 7 seconds.
- **Test scenarios (US9):** with one option, the generator creates controlled problems: one-, two- and three-phase fuse openings, faults on two branches of the same pole, loss of voltage, transformer phase drop, overload, low battery, devices that stop sending data, free fall and solar-panel alarms. It also writes the alarm events with the official Colbits codes.
- **Device inactivity (US5):** the map marks a device as inactive when a Falcon sends nothing for 9 hours or a SMAC for 3 hours. Both values are in the configuration, not in the code.
- **Clients per fuse set (US11 · F-29):** the API returns how many clients each fuse set feeds, for the node detail panel.

*Why it matters*
- We never depend on Colbits' real database to develop or test. Their production data stays read-only.
- Every map color has a known test case. For example, two openings 4 minutes apart must be one two-phase fault, but 6 minutes apart they must be two single-phase faults (5-minute window).
- Proof: with the demo scenarios, the real backend returned 6 red nodes, 1 yellow and 1 gray, exactly the ones the generator predicted. All the others were green.

*Demo idea (30–40 s)*
1. Run `npm run generate -- --reset --scenarios demo` and show the summary with "expect: …" for each node.
2. Open the map: the red, yellow and gray poles appear exactly where the summary said.

*Numbers*
- Generator: 38 automated tests (unit tests and tests on PostgreSQL with Colbits' triggers).
- Backend (F-37 and F-29): unit and end-to-end tests. Backend coverage is above 95 %.

*Next steps*
- Fault grouping engine tests on these scenarios (with the backend team).
- Node detail panel together with Sofía (F-27 + F-29).

*Speaker notes (about 45 s)*
> "My part is the data that makes testing possible. We can't change Colbits' database, so I built a generator that copies its exact structure and fills it with a realistic network. On top of that, it can create problems on purpose: open fuses, missing voltage, dropped phases, silent devices. Each problem comes with the result we expect. For example, two phases that open four minutes apart must be one fault, but six minutes apart they are two faults. When we loaded the demo, the backend showed exactly the red, yellow and gray poles that the generator predicted. So every color on the map has a test behind it."

**Corrección pendiente para cualquier diapositiva de tecnologías:** el backend es **NestJS + TypeScript** (no FastAPI) y el generador es una CLI en **TypeScript** (no Python).

---

## 5. Retrospectiva del Sprint 3 (respuesta en el hilo de Sofía)

> **RETROSPECTIVE**
> 1. **What went right?** The synthetic data generator was merged, so the team can test with realistic data without Colbits' database. F-37 (SMAC inactivity) was finished with unit and end-to-end tests. The PR template and the `pr-standards` check helped me open correct PRs.
> 2. **What went wrong?** I was not active on Slack at the start of the sprint, so the team didn't know my progress. I needed help to get access to the organization repo, and I merged into `main` by mistake once (fixed the same day). Integration with the backend came late.
> 3. **What could we do differently?** Share progress on Slack every two or three days, even when it is small. Open PRs earlier and smaller, and always check that the base branch is `develop`.
>
> **AI RETROSPECTIVE**
> 1. **AI at project level:** Yes. I used Claude Code to write and test code (generator, F-37), to check the code against the Scope and the Technical Annex, and to prepare documentation and presentation content. I logged each use in `docs/bitacora-ia`.
> 2. **What went well:** It was fast for tests and repetitive code, and it tested everything against a real PostgreSQL before I pushed.
> 3. **Difficulties:** Some AI PRs were large and harder for the team to review. I had to check details carefully against the specification, for example the event codes and thresholds.
> 4. **What to do differently:** Ask for smaller changes and explain the "why" in the PR, so reviewers can follow it.
> 5. **AI team contract:** It still reflects how we work. No changes from my side.

---

## 6. Qué probé (para que puedas defenderlo)
- **Generador:** 38 pruebas pasan (31 unitarias y 7 con PostgreSQL 16 y los triggers de Colbits), más typecheck. Sin `--scenarios`, los datos son idénticos byte a byte a la versión anterior (mismo hash), así que no le cambia nada a nadie.
- **Contra el backend real:** cargué el `demo` y llamé a `GET /nodes`: 6 CRITICAL, 1 WARNING, 1 INACTIVE y 1 `smac_inactive`, exactamente los nodos que predijo el generador. Los otros 41 quedaron NORMAL.
- **F-29:** 87 pruebas unitarias del backend, lint, build y cobertura OK. 6 pruebas e2e por HTTP contra PostgreSQL: 18 y 9 clientes en las ramas A y B, 42 en el nodo de un solo conjunto, `null` si no hay dato, 404 fuera del scope y 401 sin token.
- **Fuentes:** Scope v6.0 (RF-GEN-04/05, CU-07 2a/2b, §3.2, §3.4) y Technical Annex §4.2 (códigos `13xx20`, `13xx21`, `13xx01`, `05xx21y`).
- **Punto abierto para el motor de fallas:** con 3 aperturas, "separadas por menos de la ventana" puede leerse respecto a la apertura anterior o a la primera. El generador usa la anterior y lo dice en el README. El `demo` solo usa secuencias de 2 aperturas, que no son ambiguas.
