# ruvector-memory

An [OpenClaw](https://openclaw.ai) skill that gives AI agents semantic vector memory using [RuVector](https://github.com/ruvnet/ruvector) / [AgentDB](https://github.com/ruvnet/ruvector). Search improves automatically over time via GNN self-learning.

## Why

Keyword grep misses ~20% of conceptual queries — things like "what does liam do that isn't software" won't match anything even when the answer is clearly in your memory files. Vector search understands meaning, not just words.

**Validated against a grep baseline** across 30 test cases (easy / medium / hard):

| Tier    | System | Recall@3 | MRR   | Avg Latency |
|---------|--------|----------|-------|-------------|
| easy    | grep   | 80.0%    | 0.633 | 2107ms      |
| easy    | vector | 50.0%    | 0.433 | 794ms       |
| medium  | grep   | 80.0%    | 0.553 | —           |
| medium  | vector | 80.0%    | 0.667 | —           |
| **hard**| grep   | **50.0%**| 0.367 | —           |
| **hard**| vector | **70.0%**| 0.500 | —           |
| overall | grep   | 70.0%    | 0.518 | 2107ms      |
| overall | vector | 66.7%    | 0.533 | **794ms**   |

**Verdict: PASS** on all pre-committed criteria:
- ✅ Vector Recall@3 on hard queries beats grep by **+20pp** (threshold: ≥15pp)
- ✅ Overall vector MRR > grep MRR
- ✅ Vector latency **2.7x faster** than grep (well under 10x limit)

Vector wins where it matters: hard conceptual queries. Grep still wins on easy keyword lookups — both systems are useful.

## Architecture

```
memory/              ← existing flat files (source of truth, unchanged)
  MEMORY.md
  YYYY-MM-DD.md
agentdb/             ← vector index (managed by this skill)
  memory.db          ← RuVector backend, GNN-enhanced HNSW, EWC++ memory
```

AgentDB sits alongside your existing memory system. Flat files stay as the canonical record; the vector index is a fast semantic retrieval layer over them.

## Quick Start

### Requirements
- Node.js 18+
- `agentdb` v3.0.0-alpha.10 (`npm install -g agentdb@3.0.0-alpha.10`)
- OpenClaw (for skill triggering)

### Install

```bash
npx clawhub@latest install ruvector-memory
```

Or clone this repo and copy into your skills directory.

### First-time setup

```bash
bash skills/ruvector-memory/scripts/setup.sh
```

This installs AgentDB, initializes the vector index, and ingests all existing memory files.

### Store a memory

```bash
bash scripts/store.sh "Liam prefers Rust for performance-critical work" "MEMORY.md" "liam,preferences"
```

### Query semantically

```bash
bash scripts/query.sh "what languages does liam like" 5
```

Returns semantically ranked results — no keyword match required.

### Ingest existing files

```bash
bash scripts/ingest.sh ~/path/to/memory/
bash scripts/ingest.sh ~/path/to/MEMORY.md
```

### Check status

```bash
bash scripts/status.sh
```

## How It Works

RuVector uses a **GNN (Graph Neural Network)** layer on top of HNSW vector search. Every query teaches the system which results were relevant — after enough queries, retrieval accuracy improves automatically without retraining.

**EWC++ (Elastic Weight Consolidation)** prevents catastrophic forgetting: new patterns are learned without erasing old ones.

All local. No API calls. No cloud. No per-query cost.

## Eval Harness

A full scientific eval is included in `eval/`:

```bash
cd eval
bash run_eval.sh          # full run (~25 min, uses local Qwen for judging)
bash run_eval.sh --limit 5  # quick smoke test
```

- 30 hand-crafted test cases across easy / medium / hard tiers
- LLM-as-judge (blind, 0-3 relevance scale) with 4-worker parallelism
- Pre-committed pass/fail criteria (set before running, can't be moved)
- See `eval/results/summary.md` for latest results

## Files

```
ruvector-memory/
├── SKILL.md                  — OpenClaw skill definition
├── clawhub.json              — ClawHub manifest
├── scripts/
│   ├── setup.sh              — install + init + ingest
│   ├── store.sh              — index a memory chunk
│   ├── query.sh              — semantic search
│   ├── ingest.sh             — batch ingest markdown files
│   └── status.sh             — health check
├── references/
│   └── agentdb.md            — AgentDB CLI reference
└── eval/
    ├── test_cases.jsonl      — 30 test cases
    ├── run_eval.sh           — orchestrator
    ├── grep_search.sh        — baseline
    ├── vector_search.sh      — treatment
    ├── judge.py              — LLM judge (parallel)
    └── results/summary.md    — latest results
```

## Credits

Built on [RuVector](https://github.com/ruvnet/ruvector) by [rUv](https://ruv.io) — a self-learning vector database in Rust.

## License

MIT
