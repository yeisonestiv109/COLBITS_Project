# Segunda parte · F-44 y F-45 (alertas de energía baja) en el mismo PR #79

## 1. Subir el segundo commit al PR #79 (2 minutos)
```bash
cd COLBITS---colbits
git checkout SCRUM-115-inactive-devices-backend
curl -sL https://raw.githubusercontent.com/yeisonestiv109/COLBITS_Project/claude/modest-planck-nvemui/2026-11-12_US17_Inactividad/SCRUM-115-0002-low-energy-alerts.patch | git am
git -c credential.helper= push fork SCRUM-115-inactive-devices-backend
```
Entra encima de lo que ya subiste. El PR #79 se actualiza solo y CI vuelve a correr.

## 2. Actualizar la descripción del PR #79
PR #79 → los `···` de la descripción → **Edit** → borra todo y pega el texto **completo** de:
https://raw.githubusercontent.com/yeisonestiv109/COLBITS_Project/claude/modest-planck-nvemui/2026-11-12_US17_Inactividad/PR_SCRUM-115_inactive_devices.md
(ahora también menciona F-44/F-45 y SCRUM-242/243).

## 3. Jira
- [SCRUM-242](https://colbits-success.atlassian.net/browse/SCRUM-242) (F-44) y [SCRUM-243](https://colbits-success.atlassian.net/browse/SCRUM-243) (F-45) → **En curso**, y **Finalizado** cuando mergeen el #79 (junto con 232, 233, 234 y 244).
- [SCRUM-252](https://colbits-success.atlassian.net/browse/SCRUM-252) (F-100) → **no lo muevas todavía**: está bloqueado (mensaje para Sofía abajo).

## 4. Mensajes

### a) Sofía (sobre SCRUM-252 · F-100)
> Holaa Sofi! Ya quedaron mis tareas de US17 en el PR #79: F-38, F-39, F-40, NF-08 y también F-44/F-45 (alertas de energía baja). Me queda SCRUM-252 (F-100, el servicio de escenarios para abrir o cerrar un fusible, etc.), pero en Jira dice que depende de **US32** y de que **F-100 esté en el documento de requisitos**, y no encuentro ninguno de los dos (ni en el repo ni en la spec v2.0). ¿Me pasas qué es US32 y el texto de F-100, o si ya no aplica? Apenas lo tenga lo hago. Mientras tanto, el generador ya tiene todos los escenarios por CLI (`--scenarios`), por si sirve para avanzar con lo del panel de Mónica.

### b) Mónica (WhatsApp, cortito)
> Mónica, agregué al mismo PR #79 las alertas de energía baja (F-44/F-45): `GET /device/low-energy`. Devuelve una alerta `LOW_ENERGY` por dispositivo cuyo último nivel (supercaps del Falcon o batería del SMAC) está por debajo del 30 %, con `energy_percent`, `threshold_percent`, `since`, nodo, zona, subestación y circuito. Ya está en Swagger 🙌
