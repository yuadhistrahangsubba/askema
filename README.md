# Askema

**The AI that asks before it designs your database.**

Askema turns a plain-language description of your app into a Postgres schema. It doesn't guess at what you left unclear. It asks the questions an experienced database designer would ask, checks the result with plain code, and proves it runs in a real Postgres before handing it to you. It is open source and runs on your own computer, with a free local model by default.

> Status: **design stage**. There's no code yet. The v1 design is being written in [docs/superpowers/specs/](docs/superpowers/specs/2026-11-09-askema-v1-design.md), the research is in [docs/idea.md](docs/idea.md), and plans are in [TODO.md](TODO.md). Sections below marked _planned_ describe v1 as designed and may change.

## What it does

| Step | What happens | Uses AI? | In v1? |
| --- | --- | --- | --- |
| Describe | You type what your app is: "clinic app: patients, doctors, appointments" | No | Yes |
| Ask | 3–6 clarifying questions with multiple-choice answers: "Can one appointment involve more than one doctor?" | Yes | Yes |
| Generate | The schema as JSON in a fixed format, validated with zod | Yes | Yes |
| Check | Plain code turns the JSON into SQL and applies design rules (primary keys, foreign keys and their indexes, naming, allowed types) | No | Yes |
| Run | The SQL runs in a fresh [PGlite](https://pglite.dev) database to prove Postgres accepts it | No | Yes |
| Fix | Problems from Check and Run go back to the AI, for at most 3 rounds. Anything still wrong is shown to you, never hidden | Yes | Yes |
| Show and download | An ER diagram (Mermaid), `schema.sql`, and `decisions.md`, which explains what each of your answers changed | No | Yes |
| Prove | Realistic fake data, and your user stories turned into test queries | Partly | Later |
| Export | Migrations for Drizzle, Prisma and Supabase | No | Later |

## How it works

```
your description ─► AI asks questions ─► AI returns the schema as JSON
                                                │
                                                ▼
                               checker (plain code): JSON → SQL + design rules
                                                │
                                                ▼
                               PGlite: run the SQL in a real Postgres
                                                │
                               problems? ─► back to the AI to fix (max 3 rounds)
                                                │
                                                ▼
                               ER diagram · SQL · decision notes
```

There's no custom-trained model. Quality comes from the design rules, the checker, the sandbox and the evals around the model. More detail is in [docs/architecture.md](docs/architecture.md).

### Which AI it uses

| Provider | Cost | Where your descriptions go | Set with |
| --- | --- | --- | --- |
| A local open model through [Ollama](https://ollama.com) (default) | Free | Nowhere: everything stays on your computer | `AI_PROVIDER=ollama` |
| Claude | Paid per request, with your own API key | Anthropic's API | `AI_PROVIDER=anthropic` |

## Stack _(planned)_

- **Next.js** (App Router) and **TypeScript**
- **Tailwind CSS** and **shadcn/ui**
- **zod** for the schema JSON format
- **PGlite** to run the generated SQL
- **Mermaid** for the ER diagram
- **Ollama** (default) or the **Anthropic SDK** for the AI
- **vitest** for tests, **pnpm** as the package manager

## Getting started _(planned)_

Prerequisites: Node.js (current LTS), pnpm, and [Ollama](https://ollama.com).

```bash
# 1. Download a local model (see docs/development.md for choosing one)
ollama pull gpt-oss:20b

# 2. Get Askema
git clone https://github.com/yuadhistrahangsubba/askema.git
cd askema
pnpm install

# 3. Start it, then open http://localhost:3000
pnpm dev
```

The defaults work without a `.env` file. To change the model or use Claude, copy `.env.example` to `.env`. Full setup is in [docs/development.md](docs/development.md).

## Configuration _(planned)_

All settings are optional.

| Area | Variables |
| --- | --- |
| Provider | `AI_PROVIDER`: `ollama` (default) or `anthropic` |
| Model | `AI_MODEL`: the model name for the chosen provider, e.g. `gpt-oss:20b` |
| Ollama | `OLLAMA_HOST`: defaults to `http://localhost:11434` |
| Claude | `ANTHROPIC_API_KEY`: only needed when `AI_PROVIDER=anthropic` |

## Scripts _(planned)_

| Command | What it does |
| --- | --- |
| `pnpm dev` | Run Askema locally on http://localhost:3000 |
| `pnpm build` / `pnpm start` | Build and run a production version |
| `pnpm test` | Unit tests (vitest) |
| `pnpm lint` | ESLint |

## Project layout _(planned)_

```
src/
├── app/                 # Next.js pages and the one API route that runs the loop
└── lib/
    ├── ai/              # provider setting: Ollama or Claude behind one interface
    ├── schema/          # the schema JSON format (zod)
    ├── checker/         # JSON → SQL, and the design rules
    ├── sandbox/         # runs the SQL in PGlite
    └── export/          # Mermaid diagram, schema.sql, decisions.md
docs/
├── idea.md              # the problem, competitors and research
├── architecture.md      # how the parts fit together
├── development.md       # local setup and choosing a model
└── superpowers/specs/   # design specs, one per feature
```

## Conventions

The full rules are in [CLAUDE.md](CLAUDE.md). The most important:

- **Local first.** Everything must work with a free local model and no API key. Claude is an option, never a requirement.
- **AI proposes, code decides.** The model only returns JSON. Turning it into SQL, and checking it, is plain tested code.
- **Nothing hidden.** If problems remain after the fix rounds, show them next to the schema.
- **Every answer leaves a note.** Each clarifying answer becomes an entry in `decisions.md`.

## Further documentation

- [docs/idea.md](docs/idea.md): the problem, existing tools and research
- [docs/architecture.md](docs/architecture.md): the pipeline and its parts
- [docs/development.md](docs/development.md): setup, choosing a local model, using Claude
- [docs/superpowers/specs/](docs/superpowers/specs/2026-11-09-askema-v1-design.md): the v1 design
- [TODO.md](TODO.md): roadmap

## Contributing

Ideas and feedback are welcome. Open an issue, especially on the open questions in [docs/idea.md](docs/idea.md#open-questions). Askema has no code yet, so the most useful contributions right now are comments on the design.

## License

[MIT](LICENSE)
