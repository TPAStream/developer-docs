---
title: Security
sidebar_label: Security
---

{/* The heading anchors on this page are linked from the TPA Stream app. Keep them stable. */}

# Security

## IP allowlist {#ip-allowlist}

For additional security, every user must have at least one allowed
IP address or CIDR range. A request from any other address is refused
with a `401`, and with an empty list **every request is refused**.
The range may be `0.0.0.0/0` (meaning no restriction), but we
**highly recommend** a more restrictive range to provide an extra
layer of protection.

Edit your own list in the **Allowed IP addresses** section of the
[API access](https://app.tpastream.com/d/account/access) page (avatar
menu → **SSH keys & API access**). For a
[service account](/api/service-accounts), an administrator edits the
list on the account's **API access** tab.

The allowlist applies to **both** authentication methods:

- For [SSH-JWT](/api/authentication#ssh-jwt), a request
  with a valid signature whose source IP isn't in the allowlist gets
  a `401`.
- For [API Token Basic Auth](/api/authentication#api-tokens-legacy),
  the same allowlist is enforced, also with a `401`.

## Egress IP addresses {#egress-ip-addresses}

TPA Stream's outbound traffic, such as webhook posts and file
deliveries, comes from the source addresses published at
[`https://app.tpastream.com/ip-addresses.json`](https://app.tpastream.com/ip-addresses.json).
If your firewall restricts inbound traffic, allowlist that published
list rather than individual addresses, because the addresses can
change.
