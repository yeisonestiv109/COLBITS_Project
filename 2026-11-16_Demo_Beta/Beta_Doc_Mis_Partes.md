# Beta · lo que te toca en el doc de Mahmoud

**Cómo está la cosa:**
- **Mahmoud hace las diapositivas.** Él lo dijo en #colbits: primero se llena el documento y con eso arma el PowerPoint. Su plazo es mañana, 17.
- **"Jason" eres tú**, el segundo presentador. Según el documento, **Mahmoud presenta 1–6** (introducción, eventos, cambios, commits, Alpha→Beta y planes) y **tú presentas 7–12** (riesgos, equipo, pruebas, experiencias, horas y la demo). La 13 la presentan los dos.
- **El otro doc** (`COLBITS_Sprint4-5_Beta_Documentation.docx`): Mahmoud no compartió el link, solo dijo que está en Teams. Búscalo en **Teams → COLBITS → Archivos → 06_Presentations → 2026-11-18_Beta_Prototype**, la misma carpeta de este doc.

**Qué hacer:** en el doc de Mahmoud, busca cada **"TO FILL"** de abajo, reemplaza el texto amarillo por el que te dejo y quita el resaltado.
Link: https://studentmdh.sharepoint.com/:w:/r/sites/SUCCESSDistributedSoftwareDevelopmentCourse-Colbits26_27/Delade%20dokument/06_Presentations/2026-11-18_Beta_Prototype/COLBITS_Beta_Prototype_Presentation.docx?d=w8c47f5a487244226bddb5e22c36e36c3&csf=1&web=1&e=Bk4sF8

---

## 1. Sección 3 · las 5 preguntas de arquitectura ("TO FILL · Javier, Yeison and Mahmoud")
Contesté las cinco con lo que hay en el repo. Si Javier o Mahmoud ya escribieron algo, quédate con lo que no se repita.

1. **Live updates:** polling. The map asks the API for the node states **every 15 seconds** (it was 60 s at Alpha), and the backend recomputes every state on each request from the latest stored data. A change is shown in at most about 15 s, inside the 20 s target (NF-01). We kept polling instead of websockets because it is simpler and already meets the target.
2. **Fault engine:** it is not a background job. `GET /faults` computes the faults on each request from the `alarms` table. It groups the fuse openings of the **same fuse set** whose times are less than 5 minutes apart (configurable), and classifies them as single-, two- or three-phase by distinct phases. A fault stays open until all its phases report a close, and resolved faults keep their duration (`GET /faults/resolved`).
3. **Demo mode:** new endpoints `GET /scenarios` (is demo mode on, and which actions exist) and `POST /scenarios` (open or close a fuse, cut or restore voltage, drop or restore a phase, plus device scenarios). Each action writes the reading and the alarm a real device would send, so the map and the lists react on their own. It only works when `DEMO_SCENARIOS_ENABLED` is on, only for the administrator, and only on a database built from generated data, never on Colbits' data (NF-03).
4. **New libraries, tables or deployments:** no new libraries. No new tables in Colbits' schema; the demo safeguards add two small tables of our own on the generated database. Deployment: Render production and staging, and now **Colbits' own server (dev.colbits.com.co)** with Docker containers that run as non-root. SonarCloud now checks every pull request.
5. **Decisions we reversed:**
   - The "background fault engine" became a calculation on each request, which is simpler and always up to date.
   - After Colbits' answers of 3 Nov, the state of a fuse, the voltage and a phase comes from the **alarm events** (SET/CLEAR), and telemetry is only a fallback.

## 2. Sección 5 · una línea de cambio "que el usuario no ve" ("TO FILL · Javier, Yeison, Mahmoud")
> Yeison: the inactive-device list, the low-energy alerts and the demo scenarios are computed from the stored timestamps, with no new tables in Colbits' schema; every endpoint applies the user's scope in SQL, and the thresholds (inactivity 9 h / 3 h, low energy 30 %) are settings, not code.

## 3. Sección 7 · Riesgos · dos celdas "Result?" de la tabla
- **Backend technology change (FastAPI to NestJS):** "No problem. One language everywhere; the only cost was moving everyone and CI to Node 24, which NestJS needs."
- **Security gaps found while integrating:** "Fixed. Every endpoint applies the user's scope in SQL and answers 404 outside it, with tests for each case. No known open gap."

(La tabla inicial de riesgos la ponen Paola y Sofía; tú la presentas.)

## 4. Sección 9 · Pruebas ("TO FILL · Javier and Yeison" + problemas de integración)
Tabla, columna **Beta (now)**:
| Level | Beta (now) |
|---|---|
| Backend unit and end-to-end | **729 unit tests in 35 suites + 179 end-to-end tests in 16 suites, 93.9 % coverage** |
| Generator | **57 tests** |
| End-to-end and generator tests in CI | **Backend end-to-end: done** (runs on every pull request on an isolated PostgreSQL). **Generator: not yet in CI** (runs locally). |
| Fault scenarios tested automatically (US9) | **Done**: the backend end-to-end tests run the fuse, voltage, phase and inactivity scenarios and check the colors and alarms. |

Problemas de integración nuevos (tu parte):
- **SonarCloud rejected a pull request for duplicated test setup.** We found it in the PR checks and fixed it with one shared helper for the end-to-end tests.
- **Parallel pull requests changed the same configuration lines**, which caused merge conflicts. We found it when merging and fixed it by keeping each setting in its own place and merging `develop` before pushing.
- **The frontend waited for backend endpoints.** We found it in the sprint review, and now we publish the API contract first (for example, the scenario panel), so the frontend can start with mocks.

## 5. Sección 10 · tu fila
| Member | Experience or challenge | How we dealt with it |
|---|---|---|
| Yeison | My backend tasks blocked others: the scenario panel and the inactive-device list waited for my endpoints, and Javier's scenarios built on mine. I also did not answer on Slack fast enough. | I published the API contract first, built the scenario service so others could add to it, and tested each endpoint end to end before handing it over. Now I check Slack every day. |

## 6. Sección 11 y tablas de carga (Sprint 4 y 5) · **tus horas**
Esto solo lo sabes tú. Mándame más o menos cuántas horas le dedicaste en cada categoría y te armo las tres filas:
- Total desde el inicio.
- Sprint 4.
- Sprint 5.

Las categorías son: Meetings, Documentation, Presentation, Research, Implementation, Testing y Scrum.

## 7. Sección 12 · Demo ("TO FILL · Mahmoud and Jason")
Propuesta para cuadrar con Mahmoud:
- **Flows A, B y F** (admin, cliente, idioma): Mahmoud hace clic y tú explicas.
- **Flow C** (falla de fusible con el panel de escenarios, la principal): **tú haces clic**, porque es tu backend, y Mahmoud explica.
- **Flow D** (filtros y alarmas): Mahmoud hace clic y tú explicas.
- **Si nos pasamos de 6 minutos**, se cortan primero E (usuarios) y F (idioma).
- **Antes del miércoles:** correr la demo completa una vez en dev.colbits.com.co y tomar el tiempo ("TO FILL · Second presenter"). Para eso tiene que estar mergeado el PR del modo demo en Beta.
