# Project: Askema

Askema turns a plain-language description of an app into a Postgres schema. It asks clarifying questions first, checks the AI's output with plain code, and proves the SQL runs in PGlite. It is open source and runs on each user's own computer, with a free local model through Ollama by default.

Status: design stage, no code yet. The source of truth for v1 is [docs/superpowers/specs/2026-11-09-askema-v1-design.md](docs/superpowers/specs/2026-11-09-askema-v1-design.md). If this file and the spec disagree, the spec wins; update this file.

## Architecture

```
describe ─► ask (AI) ─► generate (AI, JSON) ─► check (code) ─► run in PGlite (code)
                                   ▲                                   │
                                   └──────── fix, max 3 rounds ◄───────┘
                                                                       │
                                                ER diagram · SQL · decisions.md
```

| Part | Job | AI? |
| --- | --- | --- |
| `lib/ai` | One interface over Ollama and Claude | Yes |
| `lib/schema` | The schema JSON format, as zod schemas | No |
| `lib/checker` | JSON → Postgres SQL, and the design rules | No |
| `lib/sandbox` | Runs the SQL in a fresh PGlite database | No |
| `lib/export` | Mermaid diagram, `schema.sql`, `decisions.md` | No |
| `app/api/...` | One route that runs the whole loop on the server | Calls the parts above |

The whole loop runs on the server in one API route (spec: approach A). The checker and sandbox may move to the browser later, so keep them free of Node-only APIs where possible.

## Rules — STRICT

### Local first

- Everything must work with `AI_PROVIDER=ollama`, a local model and **no API key**. Never make Claude required.
- All model calls go through `lib/ai`. No other file imports the Anthropic SDK or calls Ollama directly.
- Prompts and the JSON format must work with mid-sized local models (around 14–20B), not only with Claude. If a change only works on Claude, it is not done.
- Never send a user's description anywhere except the provider they chose.

### AI proposes, code decides

- The model returns **JSON only**, validated with zod. It never returns SQL.
- SQL is produced by `lib/checker` from the validated JSON. Never execute SQL that came from the model.
- Design rules live in `lib/checker` as plain, unit-tested functions, not only in the prompt.

### Checker rules (v1)

1. Every table has a primary key.
2. Every foreign key points to a table and column that exist.
3. Every foreign key column has an index.
4. Table and column names are `snake_case` and unique.
5. Column types come from the allowed list.

A rule failure is a **problem** with a table, column and message, so it can be sent back to the model and shown to the user.

### Fix loop

- At most **3** fix rounds. Send the model the exact problems, not a summary.
- If problems remain after 3 rounds, return the schema **with** the remaining problems. Never hide them, never claim success.

### Decision notes

- Every clarifying answer becomes an entry in `decisions.md`: the question, the answer, and what it changed in the schema.

## Conventions

- TypeScript, strict mode. `pnpm` only.
- Validate everything that crosses a boundary (model output, request bodies) with zod.
- Tests with vitest. Every checker rule and exporter has unit tests; the AI layer is tested with recorded model responses, never live calls.
- Names follow the generated SQL: `snake_case` in SQL, `camelCase` in TypeScript.
- Keep v1 small. Anything listed under "Not in v1" in the spec goes to [TODO.md](TODO.md), not into the code.

## Effective prompts for future use

### Add a checker rule

> Add a checker rule to `lib/checker` that <rule>. Return problems in the existing format, add unit tests for a passing and a failing schema, and add the rule to the list in CLAUDE.md and the spec.

### Try a different local model

> Run the eval prompts against `AI_MODEL=<model>` with Ollama and compare with the current default: valid JSON rate, checker problems per schema, fix rounds used, and time per schema.

### Add an export format

> Add an exporter to `lib/export` that turns the validated schema JSON into <format>. Plain code only, no AI. Add snapshot tests and a download button.
