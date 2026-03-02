---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---

**TLSleuth** is an open-source PowerShell module for inspecting TLS
endpoints and certificate details from scripts or the command line.

It provides clean, structured, script-friendly output for operators,
engineers, and automation pipelines that need reliable TLS insight.

- 🔎 Fetch a server's certificate and handshake details
- 📋 View negotiated TLS protocol and cipher information
- ⚙ Designed for automation and testing

------------------------------------------------------------------------

## Features

- **SNI-aware** -- Automatically uses SNI based on `-Hostname` (or
    `-TargetHost` override).
- **Protocol selection** -- Constrain to `Tls12`, `Tls13`, etc.
    (OS/runtime permitting).
- **Structured output** -- Stable object model with custom
    `PSTypeName`.
- **Pipeline support** -- Designed for batch processing.
- **Verbose diagnostics** -- `-Verbose` provides helper-level timing
    insight.
- **Tested** -- Unit tests with mocks; optional integration tests.

## New Feature for Version 2 - Explicit Transport Support

- Added support for specifying the transport type
- New transport option: `SmtpStartTls`
- New transport option: `ImapStartTls`
- New transport option: `Pop3StartTls`

You can now retrieve certificates from SMTP, IMAP, and POP3 servers using `STARTTLS`/`STLS` negotiation, rather than assuming implicit TLS (e.g., SMTPS on port 465, IMAPS on port 993, or POP3S on port 995).

This allows TLSleuth to:
- Connect to SMTP services on port 25 or 587
- Connect to IMAP services on port 143
- Connect to POP3 services on port 110
- Issue the STARTTLS/STLS command
- Upgrade the connection to TLS
- Retrieve certificate and negotiated TLS details

For more information see this page: [Implicit vs Explicit TLS](/blog/implicit-vs-explicit-tls.html)

------------------------------------------------------------------------

## Limitations and When to Use a Dedicated TLS Scanner

TLSleuth is designed for practical, scriptable TLS inspection - retrieving the negotiated certificate, protocol, and cipher from PowerShell.

Because it relies on `.NET SslStream` and the underlying OS TLS stack (SChannel on Windows), it has intentional limitations:

- It only shows the negotiated cipher suite (no full enumeration)
- It cannot probe for TLS vulnerabilities (Heartbleed, ROBOT, etc.)
- It cannot craft custom ClientHello messages or test fallback behavior
- TLS version and cipher availability depend on OS policy

For full TLS posture analysis, cipher enumeration, downgrade testing, and vulnerability scanning, use a [Dedicated TLS Scanner](/blog/dedicated-scanners.html)

------------------------------------------------------------------------

## Installation

### From PowerShell Gallery

``` powershell
Install-Module TLSleuth -Scope CurrentUser
Import-Module TLSleuth
```

Recommended: **PowerShell 7+**
Supported: Windows PowerShell 5.1 (reduced TLS/cipher detail)

------------------------------------------------------------------------

## Quick Start

``` powershell
# Fetch certificate + handshake details
Get-TLSleuthCertificate -Hostname github.com

# Constrain protocol
Get-TLSleuthCertificate -Hostname google.com -TlsProtocols Tls12

# Pipeline usage
'github.com','microsoft.com' |
  Get-TLSleuthCertificate |
  Select Hostname, NegotiatedProtocol, CipherAlgorithm, CipherStrength, NotAfter

# Verbose tracing
Get-TLSleuthCertificate -Hostname microsoft.com -Verbose

# New in V2.0.0 - Retrieve certificate from SMTP server
Get-TLSleuthCertificate -Hostname smtp.gmail.com -port 25 -Transport SmtpStartTls
```

> When connecting by IP but requiring proper SNI, use `-TargetHost example.com`.

------------------------------------------------------------------------

## Output Model

TLSleuth returns a structured object:

Example:

``` powershell
Hostname           : microsoft.com
Port               : 443
TargetHost         : microsoft.com
Subject            : CN=microsoft.com, O=Microsoft Corporation...
Issuer             : CN=Microsoft Azure RSA TLS Issuing CA 04...
Thumbprint         : 40B3005534C15CC035B1F0061A813B8F91D1A02A
NotBefore          : 4/02/2026 11:21:49 AM
NotAfter           : 3/08/2026 10:21:49 AM
IsValidNow         : True
DaysUntilExpiry    : 155
NegotiatedProtocol : Tls13
CipherAlgorithm    : Aes256
CipherStrength     : 256
ElapsedMs          : 50
Certificate        : X509Certificate2
```

The object includes:

- Certificate metadata
- Validity status
- Negotiated TLS protocol
- Cipher algorithm & strength
- Timing information
- Raw `X509Certificate2` for advanced use

Designed for stable automation and predictable output contracts.
