---
title: Security (IP Allowlist)
sidebar_label: Security
---

# Security (IP Allowlist)

For additional security, every user must have at least one allowed
IP address or CIDR range. A request from any other address is refused
with a `401`, and with an empty list **every request is refused**.
The range may be `0.0.0.0/0` (meaning no restriction), but we
**highly recommend** a more restrictive range to provide an extra
layer of protection.

Configure the list under **Settings**:

- Users who authenticate with their own SSH key edit **their own**
  list on the
  [SSH Settings](https://app.tpastream.com/settings/ssh) page, in the
  **Allowed IP addresses** section next to their keys. Enter single
  addresses (`203.0.113.5`) or CIDR ranges (`203.0.113.0/24`); IPv4
  and IPv6 are accepted. Saving replaces the whole list.
- Legacy API users set theirs on the
  [Manage API Access](https://app.tpastream.com/settings/api-users)
  page.

The allowlist applies to **both** authentication methods:

- For [SSH-JWT](/api/authentication#ssh-jwt-preferred), a request
  with a valid signature whose source IP isn't in the allowlist gets
  a `401`.
- For [API Token Basic Auth](/api/authentication#api-tokens-legacy),
  the same allowlist is enforced, also with a `401`.
