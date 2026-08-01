---
name: TabPFN fit and predict
description: Train a TabPFN model on a tabular dataset and generate predictions using the Prior Labs cloud REST API.
api: openapi/priorlabs-openapi-original.json
operations: [tabpfn_get_model_limits, tabpfn_prepare_train_set_upload, tabpfn_fit, tabpfn_prepare_test_set_upload, tabpfn_predict]
---

# TabPFN fit and predict

Use the TabPFN cloud API (`https://api.priorlabs.ai`) to make predictions on structured/tabular data without local GPU. Prefer the `tabpfn-client` Python SDK (`TabPFNClassifier` / `TabPFNRegressor`) which calls these routes for you; use the raw routes below when integrating directly.

## Auth
Send `Authorization: Bearer <API_KEY>` on every request. Generate the key at ux.priorlabs.ai. Never commit it — read it from an env var / secret manager.

## Steps
1. **Check limits** — `tabpfn_get_model_limits` (`GET /tabpfn/get_model_limits`) to confirm your dataset fits the target model version (rows, columns, classes; TabPFN-3 allows up to 1M rows / 2,000 columns / 160 classes).
2. **Upload the train set** — `tabpfn_prepare_train_set_upload` (`POST /tabpfn/prepare_train_set_upload`) returns a signed URL and `dataset_id`; `PUT` the CSV/Parquet to that URL. A duplicate train set returns `409`.
3. **Fit** — `tabpfn_fit` (`POST /tabpfn/fit`) referencing the uploaded train `dataset_id`; returns a fitted model id.
4. **Upload the test set** — `tabpfn_prepare_test_set_upload` (`POST /tabpfn/prepare_test_set_upload`) and `PUT` the test data.
5. **Predict** — `tabpfn_predict` (`POST /tabpfn/predict`) with the model id and test `dataset_id`; returns predictions (class probabilities for classification, values/quantiles for regression).

## Rules
- **Rate/quota:** requests draw on daily (50M) and monthly (200M) token pools; thinking-mode fits have a separate monthly quota (20). Exhaustion returns `429` with a reset time — back off and retry after reset; predictions on existing models still work when the thinking-fit quota is spent.
- **Errors** use a custom envelope `{code, detail, retryable, support}` (see errors/priorlabs-problem-types.yml). `401` = bad token, `404` = missing model/dataset, `422` = validation error.
- No idempotency-key; re-uploading identical train data yields `409` rather than a duplicate job.
- Avoid the deprecated `/v1/fit` and `/v1/predict` multipart routes.
