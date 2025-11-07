.. _api:

REST API
========

Authentication
--------------

TPA Stream’s REST API supports **two authentication methods**:

+---------------------------+----------------+-------------+----------------------------------------------+
| Method                    | Status         | Performance | Notes                                        |
+===========================+================+=============+==============================================+
| **SSH-JWT**               | Recommended ✅ | Fast        | Modern, secure, key-based authentication     |
+---------------------------+----------------+-------------+----------------------------------------------+
| **API Token (Basic Auth)**| Deprecated ⚠️  | Slower      | Still supported for backwards compatibility  |
+---------------------------+----------------+-------------+----------------------------------------------+

All API requests must be made over **HTTPS**.

You can browse and test all available API endpoints using our interactive OpenAPI documentation:

`TPA Stream API Reference (RapiDoc) <https://app.tpastream.com/rapidoc>`_

SSH-JWT (Preferred)
-------------------

SSH-JWT authentication is the **modern and preferred** way to authenticate with the TPA Stream API.

1. **Generate an SSH keypair**

   Run the following command in your terminal (Linux, macOS, or Git Bash on Windows):

   .. code-block:: bash

      ssh-keygen

   - When prompted for a file path, press Enter to accept the default.
   - You may optionally set a passphrase.

2. **Add your public key to your TPA Stream account**

   Visit your account settings at:

   https://app.tpastream.com/settings/ssh

   Paste the contents of your **public key file** (for example, ``~/.ssh/id_rsa.pub``).

3. **Make an authenticated API request**

   Use your **private key** to sign a JWT, or use a helper tool or SDK that does so automatically.
   Then include the signed token in your HTTP request as follows:

   .. code-block:: bash

      curl -L \
        -H "Authorization: SSH-JWT <your_signed_jwt>" \
        https://app.tpastream.com/api/claims

   The server will verify your JWT using your registered public key.

4. **Optional IP Allowlist**

   To provide an extra layer of security, each user can specify an **IP CIDR allowlist** (for example, ``10.0.0.0/8``).
   API requests originating outside this range will be **rejected**.

API Tokens (Legacy)
-------------------

API Tokens (Basic Auth) are still supported for older integrations but are **deprecated**.
We recommend migrating to SSH-JWT authentication for improved performance and security.

To use Basic Authentication:

.. code-block:: bash

   curl -L --user me@example.com:MY_API_KEY https://app.tpastream.com/api/claims

You may also send the same credentials base64-encoded in the ``Authorization`` header:

.. code-block:: bash

   curl -L \
     -H "Authorization: Basic $(echo -n 'me@example.com:MY_API_KEY' | base64)" \
     https://app.tpastream.com/api/claims

**Notes:**

* Basic Auth requests will be rejected if the originating IP address is not in the user’s allowlist.
* You can manage allowed IP ranges on the `Manage Users <https://app.tpastream.com/settings/users>`_ page.


Security
--------

For additional security, you must provide an IP CIDR address range.
This range may be ``0.0.0.0/0`` (meaning no restriction),
however, we **highly recommend** a more restrictive range to provide an extra layer of protection.

This range can be configured on the
`Manage Users <https://app.tpastream.com/settings/users>`_ page under Settings.


Generating an SSH Key
---------------------

To generate an SSH keypair:

1. Open your Terminal (on Linux/Mac) or Git Bash (Windows)
2. Paste the following command:

   ``ssh-keygen``

   This creates a new SSH keypair.

   a. When prompted to "Enter a file in which to save the key," press Enter to accept the default location.

      ``Enter a file in which to save the key (/home/you/.ssh/algorithm): [Press enter]``

   b. At the prompt, you may enter a secure passphrase (optional).

      ``Enter passphrase (empty for no passphrase): [Type a passphrase]``
      ``Enter same passphrase again: [Type passphrase again]``


Generating a GPG Key
--------------------

To generate a GPG keypair:

1. Download and install the `GPG command line tools <https://www.gnupg.org/download/>`_.
2. Open Terminal.
3. Generate a GPG key pair:

   ``gpg --full-generate-key``

4. Accept defaults unless you need custom options (use 4096-bit keys).
5. Use the following command to list your keys:

   .. code-block:: bash

      gpg --list-secret-keys --keyid-format=long

6. Export your public key:

   .. code-block:: bash

      gpg --armor --export YOUR_KEY_ID

7. Copy everything between:

   ``-----BEGIN PGP PUBLIC KEY BLOCK-----`` and ``-----END PGP PUBLIC KEY BLOCK-----``

8. Add your GPG key to your TPA Stream account.


A Note About Date Ranges
------------------------

Types of data that are date ranges are serialized to JSON as an object with bounds:

.. code-block:: json

    {
      "bounds": "[)",
      "end": "2025-09-16",
      "start": "2025-09-15"
    }

Since a date of service may span multiple days (for example, a hospital stay),
it is stored as a range.

*In text form, an inclusive lower bound is represented by "[" while an exclusive lower bound is "(".
Likewise, an inclusive upper bound is represented by "]", and an exclusive upper bound is ")".*

For more details and examples, see:
https://www.postgresql.org/docs/current/rangetypes.html#RANGETYPES-INCLUSIVITY

