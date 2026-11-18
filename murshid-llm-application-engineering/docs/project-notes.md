# Project Notes — Murshid Mini

**Course:** LLM Application Engineering — SDA-AIE-213  
**Training context:** SDAIA Academy  
**Instructor:** Meaad Al-Marri  
**Project mode:** Core `mock`; separate local-model homework in Days 1 and 4

> This document describes the submitted implementation and its saved artifacts. It does not claim production readiness or SDAIA endorsement.

## 1. Architecture and request pipeline

The cumulative notebook is a 14-cell, four-day progression. It is designed to be run top-to-bottom in a single Python 3.9+ runtime.

At a high level, a request follows this path:

1. **Input handling:** the question is checked for basic validity and length.
2. **Language selection:** a simple Arabic-character check selects Arabic or English response text.
3. **Ticket extraction:** the application attempts to identify the application-status service, city, and reference identifier. If the reference is missing, it asks for clarification.
4. **Scope and safety checks:** unsupported actions are not executed; invalid or suspicious inputs are rejected by the educational guards.
5. **Read-only lookup:** a valid reference is passed to the status lookup function.
6. **Response validation:** the result is checked against the expected response shape and permitted output.
7. **Caching and metrics:** only the two exact public FAQ questions are eligible for the Day 4 cache. Request traces and metrics are recorded.
8. **Evaluation and export:** ten golden cases are rerun, and JSON results and a final report are saved.

The core notebook's `FakeClient` is deterministic mock code. Its prompt-building function is a visible teaching artifact; the mock client does not execute a prompt or generate model output.

## 2. Dataset and fields

The core dataset is the hand-written synthetic fixture `synthetic-v1`, stored in `results/core/synthetic_data.json` and defined in the notebook.

### Service directory

| Field | Type | Meaning |
|---|---|---|
| `service_directory.ar` | string | Fixed Arabic FAQ response |
| `service_directory.en` | string | Fixed English FAQ response |

### Synthetic records

The `records` object is keyed by a reference number. Each record contains:

| Field | Type | Meaning |
|---|---|---|
| `owner` | string | Synthetic owner identifier used by the authorization fixture |
| `status` | string | Synthetic application status |

The supplied examples are `REF-1001` (`demo-user-1`, `under_review`) and `REF-1002` (`demo-user-2`, `approved`). These values are fictional and must not be interpreted as real application data.

### Extracted ticket

The notebook's ticket structure contains:
- `service`: the requested service category;
- `city`: extracted city or `unknown`;
- `request_id`: extracted reference number or `null`;
- `needs_clarification`: boolean indicating whether more information is needed.

The reference format accepted by the core tool is `REF-` followed by four digits.

## 3. Read-only tool and authorization fixture

The core `check_status` function accepts a reference number and a session identity. It:
- rejects malformed reference numbers;
- returns `not_found` for an unknown reference;
- returns `access_denied` when the record owner does not match the supplied session;
- otherwise returns the synthetic status.

The function does not create, edit, cancel, or otherwise mutate an application. The Day 2 core examples include:
- a valid owned record (`under_review`);
- a missing reference, which requires clarification;
- an unknown reference (`not_found`);
- an unsupported cancellation request (`unsupported_action`);
- a record owned by another synthetic user (`access_denied`).

**Authorization limitation:** the session value is supplied as trusted test setup in the notebook. It is not authenticated, signed, or verified against a real identity provider. This is an authorization demonstration fixture, not production access control.

The separate Day 2 homework notebook tests a simulated tool-call envelope. Its recorded result reports 7/7 cases passed and `records_unchanged: true`.

## 4. Validation and guards

The core notebook demonstrates limited input and output validation:
- non-string or empty questions are rejected;
- questions longer than 500 characters are rejected;
- an example of unnecessary personal data is blocked;
- unsupported actions are not routed to a write operation;
- tool outputs and response shapes are checked;
- malformed ticket schemas are rejected;
- a seeded instruction-like input is included among the golden tests.

The separate Day 3 homework notebook intentionally submits `None`, whitespace-only text, and an over-length question. It catches the expected `ValueError` exceptions and accepts a corrected input. These are deliberate test cases, not uncaught runtime errors.

These guards are narrow educational examples. They do not provide comprehensive prompt-injection detection, personal-data protection, or production security.

## 5. Cache design and rules

The Day 4 core cache is intentionally narrow:
- only the two exact public FAQ questions are eligible;
- entries have a 300-second time-to-live;
- the key incorporates the question and the prompt/directory version values;
- changing the prompt or directory version prevents an old entry from being treated as current;
- the cache is not used for personalized status lookups.

The recorded repeatable experiment sends the same FAQ three times:
- without cache: 3 mock-client calls;
- with cache: 1 mock-client call;
- cache flags: `[false, true, true]`;
- recorded hit rate: 66.7%.

This is a deterministic mock-call experiment, not a measured reduction in external API requests or provider cost. The final report labels its cost calculation as illustrative only.

## 6. Tests and evaluation

### Core golden tests

The Day 3 evaluation stores ten cases (`C01`–`C10`) covering:
- Arabic and English FAQ behavior;
- valid, unknown, and missing reference numbers;
- out-of-scope requests;
- access denial;
- an instruction-like input;
- invalid input;
- invalid output schema.

The saved core result is **10/10 passed (100%)** in `mode: mock`. Day 4 reruns the regression evaluation after the cache is introduced.

### Homework results

| Homework | Recorded result | Interpretation |
|---|---|---|
| Day 1 local model | `google/flan-t5-small`, CPU, 0 external inference API calls | One saved answer recommends using the reference number; manual review remains required |
| Day 2 tool protocol | 7/7 passed; records unchanged | Simulated tool-call validation only |
| Day 3 errors | Expected `ValueError` cases caught; corrected input accepted | Deliberate error-handling exercise |
| Day 4 local-model evaluation | 3/3 keyword hits; 0 external inference API calls | Keyword proxy only, not semantic accuracy |

The Day 1 and Day 4 exercises use the pinned model revision `0fc9ddf78a1e988dac52e2dac162b0ede4fd74ab`. The saved outputs identify CPU inference. The model and libraries require a first-time download, but inference is local rather than through a hosted inference API.

## 7. Reproducibility

### Core notebook
- Python 3.9+.
- Standard library only.
- No API key, GPU, model weights, or package installation required.
- Run all 14 cells from top to bottom in one fresh runtime.

### Day 1 and Day 4 homework
- CPU inference.
- PyTorch, Transformers 4.49.0, and SentencePiece.
- Internet access is needed for initial package and model downloads.
- No external inference API call is recorded.

**Dependency cleanup before re-running:** Day 1 currently contains repeated installation commands that pin SentencePiece to both `0.2.1` and `0.2.0`. Consolidate those cells to one version before a fresh run. Do not install two conflicting pins in sequence. Day 4 should use the same consistent dependency set.

### Saved outputs

The core notebook writes JSON files to `murshid_outputs/` when run. The repository copies are stored under `results/core/`. Homework result files are stored under `results/day1/`, `results/day4/`, and `results/homework/`.

Timing and trace identifiers may differ on a new run. Do not claim a check passed unless it has actually been rerun after the latest code change.

## 8. Known limitations

- The core client is deterministic mock code, not a trained language model.
- The core data is synthetic and finite.
- The authorization session is a test fixture, not real authentication.
- The tool is read-only and does not integrate with a real service.
- The guards cover a small list of cases and are not a complete security solution.
- The Day 4 homework keyword proxy does not measure semantic correctness.
- Local model outputs require manual review for meaning and unsupported claims.
- Runtime measurements depend on hardware and environment.
- The hypothetical cost example is not actual provider pricing or API expenditure.

## 9. Attribution

Prepared as an individual educational project for **LLM Application Engineering — SDA-AIE-213**, in the **SDAIA Academy** training context, with instructor **Meaad Al-Marri**.

- [SDAIA Academy official website](https://sdaia.gov.sa/ar/Sectors/BuildingCapacity/academy/Pages/default.aspx)
- [SDAIA Academy GitHub organization](https://github.com/SDAIAAcademy)

These links are attribution only; they do not indicate endorsement, certification, or approval of this implementation.
