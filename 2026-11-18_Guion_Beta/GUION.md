# Guion Beta · tu parte (de la diapositiva 16, LIVE DEMO, hasta la 35)

**Lo de las "dos ideas":** el número real es **unas 4 ideas a la vez** (memoria de trabajo, Cowan 2001; el famoso "7" de Miller ya se considera alto). En una charla se olvida rápido, así que la regla práctica es:
- **Una idea principal por diapositiva**, con máximo **3 puntos de apoyo**.
- **Frases cortas**: una idea por frase.
- **Avisar qué viene** ("Now, the risks") y **cerrar con el resultado** (un número o un "done").

Así está escrito el guion. Lee despacio, haz una pausa en cada línea en blanco y mira al público al decir los números.

**Credenciales** (las compartió Sofía el 6 nov en #colbits):
- Admin, ve toda la red: `admin@colbits.com` / `adminpassword`
- Cliente, ve menos nodos: `client@colbits.com` / `adminpassword`
- Página: https://dev.colbits.com.co (datos generados)

---

## 16 · LIVE DEMO (unos 5–6 min)

**Si es en vivo** (entras como admin; Mahmoud hace clic o lo haces tú):

> Now, the live demo. This is our app on Colbits' own server, with generated data.
>
> Our goal is simple. When a fuse blows, an engineer sees where it is, how many phases are open, and how many customers lost power. All of this in less than 20 seconds.
>
> I log in as the administrator. Here is the map. Green means everything is fine.
>
> Now I open the scenario panel. I choose a node, and I press "Open fuse".
>
> The backend writes the same data that a real device sends. We wait a few seconds…
>
> And there it is. The node turns red. The fault appears in the list on the right. It says single-phase, and it shows the customers affected.
>
> I click the node. Here we see the circuit, the substation, and the fuse that is open.
>
> Now I drop a phase on a transformer. The transformer turns yellow. One phase is down.
>
> Now I restore everything. The node goes back to green. The fault moves to "Resolved", with its duration.
>
> Last thing. A client user logs in and sees only their part of the network. Each client sees only their own data.
>
> So in less than 20 seconds, we know where the fault is, how serious it is, and who is affected.

**Si ponen el video de Mónica:** usa las mismas frases, pero con "here we see" en vez de "I press":
> Here the administrator opens the scenario panel and opens a fuse. … Here the node turns red, and the fault appears in the list. …

---

## 17 · Risks (1 min)

> Now, the risks.
>
> We foresaw three risks. First, no access to real data. We built a generator with Colbits' exact schema, so we were never blocked. Second, too much scope. This one happened: in Sprint 5 we closed 16 of 32 points, so we re-ranked the backlog. Third, client data exposure. That did not happen, because we only use generated data.
>
> We also found new risks. The biggest ones were the six-hour time difference, a new client request in the middle of a sprint, and late answers that changed some rules. For each one, we found a solution: fixed meetings, a new user story, and one shared mapping document.

## 18 · Team performance (45 s)

> Now, how the team works.
>
> At the start, the work was uneven. In Sprint 3, one person made 41 of 53 commits. Now all six of us commit every sprint.
>
> Reviews also improved. Now every pull request needs one approval from another member. We went from 8 percent to 77 percent of pull requests reviewed.
>
> Our next goal is to merge smaller pull requests earlier in the sprint.

## 19 · Other experiences (45 s)

> Three experiences stand out.
>
> First, working with a real client. Colbits answered all our questions, and every answer is now in the specification.
>
> Second, deploying on the client's own server. It was our first time, and the guide is now in the repository.
>
> Third, working with AI. AI helps us split tasks and write code and tests. The team always makes the final decision.

## 20 · Total working hours (30 s)

> Our total work is 528 and a half hours, from Sprint 2 to Sprint 5.
>
> Implementation is the largest part, with 181 and a half hours. That is about one third of the total.
>
> Each member also spent about 20 hours in meetings.

## 21–28 · Sprint 4 (40 s; pasa las diapositivas sin leer las tablas)

> Now, the sprint documents. Sprint 4 ran from October 19 to November 1.
>
> The goal was the Alpha prototype: the transformer layer, active faults, node detail and live updates.
>
> We planned eight user stories and 37 points, and we finished all of them. Here is the burndown chart, and here are the hours per member.

## 29–34 · Sprint 5 (40 s)

> Sprint 5 ran from November 2 to 15.
>
> The goal was to help Colbits follow and test the network: filters, alarms with their duration, device alerts, user administration and the demo panel.
>
> We finished 6 of 9 stories and 16 of 32 points. Three stories moved to Sprint 6, and most of their work is already done. Here are the burndown chart and the hours.

## 35 · Thank you (10 s)

> That is our Beta. Thank you very much. We are happy to answer your questions.

---

## Posibles preguntas y respuesta corta

- **Why polling and not websockets?** "Polling every 15 seconds already meets the 20-second target, and it is simpler to keep running."
- **Is the demo safe for real data?** "Yes. Demo mode works only for the administrator, only with a special setting, and only on a database with generated data."
- **What is left for the final version?** "History, reports, export, and finishing the three stories from Sprint 5. The optional chat assistant is the first thing we drop if time is short."
