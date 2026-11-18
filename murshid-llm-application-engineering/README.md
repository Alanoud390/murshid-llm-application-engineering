# Murshid Mini — LLM Application Engineering

**Course:** LLM Application Engineering — SDA-AIE-213  
**Training context:** [SDAIA Academy](https://sdaia.gov.sa/ar/Sectors/BuildingCapacity/academy/Pages/default.aspx)
**Instructor:** Meaad Al-Marri  
**Author:** Alanoud Abdulaziz Alwehaibi  
**Email:** aad630748@gmail.com


> This is an individual educational project prepared in the SDAIA Academy training context. This repository does not claim SDAIA endorsement, certification, or approval of this implementation.

## Project overview

Murshid Mini is a bilingual (Arabic/English) service-assistance prototype. It demonstrates how to engineer the application layer around a model-shaped interface: request handling, structured ticket extraction, a read-only status tool, input and output validation, a small regression suite, caching, and operational metrics.

The intended user is someone who wants basic guidance about checking a fictional application status. The demo can answer a fixed FAQ, ask for a missing reference number, and return a limited status for a synthetic record. It is not connected to a real government or service system.

### Important: two distinct execution tracks

- **Core cumulative notebook (`notebooks/murshid.ipynb`):** runs in `mode="mock"` with a deterministic `FakeClient`/training-mock response layer. It uses Python's standard library and synthetic data. It is not a trained language model and makes zero external inference API calls.
- **Day 1 and Day 4 homework notebooks:** use the public `google/flan-t5-small` model at a pinned Hugging Face revision. The saved results identify CPU inference and zero external inference API calls. The model and libraries must be downloaded on the first run; inference itself is local.
- **Day 2 and Day 3 homework notebooks:** demonstrate a simulated tool protocol and deliberate, caught validation errors using Python's standard library.

Do not describe the core mock results as live-LLM results, and do not describe the Day 1/Day 4 model exercises as the core application's backend.

## Problem and intended scope

A user may need to know how to check an application status but may not know which identifier is required. The prototype illustrates a constrained assistant that uses a fictional service directory and a small synthetic record set rather than inventing fees, deadlines, or application updates.

**In scope**
- Fixed bilingual FAQ responses.
- Extraction of a fictional service, city, and reference number.
- Clarification when a reference number is missing.
- Read-only lookup of a synthetic status record.
- Validation, safe error handling, a small set of golden tests, and caching of two exact public FAQ questions.

**Out of scope**
- Real application submissions, cancellations, edits, or other write actions.
- Real user accounts, identity verification, or production authorization.
- Live service data, external databases, or provider inference APIs.
- Production security, comprehensive prompt-injection defence, or a production-readiness claim.

## Four-day development journey

| Day | Core milestone | Main artifact |
|---|---|---|
| 1 | Establish the baseline bilingual FAQ flow and deterministic mock client | `results/core/baseline_results.json` |
| 2 | Add synthetic records, ticket extraction, and read-only status lookup | `results/core/day2_results.json` |
| 3 | Add input/output guards, versioned prompt metadata, and ten golden cases | `results/core/day3_results.json` |
| 4 | Add a narrow cache, repeatable before/after experiment, regression run, and final report | `results/core/day4_results.json`, `results/core/final_report.json` |

The homework notebooks are separate exercises and are listed below.

## Repository map

```text
murshid-llm-application-engineering/
├── README.md
├── notebooks/
│   └── murshid.ipynb
├── homework/
│   ├── day1_real_model.ipynb
│   ├── day2_tools.ipynb
│   ├── day3_errors.ipynb
│   └── day4_evaluation.ipynb
├── results/
│   ├── core/
│   │   ├── baseline_results.json
│   │   ├── day2_results.json
│   │   ├── day3_results.json
│   │   ├── day4_results.json
│   │   ├── final_report.json
│   │   ├── metrics.json
│   │   └── synthetic_data.json
│   ├── day1/
│   │   └── live_demo_result.json
│   ├── day4/
│   │   └── evaluation_result.json
│   └── homework/
│       ├── day2_tools_results.json
│       └── day3_errors_results.json
└── docs/
    └── project-notes.md
```

## Setup and run

### Core cumulative notebook — CPU, standard library, no key

1. Open `notebooks/murshid.ipynb` in Google Colab or Jupyter.
2. Use Python 3.9 or later.
3. Run all 14 code cells from top to bottom in one fresh runtime.
4. The notebook creates `murshid_outputs/` in the runtime and writes the core JSON artifacts there.
5. For this core notebook, no `pip install`, API key, GPU, or model download is required.

The committed copies of the core JSON outputs are in `results/core/`.

### Homework notebooks

Open each notebook separately. Day 1 and Day 4 are model-download exercises and should be run in a separate runtime from the core notebook.

| Notebook | Runtime and dependencies | Saved result |
|---|---|---|
| `homework/day1_real_model.ipynb` | CPU; PyTorch, Transformers 4.49.0, SentencePiece; internet for first-time package/model download | `results/day1/live_demo_result.json` |
| `homework/day2_tools.ipynb` | Python standard library; no model or API | `results/homework/day2_tools_results.json` |
| `homework/day3_errors.ipynb` | Python standard library; no model or API | `results/homework/day3_errors_results.json` |
| `homework/day4_evaluation.ipynb` | CPU; PyTorch, Transformers 4.49.0, SentencePiece; internet for first-time package/model download | `results/day4/evaluation_result.json` |

The model exercises use `google/flan-t5-small` at revision `0fc9ddf78a1e988dac52e2dac162b0ede4fd74ab`, with `trust_remote_code=False` and `use_safetensors=True`. They do not use a hosted inference endpoint.

**Fresh-run note:** the Day 1 notebook currently has repeated install cells and pins SentencePiece to both `0.2.1` and `0.2.0`. The saved result is present, but before a clean re-run, consolidate these to one consistent pin. Day 4 also contains setup/import cells; use the same dependency set as Day 1.

## Recorded results

These are the values in the submitted JSON files, not claims of live production performance:

- Core Day 3 golden tests: **10/10 passed (100%)** in mock mode.
- Core Day 4 cache experiment: **3 mock calls without cache**, **1 mock call with cache**, and a **66.7%** hit rate for the three repeated requests.
- Core recorded external inference API calls: **0**.
- Homework Day 2 tool-protocol checks: **7/7 passed**; the synthetic records remained unchanged.
- Homework Day 3: three deliberately invalid inputs were caught as `ValueError`; the corrected input was accepted.
- Homework Day 1: local CPU inference returned “Use its reference number” for the supplied English FAQ.
- Homework Day 4: **3/3 keyword-proxy hits**. This is not semantic accuracy; the answers still require manual review.

The core cost example is explicitly hypothetical and is not actual API spend, provider pricing, or measured savings.

## Technical documentation

See [`docs/project-notes.md`](docs/project-notes.md) for:
- pipeline stages and request flow;
- synthetic dataset and field definitions;
- validation and read-only tool behavior;
- authorization test fixture;
- cache key, expiry, and version rules;
- tests, saved metrics, and known limitations.

## Limitations and responsible interpretation

- `FakeClient` is deterministic mock code, not a trained language model.
- The core synthetic records are hand-written teaching fixtures, not real service records.
- The session identity is trusted test setup, not production authentication.
- The read-only tool can only inspect the two synthetic records included in the notebook.
- The guards cover listed examples only; they are not comprehensive security or privacy controls.
- The Day 4 homework keyword metric checks for a reference-number phrase and does not prove semantic correctness.
- Latency values vary by runtime and hardware.
- No production deployment, service integration, or SDAIA endorsement is claimed.

## Attribution

Prepared as an individual project for **LLM Application Engineering — SDA-AIE-213**, in the **SDAIA Academy** training context, with instructor **Meaad Al-Marri**.

- [SDAIA Academy — official website](https://sdaia.gov.sa/ar/Sectors/BuildingCapacity/academy/Pages/default.aspx)
- [SDAIA Academy — GitHub organization](https://github.com/SDAIAAcademy)

The links provide attribution and context only. They do not imply endorsement or certification of this project.

## Before submission

- Add your full name and the official cohort dates above.
- Restart the runtime and run the core notebook from top to bottom; rerun affected checks after any code change.
- Consolidate the duplicate SentencePiece installation pins in Day 1 before a fresh model-homework run.
- Inspect notebook outputs and JSON files for secrets, personal identifiers, or real service data.
- Use meaningful Git commits for the actual stages of your work; do not invent historical commits.
- Confirm that the instructor can access the repository, then submit the repository root URL.
