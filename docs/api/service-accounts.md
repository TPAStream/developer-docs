---
title: Service Accounts
sidebar_label: Service Accounts
---

{/* The heading anchors on this page are linked from the TPA Stream app. Keep them stable. */}

# Service Accounts

A **service account** is a TPA Stream account that belongs to an
integration rather than to a person. It authenticates exactly the way
a person does, with an SSH key and a signed JWT. The difference is
whose identity the requests carry.

## Why use a service account

Use a service account for anything that runs unattended: a nightly
export, a data pipeline, a partner system that calls the API.

- **It survives staff changes.** If an integration signs requests with
  an employee's personal key, it breaks when that person leaves and
  their account is deactivated. A service account is not tied to any
  person.
- **The audit log names the integration.** Requests, key changes and
  allowed-IP changes are attributed to the service account, not to
  whichever employee happened to set it up.
- **It has its own IP allowlist.** You can limit the service account
  to the integration host's addresses without widening any person's
  list.

### When to use a personal key {#when-to-use-a-personal-key}

A personal SSH key (see [Authentication](/api/authentication)) is the
right choice for scripts and tools a person runs themselves:
command-line tools, ad hoc scripts, and exploring the API from a
laptop. Those requests should be attributed to that person.

A small team can also run its own integration on one person's key,
if it accepts the trade-off: the integration stops working when that
person's account is deactivated.

## Create a service account (administrator) {#create-a-service-account}

If you are an administrator, open
[https://app.tpastream.com/d/manage-users/new?kind=service](https://app.tpastream.com/d/manage-users/new?kind=service)
to go straight to the new-service-account form. Alternatively, create the
account in **Manage Users** (**New service account**).

Then add the integration's public key and allowed IP addresses on the
account's **API access** tab. Step-by-step with screenshots:
[https://support.tpastream.com/kb/giving-an-integration-api-access](https://support.tpastream.com/kb/giving-an-integration-api-access)
(requires a TPA Stream login).

## Connecting the integration (developer)

1. On the integration host, generate a keypair as described in
   [Generating an SSH Key](/api/ssh-keys).
2. Send the administrator the **public** half only (the `.pub` file).
   The private key never leaves the host.
3. Sign each JWT with the private key, and put the **service
   account's email** in the `email` claim:

   ```python
   now = int(time.time())
   token = jwt.encode(
       {
           "email": "nightly-export@your-company.example",  # the service account's email
           "iat": now,
           "exp": now + 900,
       },
       private_key,  # the integration host's private key
       algorithm="EdDSA",
   )
   ```

4. Send it as `Authorization: SSH-JWT <token>`.

Everything else (loading the key, choosing the algorithm, the
`Authorization` header, and troubleshooting 4xx responses) is the
same as for a person. See
[Authentication](/api/authentication#4-sign-a-jwt-with-your-private-key).

## Rotate and revoke keys {#rotate-and-revoke-keys}

A service account can have several keys at once. To rotate without
downtime:

1. Generate a new keypair and have an administrator add the new public
   key on the account's **API access** tab.
2. Switch the integration to the new private key.
3. Once the key's **Last used** date shows the new key in use, delete
   the old key.

To revoke a key immediately, an administrator deletes it from the
**API access** tab. Adding and deleting keys, and changing the
allowed IP addresses, are recorded in the audit log.

## Deactivating a service account

Deactivating the account in **Manage Users** disables all of its
keys. Requests signed with them are refused until the account is
reactivated.

## Legacy API tokens

Older integrations may still authenticate with an API token (HTTP
Basic Auth). Existing tokens keep working, but tokens are no longer
issued, and you cannot create or rotate one yourself. To rotate or
replace a token, move the integration to a service account with an
SSH key. If the integration cannot adopt SSH keys yet, contact
TPA Stream support. See
[API tokens (legacy)](/api/authentication#api-tokens-legacy).
