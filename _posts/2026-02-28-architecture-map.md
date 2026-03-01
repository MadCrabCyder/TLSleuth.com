---
layout: post
date: 2026-02-28

title: "Architecture Map (Parts 1–7)"
excerpt: "A visual architecture map summarizing the full Certificate Infrastructure Deep Dive series (Parts 1–7)."

categories:
  - Security
  - Cryptography
  - Infrastructure
  - PKI
tags:
  - TLS
  - PKI
  - Architecture
  - Mermaid
  - X509
  - Trust Stores

series: "Certificate Infrastructure Deep Dive"
part: 0
---

# Certificate Infrastructure Deep Dive
## Architecture Map (Parts 1–7)

This page provides a **visual architecture map** of the full series, showing how each part fits into the broader certificate infrastructure story.

If you want one mental model to keep in your head when troubleshooting TLS/PKI, this is it.

---

## 1. End-to-End Trust Architecture (Big Picture)

```mermaid
flowchart TD
    A[Part 1: Cryptographic Foundations<br/>Hashing, Signatures, Key Exchange, AEAD] --> B[Part 2: TLS Handshake<br/>Negotiation, Identity, Key Schedule]
    B --> C[Part 3: X.509 Certificates<br/>ASN.1/DER, Fields, Extensions]
    C --> D[Part 4: PKI & Trust Stores<br/>Root Programs, Anchors, Chain Building]
    D --> E[Part 5: Revocation Reality<br/>CRL/OCSP/Stapling, Soft-fail]
    E --> F[Part 6: Failure Modes & Attacks<br/>Mis-issuance, CA compromise, BGP/DNS]
    F --> G[Part 7: Future of PKI<br/>Post-Quantum, ACME scale, Workload Identity]
```

---

## 2. From Wire to Root (Operational Debug View)

This diagram maps the path an engineer mentally follows when debugging a “TLS is broken” incident.

```mermaid
flowchart TD
    X[Connection Attempt] --> T[TCP Reachability<br/>routing, firewall, port]
    T --> H[TLS Handshake<br/>ClientHello / ServerHello]
    H --> S[SNI / ALPN / Cipher Negotiation]
    H --> K[Key Exchange + Key Schedule<br/>ECDHE/HKDF]
    H --> CERT[Certificate Presented<br/>leaf + intermediates]
    CERT --> VAL[Certificate Validation<br/>SAN, time, KU/EKU, constraints]
    VAL --> CHAIN[Chain Building<br/>intermediate discovery]
    CHAIN --> TRUST[Trust Anchor<br/>OS/Browser trust store]
    VAL --> REV[Revocation Check<br/>CRL/OCSP/stapling behavior]
    TRUST --> OK[Secure Channel Established]
```

Where the series fits:
- **Part 2** explains the handshake stage.
- **Part 3** explains certificate structure and extensions.
- **Part 4** explains chain building and trust anchors.
- **Part 5** explains revocation and why it fails.

---

## 3. Certificate Anatomy and Validation Gates

This map shows the most important certificate components and where they influence validation decisions.

```mermaid
flowchart LR
    Cert[X.509 Certificate] --> ID[Identity<br/>SAN]
    Cert --> Time[Validity Window<br/>NotBefore/NotAfter]
    Cert --> Key[Public Key<br/>SPKI]
    Cert --> Ext[Extensions]
    Ext --> KU[Key Usage]
    Ext --> EKU[Extended Key Usage]
    Ext --> BC[Basic Constraints]
    Ext --> NC[Name Constraints]
    Ext --> AIA[AIA<br/>Issuer + OCSP URL]
    Ext --> CRLDP[CRL Distribution Points]

    ID --> Gate[Validation Gates]
    Time --> Gate
    KU --> Gate
    EKU --> Gate
    BC --> Gate
    NC --> Gate
    Gate --> Outcome[Trusted / Rejected]
```

Series mapping:
- **Part 3** is the deep dive on every node in this diagram.

---

## 4. Global Trust Governance Map

This map shows how “trust” is distributed globally through root programs, audits, and transparency.

```mermaid
flowchart TD
    RootProg[Root Programs<br/>Microsoft / Apple / Mozilla / Android] --> Policy[CA Policy & Audit Requirements]
    Policy --> CA[CA Operators]
    CA --> Issuance[Certificate Issuance]
    Issuance --> CT[Certificate Transparency Logs]
    Issuance --> Sites[Websites / Services]
    CT --> Browsers[Browsers / OS Trust Stores]
    Browsers --> Users[Users & Systems]
    Inc[Incidents<br/>Mis-issuance/Compromise] --> RootProg
```

Series mapping:
- **Part 4** explains the governance and trust distribution model.
- **Part 6** explains how incidents feed back into governance decisions.

---

## 5. Revocation Reality Map (Why It’s Weak)

This is the “security vs availability” trade-off visualized.

```mermaid
flowchart TD
    Cert[Certificate] --> Rev[Revocation Mechanisms]
    Rev --> CRL[CRL<br/>download list]
    Rev --> OCSP[OCSP<br/>online query]
    OCSP --> Staple[OCSP Stapling<br/>server-provided]
    Rev --> Must[Must-Staple<br/>fail if missing]

    CRL --> Scale[Scale/Latency Issues]
    OCSP --> Privacy[Privacy Leakage]
    OCSP --> Avail[Responder Availability]
    Avail --> Soft[Soft-Fail Default]
    Soft --> Weak[Revocation Often Ineffective]
```

Series mapping:
- **Part 5** is the deep dive on this entire diagram.

---

## 6. Attack Surface Map

This shows where attackers target the certificate ecosystem.

```mermaid
flowchart LR
    Attacker[Attacker] --> DV[Domain Validation Abuse<br/>DNS/HTTP challenges]
    Attacker --> Routing[BGP/DNS Hijack]
    Attacker --> CAComp[CA/Intermediate Compromise]
    Attacker --> KeyLeak[Private Key Theft]
    Attacker --> Auto[ACME Automation Abuse<br/>token/secret theft]

    DV --> MisIss[Mis-Issuance]
    Routing --> MisIss
    CAComp --> MisIss
    MisIss --> MITM[MITM / Impersonation]
    KeyLeak --> Impersonation[Direct Impersonation]
    Auto --> MisIss
```

Series mapping:
- **Part 6** explores these failures and realistic mitigations.
- **Part 4** explains why governance is needed at all.
- **Part 5** explains why revocation rarely stops attacks quickly.

---

## 7. Future Trajectory Map

This shows the major trends shaping the next iteration of PKI.

```mermaid
flowchart LR
    PQ[Post-Quantum Transition] --> Hybrid[Hybrid KEX + Hybrid Signatures]
    ACME[ACME Evolution] --> Short[Shorter Lifetimes + Faster Rotation]
    Machines[Machine Identity Explosion] --> Workload[SPIFFE / Workload Identity]
    CT2[Transparency Improvements] --> Detect[Better Detection + Monitoring]
    Hardware[Hardware-backed Keys] --> Reduce[Reduced Key Theft Risk]

    Hybrid --> Future[Future PKI]
    Short --> Future
    Workload --> Future
    Detect --> Future
    Reduce --> Future
```

Series mapping:
- **Part 7** is the deep dive on these trends.

---

## How to Use This Map

When troubleshooting:
1. Start at the **Wire-to-Root** map.
2. Identify which layer is failing.
3. Drill into the corresponding series part.

When designing infrastructure:
1. Use **Global Governance** + **Attack Surface** maps to assess systemic risk.
2. Use **Future Trajectory** map to design for the next 3–5 years.

