# TODO

Future plans for Askema, roughly in order. The reasoning behind them is in [docs/idea.md](docs/idea.md).

## Decide first

- [ ] Pick the first audience: developers starting a new app, non-technical businesses, teams with an existing database, or one sector or country
- [x] Web app, CLI, or both? **Web app that people clone and run locally**
- [x] Open-source core with a hosted version, or fully open source? **Fully open source, run on your own computer**
- [x] Add a license: **MIT**
- [ ] Confirm what competitors can do today (database.build, ChartDB, Liam DB) from their own docs

## Phase 1: smallest working version

- [ ] Define the schema JSON format: tables, columns, types, keys, indexes, notes
- [ ] Write the system prompt with the design rules
- [ ] Add 5–10 good example schemas to the prompt
- [ ] Generator: prompt → schema JSON (structured output)
- [ ] Checker: schema JSON → Postgres SQL
- [ ] Checker rules: every table has a primary key, foreign keys have indexes, names are consistent
- [ ] Sandbox: run the SQL in PGlite
- [ ] Fix loop: send errors back to the generator, up to N rounds
- [ ] Show the result as an ER diagram (React Flow or Mermaid)
- [ ] Export the SQL

## Phase 2: what makes Askema different

- [ ] Question step: find unclear points in the prompt and ask about them before generating
- [ ] Decision notes: save each answer as a note explaining why the schema looks the way it does
- [ ] Turn user stories into test queries
- [ ] Fill the tables with realistic fake data
- [ ] Run the test queries and flag failures and slow queries
- [ ] Export Drizzle, Prisma and Supabase migrations

## Quality

- [ ] Eval set: 30–50 test prompts with expected results
- [ ] Rerun the evals after every prompt change
- [ ] Compare local models against Claude on the eval set, and note which ones are good enough
- [ ] Measure the real cost per schema with Claude and compare it with the $0.10–0.15 estimate

## Later ideas

- [ ] Turn a spreadsheet or CSV into a database
- [ ] Plan safe changes to an existing database ("add subscriptions with trials")
- [ ] Ready-made patterns for one sector or country (local rules, Dzongkha and other languages)
- [ ] Fine-tuning, only once there are thousands of approved schemas and API cost has become a problem
