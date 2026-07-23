---
name: Send ecommerce events to Attentive
description: Report product views, add-to-cart, and purchase events for a user so Attentive can trigger and personalize messaging.
api: openapi/attentive-v1-openapi.yml
operations: [postProductViewEvents, postAddToCartEvents, postPurchaseEvents]
---

# Send Attentive ecommerce events

Authenticate with `Authorization: Bearer <token>`. This flow needs the `ecommerce:write` scope. Default rate limit 150 req/s.

## Steps

1. **Product view** — `POST /events/ecommerce/product-view` (`postProductViewEvents`) when a user views a product. Identify the user by `phone`/`email`/client identifier and include the product payload.
2. **Add to cart** — `POST /events/ecommerce/add-to-cart` (`postAddToCartEvents`) when a user adds a product to their cart.
3. **Purchase** — `POST /events/ecommerce/purchase` (`postPurchaseEvents`) when a user completes an order.

## Rules

- Each event associates to a user via the same identifier types used across the API (phone in E.164, email, Shopify/Klaviyo/custom ids).
- These endpoints power triggered journeys (e.g. abandoned cart), so send them in real time.
- Handle `400` (bad payload), `401`/`403` (token/scope), and `429` (rate limit) per `errors/attentive-problem-types.yml`.
