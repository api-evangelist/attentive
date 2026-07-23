---
name: Subscribe to and manage Attentive webhooks
description: List, create, update, and delete webhook subscriptions so your system receives Attentive SMS/email engagement events.
api: openapi/attentive-v1-openapi.yml
operations: [getWebhooks, createWebhook, updateWebhook, deleteWebhook]
---

# Manage Attentive webhooks

Authenticate with `Authorization: Bearer <token>`; webhook management uses the `webhooks:write` scope.

## Steps

1. **List existing webhooks** — `GET /webhooks` (`getWebhooks`).
2. **Create a subscription** — `POST /webhooks` (`createWebhook`) with a delivery `url` and an `events` array. Event strings are case-sensitive; valid values include `sms.subscribed`, `sms.sent`, `sms.message_link_click`, `sms.inbound_message`, `email.subscribed`, `email.unsubscribed`, `email.sent`, `email.opened`, `email.message_link_click`, `custom_attribute.set`.
3. **Update a subscription** — `PUT /webhooks/{webhookId}` (`updateWebhook`).
4. **Delete a subscription** — `DELETE /webhooks/{webhookId}` (`deleteWebhook`).

## Rules

- All requested events are delivered to the single configured URL via HTTP POST; payloads carry `type`, `timestamp` (ms), `company`, and `subscriber`.
- Verify delivery authenticity per https://docs.attentive.com/docs/webhook-authentication.
- See the full event/webhook surface in `asyncapi/attentive-webhooks.yml`.
