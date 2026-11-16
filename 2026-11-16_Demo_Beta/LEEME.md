# Ponerse al día · 16 nov 2026

## 1. Jira: pásalas tú (sigo sin permiso para mover estados desde aquí)
Ya revisé GitHub: **todo lo tuyo está mergeado** (PR #79 energía baja e inactividad, PR #90 escenarios). Estas son las que siguen abiertas:

| Tarea | Estado ahora | Pásala a | Por qué |
|---|---|---|---|
| [SCRUM-242](https://colbits-success.atlassian.net/browse/SCRUM-242) F-44 | En curso | **Finalizado** | Mergeado en PR #79 |
| [SCRUM-243](https://colbits-success.atlassian.net/browse/SCRUM-243) F-45 | En curso | **Finalizado** | Mergeado en PR #79 |
| [SCRUM-244](https://colbits-success.atlassian.net/browse/SCRUM-244) NF-08 | En curso | **Finalizado** | Mergeado en PR #79 |
| [SCRUM-252](https://colbits-success.atlassian.net/browse/SCRUM-252) F-100 | Por hacer | **Finalizado** | Mergeado en PR #90; Paola ya lo probó con su panel |
| [SCRUM-263](https://colbits-success.atlassian.net/browse/SCRUM-263) formulario 05/11 | Por hacer | **Finalizado** | Si ya pegaste tu fila (si no, el texto está en `2026-11-12_US17_Inactividad/LEEME.md`, paso 4) |
| [SCRUM-274](https://colbits-success.atlassian.net/browse/SCRUM-274) diapositivas Beta | Por hacer | **Finalizado** | Cuando pegues tus partes en el doc de Mahmoud (`2026-11-15_US31_Escenarios/LEEME.md`, paso 4) |
| [SCRUM-277](https://colbits-success.atlassian.net/browse/SCRUM-277) presentar Beta | Por hacer | **En curso** → Finalizado el 18 | Después de presentar |

## 2. PR nuevo y pequeño: activar el panel de demo en el servidor Beta
Paola te lo pidió en #colbits: el modo demo está apagado en `dev.colbits.com.co`. Agregué las dos variables que exige la guía de Javier (`DEMO_SCENARIOS_ENABLED` y `DEMO_SCENARIOS_DATABASE`). Ese servidor solo tiene datos del generador, así que es seguro, y no choca con el PR de Javier.
```bash
cd COLBITS---colbits
git fetch origin
git checkout -B SCRUM-252-enable-demo-on-beta origin/develop
curl -sL https://raw.githubusercontent.com/yeisonestiv109/COLBITS_Project/claude/modest-planck-nvemui/2026-11-16_Demo_Beta/SCRUM-252-0002-demo-on-beta.patch | git am
git -c credential.helper= push -u fork SCRUM-252-enable-demo-on-beta
```
- **Base:** `develop`
- **Título:** `SCRUM-252 | chore(deploy): turn on the demo scenario panel on the Beta server [AI-ASSISTED]`
- **Descripción:** copia todo el texto de https://raw.githubusercontent.com/yeisonestiv109/COLBITS_Project/claude/modest-planck-nvemui/2026-11-16_Demo_Beta/PR_SCRUM-252_demo_on_beta.md

## 3. Mensajes

### a) #colbits, respondiendo a **Paola**
> Hi Paola! Thanks for testing the panel 🙌 Here's the PR that turns on demo mode on the Beta server (`deploy/docker-compose.yml`): it adds `DEMO_SCENARIOS_ENABLED: 'true'` and `DEMO_SCENARIOS_DATABASE: colbits`, the two settings Javier's guide (`docs/demo-scenarios.md`) asks for. That server only has generator data, so it's safe. Once Javier's SCRUM-253 is merged, the backend needs one data refresh on the server so the synthetic-data marker exists, and then `GET /scenarios` returns `enabled: true`.
> About resolved low-energy alerts: right now `GET /device/low-energy` only returns the active ones, and the frontend doesn't call it yet. When someone connects it, tell me and I'll add the resolved episodes (start, end, duration) the same way as the inactivity periods.

### b) DM a **Javier**
> Javier, lo vi: tu guía de escenarios quedó muy completa 👌 Paola pidió el panel en el servidor Beta, así que subí un PR pequeño que pone `DEMO_SCENARIOS_ENABLED: 'true'` y `DEMO_SCENARIOS_DATABASE: colbits` en `deploy/docker-compose.yml`, tal como dice tu guía. No choca con tu PR (lo probé), pero la fila "Beta" de `docs/demo-scenarios.md` y la línea de `docs/deploy-vps.md` dicen "Off". Si quieres, actualízalas en tu PR a "On (PR SCRUM-252-enable-demo-on-beta)".
