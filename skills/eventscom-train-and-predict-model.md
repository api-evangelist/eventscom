---
name: Train and predict with a DataGol dashboard ML model
description: Create a machine-learning model against an Events.com DataGol dashboard element, trigger training, check history, and run a prediction.
api: openapi/eventscom-datagol-ai-openapi.yml
generated: '2026-08-04'
method: generated
operations:
  - get_model_config
  - create_model
  - get_models
  - get_model
  - train_model
  - model_training_history
  - predict_using_model
  - update_model
  - delete_model
---

# Train and predict with a DataGol dashboard ML model

The DataGol AI service attaches ML models to a **dashboard element**. Every path is scoped by
`{elementType}` and `{elementId}` — the polymorphic element handle used across DataGol.

## Before you start

- **Base URL.** `https://datagol-ai.events.com/`
- **Auth.** No `securityScheme` is declared on this service (see the caveat in
  `skills/eventscom-run-ai-conversation.md`). Assume the platform JWT is required.
- **`elementType` has no published enum.** The spec declares it as a free path parameter with no valid-value
  list. Obtain a real `elementType`/`elementId` pair from the platform API rather than guessing.
- **Training is a long-running, billable-class operation with no idempotency key.** A retried `train_model`
  starts another training run. Poll `model_training_history` before retrying.

## Steps

1. **Read what model types are configurable** — `get_model_config`
   `GET /ai/api/v2/ml/dashboard/models/config`

2. **Create the model** — `create_model`
   `POST /ai/api/v2/ml/dashboard/{elementType}/{elementId}/models` → keep the `modelId`.

3. **List or read models on the element** — `get_models`
   `GET /ai/api/v2/ml/dashboard/{elementType}/{elementId}/models`, and `get_model`
   `GET /ai/api/v2/ml/dashboard/{elementType}/{elementId}/models/{modelId}`

4. **Trigger training** — `train_model`
   `POST /ai/api/v2/ml/dashboard/{elementType}/{elementId}/models/{modelId}/train`

5. **Poll training history until it settles** — `model_training_history`
   `GET /ai/api/v2/ml/dashboard/{elementType}/{elementId}/models/{modelId}/history`
   There is no webhook, no callback and no `Retry-After` on this surface, so polling with your own backoff
   is the only completion signal.

6. **Predict** — `predict_using_model`
   `POST /ai/api/v2/ml/dashboard/{elementType}/{elementId}/models/{modelId}/predict`

7. **Maintain** — `update_model` (`PUT .../models/{modelId}`) and `delete_model`
   (`DELETE .../models/{modelId}`).

## Errors

FastAPI validation errors return `422` with `{"detail":[{"loc":[...],"msg":"...","type":"..."}]}`. The
`loc` array points at the exact offending field — use it rather than re-sending blindly. No other 4xx
response is declared on this service, so treat anything else as an undocumented condition and fail loudly.
