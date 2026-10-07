# Requirements & Design Presentation, Part 2 (sections 8–14)

**Presenter:** Yeison Delgado · **Date:** Wed, Oct 7, 2026 · 10:27–10:49 Colombia / 17:27–17:49 Sweden
**Slides:** `Requirements_and_Design_Part2_v0.2.pptx`: 9 slides in the same style as Javier's deck (`COLBITS_Requirements_and_Design`). They go in as **slides 19–27**, between Javier's slide 18 ("How we will validate the requirements") and his "Thank you" slide.
**Target time:** about 7 minutes. Javier's part takes about 8.

---

## 0. What to do (in order)

1. Open Javier's deck (`COLBITS_Requirements_and_Design.pptx`, in Teams → 06_Presentations/2026-10-07_Requirements_and_Design) and my `Requirements_and_Design_Part2_v0.2.pptx`.
2. In mine, select the 9 slides in the left panel and copy them (Ctrl+C).
3. In Javier's deck, click between slide 18 and the "Thank you" slide, then paste with **Keep Source Formatting** (Ctrl+V, then the clipboard icon → "Keep source formatting"). The speaker notes come along.
4. **Slide 11 of Javier's deck (Sprint 2 · Team Workload):** your row is empty. Type: Documentation **2**, Presentation **2**, Research **3**, Implementation **0**, Testing **1**, Scrum **1** (Meetings is already 5). Fill in the same row in `COLBITS_Sprint_Documentation_Template.pptx`.
5. Check the "Thank you" slide number (it currently shows 29; after pasting it should be 28).
6. Save and upload to Teams as `Requirements_and_Design_v1.0.pptx`.
7. Tell Mahmoud and Javier, in one message:
   - Slide 24 uses a **corrected Sequence 2**. In the original, step 4 wrote into Colbits' TimescaleDB, which contradicts NF-03. Now it writes to our tables.
   - Sections 13 and 14 define the stack as **Python + FastAPI** for the backend. If the backend repo already uses something else, change slide 27 and the matching row on slide 25.

---

## 1. Content for the team document: section 13 (architectural decisions)

*Paste this under "▸ 13. Most important architectural design decisions" in `Presentacion_ReqDesign_Colbits_1.docx`.*

**Decisions and why**

1. **Modular monolith.** One REST API with feature modules (export, history, faults & alarms, network state, user administration, assistant) and three shared modules (authentication, scope & access control, data access).
   - *Why:* six part-time developers and 14 weeks. One deployable, one test suite and one CI pipeline. The module boundaries mirror the decomposition diagram, so a module can be split out later if needed.
   - *Rejected:* microservices. They bring operational overhead, distributed data and cross-service auth for no benefit at this scale.
2. **Colbits' data stays read-only. Our state lives in a separate schema.** Users, scope assignments, grouped faults, inactivity alarms and device-reassignment history go in a `platform` schema we own.
   - *Why:* NF-03 and assumption SUP-03. Production must not be modified, and a database role with SELECT-only grants on Colbits' tables makes that testable.
   - *Rejected:* adding tables or columns to Colbits' schema.
3. **Scope is enforced once, on the server, in the data-access layer.** Every query goes through a single scope filter derived from the token's user.
   - *Why:* RF-SEG-08 says no data may leak, not even through counts or totals. With one choke point there is one place to test (NF-14).
   - *Rejected:* filtering in each endpoint, or hiding data in the frontend.
4. **One fault rule, run by a background engine.** A scheduled job polls new events, groups openings per fuse set within the configurable window (5 min), flags inactivity and stores the results in our schema. Map, fault list, history, export and assistant all read the same results.
   - *Why:* every screen shows the same numbers (RF-HIS-04, RF-IA-05). Heavy work stays off the request path (NF-02). Inactivity is detected even when no new data arrives (RF-MAP-07).
   - *Rejected:* recomputing faults on every request.
5. **The live map uses short polling (≤ 10 s).**
   - *Why:* it meets the 20 s target (NF-01, RF-MAP-10) with the least complexity. The "last refresh" timestamp and the stale-data warning are trivial to show (RF-MAP-11). It also works through any proxy.
   - *Rejected for now:* WebSockets and SSE. We will revisit them if load tests show polling is too expensive.
6. **Seeded synthetic data generator, Docker and CI from day one.** The generator is a standalone command-line tool that fills a separate dev/test database with the real schema. The whole stack runs with Docker Compose, and GitHub Actions runs both test suites on every pull request.
   - *Why:* development and tests are reproducible without real data (RF-GEN-07, NF-16, NF-18, NF-21).
   - *Rejected:* developing and testing against the shared database copy.

Also decided, because the requirements already imply it:
- Reports and indicators are computed on the server, with a traceability header (RF-INF-03/04).
- The assistant only maps a question to one of the predefined, parameterised queries. It never generates SQL (RF-IA).

**Remaining technical concerns**

- **Performance on a database we cannot change.** We cannot add indexes or continuous aggregates to Colbits' database. We will pre-aggregate in our own schema and run the NF-02 load test early, in Sprint 4.
- **Real-data quirks:**
  - SMAC readings are split across 3 tables and must be joined with a time tolerance (NF-12).
  - Legacy repetition fuse IDs exist in the history, but the generator does not produce them. That case is not covered by the automated tests, which is an open risk the client already acknowledges.
- **End-to-end 20 s latency.** Poll interval, engine cycle and database time must add up to less than 20 s. We will measure it with the NF-01 test, not assume it.
- **Open points with Colbits:**
  - The mockup legend shows amber, and five states, while the written spec says yellow. To be confirmed.
  - The database copy has anonymised coordinates, and they must stay coherent per circuit (SUP-02).

---

## 2. Content for the team document: section 14 (technologies per component)

*Paste this under "▸ 14. Languages, frameworks, and databases — mapped to components".*

| Component (decomposition diagram) | Technology | Why |
|---|---|---|
| Web frontend | React + TypeScript (Vite), Atomic Design structure | Reusable components across map, history and admin screens; typed API contracts |
| (map) | Leaflet + react-leaflet + Leaflet.markercluster | Open-source map with marker clustering (RF-MAP-01); no API key or cost |
| (charts) | Recharts | Trend, distribution and ranking charts in React |
| (live refresh) | TanStack Query with 10 s polling | Caching, retries and last-refresh timestamp (RF-MAP-10/11) |
| Backend REST API, the six feature components | Python 3.12 + FastAPI + Pydantic | Generates OpenAPI documentation automatically (a contract deliverable); validated configuration for all thresholds (NF-07–12); same language as the generator and the data tooling |
| Authentication | JWT (PyJWT) + Argon2 (argon2-cffi) | Token per session, salted adaptive hashes, lockout and 15-min idle expiry (NF-04–06) |
| Scope & access control + Data access | SQLAlchemy 2 + psycopg, SELECT-only database role on Colbits' tables | A single choke point for the scope filter; parameterised SQL; read-only enforced by the database (NF-03) |
| Faults & alarms (background engine) | APScheduler job inside the API process | Periodic grouping and inactivity checks without an extra message broker |
| Export & reports | openpyxl (XLSX), Python csv, WeasyPrint (PDF) + matplotlib (charts) | Files generated on the server, deterministic (RF-INF-04) |
| Assistant (optional) | Catalogue of predefined, parameterised queries + Spanish intent matching | No free-form SQL; reuses the History and Faults services |
| Existing Colbits data | PostgreSQL + TimescaleDB (read-only) | Already in production |
| Platform-managed data | PostgreSQL schema `platform` + Alembic migrations | Our own tables, with a versioned schema |
| Development / test database | PostgreSQL + TimescaleDB in Docker | Same engine and schema as production |
| Synthetic data generator | Python CLI (Typer) + seeded NumPy random generator | Reproducible from a seed (RF-GEN-07); runs without the UI (RF-GEN-08) |
| Testing & CI | pytest + httpx + pytest-cov (API), Playwright (UI), GitHub Actions | NF-13–21: ≥ 80 % API coverage, runs on every PR to main |
| Packaging | Docker + Docker Compose | One command brings up a clean environment (NF-21) |

---

## 3. English speaker script (also in the slide notes)

> Speak calmly. Targets in brackets; total ≈ 7 min. If short on time, trim slides 19 and 26 first.

**Slide 19 · Non-functional requirements (section 8) (~60 s)**
Thank you, Javier. I'll take the second half: non-functional requirements, backlog, mockups, and then the architecture.
Besides what the system does, we have twenty-one non-functional requirements. Each one has a measurable acceptance test. They fall into four groups.
Performance: a state change in the database must reach the map within twenty seconds, and with fifty concurrent users, ninety-five percent of history queries must answer in under two seconds.
Security: Colbits' telemetry and alarm tables are read-only for us. We prove it with a test that tries to write and expects a permission error. Passwords are stored as Argon2 or bcrypt hashes, five failed logins lock the account for fifteen minutes, and fifteen idle minutes end the session.
Flexibility: every business threshold, like the inactivity times or the five-minute grouping window, lives in configuration, so changing it needs no code.
Testability: API and interface suites that run on generated data, including an explicit test that no user can see data outside their scope. They run in CI on every pull request, with at least eighty percent coverage.

**Slide 20 · Initial product backlog (section 9) (~45 s)**
This is the initial state of the product backlog: twenty-nine user stories, each traced to its requirements, and all still To Do because coding starts in Sprint 3.
We ordered them by priority and spread them over Sprints 3 to 7.
Priority 5 is the foundation: the data generator, login, scope, the map and the node colors, in Sprints 3 and 4.
Priority 4 adds filters, faults and alarms, user management, history and export.
At the bottom are the extras, including the Spanish chat, which is optional and the first thing we would drop.
The chart shows the estimated effort per sprint, around twenty-five to thirty points each, with Sprint 5 a bit lighter.
Our Sprint 3 goal is concrete: a user logs in and sees the nodes in their scope on the map, colored by fuse state, using generated data.

**Slide 21 · Initial mockups (section 10) (~45 s)**
These are our mockups in Figma, version 3. We based them on Colbits' own reference screens and already reviewed them with the client. Their feedback is tracked as tasks in Jira.
The main screen is live grid monitoring: the map with the fuse and transformer layers, each node colored by state; the node detail with its fuse groups; and on the right, the active faults, ordered by the number of affected customers.
On the right side: user management, where the administrator creates consultation users and assigns their scope; historical analysis, with date presets, filters, KPIs and charts; and the export center, which shows the traceability header that goes into every XLSX, CSV and PDF file.
The optional assistant opens as a chat panel over these screens.

**Slide 22 · System decomposition (section 11) (~60 s)**
Now the architecture. This is the inside of our backend.
A user, either the administrator or a consultation user, works only in the web frontend, and every action becomes a REST request to one of these six features: export, history, faults and alarms, network state, user administration, and the optional assistant.
Each feature relies on three shared parts: one checks who is calling, one limits what that user may see, and one reaches the data.
Features also reuse each other. History uses the same fault rule as the live alarms, so the numbers are identical on every screen.
The data comes from two places: Colbits' existing database, which we only read, shown in gray, and our own data, like users, their scope and device history.
For development and demos, our generator fills a separate database with the same structure, so we never touch Colbits' data.

**Slide 23 · Sequence 1, secure live map (section 12) (~35 s)**
The first sequence is the core flow: secure live monitoring.
The operator logs in, and the API checks the password hash and returns a token carrying the role and the scope.
When the map opens, every request carries that token. The API validates it and applies the scope filter on the server before any query runs.
Then it reads the latest Falcon and SMAC telemetry in read-only mode, computes the color of each node for the fuse and transformer layers, and returns only the nodes this user is allowed to see.

**Slide 24 · Sequence 2, faults & reports (section 12) (~40 s)**
The second sequence shows how raw telemetry becomes alarms and reports.
In the background, a fault engine polls Colbits' database every few seconds. It groups fuse openings on the same fuse set within the five-minute window into single-, two- or three-phase faults, and flags devices that stopped reporting.
The results are written only to our own tables, never to Colbits' database.
When an operator asks for thirty days of history, the API checks the scope again, combines the time-series data with our fault history, and the frontend draws the charts.
For an export, the server computes the indicators itself and adds a traceability header, so two exports with the same filters always give the same numbers.

**Slide 25 · Key architectural decisions (section 13) (~50 s)**
These are our most important design decisions.
One: a modular monolith, a single API with feature modules, instead of microservices. With six part-time developers and fourteen weeks, one deployable and one test suite is the realistic choice.
Two: Colbits' data stays read-only, and our state lives in a separate schema, enforced by a database role that can only SELECT.
Three: scope is enforced in one place, the data-access layer on the server, so nothing can leak, not even through a count or a total.
Four: one fault rule, run by a background engine, so every screen shows the same numbers and queries stay fast.
Five: the live map uses short polling instead of WebSockets. It meets the twenty-second target and makes the last-refresh warning easy.
And six: the seeded generator, Docker and CI from day one, so all development and testing works without real data.

**Slide 26 · Open technical concerns (section 13) (~30 s)**
Some technical concerns remain open.
Performance: we cannot add indexes to Colbits' database, so we will pre-aggregate in our own schema and run the load test early.
Real-data quirks: SMAC readings are split across three tables and must be joined with a time tolerance, and old repetition fuse IDs exist in history that the generator does not produce.
The twenty-second end-to-end latency must be measured, not assumed.
And a few points are still to confirm with Colbits, such as the amber versus yellow color in the mockups.

**Slide 27 · Technologies by component (section 14) (~45 s)**
Finally, the technologies, mapped to the components you saw in the diagram.
The web frontend is React with TypeScript, organized with Atomic Design. The map uses Leaflet with marker clustering, the charts use Recharts, and the live refresh is a ten-second polling query.
The backend is Python with FastAPI. It generates the API documentation automatically, which is one of our deliverables. Authentication uses JWT and Argon2. Data access uses SQLAlchemy with a read-only database role. The fault engine is a scheduled background job, and exports use openpyxl and WeasyPrint.
On the data side: PostgreSQL with TimescaleDB, the same engine as Colbits, plus our own schema with versioned migrations. The generator is a Python command-line tool with a fixed seed. Tests use pytest and Playwright, run in GitHub Actions, and everything is packaged with Docker.

**Slide 28 · Thank you (Javier's slide)**
That's our requirements and design. Thank you — we're happy to take your questions.

---

## 4. Q&A preparation (7 min)

**Why a monolith and not microservices?**
Six part-time developers and fourteen weeks. Microservices would cost us deployment, networking and distributed-data work with no benefit at this load. The modules have clear boundaries, the same as in the diagram, so we could split one out later.

**How do you guarantee a user never sees data outside their scope?**
The scope comes from the token, never from the client, and it is applied in one place: the data-access layer, which every query goes through. That includes counts and totals. We have an explicit test (NF-14): if we remove the filter from any service, the test must fail.

**How do you meet the 20-second update target?**
The frontend polls every 10 seconds and the background engine runs every few seconds, so the worst case stays under 20. We will measure it with the NF-01 test. Inactivity is also covered: the engine turns a node gray when the threshold passes, even if no new data arrives.

**Why polling instead of WebSockets?**
It is simpler, it works behind any proxy, it already meets the target, and it makes the "last refresh" warning trivial. If load tests show polling is too expensive, we will move to server-sent events.

**Where do you store faults if Colbits' database is read-only?**
In our own `platform` schema: grouped faults, inactivity alarms, users, scope assignments and device-replacement history. Colbits' tables are only read, and the database role cannot write to them.

**How do you test without the real data?**
With the synthetic generator. It uses the same schema and a fixed seed, and it can induce each scenario: single-, two- and three-phase openings, openings around the 5-minute window, voltage loss, inactivity gaps and so on. Same seed, same dataset, so the tests are reproducible.

**Why Python and FastAPI?**
One language for the API, the fault engine, the generator and the tests. FastAPI generates the API documentation automatically, which is a deliverable. Python also has mature libraries for time-series work and for producing XLSX and PDF files.

**How does the assistant work, and is it safe?**
It is optional, priority 3. It maps a Spanish question to one of the predefined, parameterised queries and reuses the same services, so the scope and the numbers are the same as in the views. It never writes SQL. If a question is outside the catalogue, it says so instead of guessing.

**What happens when a device is replaced?**
History is resolved by the stable identifier of the fuse or transformer (`custom_id`), not by the hardware ID, and we keep our own reassignment history. The series stays continuous (RF-HIS-12).

**What is your biggest risk?**
Performance on the real database, which we cannot modify: fifty users with p95 under two seconds. We will pre-aggregate in our schema and run the load test early in Sprint 4. The second risk is time: the assistant is the first thing we drop if needed.

---

## 5. Sprint 2 documentation: Team Workload (your row)

These are the values from your Slack reply of Oct 4. Put them in the Yeison row of `COLBITS_Sprint_Documentation_Template.pptx` (slide 9):

| Meetings | Documentation | Presentation | Research | Implementation | Testing | Scrum |
|---|---|---|---|---|---|---|
| 5 | 2 | 2 | 3 | 0 | 1 | 1 |

Total: 14 h. The other members have 13–15 h.
