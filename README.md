![License](https://img.shields.io/badge/license-MIT-blue.svg)
![OpenSSL](https://img.shields.io/badge/OpenSSL-3.5.x-green.svg)
![NIST PQC](https://img.shields.io/badge/NIST%20PQC-FIPS%20203%2F204%2F205-blue.svg)
![CNSA 2.0](https://img.shields.io/badge/CNSA%202.0-Compliant-brightgreen.svg)
![OQS](https://img.shields.io/badge/OQS-Non--NIST%20Algorithms-orange.svg)

# Post-Quantum Cryptography Certificate Authority Lab

## Hands-On Learning for Quantum-Resistant PKI Infrastructure

This hands-on lab guide provides tutorials for building quantum-resistant Certificate Authority (CA) infrastructure using OpenSSL. This repository provides three distinct learning paths based on your compliance requirements and algorithm interests. Who's ready to party?

<br>

## 🎯 Choose Your Learning Path

This repository offers three learning paths. Select the path that aligns with your organization's requirements:

| | **FIPS 203/204/205 Path** | **CNSA 2.0 Path** | **Alternative Algorithms Path** |
| --- | --- | --- | --- |
| **Target Audience** | Commercial organizations | Government contractors, classified systems | Researchers, international compliance, defense-in-depth |
| **Compliance Standard** | NIST FIPS standards | NSA CNSA 2.0 | Non-NIST algorithms, international standards |
| **Algorithm Coverage** | ML-DSA, ML-KEM, SLH-DSA | ML-DSA-65/87, ML-KEM-768/1024 | FrodoKEM, BIKE, HQC |
| **Use Case** | General quantum-resistant infrastructure | National security systems | Algorithm diversity, conservative security |

<br>

## 📚 [Learning Path 1: NIST FIPS 203/204/205](/fipsqs/00_fips_quantum_ca_intro.md)

**For commercial organizations implementing quantum-resistant cryptography using NIST standards.**

This path uses OpenSSL 3.5.3's native post-quantum cryptography support—no external quantum library providers required. So nice, so easy.

### Modules

| Module | Description |
| -------- | ------------- |
| [00 - Introduction](/fipsqs/00_fips_quantum_ca_intro.md) | Overview of FIPS 203/204/205, prerequisites, and lab objectives
| [01 - Environment Setup](/fipsqs/01_fips_quantum_ca_environment.md) | Verifying OpenSSL 3.5.x with PQC support
| [02 - Root CA](/fipsqs/02_fips_quantum_ca_root.md) | Building a Root CA with ML-DSA-87
| [03 - Intermediate CA](/fipsqs/03_fips_quantum_ca_intermediate.md) | Creating an Intermediate CA with ML-DSA-65
| [04 - Certificates](/fipsqs/04-fips_quantum_ca_certs.md) | Issuing end-entity certificates for servers and users
| [05 - Revocation](/fipsqs/05_fips_quantum_ca_recovation.md) | Implementing OCSP and CRL certificate revocation
| [06 - Hybrid Methods](/fipsqs/06_fips_quantum_ca_hybrid_methods.md) | IETF hybrid PQC methods (X25519MLKEM768, composite signatures)

### Algorithms Covered

- **ML-DSA-44/65/87** (FIPS 204) - Lattice-based signatures
- **ML-KEM-512/768/1024** (FIPS 203) - Lattice-based key encapsulation
- **SLH-DSA** variants (FIPS 205) - Hash-based signatures
- **X25519MLKEM768** - Hybrid TLS 1.3 key exchange (IETF-based bonus lab)

<br>

## 📚 [Learning Path 2: NSA CNSA 2.0](/cnsa2/01_cnsa_quantum_ca_intro.md)

**For government contractors and organizations requiring CNSA 2.0 compliance.**

This path uses OpenSSL 3.2+ with user-compiled Open Quantum Safe (OQS) providers for strict CNSA 2.0 algorithm compliance.

### Modules

| Module | Description |
| -------- | ------------- |
| [01 - Introduction](/cnsa2/01_cnsa_quantum_ca_intro.md) | Overview of CNSA 2.0 requirements and compliance deadlines
| [02 - Root CA](/cnsa2/02_cnsa_quantum_ca_root.md) | Building a Root CA with ML-DSA-87
| [03 - Intermediate CA](/cnsa2/03_cnsa_quantum_ca_intermediate.md) | Creating an Intermediate CA with ML-DSA-65
| [04 - Certificates](/cnsa2/04_cnsa_quantum_ca_certificates.md) | Issuing CNSA 2.0 compliant certificates
| [05 - Revocation](/cnsa2/05_cnsa_quantum_ca_revocation.md) | Implementing OCSP and CRL certificate revocation

### Algorithms Covered

| Algorithm Type | Approved Algorithms | NIST Designation |
| ---------------- | --------------------- | ------------------ |
| Digital Signatures | ML-DSA-65, ML-DSA-87 | FIPS 204 |
| Key Establishment | ML-KEM-768, ML-KEM-1024 | FIPS 203 |
| Hash Functions | SHA-384, SHA-512 | FIPS 180-4 |

**Note:** *CNSA 2.0 currently does NOT support ML-DSA-44, SLH-DSA, or Falcon algorithms.*

<br>

## 📚 [Learning Path 3: Alternative PQC Algorithms (Includes NIST proposed)](/altpqc/00_alt_pqc_introduction.md)

**For researchers, organizations requiring algorithm diversity, and those interested in international PQC implementations.**

This path explores post-quantum algorithms outside the primary NIST FIPS standards, providing options and understanding of the broader PQC landscape. We'll use OpenSSL 3.5.x with OQS provider for alternate algorithm access. The addendum to enable these algorithms is super fun and can be found here [addendum_updating_openssl_pqc](addendum_updating_openssl_pqc.md).

### Modules

| Module | Description |
| -------- | ------------- |
| [00 - Introduction](/altpqc/00_alt_pqc_introduction.md) | Overview of non-NIST algorithms, international standards, use cases |
| [01 - Environment Setup](/altpqc/01_alt_pqc_environment.md) | Ubuntu 25.10, OpenSSL 3.5.x, OQS provider configuration |
| [02 - FrodoKEM](/altpqc/02_alt_pqc_frodokem.md) | Conservative unstructured lattice KEM (European recommended; BSI, ANSSI) |
| [03 - BIKE and HQC](/altpqc/03_alt_pqc_bike_hqc.md) | Code-based KEMs (HQC is NIST-selected backup) |
| [04 - International PQC](/altpqc/04_alt_pqc_interational.md) | EU, South Korean, and Chinese algorithm standards |
| [05 - Performance Analysis](/altpqc/05_alt_pqc_perf_analysis.md) | comparing algorithms, latency impacts, use cases, nerd stats |

### Algorithms Covered

| Algorithm | Type | Mathematical Basis | Key Characteristic |
|----------- | ------ | ------------------- | ------------------- |
| **FrodoKEM** | KEM | Unstructured lattice (LWE) | Conservative security, European endorsed (BSI, ANSSI) |
| **BIKE** | KEM | Code-based (QC-MDPC) | NIST Round 4 candidate |
| **HQC** | KEM | Code-based (Quasi-cyclic) | NIST-selected backup to ML-KEM |

<br>

## 🔧 Prerequisites

### System Requirements

- **Operating System(S):** CNSA 2.0 - Ubuntu 25.04 with OpenSSL 3.2+. NIST FIPS and Alt PQC - Ubuntu 25.10 with OpenSSL 3.5.3.
- **Permissions:** Root or sudo access
- ***Note:*** *The CNSA guide is intended to require using external OQS libraries with earlier versions of OpenSSL (in this case 3.2). The FIPS and Alt PQC labs rely on a more curent release of Ubuntu (25.10) with current version of OpenSSL (3.5.3) which has all FIPS PQC requirements built in and will support newer versions of the OQS libraries. See the addendum link below for compiling OQS.*


### Required Knowledge

- Basic Linux command line familiarity
- Understanding of PKI concepts (certificates, CAs, chains)
- Familiarity with X.509 certificate structure
- TLS/SSL fundamentals (for KEM testing)

<br>

## 📖 Environment Setup

For detailed instructions on setting up your PQC environment, including building the OQS provider from source, see:

### [ADDENDUM: Compiling Open Quantum Safe (OQS) Libraries for OpenSSL Environment Setup](/addendum_updating_openssl_pqc.md)

This addendum covers:

- OQS provider installation for Ubuntu 25.10
- Building liboqs with HQC enabled
- Enabling HQC in oqs-provider
- Troubleshooting common installation issues

<br>

## ⚠️ Important Notes

### Educational Use

This lab is designed for **educational and internal testing purposes**. Production deployments should:

- Use Hardware Security Modules (HSMs) for key storage
- Implement air-gapped Root CAs, offline secured storage preferred... on zip drives
- Enable comprehensive audit logging
- Follow organizational security policies

### Manual Command Entry

All learning paths use **manual command entry only**—no scripts. This approach ensures you:

- Understand each step of the PKI workflow
- Learn proper OpenSSL syntax and options
- Build troubleshooting skills
- Develop muscle memory for cryptographic operations
- You COULD copy/paste but you're only cheating yourselves... *"sigh"*

<br>

## 📖 Additional Resources

### NIST Standards

- [FIPS 203: ML-KEM Standard](https://csrc.nist.gov/pubs/fips/203/final)
- [FIPS 204: ML-DSA Standard](https://csrc.nist.gov/pubs/fips/204/final)
- [FIPS 205: SLH-DSA Standard](https://csrc.nist.gov/pubs/fips/205/final)

### NSA CNSA 2.0

- [CNSA 2.0 Announcement](https://media.defense.gov/2022/Sep/07/2003071834/-1/-1/0/CSA_CNSA_2.0_ALGORITHMS_.PDF)
- [CNSA 2.0 FAQ](https://media.defense.gov/2022/Sep/07/2003071836/-1/-1/0/CSI_CNSA_2.0_FAQ_.PDF)

### OpenSSL and OQS

- [OpenSSL 3.5 Documentation](https://www.openssl.org/docs/)
- [Open Quantum Safe Project](https://openquantumsafe.org/)
- [OQS Provider GitHub](https://github.com/open-quantum-safe/oqs-provider)

### IETF Standards

- [RFC 9794: PQ/T Hybrid Terminology](https://datatracker.ietf.org/doc/rfc9794/)
- [Hybrid Key Exchange in TLS 1.3](https://datatracker.ietf.org/doc/draft-ietf-tls-hybrid-design/)

### International PQC

- [South Korea KpqC Competition](https://www.kpqc.or.kr/)
- [European Post-Quantum Cryptography Study](https://www.enisa.europa.eu/publications/post-quantum-cryptography-current-state-and-quantum-mitigation)

<br>

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Submit a pull request with clear documentation

Please refer to [contributing](/contributing.md) for more details.

<br>

## 📄 License

This lab guide is provided under the MIT License. See [LICENSE](/LICENSE) file for details.
