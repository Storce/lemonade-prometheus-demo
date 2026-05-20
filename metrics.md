# Lemonade Prometheus Metrics

`GET /metrics` exposes Prometheus text format metrics for Lemonade Server. It is registered as a top-level scrape endpoint, not under the versioned `/v1` or `/api/v1` prefixes. `HEAD /metrics` returns `200` without a body.

The endpoint is protected by the same API-key pre-routing logic as regular API routes when `LEMONADE_API_KEY` is configured. Either `LEMONADE_API_KEY` or `LEMONADE_ADMIN_API_KEY` is accepted as the bearer token. If only `LEMONADE_ADMIN_API_KEY` is set, `/metrics` stays unauthenticated, matching regular API endpoint behavior.

## Server Metrics

| Metric | Type | Labels | Description |
|--------|------|--------|-------------|
| `lemonade_server_up` | gauge | none | Always `1` while the server can answer the scrape. |
| `lemonade_server_info` | gauge | `version` | Build/version metadata. The sample value is always `1`. |

## Loaded Model Metrics

| Metric | Type | Labels | Description |
|--------|------|--------|-------------|
| `lemonade_loaded_models` | gauge | none | Number of models currently loaded by the router. |
| `lemonade_model_info` | gauge | `model_name`, `checkpoint`, `type`, `device`, `recipe` | Metadata sample for each loaded model. The sample value is always `1`. |

Per-model metrics use the same model labels: `model_name`, `checkpoint`, `type`, `device`, and `recipe`.

## Latest Per-Model Telemetry

| Metric | Type | Description |
|--------|------|-------------|
| `lemonade_model_input_tokens` | gauge | Input token count from the latest observed request for the model. |
| `lemonade_model_output_tokens` | gauge | Output token count from the latest observed request for the model. |
| `lemonade_model_prompt_tokens` | gauge | Prompt token count from the latest observed usage payload for the model. |
| `lemonade_model_time_to_first_token_seconds` | gauge | Latest time to first token in seconds. |
| `lemonade_model_tokens_per_second` | gauge | Latest generation throughput. |
| `lemonade_model_decode_token_time_count` | gauge | Number of per-token decode intervals captured for the latest streamed response. |
| `lemonade_model_decode_token_time_avg_seconds` | gauge | Average captured per-token decode interval for the latest streamed response. |
| `lemonade_model_decode_token_time_max_seconds` | gauge | Maximum captured per-token decode interval for the latest streamed response. |

## Cumulative Per-Model Counters

| Metric | Type | Description |
|--------|------|-------------|
| `lemonade_model_requests_total` | counter | Cumulative inference requests observed for the loaded model. |
| `lemonade_model_input_tokens_total` | counter | Cumulative input tokens observed for the loaded model. |
| `lemonade_model_output_tokens_total` | counter | Cumulative output tokens observed for the loaded model. |
| `lemonade_model_prompt_tokens_total` | counter | Cumulative prompt tokens observed for the loaded model. |

These counters are tied to the current `WrappedServer` instance. They reset when the model process is unloaded or the Lemonade server restarts.

## llama.cpp Backend Metrics

When a loaded model uses the `llamacpp` recipe, Lemonade scrapes the child llama.cpp server's private `/metrics` endpoint and republishes those samples with:

- metric names normalized from `llamacpp_*` to `lemonade_llamacpp_*`
- Lemonade model labels added to each sample
- duplicate backend `HELP` and `TYPE` lines suppressed across loaded llama.cpp models

The exact llama.cpp metric set depends on the bundled backend version. Common examples include queue depth, prompt/predicted token timing, and backend token counters.

## Configuration Metrics

| Metric | Type | Labels | Description |
|--------|------|--------|-------------|
| `lemonade_max_loaded_models` | gauge | `type` | Configured loaded-model limit for each model type. |

## Server-Wide Counters

| Metric | Type | Labels | Description |
|--------|------|--------|-------------|
| `lemonade_requests_total` | counter | none | Cumulative inference requests observed across loaded models. |
| `lemonade_input_tokens_total` | counter | none | Cumulative input tokens observed across loaded models. |
| `lemonade_output_tokens_total` | counter | none | Cumulative output tokens observed across loaded models. |
| `lemonade_prompt_tokens_total` | counter | none | Cumulative prompt tokens observed across loaded models. |

These totals are computed from the currently loaded model telemetry snapshot.

## System Metrics

| Metric | Type | Labels | Description |
|--------|------|--------|-------------|
| `lemonade_cpu_usage_percent` | gauge | none | Host CPU utilization percentage when available. |
| `lemonade_memory_used_gb` | gauge | none | Host memory usage in GiB when available. |
| `lemonade_gpu_usage_percent` | gauge | none | GPU utilization percentage when available. |
| `lemonade_vram_used_gb` | gauge | none | GPU memory usage in GiB when available. |
| `lemonade_npu_usage_percent` | gauge | none | NPU utilization percentage when available. |

Unavailable system values are omitted from the scrape instead of emitted as `NaN`.
