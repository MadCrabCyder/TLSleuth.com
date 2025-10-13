---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---

**TLSleuth** is a PowerShell module for quickly inspecting TLS/SSL endpoints and certificates from the shell or in scripts. It’s a pragmatic, scriptable helper—**not** a full-blown TLS scanner.

* 🔎 Fetch a server’s certificate and handshake details
* 📋 See the negotiated TLS protocol and (when supported) the cipher suite
* 🧩 Parse SANs, AIA and CDP URLs, and basic chain/trust information
* 🧪 Built with unit tests and a clean, mockable design

---

## Features

* **SNI-aware**: uses SNI automatically based on `-Hostname` (or `-TargetHost` override).
* **Protocol selection**: constrain to `Tls12`, `Tls13`, etc. (OS/runtime permitting).
* **Certificate details**: Subject, Subject CN, *Primary DNS name* (SAN-first), SANs\[], issuer, validity, signature/public key algorithms, key size, thumbprint, self-signed boolean.
* **Chain/trust**: optional local chain build with status details.
* **Extension parsing**: DNS SANs, AIA URLs, CRL Distribution Point URLs (empty arrays when absent).
* **Verbose diagnostics**: `-Verbose` prints begin/end + timings per helper.
* **Script-friendly**: stable object model; safe arrays (never `$null` for collections).
* **Well-tested**: Pester tests use mocks; optional live (integration) tests.

---

## Install

**From the PowerShell Gallery**:

```powershell
Install-Module TLSleuth -Scope CurrentUser
Import-Module TLSleuth
```

> **Recommended:** PowerShell 7+.
> **Supported:** Windows PowerShell 5.1 (with reduced TLS/cipher detail).

---

## Quick Start

```powershell
# Fetch cert + handshake details
Get-TLSleuthCertificate -Hostname example.com

# Constrain protocol to TLS 1.2
Get-TLSleuthCertificate -Hostname example.com -TlsProtocols Tls12

# Include local chain build + revocation checks
Get-TLSleuthCertificate -Hostname example.com -IncludeChain -CheckRevocation

# Pipeline support
'github.com','microsoft.com' |
  Get-TLSleuthCertificate -IncludeChain |
  Select Host,Protocol,CipherSuite,@{n='PrimaryDNS';e={$_.Certificate.PrimaryDnsName}},IsTrusted

# Verbose tracing (timings per helper)
Get-TLSleuthCertificate -Hostname example.com -Verbose
```

> If you connect by IP but need proper SNI, pass `-ServerName example.com`.
