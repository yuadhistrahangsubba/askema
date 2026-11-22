# Architecture

_Planned. Askema has no code yet; this describes v1 as designed in [the v1 spec](superpowers/specs/2026-11-09-askema-v1-design.md) and will be updated as it is built._

## The pipeline

```
1 describe ─► 2 ask ─► 3 generate ─► 4 check ─► 5 run ─► 7 show and download
                            ▲                     │
                            └──── 6 fix (≤ 3) ◄───┘
```

| # | Step | Done by | Input → output |
| --- | --- | --- | --- |
| 1 | Describe | The user | A plain-language description of the app |
| 2 | Ask | AI | Description → 3–6 questions, each with multiple-choice answers and an "other" box |
| 3 | Generate | AI | Description and answers → schema JSON (validated with zod) |
| 4 | Check | Plain code | Schema JSON → Postgres SQL, plus a list of rule problems |
| 5 | Run | Plain code (PGlite) | SQL → success, or Postgres errors as problems |
| 6 | Fix | AI | Schema JSON and problems → corrected schema JSON, then back to step 4 |
| 7 | Show and download | Plain code | ER diagram, `schema.sql`, `schema.mmd`, `decisions.md` |

The AI is used in steps 2, 3 and 6 only. Everything that decides whether a schema is acceptable is plain, tested code.

## Parts

### `src/lib/ai`

One small interface, two implementations:

- **Ollama** (default): talks to a local model at `OLLAMA_HOST`. Uses Ollama's structured output so replies match the schema JSON format.
- **Claude**: used when `AI_PROVIDER=anthropic` and `ANTHROPIC_API_KEY` is set.

The rest of the code never knows which one it is talking to. The provider is chosen once, from `AI_PROVIDER`.

### `src/lib/schema`

The schema JSON format as zod schemas: tables, columns, types, primary keys, foreign keys, indexes, and a short note per table. The same zod schema is used to ask the model for structured output and to validate its reply.

### `src/lib/checker`

Turns validated schema JSON into Postgres SQL, then applies the design rules:

1. every table has a primary key
2. every foreign key points to a table and column that exist
3. every foreign key column has an index
4. table and column names are `snake_case` and unique
5. column types come from the allowed list

Each failure becomes a problem with the table, the column and a message.

### `src/lib/sandbox`

Creates a fresh PGlite database for each run and executes the SQL. Postgres errors become problems in the same format as checker problems, so the fix step treats them the same way.

### `src/lib/export`

Builds the outputs from the final schema JSON: a Mermaid ER diagram (`schema.mmd`), `schema.sql`, and `decisions.md` with one entry per clarifying answer.

### `src/app`

The Next.js pages and one API route. The route runs steps 2–7 on the server, so the provider setting and any API key stay out of the browser.

## Why it is built this way

| Choice | Reason |
| --- | --- |
| The model returns JSON, never SQL | JSON can be validated and checked by code; SQL from a model can't be trusted as-is |
| Rules in code, not only in the prompt | Code catches mistakes every time, on every model, including small local ones |
| A local model by default | Free for everyone, works offline, and descriptions stay on the user's computer |
| At most 3 fix rounds, problems always shown | Bounded time and cost, and no false claims of success |
| The whole loop in one server route | One place to test and debug in v1 |

## Later

Not part of v1; tracked in [TODO.md](../TODO.md): fake data and test queries from user stories, Drizzle, Prisma and Supabase migrations, an interactive diagram, other database engines.
