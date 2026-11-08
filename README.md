# Askema

**The AI that asks before it designs your database.**

Askema turns a plain-language description of your app into a database schema. It doesn't guess at what you left unclear. It asks the questions an experienced database designer would ask, then proves the schema works before handing it to you.

> Status: **idea stage**. There's no code yet. The idea, the research and the planned architecture are in [docs/idea.md](docs/idea.md).

## What makes it different

Most prompt-to-schema tools take one prompt, guess at everything you left unclear, and return SQL. Askema works in three steps:

1. **Asks first.** "Can an order ship to more than one address?" "Are users ever deleted, or only deactivated?" Every answer is saved as a short note explaining why the schema looks the way it does.
2. **Proves it works.** The schema runs in a real Postgres inside your browser ([PGlite](https://pglite.dev)), gets filled with realistic fake data, and is tested against your user stories turned into queries.
3. **Gives you files you can use.** You get an ER diagram, SQL, and migrations for the tools you already use, such as Drizzle, Prisma and Supabase.

## How it works

```
prompt ─► LLM asks questions ─► LLM returns the schema as JSON
                                       │
                                       ▼
                      checker (plain code): JSON → SQL + design rules
                                       │
                                       ▼
                      PGlite: run SQL, add fake data, run test queries
                                       │
                      errors? ─► sent back to the LLM to fix (loop)
                                       │
                                       ▼
                 ER diagram · SQL · migrations · decision notes
```

There's no custom-trained model. Askema uses an existing LLM through its API. The work is in the rules, the checker, the test queries and the evals that sit around it.

## Contributing

Ideas and feedback are welcome. Open an issue, especially on the open questions in [docs/idea.md](docs/idea.md#open-questions).
