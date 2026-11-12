# Ponerse al día · 12 nov 2026

## Qué quedó hecho
| Jira | Qué | Estado |
|---|---|---|
| SCRUM-232 · F-38 | Alarma de inactividad cuando un dispositivo se calla | ✅ Hecho y probado → subir PR (paso 1) |
| SCRUM-233 · F-39 | La alarma se resuelve sola cuando el dispositivo vuelve a reportar | ✅ Mismo PR |
| SCRUM-234 · F-40 | Lista de dispositivos inactivos del scope, el más antiguo primero | ✅ Mismo PR |
| SCRUM-244 · NF-08 | Umbrales de energía baja configurables (30 % por defecto) | ✅ Mismo PR |
| PR #73 (Mónica) | Revisión | ✅ Revisado y probado → pegar el comentario (paso 2) |
| SCRUM-263 | Formulario de supervisión del 05/11 | ✅ Texto listo (paso 4) |

**Qué es:** un endpoint nuevo, `GET /device/inactive?hours=24`.
- Devuelve exactamente la forma que el panel "Dispositivos inactivos" de Mónica ya usa (`inactiveDevices.types.ts` de su rama SCRUM-235): dispositivo, nodo, zona, subestación, circuito, umbral y sus **periodos de inactividad** `{start, end}`.
- Cada periodo es la alarma de inactividad: se abre cuando el dispositivo pasa su umbral (F-38) y se cierra sola con su siguiente registro (F-39).
- Viene ordenado como pide F-40.

**Pruebas:**
- 510 pruebas unitarias, 95 + 3 pruebas e2e contra PostgreSQL con los triggers de Colbits, lint, build y 0 bloques duplicados (para que no se queje Sonar como la otra vez).
- Lo probé con datos del generador y el backend real: listó los dispositivos callados en orden, y el que volvió a reportar quedó con su alarma cerrada justo en ese registro.
- También probé que **no choca con el PR #73 de Mónica**: se mezclan limpio en cualquier orden.

---

## 1. Subir el PR (como siempre)
```bash
cd COLBITS---colbits
git fetch origin
git checkout -B SCRUM-115-inactive-devices-backend origin/develop
curl -sL https://raw.githubusercontent.com/yeisonestiv109/COLBITS_Project/claude/modest-planck-nvemui/2026-11-12_US17_Inactividad/SCRUM-115-0001-inactive-devices.patch | git am
git -c credential.helper= push -u fork SCRUM-115-inactive-devices-backend
```
- **Base:** `develop`
- **Título:** `SCRUM-115 | feat(device): inactive devices with their inactivity alarms (US17) [AI-GENERATED]`
- **Descripción:** abre este link y copia **todo el texto** (no el nombre del archivo):
  https://raw.githubusercontent.com/yeisonestiv109/COLBITS_Project/claude/modest-planck-nvemui/2026-11-12_US17_Inactividad/PR_SCRUM-115_inactive_devices.md
- Pide revisión a Javier o Mahmoud.

## 2. Revisar el PR #73 de Mónica
En GitHub → PR #73 → **Files changed** → **Review changes** → **Approve**, y pega esto:

> Reviewed and tested locally on the PR head. Backend: 406 unit tests, lint, build, and all e2e suites on PostgreSQL with the 01/02/03 seed (72 + 3). Generator: 57 tests and typecheck. All pass.
> I checked the changes to my code: the transformer layer now takes the existing phases from the voltage columns (CL-07/CL-24), falling back to the `-2`/`-3` suffix, and a drop on a phase the transformer does not have is ignored and logged (TD-05). That matches the client's answers, and F-23 still holds. The generator's NULL missing phase, any two-phase pair and `load_percent` from apparent power (TD-21/22/23) look right too.
> Small note, not blocking: the `#CORRECTION` comments are useful as a record, but once merged they could live only in `docs/decisions/client-clarifications-2026-11.md`, so the code stays shorter. Approving 👍

## 3. Jira
- SCRUM-232, SCRUM-233, SCRUM-234 y SCRUM-244 → **En curso** ahora, y **Finalizado** cuando mergeen el PR.
- SCRUM-263 → **Finalizado** después de pegar el formulario (paso 4).

## 4. Formulario de supervisión (SCRUM-263) · reunión del 05/11/2026
SharePoint → `COLBITS_Supervision_Meetings.docx` → tu fila en "Individual contribution this sprint":
- **Main tasks this week:** The backend of US7 (transformer-layer colours, F-19 to F-23) was merged (PR #60). `GET /nodes` sends `transformer_state`, counted against the transformer's own phases. I also fixed the SonarCloud duplication gate by moving the shared e2e setup into one helper. I confirmed with Sofía that US9 and US11 were complete, and I sent my SSH public key for the Colbits VPS.
- **Plan for next week:** The backend of US17. That means the inactivity alarm that opens and resolves by itself (F-38, F-39), the list of inactive devices in the user's scope, oldest first (F-40), and configurable low-energy thresholds (NF-08), so Mónica can connect the Inactive devices tab.

## 5. Mensajes

### a) WhatsApp a **Mónica**
> Holaa Mónica, ya está lo de US17 backend 🙌 Subí el PR (SCRUM-115): `GET /device/inactive?hours=24`. Devuelve la lista de dispositivos inactivos del scope del usuario, con la misma forma de tu `InactiveDevice`: `node`, `zone`, `substation`, `circuit`, `threshold_hours` y `periods: [{ start, end }]`. `end` es `null` mientras sigue callado; esa es la alarma de inactividad, que se abre al pasar el umbral y se cierra sola cuando vuelve a reportar (F-38/F-39). Viene ordenada como F-40: primero los inactivos ahora, del registro más viejo al más reciente, y luego los que ya volvieron.
> Además trae `kind` (FALCON/SMAC), `hw_id`, `source_id` (fusible o transformador), `inactive_now`, `last_record_at`, `energy_percent` y `low_energy` (umbral de 30 % configurable, NF-08), por si quieres mostrar que se apagó por batería baja.
> Los periodos vienen completos, no recortados; tu `clipPeriods` ya los recorta a la ventana, así que encaja directo. Para "Última hora" / "Último día" puedes pedir `?hours=1` o `?hours=24`, o siempre 24 y recortar en el front.
> Ya está en Swagger. También revisé y aprobé tu PR #73: corrí todas las pruebas y está todo bien 👍

### b) #colbits (Mónica pidió avisar de endpoints nuevos para Swagger)
> Hi team! New backend endpoint for US17: `GET /device/inactive?hours=24` (inactive devices in the user scope with their inactivity alarms, F-38 to F-40) and two new settings, `FALCON_LOW_ENERGY_PERCENT` and `SMAC_LOW_ENERGY_PERCENT` (30 % default, NF-08). Documented in Swagger. PR SCRUM-115 is up; reviews welcome 🙏
