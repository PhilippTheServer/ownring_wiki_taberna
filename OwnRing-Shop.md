---
title: OwnRing Shop
description: How the OwnRing shop is built on top of OpenTaberna
published: true
date: 2026-10-07T12:00:00.000Z
tags: ownring, shop, deployment, payments
editor: markdown
dateCreated: 2026-10-07T12:00:00.000Z
---

# OwnRing Shop

This wiki documents the OpenTaberna stack. The **OwnRing shop** is a concrete
deployment of that stack, forked and branded for the OwnRing R02 smart ring.
This page describes the OwnRing-specific parts; everything generic is on the
surrounding pages.

> The authoritative documentation for the shop as a whole lives in the
> [`ownring_shop`](https://github.com/PhilippTheServer/ownring_shop) umbrella
> repository, vendored as a submodule in each shop repo at `docs/ownring_shop`.
> This page is a summary that points back there.

## What it sells

At launch the shop sells one product:

- **OwnRing R02** — a wellness/fitness smart ring (sleep, activity, heart
  rate) plus the OwnRing app bundle, at **€99** (EUR, incl. VAT).

The product message: **no cloud, no account, no subscription** — data stays on
the device and in the app (Android / GrapheneOS). The ring is a wellness
device and **must never be marketed as detecting illness** (MDR/HWG).

Wristbands and further rings will follow. The catalog is data, not code, so a
new product is an admin operation (or a row in the backend's
`seed_products` script) — no frontend change.

## The stack

The shop runs the standard OpenTaberna stack, forked under
`PhilippTheServer`:

| Repository | Role |
|---|---|
| [`ownring_fastapi_taberna`](https://github.com/PhilippTheServer/ownring_fastapi_taberna) | FastAPI backend — catalog, orders, payments, webhooks |
| [`ownring_shop-frontend_taberna`](https://github.com/PhilippTheServer/ownring_shop-frontend_taberna) | Angular storefront (customer-facing, DE/EN) |
| [`ownring_admin-frontend_taberna`](https://github.com/PhilippTheServer/ownring_admin-frontend_taberna) | Angular admin dashboard |
| [`ownring_wiki_taberna`](https://github.com/PhilippTheServer/ownring_wiki_taberna) | This wiki |

Infrastructure (Postgres 17, Redis 8, Keycloak 26, Garage/S3) is the same as
described on [Getting Started](/Getting-Started).

## Payments: Stripe **and** PayPal

OpenTaberna ships with Stripe. The OwnRing fork adds a second provider behind
the same `PaymentProviderAdapter` interface:

- **Stripe** (`StripeAdapter`) — card, via the pre-existing PaymentIntent flow.
- **PayPal** (`PayPalAdapter`) — Checkout smart buttons / Orders API, added for
  OwnRing.

The customer chooses **Card** or **PayPal** at checkout. The backend's
`POST /orders/{id}/checkout?provider=stripe|paypal` selects the adapter and
returns the provider plus the `client_secret` (a Stripe PaymentIntent secret,
or the PayPal approval link). Webhooks arrive at
`/webhooks/stripe` and `/webhooks/paypal`, both idempotent.

See [Payments](/Orders-and-Fulfillment) for the order lifecycle the webhooks
drive.

## App delivery

After a successful purchase the customer gets the OwnRing app:

- **Android** — Play Store link **and** a direct signed-APK download.
- **iOS** — a note that the app is not yet on the App Store (no fake link).

The links and the iOS note come from the product's `custom.app` field in the
backend catalog (package id `app.ownring`, domain `ownring.app`).

## Legal

The shop is a German distance-sale business run as a **Kleinunternehmer**
(§ 19 UStG): no VAT is charged (the €99 is the final price) and the
founding-year turnover is capped at €25,000 (~250 rings at €99).

Required pages, all in the storefront footer:

- **Impressum** — Philipp Lehmann, Bochum, Deutschland.
- **Datenschutzerklärung** (DSGVO) — Stripe and PayPal are named as payment
  processors (Auftragsverarbeiter, Art. 28).
- **Widerrufsbelehrung** — 14-day withdrawal right with the mandatory
  withdrawal button.
- **GPSR** product-safety information for the ring.

Full detail, plus the open legal blockers (ElektroG, battery regulation, LUCID,
product liability, the midnight-date bug #113), is in the umbrella's
`docs/legal.md`.

## Run it

The umbrella repo ships a local/dev `docker-compose.yml` that starts the
backend, both frontends, and the infrastructure together. See the umbrella's
`docs/deployment.md` for ports and the secret checklist (Stripe, PayPal,
Garage, Keycloak). Local-only for now — there is no cluster deployment yet.
