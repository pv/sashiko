# DESIGN: Request Extras for OpenAI-Compatible Providers

## Context

Endpoints expose different request controls. A pass-through table supports
these without adding a typed setting for each endpoint-specific field.

## Decisions

| Aspect | Choice |
|---|---|
| Setting | `request_extra` in `[ai.openai_compat]`, default empty; no legacy alias |
| Scope | Both `openai` and `openai-compatible` |
| Wire format | Flatten a sorted map into the request body; preserve nested values |
| Reserved keys | Reject `model`, `messages`, `tools`, `temperature`, `max_tokens`, `max_completion_tokens`, `response_format`, and `stream` (streaming unsupported) at construction |
| Validation | Name conflicting keys; endpoint-specific values are validated by the endpoint |
| Retries | Preserve extras when retrying without an unsupported temperature |
| Response cache | Canonical extras and echo mode partition the cache; a reasoning version excludes older responses that discarded thoughts |
| Reasoning | Preserve received `reasoning_content` as `thought`; echo unchanged only when opted in |
| Echo default | `send_reasoning_content = false` in both modes for backward compatibility; enabling it partitions the cache |

## Configuration

```toml
[ai.openai_compat]
request_extra = { reasoning_effort = "high" }
# Enable only if the endpoint accepts reasoning_content.
# send_reasoning_content = true
```

For endpoints supporting nested controls, use
`request_extra = { chat_template_kwargs = { enable_thinking = true } }`.

## Verification

Cover nested serialization, reserved-key rejection, stable cache partitions,
empty defaults, retry preservation, and reasoning preservation with echo disabled.
