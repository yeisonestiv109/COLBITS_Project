# Puesta al día · 16 oct 2026 (el Sprint 3 termina el domingo 18)

## 1. Mensajes listos para pegar en Slack (no se ha enviado nada)

### a) Canal **Backend Team**, como respuesta al mensaje de Sofía sobre el PR #20
> Hi team, sorry for being off Slack these days. Quick update from my side:
> - **Synthetic data generator (US1) is ready**: F-90, F-91 and F-92. It's a TypeScript CLI (Node 24, no build step), so it fits our NestJS backend. It creates Colbits' exact schema, fills it with a seeded, configurable network (zones, substations, circuits, nodes, two-branch nodes, transformers) and Falcon/SMAC telemetry every N minutes over any range. Same seed = same data. It loads 50 nodes / 7 days (~140k rows) in about 6 s. There are 16 tests, including database tests against PostgreSQL 16 with Colbits' triggers.
> - It also has `--silent-devices N` (devices with no data in the last 10 h), so we can test the gray/inactive nodes right now.
> - I'll open a PR to `develop` with it under `generator/`, following CONTRIBUTING.md. Who can give me access to the organization repo, or send me the link?
> - I'll also review PR #20 today so it can be merged.
> - @Mahmoud: does your inactivity endpoint already use the 9 h (Falcon) and 3 h (SMAC) thresholds? If so, F-36/F-37 (SCRUM-79/80) are done and I'll link them to your work.

### b) Canal **#colbits**, como respuesta a Sofía sobre la Sprint Planning
> Sunday 8:00 AM works for me 👍

### c) DM a **Sofía**
> Holaa Sofi, perdón por desaparecerme estos días. Ya terminé el generador de datos sintéticos (US1); está listo para subirlo al repo como PR a `develop`. Hoy reviso también el PR #20 del backend. ¿Me pasas el link del repo de la organización o me agregas? Y sí, voy a estar más pendiente del Slack 🙏

### d) DM a **Mónica**, sobre lo de NestJS vs FastAPI
> Hola Mónica, tenías toda la razón y gracias por notarlo. El backend ya está en **NestJS (TypeScript)**: el login, JWT, roles y el CI ya funcionan ahí. Cambiar ahora a FastAPI nos haría perder todo eso. Propongo quedarnos con **NestJS** y justificarlo así: ya hay código funcionando y probado, un solo lenguaje (TypeScript) en front, back y generador, y NestJS genera la documentación de la API con Swagger igual que FastAPI. Yo actualizo las diapositivas de decisiones y tecnologías para la próxima presentación. ¿Te parece?

### e) DM a **Javier**
> Hola Javier, perdón la demora. Ya quedó el generador de datos sintéticos (F-90, F-91, F-92): genera la jerarquía completa y la telemetría Falcon/SMAC con semilla, y lo probé contra PostgreSQL con el esquema real de Colbits. Lo subo como PR a `develop` apenas tenga acceso al repo de la organización. Si necesitas datos para probar algo del backend, dime cuántos nodos y días y te lo corro.

---

## 2. Formulario de supervisión (SharePoint → `COLBITS_Supervision_Meetings.docx`)
Jira: **SCRUM-137** (reunión del 9 de octubre) y **SCRUM-152** (reunión del 14 de octubre). Pega esto en tu parte de cada reunión y luego mueve las dos tareas a Finalizado.

**Reunión del 9 de octubre (semana del 5 al 9)**
- **What I did:** presented the second half of the Requirements & Design presentation (sections 8–14) on Oct 7. I built those slides and the speaker script, wrote the architectural decisions and the technology mapping, and corrected sequence diagram 2 so the fault engine writes to our own tables instead of Colbits' read-only database.
- **What was left unfinished:** the synthetic data generator (US1) had not started yet.
- **Blockers / who is waiting on whom:** the backend stack was not confirmed (the presentations said FastAPI, but the repo uses NestJS).

**Reunión del 14 de octubre (semana del 12 al 16)**
- **What I did:** implemented the synthetic data generator (US1: F-90, F-91, F-92) as a TypeScript CLI. It creates Colbits' schema and fills it with a seeded, configurable network and Falcon/SMAC telemetry. It has 16 automated tests, including database tests against PostgreSQL 16.
- **What was left unfinished:** opening the PR to `develop` in the organization monorepo. Fault scenarios (F-93, F-94) belong to US9 in the next sprint.
- **Blockers / who is waiting on whom:** I need access to the organization repository to open the PR. The frontend team is waiting for generated data to connect the map.

---

## 3. Para la próxima presentación: corrección del stack
La presentación del 7 de octubre decía **Python + FastAPI**, pero el backend real está en **NestJS (TypeScript)**. Hay que actualizar dos slides:
- **Decisiones:** cambiar a "Backend en NestJS + TypeScript". La alternativa descartada es "FastAPI (la propuesta inicial)". Razones: el código ya funciona y está probado (autenticación, JWT, roles, CI), se usa un solo lenguaje en todo el proyecto y Swagger genera la documentación de la API.
- **Tecnologías:** Backend → NestJS + TypeScript, Node 24, @nestjs/jwt + bcrypt, Swagger/OpenAPI, Jest. Generador → CLI en TypeScript con `pg`.
