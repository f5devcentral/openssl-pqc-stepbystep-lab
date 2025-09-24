![License](https://img.shields.io/badge/license-MIT-blue.svg)
![OpenSSL](https://img.shields.io/badge/OpenSSL-3.2%2B-green.svg)
![CNSA](https://img.shields.io/badge/CNSA%202.0-Compliant-brightgreen.svg)
![Security Level](https://img.shields.io/badge/Security-Quantum%20Resistant-orange.svg)

# Quantum-Resistant Certificate Authority Lab Guide

This hands-on guide provides basic understanding and a wee tutorial for building a quantum-resistant Certificate Authority (CA) infrastructure using OpenSSL 3.0+ with compliance to NSA's Commercial National Security Algorithm Suite 2.0 (CNSA 2.0) standards. Fun times

## 🎯 Objective

Learn to build a complete Public Key Infrastructure (PKI) that is resistant to quantum computing attacks, following NSA and NIST requirements for evolving quantum-resistant mechanisms.

## 🔒 CNSA 2.0 Compliance

This lab uses NSA-approved quantum-resistant algorithms:

| Component | Algorithm | NIST Designation | OpenSSL Name | Security Level |
|-----------|-----------|-----------------|--------------|----------------|
| **Root CA** | ML-DSA-87 | FIPS 204 | mldsa87 | Level 5 (Highest) |
| **Intermediate CA** | ML-DSA-65 | FIPS 204 | mldsa65 | Level 3 (Standard) |
| **End-Entity Certificates** | ML-DSA-65/87 | FIPS 204 | mldsa65/87 | Level 3/5 |
| **Hash Function** | SHA-512 | FIPS 180-4 | sha512 | 256-bit security |

## 📚 Lab Modules

### [Module 1: Introduction](01_quantum_ca_intro.md)

- Prerequisites and system requirements
- Installing OpenSSL with quantum-resistant support
- Understanding CNSA 2.0 requirements
- Setting up the lab environment
- Creating directory structures and users

**Duration**: 30 minutes

### [Module 2: Building the Root CA](02_quantum_ca_root.md)

- Creating a 10-year Root CA with ML-DSA-87 (mldsa87)
- Configuring OpenSSL for quantum resistance
- Implementing proper Unix file permissions
- Setting up Subject Alternative Names (SANs)
- Creating Certificate Revocation Lists (CRL)
- Backing up the Root CA

**Duration**: 45 minutes

### [Module 3: Building the Intermediate CA](03_quantum_ca_intermediate.md)

- Creating a 5-year Intermediate CA with ML-DSA-65 (mldsa65)
- Establishing the certificate chain
- Configuring CRL Distribution Points
- Setting up OCSP responder certificates
- Implementing Authority Information Access (AIA)
- Creating certificate bundles

**Duration**: 45 minutes

### [Module 4: Building Certificates](04_quantum_ca_certificates.md)

- Generating server certificates for web services
- Creating user certificates for authentication
- Building high-security certificates with ML-DSA-87
- Configuring  SANs (DNS, IP, email, URI)
- Exporting certificates in multiple formats
- Managing certificate inventory

**Duration**: 60 minutes

### [Module 5: Certificate Revocation](05_quantum_ca_revocation.md)

- Implementing Certificate Revocation Lists (CRL)
- Setting up OCSP responders
- Revoking certificates with reason codes
- Testing revocation mechanisms
- Configuring OCSP stapling
- Creating revocation reports

**Duration**: 60 minutes

## 🎓 Learning Outcomes

After completing this lab, hopefully you will be able to:

- ✅ Build a complete PKI hierarchy
- ✅ Generate quantum-resistant certificates using ML-DSA algorithms
- ✅ Implement both CRL and OCSP revocation mechanisms
- ✅ Apply security best practices with proper file permissions
- ✅ Manage certificate lifecycle from creation to revocation
- ✅ Prepare infrastructure for the post-quantum era

## 🏗️ Infrastructure Overview

```bash
SassyCorp PKI Hierarchy
│
├── Root CA (ML-DSA-87 / mldsa87)
│   ├── Validity: 10 years
│   ├── Security Level: 5 (Highest)
│   └── Purpose: Sign Intermediate CAs only
│
└── Intermediate CA (ML-DSA-65 / mldsa65)
    ├── Validity: 5 years
    ├── Security Level: 3 (Standard)
    ├── Purpose: Issue end-entity certificates
    │
    ├── Server Certificates
    │   ├── Web Servers (ML-DSA-65 / mldsa65)
    │   ├── Database Servers (ML-DSA-87 / mldsa87)
    │   └── API Servers (ML-DSA-65 / mldsa65)
    │
    ├── User Certificates (ML-DSA-65 / mldsa65)
    │   ├── Email Protection
    │   └── Client Authentication
    │
    └── OCSP Responder (ML-DSA-65 / mldsa65)
        └── Certificate Status
```

## 🔑 Key Features

### Security Features

- **Quantum-Resistant Algorithms**: ML-DSA-65 and ML-DSA-87 only
- **SHA-512 Hashing**: Throughout the infrastructure
- **Secure Permissions**: 400 for keys, 444 for certificates
- **Complete Revocation**: CRL and OCSP support

### Compliance Features

- **CNSA 2.0**: Full compliance with NSA requirements
- **RFC 5280**: Internet X.509 PKI standards
- **RFC 6960**: OCSP implementation
- **FIPS 204**: Module-Lattice Digital Signature Standard

## 📋 Lab Approach

This lab guide uses a **manual, hands-on approach** where you:

1. Type each command directly
2. Observe the output in real-time
3. Understand what each parameter does
4. Learn to troubleshoot issues
5. Build muscle memory for OpenSSL commands

## 🏢 Organization Details

Throughout the lab, we use the fictional **SassyCorp** organization:

- **Organization**: SassyCorp
- **Country**: US
- **State**: Washington
- **Locality**: Glacier
- **Email Domain**: sassycorp.internal
- **DNS Domain**: sassycorp.lab

## ⚠️ Important Notes

### Security Warning

This lab is for **educational and internal testing purposes only**. In production:

- Use Hardware Security Modules (HSMs)
- Implement air-gapped Root CAs
- Enable  audit logging
- Conduct regular security assessments

### Algorithm Naming Compatibility

The OQS provider has updated to use NIST standard names:

- **Current**: `mldsa65` and `mldsa87`
- **Legacy**: `dilithium3` and `dilithium5`

Check your version and use the appropriate names. The lab guide uses the current NIST standard names.

### CNSA 2.0 Strict Compliance

This lab uses **ONLY** the following algorithms:

- ✅ ML-DSA-65 (mldsa65) - Standard security
- ✅ ML-DSA-87 (mldsa87) - Highest security
- ✅ SHA-512 - Hashing

The following are **NOT** used (not CNSA 2.0 compliant):

- ❌ ML-DSA-44 (mldsa44/dilithium2)
- ❌ Falcon algorithms
- ❌ SPHINCS+ algorithms
- ❌ Classic RSA/ECDSA

## 📊 Time Commitment

- **Total Lab Duration**: 4-5 hours
- **Module 1**: 30 minutes (Setup)
- **Module 2**: 45 minutes (Root CA)
- **Module 3**: 45 minutes (Intermediate CA)
- **Module 4**: 60 minutes (Certificates)
- **Module 5**: 60 minutes (Revocation)

## 📚 Additional Resources

### Standards and Specifications

- [NIST Post-Quantum Cryptography](https://csrc.nist.gov/projects/post-quantum-cryptography)
- [NSA CNSA 2.0 Suite](https://www.nsa.gov/Press-Room/News-Highlights/Article/Article/3148990/nsa-releases-future-quantum-resistant-qr-algorithm-requirements-for-national-se/)
- [FIPS 204: Module-Lattice-Based Digital Signature Standard](https://csrc.nist.gov/pubs/fips/204/ipd)
- [RFC 5280: Internet X.509 PKI](https://www.rfc-editor.org/rfc/rfc5280.html)
- [RFC 6960: OCSP](https://www.rfc-editor.org/rfc/rfc6960.html)

### OpenSSL Resources

- [OpenSSL Documentation](https://www.openssl.org/docs/)
- [Open Quantum Safe Project](https://openquantumsafe.org/)
- [OQS Provider for OpenSSL 3](https://github.com/open-quantum-safe/oqs-provider)

## 📄 License

This lab guide is provided under the MIT License. See [LICENSE](LICENSE) file for details.

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Open a Pull Request

## 💬 Support

For questions or issues:

- Open an issue in this repository
- Check the troubleshooting sections in each module
- Review the verification commands
- Ask us a quetion [@DevCentral!](https://community.f5.com)

## 🚦 Ready to Start?

Begin your journey into quantum-resistant PKI:

### [→ Start with Module 1: Introduction](01_quantum_ca_intro.md)

<<<<<<< HEAD
</br>

=======
>>>>>>> 77fbcad97512b4b564b0bece4dd8273e1dbcb874
Hack the planet! 🔐🚀
