---
layout: post
title: "Dedicated TLS Scanners"
date: 2025-09-15
permalink: /blog/dedicated-scanners.html
tags: [tls, certificates, scanners, security]
---

# When to Use Dedicated TLS Scanners

TLSleuth was designed as a **pragmatic helper**: fetch the negotiated certificate and handshake details from PowerShell, script it into your workflows, and give operators a quick look at what an endpoint is actually presenting.

But there are hard limits to what we can achieve from inside PowerShell and .NET. This post looks at those limits, and then surveys the dedicated tools that go much further.

---

## The Limits of PowerShell & .NET

TLSleuth rides on the .NET `SslStream` and the underlying OS TLS stack (SChannel on Windows). That means:

- ❌ **No cipher suite enumeration**
  You only see the *negotiated* suite. You cannot coerce `SslStream` to test every possible cipher the server supports.

- ❌ **No vulnerability probes**
  There’s no built-in way to test for protocol-level CVEs like Heartbleed, ROBOT, BEAST, Lucky13, etc.

- ❌ **No ALPN/HTTP2 visibility**
  `SslStream` doesn’t expose ALPN negotiation results. You can’t easily see if the server supports HTTP/2 or custom protocols.

- ❌ **Limited TLS 1.3 support**
  Availability depends on the host OS and .NET runtime. PowerShell 5.1 on older Windows has no TLS 1.3 at all.

- ❌ **Minimal handshake tweaking**
  Things like client hello customization, curve ordering, compression flags or resumption tickets are out of reach.

That’s fine for **scripting, monitoring, and quick cert checks**, but not for **auditing a server’s full TLS posture**.

---

## When TLSleuth is “enough”

Use TLSleuth when you need:

- ✅ Quick confirmation of a cert’s CN/SAN, expiry, and issuer
- ✅ Pipeline-friendly object output for reporting
- ✅ Diagnostics of the negotiated protocol and cipher *your client* will use
- ✅ Integration into PowerShell automation (monitoring, CI pipelines, scheduled checks)

---

## When to bring in a dedicated scanner

Reach for a scanner if you need to answer questions like:

- *“What ciphers and protocols does this server accept?”*
- *“Is it vulnerable to Heartbleed/ROBOT/BEAST/POODLE/etc.?”*
- *“Does it support ALPN/HTTP2, session resumption, or OCSP stapling?”*
- *“What curves and key exchanges are supported?”*

---

## Popular TLS Scanners

### 🔍 **sslyze**
<https://github.com/nabla-c0d3/sslyze>
Python-based, modular, actively maintained. Enumerates supported protocols, ciphers, compression, renegotiation, resumption, stapling, ALPN, etc. Exports JSON for integration.

### 🔍 **sslscan**
<https://github.com/rbsec/sslscan>
Fast, OpenSSL-backed. Reports supported ciphers/protocols, certificate info, and common vulnerabilities. Lightweight and scriptable.

### 🔍 **testssl.sh**
<https://testssl.sh/>
A comprehensive Bash script wrapping OpenSSL. Tests weak ciphers, curves, protocol fallbacks, CVEs, STARTTLS upgrades, ALPN, and more. Portable and very thorough.

### 🔍 **Nmap NSE scripts**
<https://nmap.org/nsedoc/>
- `ssl-enum-ciphers`: Enumerates supported ciphers/protocols
- `ssl-cert`: Fetches and parses certificate details
- `ssl-dh-params`: Checks Diffie-Hellman parameter strength
Great for quick scans against a range of hosts/ports.

### 🔍 **OpenSSL CLI**
While not a full scanner, `openssl s_client` is invaluable for ad-hoc inspection and verifying cert chains or protocol handshakes.

### 🔍 **Qualys SSL Labs API**
<https://www.ssllabs.com/ssltest/>
Cloud-based analysis with deep grading and vulnerability checks. Slow, but definitive. Also has an API for automation.

### 🔍 **Other honorable mentions**
- **ZGrab2**: JSON-first TLS/HTTP grabber (used in ZMap ecosystem)
- **CryptCheck**: Online TLS test with a focus on ciphers/protocols
- **tls-scan**: Lightweight Go-based scanner with JSON output

---

## TLSleuth + Scanners: Best of Both

Think of TLSleuth as the **PowerShell detective for day-to-day cert inspection**, and scanners as the **forensic lab** when you need the full story.

- Use TLSleuth to watch cert expiry and trust on your production hosts.
- Use scanners during audits, red-team exercises, or when validating server hardening.

Each tool has its place. Together, they cover both the **quick check** and the **deep dive**.

---

✍️ *Have a favorite TLS scanner not listed here? Open an issue or PR — we’ll add it to the toolbox!*
