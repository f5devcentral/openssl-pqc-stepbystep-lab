# Module 0: Introduction to Alternative PQC Algorithms

## Overview

This learning path explores post-quantum cryptographic algorithms outside the primary NIST FIPS standards (203/204/205). While ML-KEM and ML-DSA form the foundation of many PQC deployments, a comprehensive security strategy benefits from understanding the broader algorithm landscape.

These alternative algorithms offer different mathematical foundations, security properties, and trade-offs that may be critical for specific use cases, defense-in-depth strategies, or international compliance requirements. This is where crypto agility will start to become a thing as new algorithms are adopted and we quickly need to swap out or add new methods of key encapsulation or digital signing as standards evolve.

---

## Learning Objectives

After completing this module, you will be able to:

- Explain why algorithm diversity matters for quantum-resistant security
- Improve understanding of international PQC standardization efforts beyond NIST
- Recognize some use cases for each alternative algorithm
- Throw shade at your friends who still use AES128. Cowards.

<br>

## Why Alternative Algorithms Matter

### Defense in Depth

Relying solely on one algorithm family creates systemic risk. If a breakthrough in lattice cryptanalysis occurs (which does happen), organizations using only ML-KEM/ML-DSA would be vulnerable. Alternative algorithms based on different mathematical problems provide backup options.

| Algorithm Family | Mathematical Problem | Primary NIST Standard | Alternative Options |
| ----------------- | --------------------- | ---------------------- | --------------------- |
| Structured Lattice | Module-LWE | ML-KEM, ML-DSA | NTRU |
| Unstructured Lattice | LWE | None | FrodoKEM |
| Code-based | Syndrome Decoding | HQC (2027) | BIKE, Classic McEliece |
| Hash-based | Hash Function Security | SLH-DSA | LMS, XMSS |

### International Compliance

Different regions have different cryptographic requirements and recommendations:

- **Europe (BSI, ANSSI)**: Recommend FrodoKEM and Classic McEliece for high-security applications
- **South Korea**: Developing national standards including NTRU+, SMAUG-T, HAETAE
- **China**: Independent PQC standards including Aigis-sig, Aigis-enc

### Conservative Security

Some alternative algorithms provide more conservative security guarantees at the cost of performance:

- **FrodoKEM**: Uses unstructured lattices without algebraic ring structure—harder to attack but slower
- **Classic McEliece**: 40+ years of cryptanalysis without practical attacks—extremely conservative

<br>

## Algorithm Overview

### Key Encapsulation Mechanisms (KEMs)

This learning path currently focuses on KEMs because they protect data in transit and are critical for TLS connections, VPN tunnels, and key exchange protocols.

| Algorithm | Basis | Key Sizes | Performance | Status |
| ----------- | ------- | ----------- | ------------- | -------- |
| **FrodoKEM** | Unstructured lattice | Large (9-21 KB) | Slow | NIST Round 3 alternate |
| **NTRU** | Structured lattice | Small (0.7-1.2 KB) | Fast encaps/decaps | Not currently available in this lab |
| **Classic McEliece** | Code-based | Huge (261 KB-1 MB) | Very slow keygen | Not currently available in this lab |
| **BIKE** | Code-based | Medium (1.5-5 KB) | Moderate | NIST Round 4 |
| **HQC** | Code-based | Medium (2-7 KB) | Good | NIST selected backup, disabled for KEM pending bug fixes |

### Why No Signature Algorithms?

This path focuses on KEMs because:

1. **NIST coverage is comprehensive**: ML-DSA and SLH-DSA cover most signature use cases
2. **FN-DSA (FALCON) is coming**: Expected FIPS 206 standardization addresses remaining gaps
3. **LMS/XMSS are already standardized**: SP 800-208 covers hash-based signatures for firmware
4. **KEMs have more diversity**: Code-based alternatives provide meaningful differentiation

<br>

### European Recommendations

European cybersecurity agencies (BSI, ANSSI, NLNCSA) have endorsed:

- **FrodoKEM**: For applications requiring conservative unstructured lattice security
- **Classic McEliece**: For high-security applications willing to accept large key sizes

***Note*** *Some documentation is stagnant but agencies like BSI note future support for NIST approved alogrithms dependent on algorithm strength and use*

---

## What You Will Build

Throughout this learning path, you will:

1.  **Configure the OQS Environment**: Set up OpenSSL 3.5.x with the OQS provider for access to alternative algorithms
2.  **Test FrodoKEM**: Experience the conservative unstructured lattice approach and understand its security/performance trade-offs
6.  **Compare BIKE and HQC**: Understand code-based alternatives and why NIST selected HQC as the ML-KEM backup
7.  **Analyze Performance**: Measure and compare handshake sizes, latency impacts, and operational characteristics
8.  **Understand Use Cases**: Learn when each algorithm is the right choice for specific requirements

<br>

## Prerequisites

### Required Knowledge

- ***`Completion of FIPS Path Module 00-01 OR CNSA Path Module 00-01`***
- Understanding of TLS handshake process
- Familiarity with key encapsulation concepts
- Basic PKI knowledge

<br>

## Lab Architecture

Throughout this path, you will work with the same Sassy Corp infrastructure, but focus on TLS key exchange rather than certificate signing:

```ini
┌─────────────────────────────────────────────────────────────┐
│                    Sassy Corp Lab Environment               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────┐     TLS with Alt KEM     ┌───────────┐    │
│   │ Test Client │◄────────────────────────►│Test Server│    │
│   │ s_client    │    FrodoKEM, BIKE, HQC   │ s_server  │    │
│   └─────────────┘                          └───────────┘    │
│                                                             │
│   Certificates: ML-DSA-65 (from FIPS/CNSA path)             │
│   Key Exchange: Alternative algorithms (this path)          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

You will use ML-DSA certificates created in previous learning paths (or create new ones) while experimenting with different key exchange algorithms.

<br>

## Module Structure

| Module | Focus | Key Outcome |
| -------- | ------- | ------------- |
| 01 - Environment Setup | Ubuntu 25.10 + OQS | Working alternative algorithm environment |
| 02 - FrodoKEM | Unstructured lattice | Conservative KEM implementation |
| 03 - BIKE and HQC | Code-based alternatives | NIST Round 4 and backup algorithms |
| 04 - International PQC | South Korea, China | Global standards awareness |
| 05 - Performance Analysis | Comprehensive comparison | Algorithm selection guidance |

<br>

## Security Considerations

### Known Considerations

- **BIKE**: Historical timing attack vulnerabilities (patched)
- **HQC**: Some implementation concerns (being addressed)
- **FrodoKEM**: Performance overhead may be prohibitive for some applications

<br>

## Next Steps

Proceed to **[Module 01: Environment Setup](01_alt_pqc_environment.md)** to validate your Ubuntu 25.10 system with OpenSSL 3.5.x and the oqsprovider for alternative algorithm access.


**Module Navigation:**

| Previous | Current | Next |
|----------|---------|------|
| [Main README](../README.md) | **00 - Introduction** | [01 - Environment Setup](01_alt_pqc_environment.md) |
