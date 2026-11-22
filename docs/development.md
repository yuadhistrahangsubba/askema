# Setup and development

_Planned. Askema has no code yet; these are the setup steps v1 is designed for._

## First-time setup

You need:

- **Node.js**, current LTS
- **pnpm** (`npm install -g pnpm`, or `corepack enable`)
- **Ollama** from [ollama.com](https://ollama.com), unless you will only use Claude

```bash
git clone https://github.com/yuadhistrahangsubba/askema.git
cd askema
pnpm install
```

## Choosing a local model

Askema needs a model that is good at following a JSON format and reasoning about data. Bigger models give better schemas and need fewer fix rounds, but are slower and need more memory.

| Your computer's memory | What fits | Expect |
| --- | --- | --- |
| 8 GB | Small models, around 3–8B | Works, but more checker problems and fix rounds |
| 16 GB | Models up to around 14B | A reasonable starting point |
| 24 GB or more | Models around 20B, such as `gpt-oss:20b` | Noticeably better questions and schemas |

```bash
# Download a model once
ollama pull gpt-oss:20b

# Check that Ollama is running and the model is there
ollama list
```

Which local models work best for Askema will be measured with the eval set (see [TODO.md](../TODO.md)) and recorded here.

## Configuration

Copy `.env.example` to `.env` only if you want to change the defaults.

```bash
# Use a different local model
AI_PROVIDER=ollama
AI_MODEL=qwen2.5-coder:14b

# Or use Claude with your own key
AI_PROVIDER=anthropic
ANTHROPIC_API_KEY=sk-ant-...
```

| Variable | Default | Notes |
| --- | --- | --- |
| `AI_PROVIDER` | `ollama` | `ollama` or `anthropic` |
| `AI_MODEL` | provider's default | Model name for the chosen provider |
| `OLLAMA_HOST` | `http://localhost:11434` | Change it if Ollama runs on another machine |
| `ANTHROPIC_API_KEY` | none | Only for `AI_PROVIDER=anthropic`. Never commit `.env` |

## Dev server

```bash
# Start Askema on http://localhost:3000
pnpm dev

# Run the tests
pnpm test

# Lint
pnpm lint
```

## Troubleshooting

| Problem | Likely cause |
| --- | --- |
| "Can't reach Ollama" | Ollama isn't running. Open the Ollama app, or run `ollama serve` |
| "Model not found" | Run `ollama pull <model>` for the model in `AI_MODEL` |
| Slow answers | The model is too large for your memory. Try a smaller one |
| Many problems left after 3 fix rounds | The model is too small for the description. Try a larger model, or Claude |
