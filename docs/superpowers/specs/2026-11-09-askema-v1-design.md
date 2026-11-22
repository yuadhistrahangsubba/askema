# Askema v1: design

_Status: in progress. Written one section at a time, each approved before the next._

## Decisions so far

| Decision | Choice | Why |
|---|---|---|
| Form | Open-source web app that people clone and run on their own computer (`pnpm install`, `pnpm dev`) | Free for everyone, nothing to host, and anyone can read and change the code |
| Stack | Next.js (App Router), TypeScript, Tailwind, shadcn/ui, zod, vitest, pnpm | Same stack as the author's other projects |
| AI | A free open model running locally through [Ollama](https://ollama.com) by default; Claude as an option for people who add their own key to `.env` | Costs nobody anything, works offline, and descriptions of people's apps never leave their computer |
| Where the loop runs | On the server, in one API route (approach A) | One place to test and debug; the checker and sandbox can move to the browser later |
| Database engine | Postgres only | PGlite is Postgres; one dialect keeps v1 small |
| AI provider setting | `AI_PROVIDER=ollama` (default) or `anthropic`, plus `AI_MODEL` | One small interface for both, so the rest of the code never knows which model it is talking to |
| License | MIT | Lets anyone use, change and share Askema, including at work |

## 1. Scope

### In v1

1. **Describe.** The user types what the app is ("clinic app: patients, doctors, appointments").
2. **Ask.** The AI asks one round of 3–6 clarifying questions, each with multiple-choice answers plus an "other" box. Example: "Can one appointment involve more than one doctor?"
3. **Generate.** The AI returns the schema as JSON in a fixed format (tables, columns, types, keys, indexes, a note per table), validated with zod.
4. **Check.** Plain code turns the JSON into Postgres SQL and applies these rules:
   - every table has a primary key
   - every foreign key points to a table and column that exist
   - every foreign key column has an index
   - table and column names are snake_case and unique
   - column types come from an allowed list
5. **Run.** The SQL runs in a fresh PGlite database to prove that Postgres accepts it.
6. **Fix.** Problems from steps 4–5 go back to the AI, for at most 3 rounds. If problems remain after that, the user sees the schema along with the remaining problems, so nothing is hidden.
7. **Show.** An ER diagram drawn with Mermaid, plus the SQL.
8. **Explain.** Each answer from step 2 becomes a decision note (`decisions.md`): the question, the answer, and what it changed in the schema.
9. **Download.** `schema.sql`, `schema.mmd` (the diagram source) and `decisions.md`.

### Not in v1 (stays in TODO.md)

- fake data and test queries built from user stories
- Drizzle, Prisma and Supabase migration exports
- an interactive diagram (React Flow) or a Postgres playground in the browser
- accounts, or saving past sessions (a session lives only in the open page)
- changing an existing database, importing spreadsheets, other database engines
- a hosted public version. Askema runs on each person's own computer, so there is no shared key to protect. If a hosted version is ever added, it needs a rate limit first.
