---
name: TabPFN predictions via MCP
description: Run TabPFN tabular predictions from an AI agent through the Prior Labs Model Context Protocol server.
api: mcp/priorlabs-mcp.yml
operations: [upload_dataset, fit_and_predict_from_dataset, predict_from_dataset, fit_and_predict_inline, predict_inline]
---

# TabPFN predictions via MCP

Connect an agent (Claude, ChatGPT, Cursor, Codex CLI, n8n) to the TabPFN MCP server for natural-language predictions on tabular data.

## Connect
- Server: `https://api.priorlabs.ai/mcp/server` (HTTP transport).
- Auth: OAuth (Claude.ai / Claude Desktop / ChatGPT / Codex CLI / Cursor) or a manual `Authorization: Bearer <API_KEY>` header from ux.priorlabs.ai.
- Claude Code: `claude mcp add --transport http tabpfn https://api.priorlabs.ai/mcp/server`.

## Tools
- **upload_dataset** — get a secure upload URL for train/test data; returns a `dataset_id` valid 60 minutes.
- **fit_and_predict_from_dataset** — fit on pre-uploaded data and predict.
- **predict_from_dataset** — predict with an existing model against a new uploaded test set.
- **fit_and_predict_inline** — fit from inline data arrays and return predictions plus a model id (best for small datasets that fit in the conversation).
- **predict_inline** — predict from a previously fitted model id with inline test features.

## Flow
1. For large data: `upload_dataset` (train), then `upload_dataset` (test), then `fit_and_predict_from_dataset`.
2. For small data: `fit_and_predict_inline`, keep the returned model id, then `predict_inline` for new rows.

## Rules
- Same daily/monthly token pools and thinking-fit quota as the REST API apply; a `429`-equivalent means back off until the reset time.
- A `dataset_id` expires after 60 minutes — re-upload if a later call fails to find it.
