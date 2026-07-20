# CLAUDE.md — A9_Prithvi / P-PFE Tool

**Repo:** `C:\Users\tjmayer\Documents\GitHub\A9_Prithvi`
**Purpose of this file:** Instructions for a Claude Code CLI instance working *locally* in this repo. Read this fully before making any changes. Keep this file up to date as the project evolves — it is the source of truth for scope, current phase, and conventions.

---

## 1. Project Summary

We are building **P-PFE** (Prithvi-Powered [Forecasting/Fusion] Engine — expand acronym once finalized), a decision-support tool that follows a **NASA VEDA-aligned architecture** so it integrates cleanly with **NASA IMPACT** platform services. The contract framing (see `docs/task_order/` once populated) organizes the work into three elements:

- **Element 1 — Architecture & Standards Alignment**: VEDA/IMPACT/OSS review → architecture blueprint (components, APIs, data flows, security, AWS model hosting/inference).
- **Element 2 — Core Platform Development**: spatial scaling, data ingest, inference, prediction serving, automation, user interaction; use cases + integration tests.
- **Element 3 — Documentation, Knowledge Transfer, VEDA Readiness**: developer + user docs, contribution framework, onboarding materials.

Deliverables D1–D5 (blueprint, core build, doc suite, integration/validation, sustainability framework) are tracked in `docs/deliverables/STATUS.md`.

### Pivot decision (current)
Rather than building the model-serving/orchestration layer from scratch, we are **building on top of TerraStackAI's Geospatial Studio stack**:

- Platform/docs: https://terrastackai.github.io/geospatial-studio/
- SDK + QGIS plugin: https://github.com/terrastackai/geospatial-studio-toolkit (PyPI: `geostudio`)

Geospatial Studio already provides a no-code UI, low-code Python SDK, and REST API for **fine-tuning, inference, and orchestration of geospatial AI models** (TerraTorch-based foundation models, e.g. Prithvi family). It ships example notebooks for dataset onboarding, fine-tuning, HPO, inference, and precomputed-layer uploads, plus end-to-end walkthroughs (burn scars, flooding). This gives us a working reference architecture for Element 1 and a set of building blocks for Element 2 — our job shifts from "build an inference platform" to "integrate with / extend Geospatial Studio to serve the P-PFE decision-support use case," and document that integration for Element 3.

**Immediate technical goal:** produce a **dummy Prithvi-WxC-like output** (synthetic forecast tensor / plausible-shaped NetCDF or GeoTIFF stack) so we can build and test the downstream inference → decision-support pipeline *without waiting on a real Prithvi-WxC checkpoint or GPU access*. Once the pipeline is proven end-to-end on dummy data, swap in the real model.

---

## 2. Current Phase

> Update this section at the start of every work session.

- [ ] Phase 0: Repo scaffolding + this CLAUDE.md (in progress)
- [ ] Phase 1: Dummy Prithvi-WxC output generator
- [ ] Phase 2: Geospatial Studio SDK integration (local/dev deployment or mocked client)
- [ ] Phase 3: Inference → decision-support pipeline (ingest → infer → postprocess → decision layer)
- [ ] Phase 4: Architecture blueprint doc (D1)
- [ ] Phase 5: Tests, validation, docs suite (D3/D4)

**Working on right now:** _fill in before each session_

---

## 3. Repo Layout (target — create as needed, don't assume it exists yet)

```
A9_Prithvi/
├── CLAUDE.md                     # this file
├── README.md
├── pyproject.toml                # or requirements.txt — pick one, don't mix
├── src/
│   └── ppfe/
│       ├── __init__.py
│       ├── dummy_wxc.py          # Phase 1: synthetic Prithvi-WxC-like output generator
│       ├── ingest/                # data ingest connectors (EO sources, VEDA STAC, etc.)
│       ├── inference/             # geostudio SDK client wrapper + model calls
│       ├── decision/              # decision-support logic layer (rules/ML on top of inference)
│       └── serving/               # API layer for exposing pipeline outputs
├── notebooks/                     # exploratory notebooks (mirror geostudio-toolkit examples)
├── tests/
│   ├── unit/
│   └── integration/
├── docs/
│   ├── architecture/              # D1 blueprint, diagrams
│   ├── developer_guide/
│   ├── user_guide/
│   ├── deliverables/
│   │   └── STATUS.md
│   └── task_order/                # original SOW/task text, for reference — do not treat as code
├── .github/
│   ├── workflows/                 # CI: lint, test, (later) build
│   ├── ISSUE_TEMPLATE/
│   └── PULL_REQUEST_TEMPLATE.md
└── data/                          # gitignored — local dummy/sample data only, never commit real data
```

If the repo currently doesn't match this, propose the diff and create missing pieces incrementally — don't do a disruptive one-shot restructure without flagging it first.

---

## 4. Working Agreements for Claude Code in this repo

1. **Always check current phase (Section 2) before starting work.** Don't jump ahead to Phase 3 code if Phase 1 isn't done.
2. **Prefer extending Geospatial Studio's SDK (`geostudio` package) over reinventing inference/orchestration.** Read `examples/` in a local clone of `geospatial-studio-toolkit` before writing new ingest/inference code — mirror their patterns (`Client`, `list_models`, `list_tunes`) so we stay compatible with upstream.
3. **No real credentials or API keys in this repo, ever.** Geostudio auth uses a local credentials file (e.g. `.geostudio_config_file` with `GEOSTUDIO_API_KEY` / `BASE_STUDIO_UI_URL`) — this must be `.gitignore`d and referenced only via a documented env var pattern.
4. **Dummy data first.** For Phase 1–2, never assume a live Geospatial Studio deployment or GPU is available. Build a mock/stub `geostudio` client and a synthetic Prithvi-WxC output generator so the pipeline is testable offline. Real model/API swap-in should be a config flag, not a rewrite.
5. **VEDA/IMPACT compatibility checkpoints.** Whenever adding a data ingest or metadata step, check it against VEDA conventions (STAC catalogs, COG/Zarr outputs, standard CRS handling) before inventing a custom format.
6. **Every new module gets a matching test in `tests/unit/` before being marked done.** Integration tests against a real Geospatial Studio deployment go in `tests/integration/` and must be skippable (`pytest -m "not integration"`) when no deployment is configured.
7. **Documentation is not an afterthought.** When a module reaches a working state, add/update its section in `docs/developer_guide/` in the same session — don't defer all docs to Element 3.
8. **Windows/PowerShell environment.** This repo lives at `C:\Users\tjmayer\Documents\GitHub\A9_Prithvi` on Windows. Give PowerShell-compatible commands (not bash) when suggesting terminal steps, unless working inside WSL — confirm which shell is in use if unclear.
9. **Update Section 2 (Current Phase) and `docs/deliverables/STATUS.md`** at the end of each session so the next session (or next Claude Code invocation) picks up context correctly.
10. **Don't fabricate NASA/IMPACT/VEDA architectural claims.** If unsure whether something matches VEDA's actual technical framework, say so and flag it for verification against https://nasa-impact.github.io/veda-docs/ or the VEDA GitHub org, rather than asserting compatibility.

---

## 5. Phase 1 Spec — Dummy Prithvi-WxC Output Generator

Goal: a script/module (`src/ppfe/dummy_wxc.py`) that produces output shaped like a real Prithvi-WxC forecast so downstream code can be built against it immediately.

Requirements:
- Configurable spatial extent, resolution, number of forecast lead times, and variable list (surface + upper-air placeholders).
- Output format matching what the real pipeline will consume downstream (likely NetCDF or a stack of GeoTIFFs — confirm against Geospatial Studio's expected inference output format from the SDK examples before finalizing).
- Include realistic-ish spatial/temporal structure (e.g. smoothed noise + diurnal/seasonal signal) rather than pure random noise, so downstream visualization and decision logic can be sanity-checked visually.
- CLI entry point: `python -m ppfe.dummy_wxc --bbox ... --lead-times ... --out ...`
- Unit test asserting output shape/CRS/variable list matches spec.

## 6. Phase 2 Spec — Geospatial Studio Integration

- Wrap `geostudio.Client` in `src/ppfe/inference/client.py` with a **mockable interface** (dependency-injected client) so tests can run without a live deployment.
- Support both: (a) a real Geospatial Studio deployment via `.geostudio_config_file`, and (b) a local/offline mode that reads from the Phase 1 dummy output instead of calling the API.
- Document both modes clearly in `docs/developer_guide/inference.md`.

## 7. Phase 3 Spec — Inference → Decision-Support Pipeline

- `ingest` → `inference` → `decision` → `serving`, each a swappable stage.
- Decision layer starts simple (threshold/rule-based on forecast variables) with a clear extension point for a learned model later.
- End-to-end integration test running the full pipeline on dummy data, producing a decision-support artifact (e.g. a risk map or ranked alert list).

---

## 8. Open Questions to Resolve Early (flag, don't silently assume)

- Exact meaning of "P-PFE" acronym and the specific decision the tool supports (what hazard/variable, what end user).
- Whether we have or will have access to an actual Geospatial Studio deployment (local Docker install per https://github.com/terrastackai/geospatial-studio) vs. SDK-only against a remote IMPACT-hosted instance.
- Target AWS hosting pattern for D1 (Lambda/Batch vs. SageMaker vs. ECS) — needs a decision before the blueprint can be finalized.
- Whether real Prithvi-WxC checkpoints/weights are available yet, and under what license/access terms.

---

## 9. Session Startup Checklist (for Claude Code to run at the start of every session)

1. Read this file in full.
2. Read `docs/deliverables/STATUS.md` for latest phase/status notes.
3. Run `git status` and `git log -5 --oneline` to see uncommitted work and recent history.
4. State current phase and proposed next action before writing code.
5. If anything in this file seems stale relative to the repo's actual state, say so and propose an update to this file.
