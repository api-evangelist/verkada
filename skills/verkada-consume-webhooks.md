---
name: Consume and verify Verkada webhooks
description: Stand up a Verkada Command webhook endpoint and verify HMAC-SHA256 event signatures before processing.
api: https://apidocs.verkada.com/reference/event-based-webhooks
operations: []
---

# Consume and verify Verkada webhooks

Verkada pushes real-time events (cameras, access, new alarms, guest) to an HTTPS endpoint you configure in Command (Admin > APIs and Integrations > Webhooks) with a URL and a shared secret. This flow is push-only — there are no polling operationIds.

## Endpoint requirements
- HTTPS with a trusted-CA certificate; accept and return JSON.
- Return a `2xx` within **2000 ms** (before heavy logic) or Verkada retries once and then drops the event.
- Handle concurrent requests.

## Verify the signature (do this first)
1. Read the `Verkada-Signature` header; split on `|` into `timestamp` and `signature`.
2. Build `signed_payload = <raw_json_body> + "|" + timestamp`.
3. Compute `HMAC-SHA256(shared_secret, signed_payload)` and hex-encode it.
4. Constant-time compare against `signature`. Reject (`403`) on mismatch.
5. Reject stale events: if `now - timestamp` exceeds your tolerance (docs sample: 60s), treat as replay and drop.

## Process the payload
- Common envelope: `org_id`, `webhook_type`, `created_at`, `webhook_id`, `data{...}`.
- `data.alert_type` + `data.product_type` classify the event; `data.details{}` carries event-specific fields (e.g. `object_type` for motion/line-crossing, `activity_type` for activity detection, `ai_query` for AI-powered alerts).

## Rules
- Only act after the signature and timestamp checks pass.
- See `asyncapi/verkada-webhooks.yml` for the full event catalog and `conventions/verkada-conventions.yml` for signing details.
