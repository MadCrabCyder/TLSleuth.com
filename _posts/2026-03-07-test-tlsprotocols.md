---
layout: post
date: 2026-03-07 09:00:00 +1100
title: "Testing Supported TLS Protocols with TLSleuth"
excerpt: "TLSleuth adds Test-TLSleuthProtocol, a PowerShell command that tests which TLS protocol versions actually succeed against a target endpoint."

tags:
  - powershell
  - tls
  - security
  - tlsleuth
  - networking
---

# Testing Supported TLS Protocols with TLSleuth

TLS endpoints often support a subset of the TLS protocol versions that exist in a runtime environment.
Determining which protocol versions actually succeed usually requires running multiple connection attempts manually.

TLSleuth introduces a new command to simplify this workflow:

```
Test-TLSleuthProtocol
```

This command attempts a TLS handshake using each protocol version available in the current runtime and returns a structured result for each attempt.

This feature is scheduled for the **next release after TLSleuth 2.2.0**, with a release date of **2026‑03‑07**.

---

# Why This Feature Matters

Before this feature existed, validating supported TLS protocol versions required running multiple commands manually.

For example, a user might attempt several separate connections or write custom scripts to test TLS versions individually.

`Test-TLSleuthProtocol` automates this process by:

- discovering runtime‑available TLS protocol enum values
- attempting one handshake per protocol
- returning one structured result row per protocol attempt

The result is a **single command that shows exactly which TLS versions succeed and fail**.

This is useful for:

- validating TLS configuration changes
- verifying deprecation of older protocols
- quickly testing STARTTLS‑enabled services
- generating script‑friendly results for automation

---

# How the Command Works

The command loops through TLS protocol values that exist in the current runtime.

The candidate list is built from the following known names:

```
Ssl3
Tls
Tls11
Tls12
Tls13
```

Only enum values that exist in the runtime's
`[System.Security.Authentication.SslProtocols]` enumeration are tested.

For each protocol attempt:

1. A TCP connection is established.
2. If required, a STARTTLS/STLS negotiation is performed.
3. A TLS handshake is attempted using that specific protocol.
4. Handshake and certificate details are extracted.
5. Results are emitted as a structured object.

Failures are captured per protocol attempt and stored in the `ErrorMessage` field.

Importantly:

- a failure does **not stop subsequent protocol tests**
- resources are always cleaned up after each attempt

The implementation ensures consistent behavior with the existing command:

```
Get-TLSleuthCertificate
```

by reusing the same connection, negotiation, handshake, and extraction helpers.

---

# Supported Transports

`Test-TLSleuthProtocol` supports the same transport modes as TLSleuth's certificate inspection workflow.

| Transport | Description |
|---|---|
ImplicitTls | Direct TLS connection (e.g. HTTPS, IMAPS) |
SmtpStartTls | SMTP STARTTLS negotiation |
ImapStartTls | IMAP STARTTLS negotiation |
Pop3StartTls | POP3 STLS negotiation |

When STARTTLS transports are used, the protocol negotiation step runs **before every protocol handshake attempt**.

For SMTP, the EHLO name is derived from:

1. `-SmtpEhloName` parameter if provided
2. local DNS hostname
3. `localhost` as a final fallback

---

# Output Model

Each protocol attempt produces a structured object with the type:

```
TLSleuth.ProtocolTestResult
```

One object is emitted **per protocol tested**.

The output includes three main categories of information.

## Endpoint Context

```
Hostname
Port
TargetHost
Transport
Protocol
```

## Outcome Information

```
ConnectionSuccessful
ErrorMessage
ElapsedMs
```

## TLS Session Details (when successful)

```
NegotiatedProtocol
CipherAlgorithm
CipherStrength
NegotiatedCipherSuite
HashAlgorithm
HashStrength
KeyExchangeAlgorithm
KeyExchangeStrength
IsMutuallyAuthenticated
IsEncrypted
IsSigned
NegotiatedApplicationProtocol
ForwardSecrecy
```

## Validation Information (when available)

```
CertificateValidationPassed
CertificatePolicyErrors
CertificatePolicyErrorFlags
CertificateChainStatus
```

These fields match the design goal of TLSleuth: **stable property names suitable for automation**.

---

# Example: Testing a Standard HTTPS Endpoint

```
Test-TLSleuthProtocol -Hostname github.com |
Select Protocol, ConnectionSuccessful, NegotiatedProtocol, NegotiatedCipherSuite, ForwardSecrecy, ErrorMessage
```

Example workflow:

- The command attempts connections for all runtime‑available protocols.
- Each attempt produces one row.
- Successful handshakes include negotiated TLS details.

This makes it easy to identify which protocols succeed and which fail.

---

# Example: Testing SMTP STARTTLS

```
Test-TLSleuthProtocol -Hostname smtp.gmail.com -Port 587 -Transport SmtpStartTls |
Select Protocol, ConnectionSuccessful, NegotiatedProtocol, ErrorMessage
```

For this transport:

1. A plaintext SMTP session is established.
2. STARTTLS negotiation is performed.
3. The TLS handshake attempt runs using the selected protocol.

The process repeats independently for each protocol value.

---

# Example: Testing IMAP STARTTLS

```
Test-TLSleuthProtocol -Hostname outlook.office365.com -Port 143 -Transport ImapStartTls
```

The command performs:

1. IMAP connection
2. STARTTLS command negotiation
3. TLS handshake attempt for each protocol

Results are returned in the same structured format.

---

# Implementation Details

The new command is implemented in:

```
source/public/Test-TLSleuthProtocol.ps1
```

It reuses existing internal helpers to ensure consistent behavior across TLSleuth commands.

Helpers reused include:

```
Connect-TcpWithTimeout
Invoke-WithRetry
Invoke-SmtpStartTlsNegotiation
Invoke-ImapStartTlsNegotiation
Invoke-Pop3StartTlsNegotiation
Start-TlsHandshake
Get-TlsHandshakeDetails
Close-NetworkResources
```

This reuse ensures:

- consistent handshake behavior
- consistent certificate validation handling
- identical TLS detail extraction

Connection lifecycle management guarantees that **TLS and network resources are always closed**, even if a handshake fails.

---

# Observability

The command includes verbose lifecycle logging.

Verbose output records:

- begin/end execution timing
- overall elapsed time

Debug logging records:

- per‑protocol failures
- protocol attempt diagnostics

This allows engineers to inspect failures without interrupting the testing process.

---

# Test Coverage

Unit tests for the new command are located at:

```
source/tests/unit/Test-TLSleuthProtocol.Tests.ps1
```

Test coverage verifies:

- all runtime‑available explicit protocols are tested
- successful rows contain negotiated TLS details
- single protocol failures do not stop remaining attempts
- SMTP STARTTLS negotiation runs per protocol attempt

Tests can be executed using:

```
Invoke-Pester -Path source/tests/unit/Test-TLSleuthProtocol.Tests.ps1
```

Integration tests for this command have **not yet been added** in the current branch.

---

# Operational Caveats

Several operational considerations should be kept in mind when using this feature.

## This Is Not a TLS Vulnerability Scanner

The command reports **what the runtime successfully negotiates**, not all possible server capabilities.

It does not enumerate server cipher support or detect vulnerabilities.

---

## Protocol Availability Depends on Runtime

The set of protocols tested depends on the runtime enum values and OS policy.

If a protocol is disabled by the OS or not present in the runtime enumeration, it will not be tested.

---

## Certificate Validation May Be Skipped

`-SkipCertificateValidation` defaults to `$true`.

This means trust validation may be bypassed unless the user explicitly changes behavior.

This design choice prioritizes **endpoint inspection** over strict trust enforcement.

---

## Runtime Duration Scales with Protocol Count

The command performs **one full connection and handshake per protocol**.

Total runtime therefore depends on:

- number of runtime‑available protocols
- responsiveness of the target endpoint

---

# When to Use This Feature

`Test-TLSleuthProtocol` is useful when you need to quickly answer:

- Which TLS versions succeed against a service?
- Did a server configuration change successfully disable older protocols?
- Does a STARTTLS service negotiate TLS correctly?

It is particularly useful for **script‑based validation and operational troubleshooting**.

---

# Summary

`Test-TLSleuthProtocol` extends TLSleuth with a straightforward but powerful capability:

- automatically test runtime‑available TLS protocols
- produce structured results for each attempt
- reuse TLSleuth’s existing handshake and negotiation logic

The result is a practical diagnostic command for engineers working with TLS endpoints in PowerShell environments.
