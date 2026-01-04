# Module 00: Introduction to Alternative PQC Algorithms

## Overview

This learning path explores post-quantum cryptographic algorithms outside the primary NIST FIPS standards (203/204/205). While ML-KEM and ML-DSA form the foundation of most PQC deployments, a comprehensive security strategy benefits from understanding the broader algorithm landscape.

These alternative algorithms offer different mathematical foundations, security properties, and trade-offs that may be critical for specific use cases, defense-in-depth strategies, or international compliance requirements.

---

## Learning Objectives

After completing this module, you will be able to:

- Explain why algorithm diversity matters for quantum-resistant security
- Identify the mathematical foundations of different PQC algorithm families
- Understand international PQC standardization efforts beyond NIST
- Recognize appropriate use cases for each alternative algorithm
- Make informed decisions about algorithm selection for specific requirements

---

## Why Alternative Algorithms Matter

### Defense in Depth

Relying solely on one algorithm family creates systemic risk. If a breakthrough in lattice cryptanalysis occurs, organizations using only ML-KEM/ML-DSA could be vulnerable. Alternative algorithms based on different mathematical problems provide backup options.

| Algorithm Family | Mathematical Problem | Primary NIST Standard | Alternative Options |
|-----------------|---------------------|----------------------|---------------------|
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

---

## Algorithm Overview

### Key Encapsulation Mechanisms (KEMs)

This learning path focuses on KEMs because they protect data in transit and are critical for TLS connections, VPN tunnels, and key exchange protocols.

| Algorithm | Basis | Key Sizes | Performance | Status |
|-----------|-------|-----------|-------------|--------|
| **FrodoKEM** | Unstructured lattice | Large (9-21 KB) | Slow | NIST Round 3 alternate |
| **NTRU** | Structured lattice | Small (0.7-1.2 KB) | Fast encaps/decaps | ANSI standardized |
| **Classic McEliece** | Code-based | Huge (261 KB-1 MB) | Very slow keygen | NIST Round 4 |
| **BIKE** | Code-based | Medium (1.5-5 KB) | Moderate | NIST Round 4 |
| **HQC** | Code-based | Medium (2-7 KB) | Good | NIST selected backup |

### Why No Signature Algorithms?

This path focuses on KEMs because:

1. **NIST coverage is comprehensive**: ML-DSA and SLH-DSA cover most signature use cases
2. **FN-DSA (FALCON) is coming**: Expected FIPS 206 standardization addresses remaining gaps
3. **LMS/XMSS are already standardized**: SP 800-208 covers hash-based signatures for firmware
4. **KEMs have more diversity**: Code-based alternatives provide meaningful differentiation

---

## International PQC Landscape

### South Korea

South Korea's KpqC (Korean Post-Quantum Cryptography) competition has produced several algorithms:

**KEMs:**
- **NTRU+**: Enhanced NTRU variant
- **SMAUG-T**: Module-LWE based KEM

**Signatures:**
- **HAETAE**: Lattice-based signature scheme
- **ALMer**: Alternative lattice signature

### China

China's approach prioritizes cryptographic sovereignty:

**KEMs:**
- **Aigis-enc**: LWE-based encryption
- **LAC.PKE**: Lattice-based encryption

**Signatures:**
- **Aigis-sig**: Lattice-based signature

The February 2025 ICCS (Institute of Commercial Cryptography Standards) call for new algorithm proposals indicates continued independent development.

### European Recommendations

European cybersecurity agencies (BSI, ANSSI, NLNCSA) have endorsed:

- **FrodoKEM**: For applications requiring conservative unstructured lattice security
- **Classic McEliece**: For high-security applications willing to accept large key sizes

---

## What You Will Build

Throughout this learning path, you will:

1. **Configure the OQS Environment**: Set up OpenSSL 3.5.x with the OQS provider for access to alternative algorithms

2. **Test FrodoKEM**: Experience the conservative unstructured lattice approach and understand its security/performance trade-offs

3. **Explore NTRU**: Work with the most compact lattice-based KEM and understand its 25-year security history

4. **Implement Classic McEliece**: Handle the largest keys in post-quantum cryptography and understand when this trade-off makes sense

5. **Compare BIKE and HQC**: Understand code-based alternatives and why NIST selected HQC as the ML-KEM backup

6. **Analyze Performance**: Measure and compare handshake sizes, latency impacts, and operational characteristics

7. **Understand Use Cases**: Learn when each algorithm is the right choice for specific requirements

---

## Prerequisites

### System Requirements

| Requirement | Specification |
|-------------|---------------|
| Operating System | Ubuntu 25.10 |
| OpenSSL | 3.5.x |
| OQS Provider | Latest version |
| RAM | 8GB recommended |
| Disk Space | 15GB free |
| CPU | Multi-core recommended for build |

### Required Knowledge

- Completion of FIPS Path Module 00-01 OR CNSA Path Module 00-01
- Understanding of TLS handshake process
- Familiarity with key encapsulation concepts
- Basic PKI knowledge

### Why Ubuntu 25.10?

Ubuntu 25.10 provides:
- Native OpenSSL 3.5.x support
- Updated build toolchain for liboqs
- Kernel optimizations for cryptographic operations

---

## Lab Architecture

Throughout this path, you will work with the same Sassy Corp infrastructure, but focus on TLS key exchange rather than certificate signing:

```
┌─────────────────────────────────────────────────────────────┐
│                    Sassy Corp Lab Environment               │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌─────────────┐     TLS with Alt KEM      ┌───────────┐  │
│   │ Test Client │◄────────────────────────►│Test Server │  │
│   │ s_client    │    FrodoKEM, NTRU,       │ s_server   │  │
│   └─────────────┘    McEliece, BIKE, HQC   └───────────┘  │
│                                                             │
│   Certificates: ML-DSA-65 (from FIPS/CNSA path)            │
│   Key Exchange: Alternative algorithms (this path)          │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

You will use ML-DSA certificates created in previous learning paths (or create new ones) while experimenting with different key exchange algorithms.

---

## Module Structure

| Module | Focus | Key Outcome |
|--------|-------|-------------|
| 01 - Environment Setup | Ubuntu 25.10 + OQS | Working alternative algorithm environment |
| 02 - FrodoKEM | Unstructured lattice | Conservative KEM implementation |
| 03 - NTRU | Compact structured lattice | Smallest-key KEM demonstration |
| 04 - Classic McEliece | Code-based (Goppa) | Largest-key conservative KEM |
| 05 - BIKE and HQC | Code-based alternatives | NIST Round 4 and backup algorithms |
| 06 - International PQC | South Korea, China | Global standards awareness |
| 07 - Performance Analysis | Comprehensive comparison | Algorithm selection guidance |

---

## Security Considerations

### Algorithm Maturity

| Algorithm | Years of Analysis | Standardization Status |
|-----------|------------------|----------------------|
| Classic McEliece | 40+ years | NIST Round 4, ISO process |
| NTRU | 25+ years | ANSI X9.98-2010, IEEE P1363 |
| FrodoKEM | 10+ years | NIST Round 3 alternate |
| BIKE | 8+ years | NIST Round 4 |
| HQC | 8+ years | NIST selected (2027 standard) |

### Known Considerations

- **BIKE**: Historical timing attack vulnerabilities (patched)
- **HQC**: Some implementation concerns (being addressed)
- **Classic McEliece**: Key generation is extremely slow
- **FrodoKEM**: Performance overhead may be prohibitive for some applications

---

## Next Steps

Proceed to **[Module 01: Environment Setup](01-ENVIRONMENT-SETUP.md)** to configure your Ubuntu 25.10 system with OpenSSL 3.5.x and the OQS provider for alternative algorithm access.

---

**Module Navigation:**

| Previous | Current | Next |
|----------|---------|------|
| [Main README](../README.md) | **00 - Introduction** | [01 - Environment Setup](01-ENVIRONMENT-SETUP.md) |
