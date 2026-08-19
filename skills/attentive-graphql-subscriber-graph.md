---
name: Query and mutate the Attentive subscriber graph over GraphQL
description: >-
  Use Attentive's GraphQL API (beta) to identify the calling app, look a user up by phone or
  email, run a filtered user search that REST cannot do, and opt users in or out - with the
  scope, rate-limit and error rules that apply.
api: graphql/attentive.graphql
endpoint: https://api.attentivemobile.com/v1/graphql
operations: [viewer, subscribe, unsubscribe, setCustomAttributes, createCustomEvent, createWebhook, updateWebhook, deleteWebhook]
method: generated
source: >-
  graphql/attentive.graphql (anonymous introspection of https://api.attentivemobile.com/v1/graphql,
  2026-08-13), https://docs.attentive.com/reference/graphql-introduction
---

# Attentive GraphQL — subscriber graph

Every field and mutation named below was read from a live introspection of
`https://api.attentivemobile.com/v1/graphql` on 2026-08-13. Nothing here is invented; if a field
is not in `graphql/attentive.graphql`, it does not exist.

## Before you call

- **Endpoint**: `POST https://api.attentivemobile.com/v1/graphql`. One URL, one method.
- **Auth**: `Authorization: Bearer <application token>` — the *same* token the REST API uses.
  Introspection answers without a token; nothing else does.
- **Scopes**: fields and mutations are gated by the scopes attached to your app, so a valid token
  can still be refused per-field. Scope names are the REST ones (see `scopes/attentive-scopes.yml`).
- **Status**: the graph is in **beta**. Prefer REST for anything in
  `mcp/attentive-tool-crosswalk.yml` `rest_only[]` — bulk ingestion, segments, product catalog,
  privacy requests, identity resolution and OAuth token exchange have no GraphQL equivalent.
- **Rate limits**: the published per-second limits in `rate-limits/attentive-rate-limits.yml` are
  the operative ceiling (default 150 rps; subscribe/unsubscribe are the tight ones at 10 and 5 rps).
  On `429`, back off exponentially with jitter.
- **No idempotency**: Attentive documents no idempotency key on any surface
  (`conventions/attentive-conventions.yml`). A retried `subscribe` is a *second* opt-in attempt,
  which for a TEXT subscription can send the person another "you are already subscribed" message.
  Retry only on transport failures, never blindly on a timeout.

## 1. Confirm who the token is

```graphql
query {
  viewer {
    installedApplication {
      id
      application { name }
      installerCompany { id name }
    }
  }
}
```

This is the GraphQL form of the REST `getMe` / `getMeV2` test-authentication calls. Run it first;
it is the cheapest way to prove the token and scopes are live.

## 2. Look one user up

```graphql
query($phone: String) {
  viewer { installedApplication { installerCompany {
    user(phone: $phone) { ... }
  } } }
}
```

`Company.user` takes `phone` **or** `email`. Phone numbers must be E.164 (`+19148440001`). One
GraphQL read here replaces two REST reads (`getSubscriptions` + `getCustomAttributes`), because the
`User` type carries both subscription eligibility and custom attributes.

## 3. Search users — the thing REST cannot do

`Company.usersExperimental(filter, first, after)` is a Relay connection with a rich
`ListUsersFilter` (subscription opt-in, location, click events, purchase events, attribute
predicates). **There is no public REST operation that lists or filters users** — REST can only look
one user up by phone or email. Page with `first` + `after` from `pageInfo.endCursor`; never assume
a page size. The field is named *experimental*: treat its shape as unstable and pin nothing to it.

## 4. Opt a user in and out

```graphql
mutation { subscribe(input: {...}) { ... } }
mutation { unsubscribe(input: {...}) { ... } }
```

The legal rules from the REST docs apply unchanged: a programmatic opt-in requires the disclosure
language, and the request must carry either a `signUpSourceId` **or** both a `locale` and a
`subscriptionType` that resolve to exactly one API sign-up unit. Zero or multiple matches is a
`400`, and no subscription is created.

## 5. Events, attributes and webhooks

Mutations that mirror REST 1:1 — use whichever surface your client already speaks:

| GraphQL mutation | REST operationId |
|---|---|
| `createCustomEvent` | `postCustomEvents` |
| `setCustomAttributes` | `postCustomAttributes` |
| `productViewEvent` | `postProductViewEvents` |
| `addToCartEvent` | `postAddToCartEvents` |
| `productPurchaseEvent` | `postPurchaseEvents` |
| `createWebhook` / `updateWebhook` / `deleteWebhook` | `createWebhook` / `updateWebhook` / `deleteWebhook` |
| `InstalledApplication.webhooks` | `getWebhooks` |

Attribute values must be scalars — passing an array or a map (`["chicago","new york"]`) is a `400`,
same as on REST.

## Errors

Attentive does not use RFC 9457 problem+json on either surface
(`errors/attentive-problem-types.yml`). GraphQL returns HTTP 200 with an `errors[]` array on the
body, so **check `errors[]` even on a 200** — a naive `if (res.ok)` will read a failed mutation as a
success. Responses also carry `extensions.traceId`; log it, it is the only correlation handle.
