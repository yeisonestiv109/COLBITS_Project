# Alpha Prototype · contenido para el documento de Paola

## 8. Current product architecture and technology stack · ~2.5 min
(Backend team: Javier, Mahmoud and Yeison. Each can take one block: 1 = architecture, 2 = stack, 3 = changes.)

**How the system is built today**
- **One repository, three parts:** a React web app (`frontend/`), a NestJS REST API (`backend/`) and the synthetic data generator (`generator/`). One `docker compose up` starts the database for local work.
- **Request flow:** the user logs in, and the API returns a JWT token. Every map request sends the token. The API checks it, reads the user's scope (zone, substation, circuit or node) and filters **in SQL on the server**, so a client engineer never receives data from another area.
- **Map state:** the API reads the last reading of each Falcon and SMAC (`last_*` tables, filled by Colbits' own triggers). It then computes one color per node: red = fuse open, yellow = no voltage, gray = no data for 9 h (Falcon) or 3 h (SMAC), green = normal. The thresholds are in the configuration, not in the code.
- **Data:** Colbits' schema is used **read-only**. Our own tables (users, scopes, token blacklist for logout) are separate. For development and the demo, the generator fills a copy of the schema with a realistic network and test scenarios.
- **Public demo:** the whole stack runs on Render. Every push to `develop` redeploys it, and the backend refreshes the demo data by itself with the generator.

**Technologies and why**

| Part | Technology | Why |
|---|---|---|
| Frontend | React + TypeScript (Vite), React Query, Leaflet, Recharts, i18next | Typed code. Leaflet is free and groups nearby poles. i18next gives the language switch (F-99). |
| Backend | NestJS + TypeScript (Node 24), TypeORM, @nestjs/jwt, bcrypt | Modules for auth, scope, nodes and devices. The same language as the frontend and the generator. |
| API docs | Swagger / OpenAPI | Generated from the code, so the docs stay up to date. It is a deliverable. |
| Database | PostgreSQL (Colbits uses TimescaleDB on PostgreSQL) | The same schema and triggers as production. |
| Test data | TypeScript CLI generator, seeded | The same seed always gives the same data. It induces faults, voltage loss, inactivity and more, without touching Colbits' database. |
| Quality | Jest, Vitest, GitHub Actions, oxlint, PR checks | CI runs on every pull request. Backend coverage is above 95 % (the target is 80 %). |
| Deploy | Docker Compose (local), Render (public link) | Anyone can open the link. No local setup is needed. |

**What changed since the Requirements & Design presentation**
- **Backend: FastAPI (Python) → NestJS (TypeScript).** Login, JWT, roles and CI were already working in NestJS, so changing would have meant losing that work. Now one language runs everywhere, and Swagger still gives the automatic API docs.
- **Generator: Python → TypeScript CLI**, for the same reason.
- **Password hashing: Argon2 → bcrypt** (what the NestJS code already used).
- **Two repositories → one monorepo with CI and a public deployment.**
- **Still to come:** the background fault engine (grouping openings within 5 minutes per fuse set), live updates within 20 s (US12; the map currently refreshes every 60 s) and the node detail panel (US11, Sprint 5).

*Speaker notes (B2, about 2.5 min, can be split in three)*
> "Our system has three parts in one repository: the web app, the REST API and a data generator. When a user logs in, the API gives them a token. Every request carries that token, and the API filters the data on the server, so a client engineer only receives their own part of the network.
> To color the map, the API reads the last reading of every device. Colbits' own triggers keep it up to date. Red means a fuse is open, yellow means no voltage, gray means the device stopped sending data, and green means everything is normal. We never write to Colbits' data. Our users and permissions live in separate tables.
> For technologies, we use React and TypeScript in the frontend and NestJS with TypeScript in the backend. So the whole team works in one language, and Swagger documents the API automatically. Tests run on every pull request, and the backend coverage is above 95 percent.
> The biggest change since our last presentation is the backend. We planned FastAPI, but the working code was already in NestJS, with login, roles and CI. So we kept it. Next, we will build the fault engine and bring the map updates under 20 seconds."

## 10. Main challenges · fila de Yeison

| Member | Challenge | How we dealt with it |
|---|---|---|
| Yeison | We could not test with Colbits' real data. Their database is read-only for us, and we needed known cases (open fuses, voltage loss, silent devices) to check every map color. | I built a seeded generator that copies Colbits' exact schema and can induce each scenario on purpose, with the expected result. Our tests use it to check the map colors on known data, and the public demo uses it to refresh its data by itself. |
