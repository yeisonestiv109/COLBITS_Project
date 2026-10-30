# Ponerse al día · 30 oct 2026 (el Sprint 4 cierra el 1 de noviembre)

## Qué hice
| Qué | Estado |
|---|---|
| **US7 backend: color de transformadores** (`transformer_state` en `GET /nodes`). Era lo que bloqueaba a Mónica (colores rojo/amarillo/verde por fases) y a Sofía (F-24). Nadie lo tenía en Jira. | ✅ Hecho y probado → falta subir el PR (paso 1) |
| US9 escenarios del generador (SCRUM-169/170) | ✅ Ya mergeado (PR #46) |
| F-29 clientes por conjunto (SCRUM-180) | ✅ Ya mergeado (PR #47) |
| Respuestas a Mónica (su reporte de dudas), a Javier, a Sofía y al grupo backend | ✅ Textos abajo, listos para pegar |

**Pruebas de US7:**
- 396 pruebas unitarias, 11 pruebas e2e contra PostgreSQL, lint y build: todo verde.
- Con datos del generador, el backend real dio exactamente lo esperado: 1 fase caída = amarillo, 2 de 2 y 3 de 3 = rojo, fase recuperada = verde, SMAC sin datos = gris.
- El mapa del front lo pintó solo, sin cambiar nada en el front. Captura: `mapa_transformadores_real.png`.

---

## 1. Subir el PR (igual que las otras veces)
```bash
cd COLBITS---colbits
git fetch origin
git checkout -B SCRUM-105-transformer-state origin/develop
curl -sL https://raw.githubusercontent.com/yeisonestiv109/COLBITS_Project/claude/modest-planck-nvemui/2026-10-30_Ponerse_al_dia/SCRUM-105-0001-transformer-state.patch | git am
git -c credential.helper= push -u fork SCRUM-105-transformer-state
```
- **Base:** `develop`.
- **Título:** `SCRUM-105 | feat(nodes): transformer-layer state per node (US7) [AI-GENERATED]`
- **Descripción:** abre este link, copia **todo el texto** (Ctrl+A, Ctrl+C) y pégalo. Ojo: el contenido, no el nombre del archivo.
  https://raw.githubusercontent.com/yeisonestiv109/COLBITS_Project/claude/modest-planck-nvemui/2026-10-30_Ponerse_al_dia/PR_SCRUM-105_transformer_state.md

## 2. Jira (yo ya no tengo acceso desde aquí, hazlo tú en 1 minuto)
- **SCRUM-169, SCRUM-170 y SCRUM-180 → Finalizado** (sus PRs #46 y #47 ya están mergeados).
- **US7 backend:** crea una subtarea en **SCRUM-105** llamada `[US7] Backend: transformer_state in GET /nodes (F-19 to F-23)`, asignada a ti y **En curso**. Pásala a Finalizado cuando mergeen el PR. (Sofía ofreció crearla. Si prefieres, pídeselo a ella con el mensaje 3c.)

## 3. Mensajes para Slack

### a) WhatsApp / DM a **Mónica** (responde a lo de hoy)
> Holaa Mónica, perdón por la demora y tienes razón con lo de Slack, desde hoy lo reviso a diario 🙏
> Ya hice lo que te bloqueaba: `GET /nodes` ahora manda **`transformer_state`** por nodo: `CRITICAL` (2 o más fases caídas), `WARNING` (1 fase caída), `INACTIVE` (SMAC sin datos > 3 h) y `NORMAL`, o `null` si el nodo no tiene transformador. Cuenta las fases según el tipo de transformador, así que uno bifásico con sus 2 fases caídas sale rojo. Lo probé con el backend real y el mapa de ustedes ya lo pinta solo, sin cambiar nada en el front (te paso la captura). Ahorita subo el PR (SCRUM-105).
> Sobre tu reporte de dudas, así lo muestra el backend:
> • **Fusible abierto:** `fuse_state: "CRITICAL"` en `GET /nodes`. En el detalle (`GET /nodes/:name/detail`) cada fusible trae `fuse_state: "Abierto"`.
> • **Sin tensión:** `fuse_state: "WARNING"` (fusible cerrado pero sin tensión). En el detalle viene `is_voltage_present: false`.
> • **Fases caídas del SMAC:** `transformer_state` (lo nuevo). En el detalle vienen `va`, `vb`, `vc` por transformador.
> • **`fuse_state: "INACTIVE"`:** algún Falcon del nodo lleva más de 9 h sin datos (se configura con `FALCON_INACTIVITY_HOURS`).
> • **`smac_inactive: null`:** el nodo no tiene transformador. `true`/`false` es si su SMAC está inactivo.
> • Ojo: ya existe **`GET /faults`** (lo hizo Javier, ya está en develop). El panel de fallas todavía usa datos de prueba del front (`liveMonitoring.mock.ts`), por eso salen nodos como `ND04821A07`.
> • La idea del panel visual para manipular variables me parece buenísima. El generador ya tiene `--scenarios` (aperturas, caídas de fase, inactividad…), así que se puede conectar a eso. Hablamos el viernes en la reunión 🙌

### b) DM a **Javier** (te preguntó por US9)
> Hola Javier! US9 de mi parte ya está: F-93 y F-94 (SCRUM-169/170) se mergearon en el PR #46. Con `npm run generate -- --reset --scenarios demo` salen todos los escenarios: aperturas de 1/2/3 fases, ramas A/B, aperturas dentro y fuera de la ventana de 5 min, pérdida de tensión, caída de fase, sobrecarga, batería baja, inactividad, caída libre y panel solar, con sus alarmas en `alarms`. Para tu F-95 (huecos de datos) puedes usar el escenario `inactivity` con `durationMinutes`: el dispositivo deja de reportar y vuelve. Está en el README del generador. Además, ahorita subo US7 backend (`transformer_state`), que usa esos mismos eventos `05xx21y` que tu lista de fallas.

### c) Responder el `.md` de **Sofía** en #frontend (lo de US7)
> Hola Sofi! Yo tomé el backend de US7: `GET /nodes` ya manda `transformer_state` con la regla RF-MAP-04/05 (y F-23 para bifásicos), así que F-24 ya no está bloqueado. Ahorita subo el PR (SCRUM-105). ¿Me ayudas creando la subtarea de backend en Jira y asignándomela, o la creo yo?

### d) Grupo **backend** (Mahmoud: ¿polling o websockets?)
> Sorry for the late reply! I vote **polling**. It's what we decided in the Requirements & Design presentation (simpler, and enough for the 20-second target), and the map already does it (SCRUM-181/182). The state is recomputed on every request, so lowering the refresh interval to 15–20 s meets NF-01 without adding websockets.

### e) #colbits (Mónica pidió avisar de endpoints nuevos para Swagger)
> Hi team! New backend change for Swagger: `GET /nodes` now includes `transformer_state` (US7: CRITICAL = 2+ phases down, WARNING = 1 phase down, INACTIVE = SMAC without data > 3 h, NORMAL, or null without a transformer). It's already documented in the Swagger DTO. PR SCRUM-105 coming in a few minutes, reviews welcome 🙏 @Mónica

## 4. VPS de Colbits: tu llave SSH (Katerine la pidió)
En tu computador:
```bash
ssh-keygen -t ed25519 -C "yeison-colbits"     # Enter a todo
cat ~/.ssh/id_ed25519.pub                      # copia esa línea
```
Mándale a Mónica **solo la línea `.pub`** (la pública). Nunca mandes `id_ed25519` sin `.pub`.

## 5. Horas reales del Sprint 3 / 4
Si aún no las llenaste en `COLBITS_Sprint_Backlog.xlsx`, hazlo hoy: el sprint cierra el 1 de noviembre.
