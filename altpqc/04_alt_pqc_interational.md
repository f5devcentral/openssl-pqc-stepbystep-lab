# Module 4: International PQC Standards

## Overview

While NIST's post-quantum cryptography standardization has received the most attention, several countries are developing their own PQC standards. Understanding the global PQC landscape is crucial for organizations operating internationally or requiring algorithm diversity beyond North American standards (US NIST partners with Candada's CCCS for FIPS and CSPRNG selections). BTW, the Open Quantum Safe project was started at University of Waterloo. So most of what we're doing we have Bob and Doug Makenzie to thank.

This module provides an overview of PQC standardization efforts in South Korea, China, and European recommendations.

## Learning Objectives

After completing this module, you will be able to:

- Identify key PQC algorithms from South Korean standardization
- Understand China's approach to cryptographic sovereignty
- Recognize European agency recommendations
- Assess implications for international compliance
- Make reasonable decisions about global PQC strategy
- Look cool to all your friends at the bar

<br>

## South Korea: KpqC Competition

### Overview

South Korea's National Intelligence Service (NIS) and related agencies conducted the Korean Post-Quantum Cryptography (KpqC) competition to develop national PQC standards.

### Timeline

| Year | Milestone |
| ------ | ----------- |
| 2021 | KpqC competition announced |
| 2022 | Round 1 submissions |
| 2023 | Round 2 selections |
| 2024 | Final algorithm selections |
| 2025+ | Standardization in progress |

### Selected Algorithms

#### Key Encapsulation Mechanisms

| Algorithm | Mathematical Basis | Notes |
| ----------- | ------------------- | ------- |
| **NTRU+** | NTRU lattice (enhanced) | Enhanced NTRU variant |
| **SMAUG-T** | Module-LWE | Similar basis to ML-KEM |

#### Digital Signatures

| Algorithm | Mathematical Basis | Notes |
| ----------- | ------------------- | ------- |
| **HAETAE** | Lattice-based | Fiat-Shamir signature |
| **ALMer** | Lattice-based | Alternative construction |

### Interoperability Considerations

South Korean algorithms may not be available in standard OQS distributions. Organizations working with South Korean entities should:

1. Monitor NIS publications for official specifications
2. Consider direct implementation if required
3. Plan for potential hybrid deployments (NIST + KpqC)

<br>

## China: Independent PQC Development

### Cryptographic Sovereignty

China prioritizes cryptographic independence from Western standards. While NIST algorithms are based on well-understood mathematical problems, China is developing algorithms with Chinese-designed implementations.

### Key Organizations

| Organization | Role |
| -------------- | ------ |
| ICCS | Institute of Commercial Cryptography Standards |
| CACR | Chinese Association for Cryptologic Research |
| OSCCA | Office of State Commercial Cryptography Administration |

### Timeline

| Year | Milestone |
| ------ | ----------- |
| 2019-2020 | Initial PQC competition completed |
| 2020-2024 | Algorithm refinement |
| Feb 2025 | ICCS calls for new algorithm proposals |
| 2025+ | Continued development |

### Known Algorithms

#### Key Encapsulation

| Algorithm | Mathematical Basis | Notes |
| ----------- | ------------------- | ------- |
| **Aigis-enc** | LWE-based | Chinese lattice KEM |
| **LAC.PKE** | Lattice-based | Alternative construction |

#### Digital Signatures

| Algorithm | Mathematical Basis | Notes |
| ----------- | ------------------- | ------- |
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

<br>

## European Recommendations

### European Agency Positions

European cybersecurity agencies have evaluated PQC algorithms and provided recommendations:

#### Germany (BSI - Bundesamt für Sicherheit in der Informationstechnik)

| Recommendation | Algorithm | Rationale |
| ---------------- | ----------- | ----------- |
| Primary | ML-KEM, ML-DSA | NIST standards |
| Conservative | **FrodoKEM** | Unstructured lattice |
| Maximum security | **Classic McEliece** | Longest security history |

#### France (ANSSI - Agence nationale de la sécurité des systèmes d'information)

| Recommendation | Algorithm | Rationale |
| ---------------- | ----------- | ----------- |
| Primary | ML-KEM, ML-DSA | NIST standards |
| Conservative | **FrodoKEM** | Conservative security margin |

#### Netherlands (NLNCSA - National Cyber Security Agency)

Similar recommendations to BSI and ANSSI, emphasizing hybrid approaches during transition.

### European Union Considerations

The EU's approach to PQC includes:

1. **eIDAS 2.0**: May incorporate PQC requirements
2. **ENISA guidance**: Agency publishes PQC transition recommendations
3. **NIS2 Directive**: May influence cryptographic requirements

<br>

## Russia: Developing Standards

### Status

Russia's TK26 Working Group 2.5 has been evaluating PQC submissions since 2019. Limited public information is available about specific algorithm selections.

### Implications

- Russian standards likely to be independent from NIST
- Based on similar mathematical foundations (lattice, code-based)
- May require separate compliance for organizations operating in Russia

<br>

## Algorithm Comparison: Global Landscape

### KEMs

| Region | Primary KEM | Alternative KEMs |
| -------- | ------------- | ------------------ |
| NIST | ML-KEM | HQC (2027) |
| Europe | ML-KEM | FrodoKEM, Classic McEliece |
| South Korea | SMAUG-T | NTRU+ |
| China | Aigis-enc | LAC.PKE |

### Signatures

| Region | Primary Signature | Alternative Signatures |
| -------- | ------------------ | ---------------------- |
| NIST | ML-DSA | SLH-DSA, FN-DSA (2025) |
| Europe | ML-DSA | SLH-DSA |
| South Korea | HAETAE | ALMer |
| China | Aigis-sig | TBD |

### Mathematical Basis Overlap

Despite independent development, most algorithms use similar mathematical foundations:

| Problem | NIST | South Korea | China | Europe |
| --------- | ------ | ------------- | ------- | -------- |
| Module-LWE | ML-KEM | SMAUG-T | Aigis-enc | ML-KEM |
| Plain LWE | - | - | LAC.PKE | FrodoKEM |
| NTRU lattices | - | NTRU+ | - | - |
| Code-based | HQC | - | - | Classic McEliece |

Overlap suggests that cryptanalytic advances against one algorithm type would likely affect algorithms from multiple regions.

<br>

## Next Steps

Proceed to **[Module 05: Performance Analysis](05_alt_pqc_perf_analysis.md)** for performance comparisons across all algorithms covered in this learning path.

**Module Navigation:**

| Previous | Current | Next |
|----------|---------|------|
| [03 - BIKE and HQC](05_alt_pqc_perf_analysis.md) | **04 - International PQC** | [05 - Performance Analysis](05_alt_pqc_perf_analysis.md.md) |
