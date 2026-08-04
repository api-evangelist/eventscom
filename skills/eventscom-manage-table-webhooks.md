---
name: Configure and test a DataGol table webhook
description: Create, test, update and remove an outbound HTTP webhook on an Events.com DataGol table.
api: openapi/eventscom-datagol-platform-openapi.yml
generated: '2026-08-04'
method: generated
operations:
  - getWebhookList
  - testWebhook_2
  - createWebhook
  - testWebhook
  - testWebhook_1
  - updateWebhook
  - deleteWebhook
---

# Configure and test a DataGol table webhook

DataGol table webhooks are a **generic HTTP notifier**: the subscriber supplies the method, URL, headers,
query parameters and payload template, and DataGol fires it on a table event.

## Before you start

- **Auth.** `Authorization: Bearer <jwt>` (`s_jwt`).
- **Base URL.** `https://datagol-be.events.com/`
- **There is no signature scheme.** The spec declares no HMAC signing secret and no signature header, so a
  receiver **cannot cryptographically verify** that a delivery came from DataGol. If you own the receiver,
  put a shared secret in `httpNotificationDetails.headerDetails` yourself and check it, and terminate TLS
  on an endpoint that is not guessable.
- **There is no retry policy.** Delivery guarantees are undocumented — make your receiver idempotent and
  reconcile against the table rather than trusting delivery.

## The subscription object

`WebhookEventDTO`:

| Field | Type | Notes |
|---|---|---|
| `id` | string | server-assigned |
| `title` | string | display name |
| `tableId` | string | the table to watch |
| `tableName` | string | |
| `eventName` | string | **free-form; no enum and no published list of valid values** |
| `notificationType` | string | |
| `httpNotificationDetails` | object | `{method, url, headerDetails, parameterDetails, payload}` |

The delivered row payload is `WebhookEventDataDTO`: `{id, tableId, tableName, data}` — note `data` arrives
as a **string**, not a nested object, so parse it before use.

## Steps

1. **See what already exists** — `getWebhookList`
   `GET /noCo/api/v1/tables/webhooks`
   Check for a duplicate before creating; there is no idempotency key protecting you from double-creates.

2. **Fetch the sample payload first** — `testWebhook_2`
   `GET /noCo/api/v1/tables/webhooks/test/payload`
   Use this to learn the real delivery shape before you build the receiver.

3. **Create the subscription** — `createWebhook`
   `POST /noCo/api/v1/tables/webhooks` with a `WebhookEventDTO` body.

4. **Fire a test delivery** — `testWebhook`
   `POST /noCo/api/v1/tables/webhooks/test`
   To test against an explicit payload instead, use `testWebhook_1`
   (`POST /noCo/api/v1/tables/webhooks/test/payload`).

5. **Update** — `updateWebhook`
   `PUT /noCo/api/v1/tables/webhooks/{id}`

6. **Remove** — `deleteWebhook`
   `DELETE /noCo/api/v1/tables/webhooks/{id}`

## Do not confuse these with

- `sendNotification` (`POST /noCo/api/v1/tables/webhooks/notify`) dispatches a notification; it does not
  manage subscriptions.
- `listenToSNSMessages` (`POST /workflow/api/v1/sns/webhook`) is an **inbound** AWS SNS receiver on
  Events.com's side, and is marked `deprecated: true`.
- The `/api/v1/alerts/notification-history/*` family is a **poll-based** alert log, not a push surface.

See `asyncapi/eventscom-webhooks.yml` for the full catalog.
