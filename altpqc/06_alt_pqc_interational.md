# Module 06: International PQC Standards

## Overview

While NIST's post-quantum cryptography standardization has received the most attention, several countries are developing their own PQC standards. Understanding the global PQC landscape is essential for organizations operating internationally or requiring algorithm diversity beyond Western standards.

This module provides an overview of PQC standardization efforts in South Korea, China, and European recommendations.

---

## Learning Objectives

After completing this module, you will be able to:

- Identify key PQC algorithms from South Korean standardization
- Understand China's approach to cryptographic sovereignty
- Recognize European agency recommendations
- Assess implications for international compliance
- Make informed decisions about global PQC strategy

---

## South Korea: KpqC Competition

### Overview

South Korea's National Intelligence Service (NIS) and related agencies conducted the Korean Post-Quantum Cryptography (KpqC) competition to develop national PQC standards.

### Timeline

| Year | Milestone |
|------|-----------|
| 2021 | KpqC competition announced |
| 2022 | Round 1 submissions |
| 2023 | Round 2 selections |
| 2024 | Final algorithm selections |
| 2025+ | Standardization in progress |

### Selected Algorithms

#### Key Encapsulation Mechanisms

| Algorithm | Mathematical Basis | Notes |
|-----------|-------------------|-------|
| **NTRU+** | NTRU lattice (enhanced) | Enhanced NTRU variant |
| **SMAUG-T** | Module-LWE | Similar basis to ML-KEM |

#### Digital Signatures

| Algorithm | Mathematical Basis | Notes |
|-----------|-------------------|-------|
| **HAETAE** | Lattice-based | Fiat-Shamir signature |
| **ALMer** | Lattice-based | Alternative construction |

### Interoperability Considerations

South Korean algorithms may not be available in standard OQS distributions. Organizations working with South Korean entities should:

1. Monitor NIS publications for official specifications
2. Consider direct implementation if required
3. Plan for potential hybrid deployments (NIST + KpqC)

### Testing South Korean Algorithms

**Note:** As of this writing, South Korean algorithms may require specialized implementations. Check the OQS provider for current support:

```bash
openssl list -kem-algorithms | grep -i smaug
openssl list -kem-algorithms | grep -i ntruplus
openssl list -signature-algorithms | grep -i haetae
```

If not available, the algorithms would require custom implementation or waiting for OQS updates.

---

## China: Independent PQC Development

### Cryptographic Sovereignty

China prioritizes cryptographic independence from Western standards. While NIST algorithms are based on well-understood mathematical problems, China is developing algorithms with Chinese-designed implementations.

### Key Organizations

| Organization | Role |
|--------------|------|
| ICCS | Institute of Commercial Cryptography Standards |
| CACR | Chinese Association for Cryptologic Research |
| OSCCA | Office of State Commercial Cryptography Administration |

### Timeline

| Year | Milestone |
|------|-----------|
| 2019-2020 | Initial PQC competition completed |
| 2020-2024 | Algorithm refinement |
| Feb 2025 | ICCS calls for new algorithm proposals |
| 2025+ | Continued development |

### Known Algorithms

#### Key Encapsulation

| Algorithm | Mathematical Basis | Notes |
|-----------|-------------------|-------|
| **Aigis-enc** | LWE-based | Chinese lattice KEM |
| **LAC.PKE** | Lattice-based | Alternative construction |

#### Digital Signatures

| Algorithm | Mathematical Basis | Notes |
|-----------|-------------------|-------|
| **Aigis-sig** | Lattice-based | Chinese signature scheme |

### Practical Implications

For organizations operating in China:

1. **Monitor developments**: Chinese standards may become mandatory for certain sectors
2. **Plan for dual compliance**: May need both NIST and Chinese algorithms
3. **Consider hybrid approaches**: Use multiple algorithms in parallel
4. **Engage local expertise**: Chinese cryptographic requirements evolve independently

### Availability

Chinese PQC algorithms are generally **not available** in Western cryptographic libraries. Implementation would require:

- Direct reference implementation from Chinese sources
- Custom integration
- Potential licensing considerations

---

## European Recommendations

### European Agency Positions

European cybersecurity agencies have evaluated PQC algorithms and provided recommendations:

#### Germany (BSI - Bundesamt für Sicherheit in der Informationstechnik)

| Recommendation | Algorithm | Rationale |
|----------------|-----------|-----------|
| Primary | ML-KEM, ML-DSA | NIST standards |
| Conservative | **FrodoKEM** | Unstructured lattice |
| Maximum security | **Classic McEliece** | Longest security history |

#### France (ANSSI - Agence nationale de la sécurité des systèmes d'information)

| Recommendation | Algorithm | Rationale |
|----------------|-----------|-----------|
| Primary | ML-KEM, ML-DSA | NIST standards |
| Conservative | **FrodoKEM** | Conservative security margin |

#### Netherlands (NLNCSA - National Cyber Security Agency)

Similar recommendations to BSI and ANSSI, emphasizing hybrid approaches during transition.

### European Union Considerations

The EU's approach to PQC includes:

1. **eIDAS 2.0**: May incorporate PQC requirements
2. **ENISA guidance**: Agency publishes PQC transition recommendations
3. **NIS2 Directive**: May influence cryptographic requirements

### Testing European-Recommended Algorithms

FrodoKEM and Classic McEliece (covered in earlier modules) are the primary European recommendations beyond NIST standards:

```bash
openssl list -kem-algorithms | grep -i frodo
openssl list -kem-algorithms | grep -i mceliece
```

---

## Russia: Developing Standards

### Status

Russia's TK26 Working Group 2.5 has been evaluating PQC submissions since 2019. Limited public information is available about specific algorithm selections.

### Implications

- Russian standards likely to be independent from NIST
- Based on similar mathematical foundations (lattice, code-based)
- May require separate compliance for organizations operating in Russia

---

## Algorithm Comparison: Global Landscape

### KEMs

| Region | Primary KEM | Alternative KEMs |
|--------|-------------|------------------|
| NIST | ML-KEM | HQC (2027) |
| Europe | ML-KEM | FrodoKEM, Classic McEliece |
| South Korea | SMAUG-T | NTRU+ |
| China | Aigis-enc | LAC.PKE |

### Signatures

| Region | Primary Signature | Alternative Signatures |
|--------|------------------|----------------------|
| NIST | ML-DSA | SLH-DSA, FN-DSA (2025) |
| Europe | ML-DSA | SLH-DSA |
| South Korea | HAETAE | ALMer |
| China | Aigis-sig | TBD |

### Mathematical Basis Overlap

Despite independent development, most algorithms use similar mathematical foundations:

| Problem | NIST | South Korea | China | Europe |
|---------|------|-------------|-------|--------|
| Module-LWE | ML-KEM | SMAUG-T | Aigis-enc | ML-KEM |
| Plain LWE | - | - | LAC.PKE | FrodoKEM |
| NTRU lattices | - | NTRU+ | - | - |
| Code-based | HQC | - | - | Classic McEliece |

This overlap suggests that cryptanalytic advances against one algorithm type would likely affect algorithms from multiple regions.

---

## Practical Guidance for International Operations

### Compliance Matrix

| Operating In | Primary Algorithms | Backup/Alternative |
|--------------|-------------------|-------------------|
| United States | ML-KEM, ML-DSA | HQC, SLH-DSA |
| European Union | ML-KEM, ML-DSA | FrodoKEM, Classic McEliece |
| South Korea | Monitor KpqC | NIST algorithms may be accepted |
| China | Monitor developments | Plan for Aigis-enc/sig |
| Global | ML-KEM, ML-DSA | Multiple backups |

### Recommended Strategy

For organizations operating internationally:

1. **Baseline**: Implement NIST standards (ML-KEM, ML-DSA)
2. **European enhancement**: Add FrodoKEM capability if needed
3. **Monitor Asia-Pacific**: Track South Korean and Chinese developments
4. **Hybrid readiness**: Design systems to support multiple algorithms

### Implementation Approach

```
International PQC Strategy
├── Tier 1: NIST FIPS (required everywhere)
│   ├── ML-KEM (FIPS 203)
│   ├── ML-DSA (FIPS 204)
│   └── SLH-DSA (FIPS 205)
├── Tier 2: European recommendations (if operating in EU)
│   ├── FrodoKEM (conservative)
│   └── Classic McEliece (maximum security)
├── Tier 3: Regional specifics (as required)
│   ├── South Korea: Monitor SMAUG-T, HAETAE
│   ├── China: Monitor Aigis family
│   └── Russia: Monitor TK26 developments
└── Tier 4: Future standards
    ├── HQC (2027)
    └── FN-DSA (2025)
```

---

## Cryptographic Agility

### Why Agility Matters

The diversity of international PQC standards reinforces the need for **cryptographic agility**—the ability to quickly switch algorithms when needed.

### Agility Requirements

| Capability | Implementation |
|------------|----------------|
| Algorithm negotiation | TLS 1.3 group selection |
| Multiple algorithm support | OQS provider with multiple KEMs |
| Configuration flexibility | Algorithm selection via config |
| Monitoring | Track algorithm security status |

### Testing Multiple Algorithms

Configure a server to offer multiple algorithms:

```bash
openssl s_server \
    -key keys/test.key \
    -cert certs/test.crt \
    -port 4433 \
    -tls1_3 \
    -groups MLKEM768:frodo640aes:hqc192:ntru_hps2048677 \
    -www
```

Client can select preferred algorithm:

```bash
openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups frodo640aes \
    -CAfile certs/test.crt
```

---

## Summary

The global PQC landscape is diverse but converging on similar mathematical foundations:

| Region | Status | Key Algorithms |
|--------|--------|----------------|
| NIST (US) | Standards finalized | ML-KEM, ML-DSA, SLH-DSA |
| Europe | Recommendations published | NIST + FrodoKEM + Classic McEliece |
| South Korea | Standardization in progress | SMAUG-T, NTRU+, HAETAE |
| China | Active development | Aigis-enc, Aigis-sig |
| Russia | Limited public info | TBD |

**Bottom line:** Organizations operating internationally should implement NIST standards as a baseline while maintaining cryptographic agility for regional requirements. The mathematical overlap between standards suggests that algorithm diversity (lattice + code-based) is more important than regional algorithm selection.

---

## Next Steps

Proceed to **[Module 07: Performance Analysis](07-PERFORMANCE-ANALYSIS.md)** for comprehensive performance comparisons across all algorithms covered in this learning path.

---

**Module Navigation:**

| Previous | Current | Next |
|----------|---------|------|
| [05 - BIKE and HQC](05-BIKE-HQC.md) | **06 - International PQC** | [07 - Performance Analysis](07-PERFORMANCE-ANALYSIS.md) |
