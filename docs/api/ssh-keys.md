---
title: Generating an SSH Key
sidebar_label: Generating an SSH Key
---

{/* The heading anchors on this page are linked from the TPA Stream app. Keep them stable. */}

# Generating an SSH Key

To generate an SSH keypair:

1. Open your Terminal (Linux / macOS) or Git Bash (Windows).
2. Paste one of the following commands, depending on which key type
   you want to use:

   ```bash
   # Ed25519 (recommended for new keys: short, fast, modern)
   ssh-keygen -t ed25519
   # RSA (4096-bit, broadly compatible)
   ssh-keygen -t rsa -b 4096
   # ECDSA on NIST P-521
   ssh-keygen -t ecdsa -b 521
   ```

   This creates a new SSH keypair.

   a. When prompted to "Enter a file in which to save the key," press
      Enter to accept the default location:

      ```
      Enter a file in which to save the key (/home/you/.ssh/id_<type>): [Press enter]
      ```

   b. At the prompt, you may enter a secure passphrase (optional):

      ```
      Enter passphrase (empty for no passphrase): [Type a passphrase]
      Enter same passphrase again: [Type passphrase again]
      ```

Once the keypair is created, upload the **public** half on the
[API access](https://app.tpastream.com/d/account/access) page (avatar
menu → **SSH keys & API access** → **New SSH key**). For a key that
belongs to an integration rather than a person, an administrator adds
it to a [service account](/api/service-accounts) instead.
See [Authentication](/api/authentication#2-add-your-public-key-to-your-tpa-stream-account)
for the rest of the setup, including the **required** step of adding
at least one allowed IP address on the same page. Until you do, every
request is refused with a `401`.

## Fingerprints {#fingerprints}

The keys table on the API access page shows each key's **SHA256
fingerprint**: a short hash of the public key, such as
`SHA256:3x4mpl3...`. It identifies the key without showing the key
itself.

To print the fingerprint of a key you hold, run:

```bash
ssh-keygen -lf ~/.ssh/id_ed25519.pub
```

Use the path to your own `.pub` file. If the `SHA256:` value printed
matches the one shown in the app, the key you uploaded is the key you
are signing with. If they differ, you uploaded a different key, and
your requests will fail with `401 — JWT could not be verified`.
