---
layout: post
date: 2026-02-28

title: "The Cryptographic Foundations"
excerpt: "An advanced technical exploration of the cryptographic primitives that underpin TLS and certificate infrastructure — symmetric crypto, asymmetric keys, digital signatures, forward secrecy, and modern TLS design."

categories:
  - Security
  - Cryptography
  - Infrastructure
tags:
  - TLS
  - PKI
  - X509
  - Cryptography
  - Digital Signatures
  - RSA
  - ECC
  - ECDHE
  - Forward Secrecy
  - AEAD
  - TLS13

series: "Certificate Infrastructure Deep Dive"
part: 1
---

# Certificate Infrastructure Deep Dive - Part 1
## The Cryptographic Foundations

Modern certificate infrastructure rests on a small number of
cryptographic primitives.
If you understand these deeply, TLS and PKI stop being "magic" and start
being engineering.

This article assumes you are technically comfortable and want
protocol-level clarity --- not analogies about passports.

------------------------------------------------------------------------

# 1. Symmetric vs Asymmetric Cryptography

At the heart of TLS are two families of cryptography:

## Symmetric Cryptography

-   Same key used for encryption and decryption
-   Extremely fast
-   Suitable for bulk data transfer

Examples: - AES-GCM - ChaCha20-Poly1305

Symmetric crypto provides:

-   Confidentiality
-   Integrity (when used in AEAD modes)

But symmetric crypto has a distribution problem:

> How do two parties securely agree on a shared secret over an untrusted
> network?

That problem is solved using asymmetric cryptography.

------------------------------------------------------------------------

## Asymmetric Cryptography

Asymmetric systems use a **key pair**:

-   Public key (shared)
-   Private key (kept secret)

Two core uses:

1.  Encryption (rare in modern TLS)
2.  Digital signatures (critical for certificates)

Examples: - RSA - ECDSA - Ed25519

Key insight:

> Asymmetric crypto is slow and computationally expensive.

It is not used for bulk encryption --- only for identity and key
agreement.

------------------------------------------------------------------------

# 2. Digital Signatures

Certificates are fundamentally about **digital signatures**.

A digital signature provides:

-   Authenticity (who signed it)
-   Integrity (was it modified?)
-   Non-repudiation (in some contexts)

Mechanism:

1.  Hash the message
2.  Encrypt the hash with the private key
3.  Verify using the public key

Important distinction:

Encryption protects confidentiality.
Signatures protect integrity and authenticity.

TLS relies on signatures to validate:

-   Certificates
-   Handshake messages
-   Key exchange parameters

------------------------------------------------------------------------

# 3. Hash Functions

Hash functions convert arbitrary data into fixed-length output.

Properties required for TLS:

-   Preimage resistance
-   Second preimage resistance
-   Collision resistance

Modern TLS uses:

-   SHA-256
-   SHA-384

Why this matters:

Certificates sign the hash of data --- not the raw data.
The security of signatures depends on the hash being
collision-resistant.

If collisions become feasible, signature trust collapses.

------------------------------------------------------------------------

# 4. Key Exchange and Forward Secrecy

Modern TLS does **not** use RSA key transport anymore.

Instead, it uses ephemeral key exchange:

-   Diffie-Hellman (DH)
-   Elliptic Curve Diffie-Hellman (ECDHE)

Key idea:

Two parties can derive a shared secret without transmitting it.

### Why Ephemeral?

Ephemeral key exchange provides:

> Forward Secrecy (PFS)

If a server's private key is compromised later:

-   Past TLS sessions remain secure
-   Recorded traffic cannot be decrypted

Without PFS:

-   Compromise of one private key can decrypt historical traffic

This is why RSA key exchange was removed in TLS 1.3.

------------------------------------------------------------------------

# 5. RSA vs ECC

## RSA

-   Based on integer factorization
-   Large key sizes (2048--4096 bits)
-   Slower
-   Mature and widely supported

## Elliptic Curve Cryptography (ECC)

-   Based on discrete logarithm over elliptic curves
-   Smaller key sizes
-   Faster
-   Stronger per bit of key length

Example equivalence:

-   RSA 2048 ≈ ECC 256-bit security

Most modern TLS deployments prefer:

-   ECDHE for key exchange
-   ECDSA for signatures

Because:

-   Better performance
-   Lower CPU usage
-   Smaller handshake size

------------------------------------------------------------------------

# 6. Authenticated Encryption (AEAD)

After handshake completes, TLS switches to symmetric encryption.

Modern TLS uses AEAD modes:

-   AES-GCM
-   ChaCha20-Poly1305

AEAD provides:

-   Confidentiality
-   Integrity
-   Replay protection
-   Authenticated additional data (AAD)

Older constructions like:

-   AES-CBC + HMAC

Are deprecated due to padding oracle vulnerabilities and complexity.

TLS 1.3 removed non-AEAD ciphers entirely.

------------------------------------------------------------------------

# 7. TLS 1.2 vs TLS 1.3 Cryptographic Differences

TLS 1.3 simplified and hardened cryptography:

Removed: - RSA key exchange - Static DH - CBC cipher suites - SHA-1
usage in handshake

Mandated: - Ephemeral key exchange - AEAD ciphers - HKDF-based key
schedule

TLS 1.3's key schedule is derived through HKDF chaining, ensuring:

-   Clean separation of secrets
-   Strong forward secrecy
-   Reduced attack surface

------------------------------------------------------------------------

# 8. Threat Model Considerations

Cryptography is only meaningful relative to threat models.

Modern TLS defends against:

-   Passive eavesdropping
-   Active MitM attempts
-   Replay (with caveats in 0-RTT)
-   Key compromise (limited by PFS)

It does not defend against:

-   Compromised endpoints
-   Malicious root CAs
-   BGP hijacking without additional controls
-   Application-layer vulnerabilities

Understanding these boundaries is critical.

------------------------------------------------------------------------

# 9. Why This Foundation Matters

Everything in certificate infrastructure builds on:

-   Hash functions
-   Digital signatures
-   Asymmetric identity validation
-   Ephemeral key agreement
-   Symmetric authenticated encryption

In the next part, we move from primitives to protocol:

> How the TLS handshake actually works on the wire.

------------------------------------------------------------------------

Understanding cryptography removes mystery from certificates.

Once the primitives are clear, PKI becomes architecture --- not magic.
