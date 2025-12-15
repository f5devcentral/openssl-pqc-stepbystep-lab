![License](https://img.shields.io/badge/license-MIT-blue.svg)
![OpenSSL](https://img.shields.io/badge/OpenSSL-3.2%2B-green.svg)
![NIST PQC](https://img.shields.io/badge/NIST%20PQC-FIPS%20203%2F204%2F205-blue.svg)
![CNSA 2.0](https://img.shields.io/badge/CNSA%202.0-Compliant-brightgreen.svg)
![Security Level](https://img.shields.io/badge/Security-Quantum%20Resistant-orange.svg)

# Post-Quantum Cryptography Certificate Authority Lab

## Hands-On Learning for Quantum-Resistant PKI Infrastructure

This hands-on lab guide provides basic understanding and a tiny tutorial for building quantum-resistant Certificate Authority (CA) infrastructure using OpenSSL. This repository provides two distinct learning paths based on your compliance requirements

---
<br>

## 🎯 Choose Your Learning Path

This repository offers two parallel learning tracks. Select the path that aligns with your organization's requirements:

| | **FIPS 203/204/205 Path** | **CNSA 2.0 Path** |
|---|---|---|
| **Target Audience** | Commercial organizations, compliance needs | Government contractors, classified systems |
| **Compliance Standard** | NIST FIPS standards | NSA Commercial National Security Algorithm Suite 2.0 |
| **Algorithm Flexibility** | Full FIPS algorithm suites (ML-DSA-44/65/87, SLH-DSA) | Restricted to CNSA 2.0 approved (ML-DSA-65/87 only) |
| **Use Case** | General quantum-resistant infrastructure | National security systems, defense contracts |

---
<br>

## 📚 [Learning Path: NIST FIPS 203/204/205](/fipsqs/00_fips_quantum_ca_intro.md)

**For commercial organizations implementing quantum-resistant cryptography using NIST standards.**

This path uses OpenSSL 3.5.3's native post-quantum cryptography support—no external quantum library providers required. So nice, so easy.

### Modules

| Module | Description | Duration |
|--------|-------------|----------|
| [00 - Introduction](/fips/00_fips_quantum_ca_intro.md) | Overview of FIPS 203/204/205, prerequisites, and lab objectives | 15 min |
| [01 - Environment Setup](/fipsqs/01_fips_quantum_ca_environment.md) | Installing and configuring OpenSSL 3.5.3 with PQC support | 30 min |
| [02 - Root CA](/fipsqs/02_fips_quantum_ca_root.md) | Building a Root CA with ML-DSA-87 | 45 min |
| [03 - Intermediate CA](/fipsqs/03_fips_quantum_ca_intermediate.md) | Creating an Intermediate CA with ML-DSA-65 | 45 min |
| [04 - Certificates](/fipsqs/04-fips_quantum_ca_certs.md) | Issuing end-entity certificates for servers and users | 60 min |
| [05 - Revocation](/fipsqs/05_fips_quantum_ca_recovation.md) | Implementing OCSP and CRL certificate revocation | 60 min |
| [06 - Hybrid Methods](/fipsqs/06_fips_quantum_ca_hybrid_methods.md) | IETF hybrid PQC methods (X25519MLKEM768, composite signatures) | 45 min |



### Algorithms Covered

- **ML-DSA-44** (FIPS 204) - NIST Security Level 2
- **ML-DSA-65** (FIPS 204) - NIST Security Level 3
- **ML-DSA-87** (FIPS 204) - NIST Security Level 5
- **SLH-DSA** variants (FIPS 205) - Hash-based signatures
- **ML-KEM** variants (FIPS 203) - Key encapsulation
- **Hybrid Key Exchange (Hybrid KEX)** - X25519MLKEM768 for TLS 1.3 *(IETF-based bonus learning)*

---
<br>

## 📚 [Learning Path: NSA CNSA 2.0](/cnsa2/01_cnsa_quantum_ca_intro.md)

**For government contractors and organizations requiring CNSA 2.0 compliance.**

This path uses OpenSSL 3.2+ with the Open Quantum Safe (OQS) provider for strict CNSA 2.0 algorithm compliance.

### Modules

| Module | Description | Duration |
|--------|-------------|----------|
| [00 - Introduction](/cnsa2/00-INTRODUCTION.md) | Overview of CNSA 2.0 requirements and compliance deadlines | 15 min |
| [01 - Environment Setup](/cnsa2/01-ENVIRONMENT-SETUP.md) | Installing OpenSSL with OQS provider | 45 min |
| [02 - Root CA](/cnsa2/02-ROOT-CA.md) | Building a Root CA with ML-DSA-87 (Dilithium5) | 45 min |
| [03 - Intermediate CA](/cnsa2/03-INTERMEDIATE-CA.md) | Creating an Intermediate CA with ML-DSA-65 (Dilithium3) | 45 min |
| [04 - Certificates](/cnsa2/04-CERTIFICATES.md) | Issuing CNSA 2.0 compliant certificates | 60 min |
| [05 - Revocation](/cnsa2/05-REVOCATION.md) | Implementing OCSP and CRL certificate revocation | 60 min |

### CNSA 2.0 Algorithm Requirements

| Algorithm Type | Approved Algorithms | NIST Designation |
|----------------|---------------------|------------------|
| Digital Signatures | ML-DSA-65, ML-DSA-87 | FIPS 204 |
| Key Establishment | ML-KEM-768, ML-KEM-1024 | FIPS 203 |
| Hash Functions | SHA-384, SHA-512 | FIPS 180-4 |

**Note:** *CNSA 2.0 currently does NOT approve ML-DSA-44, SLH-DSA, or Falcon algorithms.* 

---
<br>

## 🔧 Prerequisites

### System Requirements

- **Operating System(S):** For CNSA 2.0, use a recent Ubuntu LTS with OpenSSL 3.2+. For the FIPS lab, we relied on Ubuntu 25.10 with OpenSSL 3.5.3.
- **Permissions:** Root or sudo access
- **Note** *The CNSA guide is intended to require using external OQS libraries with earlier versions of OpenSSL (in this case 3.2).  The FIPS lab relies on a curent release of Ubuntu (25.10) with current version of OpenSSL (3.5.3) which has all FIPS quantum requirements built in. I mean, you can compile if you want to.... We're not your mom.*

### Required Knowledge

- Basic Linux command line familiarity
- Understanding of PKI concepts (certificates, CAs, chains)
- Familiarity with X.509 certificate structure

---
<br>

## ⚠️ Important Notes

### Security Considerations

This lab guide is for **internal testing and educational purposes only**. In a production environment:

1. **Hardware Security Modules (HSMs)** should be used to protect private keys
2. **Air-gapped systems** should be used for Root CA operations
3. **Multi-person control** should be implemented for Root CA access
4. **Audit logging** must be enabled and monitored
5. **Regular security assessments** should be conducted

### Manual Command Entry

Both learning paths use **manual command entry only**—no scripts. This approach ensures you:

- Understand each step of the PKI workflow
- Observe the output in real-time
- Learn proper OpenSSL syntax and options
- Build troubleshooting skills
- Develop muscle memory for cryptographic operations

### Working with Configuration Files

Throughout this lab, you'll create several configuration files. When you see a configuration file in the guide:

1. Create the file using your preferred text editor
2. Copy the configuration content carefully
3. Save the file with the specified name and path
4. Verify the file permissions match what's specified

Example of creating a configuration file:

```bash
# Create and edit a file
vim /path/to/config.cnf

# After saving, set appropriate permissions
chmod 644 /path/to/config.cnf

# Verify the file was created correctly
ls -la /path/to/config.cnf
```

---
<br>

## 📖 Additional Resources

### NIST Standards

- [FIPS 203: ML-KEM Standard](https://csrc.nist.gov/pubs/fips/203/final)
- [FIPS 204: ML-DSA Standard](https://csrc.nist.gov/pubs/fips/204/final)
- [FIPS 205: SLH-DSA Standard](https://csrc.nist.gov/pubs/fips/205/final)
- [NIST Post-Quantum Cryptography](https://csrc.nist.gov/projects/post-quantum-cryptography)

### NSA CNSA 2.0

- [CNSA 2.0 Announcement](https://media.defense.gov/2022/Sep/07/2003071834/-1/-1/0/CSA_CNSA_2.0_ALGORITHMS_.PDF)
- [CNSA 2.0 FAQ](https://media.defense.gov/2022/Sep/07/2003071836/-1/-1/0/CSI_CNSA_2.0_FAQ_.PDF)

### OpenSSL

- [OpenSSL 3.5 Documentation](https://www.openssl.org/docs/)
- [OpenSSL PQC Announcement](https://openssl-library.org/post/2025-02-04-release-announcement-3.5/)
- [Open Quantum Safe Project](https://openquantumsafe.org/)
- [OQS Provider for OpenSSL 3](https://github.com/open-quantum-safe/oqs-provider)

### IETF Standards

- [RFC 9794: PQ/T Hybrid Terminology](https://datatracker.ietf.org/doc/rfc9794/)
- [Hybrid Key Exchange in TLS 1.3](https://datatracker.ietf.org/doc/draft-ietf-tls-hybrid-design/)
- [Composite ML-DSA for X.509](https://datatracker.ietf.org/doc/draft-ietf-lamps-pq-composite-sigs/)

---

## 🤝 Contributing

Contributions are welcome! Please:

Please refer to [contributing.md](contributing.md) for more details.

---

## 📄 License

This lab guide is provided under the MIT License. See [LICENSE](LICENSE) file for details.

---
<br>
<br>
<br>
I know someone who LOVES emojis... 🫵👻😻🤡🦹‍♀️🙆‍♂️🧚‍♀️🧛‍♀️🫵