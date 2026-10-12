---
title: Authentication
sidebar_label: Authentication
---

{/* The heading anchors on this page are linked from the TPA Stream app. Keep them stable. */}

# Authentication

## SSH-JWT (preferred) {#ssh-jwt}

<a id="ssh-jwt-preferred"></a>

SSH-JWT is the **modern and preferred** way to authenticate with the
TPA Stream API.

:::tip Authenticating as an integration rather than a person?
Use a [service account](/api/service-accounts). It is a separate
account for the integration, with its own SSH keys and allowed IP
addresses, so it keeps working when staff change and shows up in the
audit log as the integration.
:::

### 1. Generate an SSH keypair

Generate an RSA, Ed25519, or ECDSA keypair as described in
[Generating an SSH Key](/api/ssh-keys).

### 2. Add your public key to your TPA Stream account

Upload the public key (for example `~/.ssh/id_ed25519.pub`) from the
avatar menu → **SSH keys & API access** → **New SSH key**
([`https://app.tpastream.com/d/account/access`](https://app.tpastream.com/d/account/access)).

You can upload **multiple keys** to the same account. The server tries
each one in turn during verification, so this is the right way to
rotate: upload the new key, leave the old one live until every client
has switched, then delete the old key.

### 3. Add your allowed IP addresses

Add at least one address or CIDR range your requests will come from
in the **Allowed IP addresses** section of the same
[API access](https://app.tpastream.com/d/account/access) page.

:::warning Required
Requests signed with your SSH key are accepted only from an IP address
in your list. With an empty list, **every request is refused with a
`401`**, even when the signature is valid. The page warns you when you
have keys but no addresses.
:::

See [Security](/api/security#ip-allowlist) for details.

### 4. Sign a JWT with your private key

The JWT must be:

- Signed with the **private key** whose public half you uploaded in
  step 2.
- Carry exactly these three claims: `email`, `iat`, `exp`.
- Use the algorithm matching your key type:

| Key type | JWT `alg` |
|---|---|
| RSA | `RS256` |
| Ed25519 | `EdDSA` |
| ECDSA P-256 | `ES256` |
| ECDSA P-384 | `ES384` |
| ECDSA P-521 | `ES512` |

`email` must match the email on your TPA Stream account
(case-insensitive). `iat` and `exp` are seconds since the Unix epoch;
a short TTL (15 minutes is plenty) is recommended. The server uses
**no clock-skew tolerance**, so a host with a drifted clock will see
tokens rejected as expired or "not yet valid" even when the TTL looks
right. Run NTP, and prefer building a fresh token per request over
caching one for hours.

You can reuse the same signed token across as many requests as you
like until `exp` lapses; each request stamps `last_accessed` on the
matching key but does not consume the token.

Complete working example in Python (`pip install pyjwt cryptography
requests`). It loads an Ed25519 or RSA private key, signs a token
that is valid for 15 minutes, and makes one request:

```python
import time

import jwt
import requests
from cryptography.hazmat.primitives import serialization
from cryptography.hazmat.primitives.asymmetric import ed25519, rsa

EMAIL = "you@example.com"  # the email on your TPA Stream account
KEY_PATH = "/path/to/id_ed25519"  # or /path/to/id_rsa

with open(KEY_PATH, "rb") as fh:
    data = fh.read()

# ssh-keygen writes "-----BEGIN OPENSSH PRIVATE KEY-----" files, which
# need load_ssh_private_key; PEM files need load_pem_private_key.
if b"OPENSSH PRIVATE KEY" in data:
    private_key = serialization.load_ssh_private_key(data, password=None)
else:
    private_key = serialization.load_pem_private_key(data, password=None)

if isinstance(private_key, ed25519.Ed25519PrivateKey):
    algorithm = "EdDSA"
elif isinstance(private_key, rsa.RSAPrivateKey):
    algorithm = "RS256"
else:
    raise ValueError("this example handles Ed25519 and RSA keys only")

now = int(time.time())
token = jwt.encode(
    {"email": EMAIL, "iat": now, "exp": now + 15 * 60},
    private_key,
    algorithm=algorithm,
)

response = requests.get(
    "https://app.tpastream.com/api/claims",
    headers={"Authorization": f"SSH-JWT {token}"},
)
response.raise_for_status()
print(response.json())
```

If your key file is passphrase-protected, pass the passphrase as bytes
in `password=`. For an ECDSA key, use `ES256`, `ES384`, or `ES512` to
match the curve (see the table above).

### 5. Make an authenticated API request

Include the signed token in the `Authorization` header. The scheme
**must** be `SSH-JWT` (case-insensitive), not the `Bearer` that most
JWT tooling defaults to. Sending `Authorization: Bearer <jwt>` falls
through to the legacy API-key parser and authentication fails.

```bash
curl -L \
  -H "Authorization: SSH-JWT <your_signed_jwt>" \
  https://app.tpastream.com/api/claims
```

The server verifies the signature against your registered public key.

## Common authentication mistakes {#common-authentication-mistakes}

If your request returns a 4xx, the response body's `message` field
will tell you what's wrong. The most frequent causes:

- **400 — JWT could not be decoded**: the token is malformed (bad
  base64, wrong number of segments, not actually a JWT). Re-encode
  with a known-good library.
- **400 — JWT missing required claim(s)**: at least one of `email`,
  `iat`, or `exp` is absent from the payload. All three are required.
- **401 — JWT could not be verified**: covers two cases on purpose.
  Either the token's signature didn't match any uploaded public key
  for that email, or the email isn't on file at all. The server
  returns the same response for both to keep unauthenticated clients
  from probing which emails have keys uploaded by watching the
  response shape. Confirm the email matches your account exactly and
  you are signing with the private key whose public half you uploaded
  in step 2.
- **401 — JWT has expired**: `exp` is in the past relative to server
  time. If your laptop clock is drifted this can fire even on freshly
  minted tokens, so check NTP before chasing token generation.
- **401 — IP not allowed**: the signature is valid, but the request's
  source IP is not in your allowed IP addresses (or the list is
  empty). Add the address under **Allowed IP addresses** on the
  [API access](https://app.tpastream.com/d/account/access) page
  (avatar menu → **SSH keys & API access**; step 3).
- **403** with no descriptive body: authentication succeeded but the
  user lacks permission for the endpoint.
- **Wrong algorithm**: the JWT `alg` header must match the key type
  you uploaded (see the table above). For example, an Ed25519 key
  paired with `RS256` will be rejected as `400 — JWT 'alg' header is
  'RS256'`.
- **DSA keys / unusual ECDSA curves**: TPA Stream accepts RSA,
  Ed25519, and ECDSA on the NIST P-256, P-384, and P-521 curves. DSA
  keys and ECDSA on other curves (e.g. `secp256k1`) are rejected at
  upload time. If you generated a DSA key, regenerate with one of the
  supported types (`ssh-keygen -t ed25519` is the modern default).

## API tokens (legacy) {#api-tokens-legacy}

API tokens (HTTP Basic Auth) are still supported for older
integrations but are **legacy**. Tokens are no longer issued, and you
cannot create or rotate one yourself. Existing tokens keep working.

To rotate or replace a token, switch the integration to a
[service account](/api/service-accounts) with an SSH key, which
authenticates with SSH-JWT as described above. If an integration
cannot adopt SSH keys yet, contact TPA Stream support: only TPA Stream
staff can issue a token, on request, for that case.

To use Basic authentication:

```bash
curl -L --user me@example.com:MY_API_KEY https://app.tpastream.com/api/claims
```

You may also send the same credentials base64-encoded in the
`Authorization` header:

```bash
curl -L \
  -H "Authorization: Basic $(echo -n 'me@example.com:MY_API_KEY' | base64)" \
  https://app.tpastream.com/api/claims
```

Notes:

- Basic Auth requests are rejected with a `401` if the originating IP
  address is not in the user's allowed IP addresses.
- An administrator manages the account's allowed IP addresses under
  **Manage Users** → the account's **API access** tab
  ([`https://app.tpastream.com/d/manage-users?includeServiceAccounts=true`](https://app.tpastream.com/d/manage-users?includeServiceAccounts=true)).
