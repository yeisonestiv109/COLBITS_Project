# Requirements & Design Presentation, Part 2 (sections 8–14)

**Presenter:** Yeison Delgado · **Date:** Wed, Oct 7, 2026 · 10:27–10:49 Colombia / 17:27–17:49 Sweden
**Slides:** `Requirements_and_Design_Part2_v0.3.pptx`: 9 slides in the same style as Javier's deck (`COLBITS_Requirements_and_Design`). They go in as **slides 19–27**, between Javier's slide 18 ("How we will validate the requirements") and his "Thank you" slide.
**Target time:** about 7 minutes. Javier's part takes about 8.

---

## 0. What to do (in order)

1. Open Javier's deck (`COLBITS_Requirements_and_Design.pptx`, in Teams → 06_Presentations/2026-10-07_Requirements_and_Design) and my `Requirements_and_Design_Part2_v0.3.pptx`.
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

## 3. English speaker script, B2 level (also in the slide notes)

> Short sentences, simple words, technical only where it is needed. Total ≈ 7 min. Pause briefly between slides.

**Slide 19 · Non-functional requirements (section 8) (~60 s)**
Thank you, Javier. Now I will present the second half: requirements, backlog, mockups and architecture.

Let's start with the non-functional requirements. They are not about what the system does, but how well it does it. We have twenty-one, in four groups, and each one has a test.

First, speed. When something changes in the field, the map must show it in less than twenty seconds. And with fifty people using the system at the same time, history queries must answer in under two seconds.

Second, security. We can only read Colbits' data, never change it. Passwords are never saved as plain text, only as secure hashes. After five wrong passwords, the account is locked for fifteen minutes, and after fifteen minutes without activity, you must log in again.

Third, flexibility. All the important numbers, like the inactivity time or the five-minute window for faults, are in a configuration file. They can change without touching the code.

And fourth, testing. Automatic tests run every time we change the code, and they must cover at least eighty percent of the backend.

**Slide 20 · Initial product backlog (section 9) (~45 s)**
Here is our initial product backlog: twenty-nine user stories, each linked to the requirements. All are still "To Do", because coding starts now, in Sprint 3.

Priority five, in dark blue, is the base of the system: the data generator, login, the map and the colors of each node.

Priority four adds faults, history and export.

The last group has extras, like the chat in Spanish. It is optional, so if we run out of time, it is the first thing we drop.

On the chart you can see the effort is balanced: around twenty-five to thirty points per sprint.

And our goal for this sprint is simple: a user logs in and sees their part of the network on the map, with the right colors.

**Slide 21 · Initial mockups (section 10) (~45 s)**
These are our mockups, made in Figma. We based them on Colbits' reference screens and already reviewed them with the client.

The big one is the main screen: live monitoring. You see the map with each pole as a colored point. Red means a fuse is open, yellow means no voltage, gray means the device stopped sending data, and green means everything is OK. When you click a node, you see its details. On the right, there is the list of active faults, with the most serious ones first.

On the right, three more screens: user management, where the administrator decides who sees what; historical analysis, with charts and indicators; and the export center, to download reports in Excel, CSV or PDF.

**Slide 22 · System decomposition (section 11) (~60 s)**
Now, the architecture. This diagram shows how our system is built inside.

At the top are the two types of users. Both only use the web page, and every click sends a request to our backend. Inside the backend, we have six features: the map state, faults and alarms, history, export, user administration, and the optional assistant.

Below them, there are three shared parts that every feature uses. The first one checks who you are. The second one decides what you are allowed to see. And the third one is the only part that talks to the database.

At the bottom is the data. The gray one is Colbits' database: we only read it, never write to it. The green ones are ours: one for users and faults, and one with fake data for development and testing.

So, in short: Colbits' production data is always safe.

**Slide 23 · Sequence 1, secure live map (section 12) (~35 s)**
This sequence shows the most important flow: opening the live map safely.

Step one: the user logs in, and the backend gives them a token. Think of it as a digital pass that says who they are and what they can see.

Step two: every request from the map carries that pass.

Step three: the backend checks the pass and applies the filter on the server, not in the browser. So a user can never see data from another zone.

Finally, the backend reads the latest data, calculates the color of each node, and sends back only the nodes that this user is allowed to see.

**Slide 24 · Sequence 2, faults & reports (section 12) (~40 s)**
The second sequence shows how raw data becomes alarms and reports.

In the loop at the top, a background process runs every few seconds. It reads the new events from Colbits' database. Then it groups the fuses that opened within five minutes into one fault, and it decides if it is a single-, two- or three-phase fault. It also detects devices that stopped sending data.

The important detail: it saves the results in our own database, never in Colbits' database.

When a user asks for one month of history, the backend checks their permissions again and sends the data for the charts.

And when they export a report, the server calculates everything. So the same filters always give the same numbers.

**Slide 25 · Key architectural decisions (section 13) (~50 s)**
These are our six key decisions, and why we made them.

One: we build one single backend, divided into modules, and not microservices. We are six part-time students with fourteen weeks, so we need something simple to build and test.

Two: Colbits' data is read-only. Our own data lives in a separate space. The database itself blocks any writing, so it is not only a promise.

Three: the permission filter lives in one single place, on the server. One place is easier to protect, and easier to test.

Four: faults are calculated once, in the background. That way, every screen shows the same numbers, and the app stays fast.

Five: the map asks for new data every ten seconds. It is simpler than a live connection, and it is fast enough for the twenty-second goal.

And six: we use fake data, Docker and automatic tests from day one, so we never depend on the real database to work.

**Slide 26 · Open technical concerns (section 13) (~30 s)**
Of course, some risks are still open.

First, performance. We can't optimize Colbits' database, because we can't change it. So we will prepare the heavy calculations in our own database, and we will test the speed early.

Second, the real data has some unusual details. For example, one transformer reading is split across three tables.

Third, we must measure the twenty-second goal, not just assume it.

And finally, we still need to confirm a few small details with Colbits, like the exact colors in the mockups.

**Slide 27 · Technologies by component (section 14) (~45 s)**
Last, the technologies, one column per part of the diagram.

For the frontend, we use React with TypeScript. The map uses Leaflet, which is free and groups nearby points when you zoom out. Charts are made with Recharts.

For the backend, we use Python with FastAPI. One big advantage: it creates the API documentation automatically, and that is one of our deliverables. Login uses JWT tokens, and passwords are protected with Argon2. The fault engine is a background job, and reports are created with standard Python libraries.

For data, we use PostgreSQL with TimescaleDB, the same as Colbits. The fake data generator is also in Python. Tests run automatically on GitHub, and everything runs in Docker, so any team member can start the project with one command.

That's all from my side.

**Slide 28 · Thank you (Javier's slide)**
Thank you very much. We are happy to answer your questions.

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
