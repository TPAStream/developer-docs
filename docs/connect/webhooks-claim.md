---
title: Claim Webhook
sidebar_label: Claim Webhook
---

# Claim Webhook

TPA Stream POSTs new claims to a customer-provided URL.

## Trigger

The claim webhook fires whenever a new claim is processed in TPA
Stream.

## Configuring the URL

Your TPA Stream administrator sets the destination URL under Settings > Webhook Settings and can replay a delivery from the claims page. Step-by-step (requires a TPA Stream login): https://support.tpastream.com/kb/setting-up-webhooks

## Request shape

POST with `Content-Type: application/json`. See
[Webhook Examples](/connect/webhook-examples) for the full payload
shape, and [Claim Types](/api/claim-types) for the values the `type`
field can take.

## Retry behavior and 406

See [Webhook Security](/connect/webhook-security#response-codes-and-retries)
for the full table.
