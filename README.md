<div align="center">

<img src="./miroshark-banner.jpg" alt="MiroShark Logo" width="75%"/>

<em>A Simple and Universal Swarm Intelligence Engine, Predicting Anything — Run Locally or with Any Cloud API</em>

</div>

## What is this?

**MiroShark** is a multi-agent simulation engine: upload any document (press release, policy draft, financial report), and it generates hundreds of AI agents with unique personalities that simulate the public reaction on social media. Posts, arguments, opinion shifts — hour by hour.

MiroShark runs on **Neo4j** for the knowledge graph and any **OpenAI-compatible API** for LLM inference and embeddings — use local Ollama (no cloud needed) or cloud providers like OpenRouter, OpenAI, or Anthropic.

> All you need to do: upload seed materials and describe your prediction requirements in natural language.
> MiroShark will return: a detailed prediction report and a high-fidelity digital world you can deeply interact with.

## Screenshots

<div align="center">
<table>
<tr>
<td><img src="./screen1.png" alt="Screenshot 1" width="100%"/></td>
<td><img src="./screen2.png" alt="Screenshot 2" width="100%"/></td>
</tr>
<tr>
<td><img src="./screen3.png" alt="Screenshot 3" width="100%"/></td>
<td><img src="./screen4.png" alt="Screenshot 4" width="100%"/></td>
</tr>
<tr>
<td><img src="./screen5.png" alt="Screenshot 5" width="100%"/></td>
<td><img src="./screen6.png" alt="Screenshot 6" width="100%"/></td>
</tr>
</table>
</div>

## Workflow

1. **Graph Build** — Extracts entities (people, companies, events) and relationships from your document. Builds a knowledge graph with individual and group memory via Neo4j.
2. **Agent Setup** — Generates hundreds of agent personas, each with unique personality, opinion bias, reaction speed, influence level, and memory of past events.
3. **Simulation** — Agents interact on simulated social platforms: posting, replying, arguing, shifting opinions. The system tracks sentiment evolution, topic propagation, and influence dynamics in real time. Supports **pause, resume, and restart** — simulations survive interruptions.
4. **Report** — A ReportAgent analyzes the post-simulation environment, interviews a focus group of agents, searches the knowledge graph for evidence, and generates a structured analysis. Reports are cached and reused.
5. **Interaction** — Chat with any agent from the simulated world via **persona chat** or send **group questions** to multiple agents at once. Click any agent to view their full profile and simulation activity. The environment auto-restarts for interviews if needed.

## Quick Start

### Prerequisites

- Docker & Docker Compose (recommended), **or**
- Python 3.11+, Node.js 18+, Neo4j 5.15+
- Ollama (for local inference) **or** an OpenRouter/OpenAI API key (for cloud inference)

### Option A: Docker (easiest)

```bash
git clone https://github.com/aaronjmars/MiroShark.git
cd MiroShark

# Start all services (Neo4j, Ollama, MiroShark)
docker compose up -d

# Pull the required models into Ollama
docker exec miroshark-ollama ollama pull qwen2.5:32b
docker exec miroshark-ollama ollama pull nomic-embed-text
```

Open `http://localhost:3000` — that's it.

### Option B: Manual (Local Ollama)

**1. Start Neo4j**

```bash
docker run -d --name neo4j \
  -p 7474:7474 -p 7687:7687 \
  -e NEO4J_AUTH=neo4j/miroshark \
  neo4j:5.15-community
```

**2. Start Ollama & pull models**

```bash
ollama serve &
ollama pull qwen2.5:32b      # LLM (or qwen2.5:14b for less VRAM)
ollama pull nomic-embed-text  # Embeddings (768d)
```

**3. Configure & run**

```bash
cp .env.example .env
# Edit .env if your Neo4j/Ollama are on non-default ports

# Install all dependencies
npm run setup:all

# Start both frontend and backend
npm run dev
```

### Option C: Cloud API (no GPU needed)

Only Neo4j is required locally. LLM and embeddings use a cloud API.

**1. Start Neo4j** (same as above, or `brew install neo4j && brew services start neo4j`)

**2. Configure & run**

```bash
cp .env.example .env
```

Edit `.env` with your API key (e.g. OpenRouter):

```bash
LLM_API_KEY=sk-or-v1-your-key
LLM_BASE_URL=https://openrouter.ai/api/v1
LLM_MODEL_NAME=qwen/qwen-2.5-72b-instruct

EMBEDDING_PROVIDER=openai
EMBEDDING_MODEL=openai/text-embedding-3-small
EMBEDDING_BASE_URL=https://openrouter.ai/api
EMBEDDING_API_KEY=sk-or-v1-your-key
EMBEDDING_DIMENSIONS=768

OPENAI_API_KEY=sk-or-v1-your-key
OPENAI_API_BASE_URL=https://openrouter.ai/api/v1
```

```bash
npm run setup:all
npm run dev
```

Open `http://localhost:3000`.

**Service addresses:**
- Frontend: `http://localhost:3000`
- Backend API: `http://localhost:5001`

## Configuration

### Recommended Models (OpenRouter)

**Primary simulation LLM — best all-round pick**

**Gemini 3 Flash Preview** (`google/gemini-3-flash-preview`)
- 1M context — zero risk of crashing on any simulation size
- Quality score 87 — strong reasoning for agent personas
- $0.50 in / $3.00 out — at 10k calls with ~2k tokens avg, ~$10–15 total

**Budget simulation LLM — max scale / cost control**

**DeepSeek Chat V3** (`deepseek/deepseek-chat`)
- 164K context — covers most simulations comfortably
- $0.32 in / $0.89 out — cheapest non-free model with real quality (score 79)
- Great for 100+ agent simulations where cost is the ceiling

**GPT-5 Nano** (`openai/gpt-5-nano`) as a fallback
- 400K context, $0.05 in / $0.40 out — almost free at scale
- Lower quality (51) but fine for simple agent personas or early-stage testing

**High-quality simulation (worth the extra cost)**

**Kimi K2** (`moonshotai/kimi-k2`)
- 131K context, quality score 89 — nearly Opus-level quality
- $0.55 in / $2.20 out — remarkably cheap for the capability
- Best pick if your agents need nuanced reasoning

**Report generation only (not simulation)**

**Claude Opus 4.6** or **GPT-5** — save these for the final report step where you make one high-quality call, not thousands.

**Embeddings** — `openai/text-embedding-3-small` on OpenRouter. Keep `EMBEDDING_DIMENSIONS=768` to match the Neo4j index.

| Situation | Pick |
|---|---|
| Default / balanced | Gemini 3 Flash Preview |
| Cost is critical | DeepSeek Chat V3 |
| Max agent quality | Kimi K2 |
| Prototype / testing | GPT-5 Nano |
| Final report | Claude Opus 4.6 |

### Recommended Models (Ollama — Local)

> **Important:** Ollama defaults to only **4096 tokens** of context regardless of what the model supports. For MiroShark's 10–30k token prompts you *must* override this:
>
> ```bash
> cat > Modelfile << 'EOF'
> FROM qwen3:14b
> PARAMETER num_ctx 32768
> EOF
> ollama create mirosharkai -f Modelfile
> ```

**Best overall — Qwen3 family**

`qwen3.5:27b` hits 72.4% on SWE-bench, putting it in the same range as GPT-5 Mini — an open-weight model on a single consumer GPU.

| Model | Pull command | VRAM needed | Context | Speed |
|---|---|---|---|---|
| `qwen3.5:27b` | `ollama pull qwen3.5:27b` | 20GB+ | 128K | ~40 t/s on 3090 |
| `qwen3.5:35b-a3b` *(MoE)* | `ollama pull qwen3.5:35b-a3b` | 16GB | 128K | ~112 t/s (MoE!) |
| `qwen3:14b` | `ollama pull qwen3:14b` | 12GB | 128K | ~60 t/s |
| `qwen3:8b` | `ollama pull qwen3:8b` | 8GB | 40K* | ~42 t/s |

The `35b-a3b` is a mixture-of-experts model that only activates 3B parameters per forward pass — 112 tokens/second on an RTX 3090. Quality is lower than the 27B dense model on hard problems, but for simulation agent calls it's fast enough to feel like a cloud API.

*\*Qwen3 8b: 40K context on Ollama — tight but workable for smaller simulations.*

**Budget / low VRAM**

- `llama3.3:70b-instruct-q4_K_M` — 40GB+ RAM (CPU offload), strong instruction following, 128K context. Slow though.
- `qwen2.5:14b` — 128K context, solid instruction following. Good fallback if Qwen3 is too new for your setup.

**Hardware decision tree**

| Your hardware | Pick |
|---|---|
| 24GB+ VRAM (RTX 3090/4090, M2 Pro 32GB+) | `qwen3.5:27b` — best quality |
| 16GB VRAM (RTX 4080, M2 Pro 16GB) | `qwen3.5:35b-a3b` — fastest via MoE |
| 12GB VRAM (RTX 4070, M1 Pro) | `qwen3:14b` — solid balance |
| 8GB VRAM / laptop | `qwen3:8b` — minimum viable, watch context limits |

**Embeddings locally**

```bash
ollama pull nomic-embed-text   # 768 dimensions — matches Neo4j default
```

Replaces `openai/text-embedding-3-small` for fully offline operation. Same 768-dim output, no API calls.

**Hybrid setup**

Run the local model for simulation rounds (cheap, high volume), then hit Gemini 3 Flash or Kimi K2 on OpenRouter for the final report generation step. Most users end up with this approach — local handles 70% of calls, cloud handles the quality-sensitive ones.

### Environment Variables

All settings are in `.env` (copy from `.env.example`):

```bash
# LLM — points to local Ollama (OpenAI-compatible API)
LLM_API_KEY=ollama
LLM_BASE_URL=http://localhost:11434/v1
LLM_MODEL_NAME=qwen2.5:32b

# Neo4j
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=miroshark

# Embeddings — "ollama" or "openai" provider
EMBEDDING_PROVIDER=ollama
EMBEDDING_MODEL=nomic-embed-text
EMBEDDING_BASE_URL=http://localhost:11434
EMBEDDING_API_KEY=
EMBEDDING_DIMENSIONS=768
```

Works with **any OpenAI-compatible API** — swap Ollama for OpenRouter, OpenAI, Claude, or any other provider by changing `LLM_BASE_URL` and `LLM_API_KEY`.

**Example: OpenRouter (no local GPU needed)**

```bash
LLM_API_KEY=sk-or-v1-your-key
LLM_BASE_URL=https://openrouter.ai/api/v1
LLM_MODEL_NAME=qwen/qwen-2.5-72b-instruct

EMBEDDING_PROVIDER=openai
EMBEDDING_MODEL=openai/text-embedding-3-small
EMBEDDING_BASE_URL=https://openrouter.ai/api
EMBEDDING_API_KEY=sk-or-v1-your-key
EMBEDDING_DIMENSIONS=768
```

## Architecture

```
┌─────────────────────────────────────────┐
│              Flask API                   │
│  graph.py  simulation.py  report.py     │
└──────────────┬──────────────────────────┘
               │ app.extensions['neo4j_storage']
┌──────────────▼──────────────────────────┐
│           Service Layer                  │
│  EntityReader  GraphToolsService         │
│  GraphMemoryUpdater  ReportAgent         │
└──────────────┬──────────────────────────┘
               │ storage: GraphStorage
┌──────────────▼──────────────────────────┐
│         GraphStorage (abstract)          │
│              │                            │
│    ┌─────────▼─────────┐                │
│    │   Neo4jStorage     │                │
│    │  ┌───────────────┐ │                │
│    │  │ EmbeddingService│ ← Ollama/OpenAI │
│    │  │ NERExtractor   │ ← Ollama LLM   │
│    │  │ SearchService  │ ← Hybrid search │
│    │  └───────────────┘ │                │
│    └───────────────────┘                │
└─────────────────────────────────────────┘
               │
        ┌──────▼──────┐
        │  Neo4j CE   │
        │  5.15       │
        └─────────────┘
```

**Key design decisions:**

- `GraphStorage` is an abstract interface — swap Neo4j for any other graph DB by implementing one class
- `EmbeddingService` supports both Ollama (`/api/embed`) and OpenAI-compatible (`/v1/embeddings`) providers
- Dependency injection via Flask `app.extensions` — no global singletons
- Hybrid search: 0.7 × vector similarity + 0.3 × BM25 keyword search
- Simulation supports pause/resume/restart with action log persistence
- Auto-restart environment for interviews when simulation is not running
- All original simulation tools (InsightForge, Panorama, Agent Interviews) preserved

## Hardware Requirements

**Local mode (Ollama):**

| Component | Minimum | Recommended |
|---|---|---|
| RAM | 16 GB | 32 GB |
| VRAM (GPU) | 10 GB (14b model) | 24 GB (32b model) |
| Disk | 20 GB | 50 GB |
| CPU | 4 cores | 8+ cores |

CPU-only mode works but is significantly slower for LLM inference. For lighter setups, use `qwen2.5:14b` or `qwen2.5:7b`.

**Cloud mode (OpenRouter/OpenAI):** No GPU required — just Neo4j and an API key. Any machine with 4 GB RAM can run the frontend + backend.

## Use Cases

- **PR crisis testing** — simulate the public reaction to a press release before publishing
- **Trading signal generation** — feed financial news and observe simulated market sentiment
- **Policy impact analysis** — test draft regulations against simulated public response
- **Creative experiments** — feed a novel with a lost ending; the agents write a narratively consistent conclusion

## License

AGPL-3.0 — same as the original MiroFish project. See [LICENSE](./LICENSE).

## Credits & Acknowledgments

Built on top of [MiroFish](https://github.com/666ghj/MiroFish) by [666ghj](https://github.com/666ghj), originally supported by [Shanda Group](https://www.shanda.com/).

The local Neo4j + Ollama storage layer (replacing Zep Cloud) was adapted from [MiroFish-Offline](https://github.com/nikmcfly/MiroFish-Offline) by [nikmcfly](https://github.com/nikmcfly). Their work on making MiroFish fully local — including the `GraphStorage` abstraction, Neo4j schema, embedding service, NER extractor, hybrid search, and the translated service layer — was the foundation for MiroShark's offline capabilities.

MiroShark's simulation engine is powered by **[OASIS](https://github.com/camel-ai/oasis)** from the CAMEL-AI team.
