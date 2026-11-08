# Askema: idea and research

_Written 2026-11-08. Idea stage, nothing built yet._

## The problem

Asking an AI for a database schema is easy. Getting one you can trust is not. Today's tools turn a single prompt into tables and foreign keys, but they:

- **guess** wherever the prompt is unclear, without telling you they guessed
- rarely add the parts a real system needs: indexes, constraints, soft deletes, audit columns, multi-tenant setup
- don't **test** the result: nothing checks that the schema can answer the questions your app will ask
- return plain SQL, not migrations for the tools developers actually use

## What already exists

Checked in November 2026. Some details come from third-party listings and still need confirming against each project's own docs.

| Project | What it does | Open source |
|---|---|---|
| database.build (Supabase, previously postgres.new) | Chat → Postgres in the browser → ER diagram | Yes |
| ChartDB | ER editor; AI builds a diagram from a prompt and converts SQL between database types | Yes, self-hostable; AI needs your own key |
| Liam ERD / Liam DB | ER diagrams from schema files; Liam DB adds an AI agent that designs and reviews schemas | Yes |
| drawDB, StackRender, Grimoire | Visual schema editors with no built-in AI | Yes |
| Visual Paradigm, Taskade, dbdiagram AI, Xano | Prompt-to-ERD | No (commercial) |

Text-to-SQL tools such as Vanna and WrenAI turn questions into queries against a database that already exists. That's a different problem from designing a schema.

## Where Askema fits

What competitors don't do yet:

1. **Ask questions instead of guessing**, and keep a decision note for each answer.
2. **Prove the schema works** by running it in a real Postgres with fake data, testing it against user stories turned into queries, and flagging slow queries.
3. **Export migrations for real tools** (Drizzle, Prisma, Supabase), not just raw SQL.

Ideas set aside for later:

- turning a spreadsheet or CSV into a database
- planning safe changes to a database that already exists
- ready-made patterns for one sector or country

## Planned architecture

| Part | Job | Uses AI? |
|---|---|---|
| Question step | Finds unclear points in the prompt and asks about them | Yes |
| Generator | Returns the schema as JSON in a fixed format (structured output) | Yes |
| Checker | JSON → SQL; rule checks: every table has a primary key, foreign keys have indexes, names are consistent | No |
| Sandbox | PGlite in the browser: runs the SQL, adds fake data, runs test queries | No |
| Fix loop | Sends checker and sandbox errors back to the generator, up to N rounds | Yes |
| Exporters | ER diagram, SQL, Drizzle, Prisma, decision notes | No |

### No custom model training

Askema calls an existing LLM through its API. Quality comes from:

- a **system prompt** containing the design rules
- a handful of **good example schemas** included in the prompt
- **structured output**, so the model always returns valid JSON in the expected format
- the **checker and sandbox**, which catch mistakes the model makes
- **evals**: 30–50 test prompts with expected results, rerun after every prompt change

Fine-tuning is worth looking at only after there are thousands of real, approved schemas and API cost has become a problem.

### Rough cost

An estimate, not a measurement: about 5–10K input tokens and 5K output tokens per schema, which comes to roughly $0.10–0.15 per schema on a top-tier model, plus fix-up rounds. Simple steps can go to a cheaper model.

### Planned stack

TypeScript, the Anthropic SDK, PGlite, and React Flow or Mermaid for the diagrams.

## Open questions

- **Who is it for first?** Developers starting a new app, non-technical businesses, teams with an existing database, or one sector or country?
- Web app, CLI, or both?
- Which export formats come first?
- Open-source core with a hosted version, or fully open source?
