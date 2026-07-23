---
name: Subscribe and manage a user's marketing subscriptions
description: Opt a user into SMS/email marketing, check their subscription eligibility, and unsubscribe them, using the Attentive API v1.
api: openapi/attentive-v1-openapi.yml
operations: [addSubscriptions, getSubscriptions, unsubscribeSubscriptions]
---

# Subscribe and manage an Attentive user

Authenticate every request with `Authorization: Bearer <token>` (the token is a scoped API key from the Attentive product, or an OAuth access token). This flow needs the `subscriptions:write` scope.

## Steps

1. **Opt a user in** — `POST /subscriptions` (`addSubscriptions`). Send the user's `phone` in E.164 format (e.g. `+19148440001`) and/or `email`, plus either a `signUpSourceId` OR both a `locale` and `subscriptionType`. A legal disclosure is required for programmatic opt-in. Rate limit: 10 req/s.
2. **Check eligibility** — `GET /subscriptions` (`getSubscriptions`) with a single `phone` or `email` query param to list the subscription types/channels the user is subscribed to before messaging them.
3. **Opt a user out** — `POST /subscriptions/unsubscribe` (`unsubscribeSubscriptions`). With no subscriptions in the body the user is removed from all; otherwise only the specified type/channel. Note `email` only affects email subscriptions and `phone` only affects SMS. Rate limit: 5 req/s.

## Rules

- Phone numbers must be E.164; malformed numbers are rejected with `400`.
- A `409 Conflict` on opt-in means the subscription already exists.
- Handle `429` by backing off using the `x-ratelimit-remaining` header with exponential backoff + jitter.
- Errors are plain HTTP status + JSON message (no problem+json). See `errors/attentive-problem-types.yml`.
