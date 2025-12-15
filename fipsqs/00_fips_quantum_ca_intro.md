# FIPS 203/204/205 Post-Quantum Certificate Authority Lab Guide

## Overview

This learning path guides you through building a complete quantum-resistant Public Key Infrastructure (PKI) using the NIST post-quantum cryptography standards: FIPS 203, FIPS 204, and FIPS 205. You will use OpenSSL 3.5.3's native support for these algorithms—no external providers required. For the enjoyment of compiling OQS libraries, we created an addenum for [OpenSSL alternate installs](/addenum_updating_openssl_pqc.md).

By the end of this lab, you will have built a fully functional Certificate Authority hierarchy for Sassy Corp, resistant to attacks from quantum relevant systems.

---

## Learning Objectives

After completing this module, you will be able to:

- Explain the purpose of FIPS 203, 204, and 205
- Identify appropriate algorithm choices for different security levels
- Understand the structure of a quantum-resistant PKI
- Describe the components you will build in this lab

---

## Understanding the NIST Post-Quantum Standards

In August 2024, NIST finalized the first set of post-quantum cryptography standards. These standards define algorithms that are believed to be secure against both classical and quantum computer attacks... for now.

### FIPS 203: ML-KEM (Module-Lattice-Based Key Encapsulation Mechanism)

**Purpose:** Secure key exchange between parties

**Based on:** [CRYSTALS-Kyber](https://pq-crystals.org/kyber/) algorithm

**Use cases:**

- TLS key exchange
- VPN tunnel establishment
- Encrypted messaging key agreement
- Any scenario requiring secure key establishment

**Parameter Sets:**

| Variant | Security Level | Public Key Size | Ciphertext Size |
|---------|---------------|-----------------|-----------------|
| ML-KEM-512 | Level 1 (128-bit) | 800 bytes | 768 bytes |
| ML-KEM-768 | Level 3 (192-bit) | 1,184 bytes | 1,088 bytes |
| ML-KEM-1024 | Level 5 (256-bit) | 1,568 bytes | 1,568 bytes |

### FIPS 204: ML-DSA (Module-Lattice-Based Digital Signature Algorithm)

**Purpose:** Digital signatures for authentication and non-repudiation

**Based on:** [CRYSTALS-Dilithium](https://pq-crystals.org/dilithium/index.shtml) algorithm

**Use cases:**

- Certificate signing
- Code signing
- Document signing
- Any scenario requiring digital signatures

**Parameter Sets:**

| Variant | Security Level | Public Key Size | Signature Size |
|---------|---------------|-----------------|----------------|
| ML-DSA-44 | Level 2 (128-bit) | 1,312 bytes | 2,420 bytes |
| ML-DSA-65 | Level 3 (192-bit) | 1,952 bytes | 3,293 bytes |
| ML-DSA-87 | Level 5 (256-bit) | 2,592 bytes | 4,595 bytes |

### FIPS 205: SLH-DSA (Stateless Hash-Based Digital Signature Algorithm)

**Purpose:** Digital signatures using hash-based cryptography

**Based on:** [SPHINCS+](https://sphincs.org/) algorithm

**Use cases:**

- Long-term signatures (documents that must remain valid for decades)
- High-assurance environments requiring conservative security assumptions
- Backup signature algorithm for defense-in-depth

**Key Characteristics:**

- Based on well-understood hash function security
- Larger signatures than ML-DSA but smaller public keys
- Two variants: "f" (fast) and "s" (small signatures)
- Multiple hash function options: SHA-2 and SHAKE

**Common Parameter Sets:**

| Variant | Security Level | Public Key | Signature Size |
|---------|---------------|------------|----------------|
| SLH-DSA-SHA2-128f | Level 1 | 32 bytes | 17,088 bytes |
| SLH-DSA-SHA2-192f | Level 3 | 48 bytes | 35,664 bytes |
| SLH-DSA-SHA2-256f | Level 5 | 64 bytes | 49,856 bytes |

---

## Lab Architecture

In this lab, you will build a three-tier PKI hierarchy for Sassy Corp:

```
┌─────────────────────────────────────────────────────────────────┐
│                     Sassy Corp Root CA                          │
│                     (ML-DSA-87 / Level 5)                       │
└─────────────────────────────────────────────────────────────────┘
                                │
                                │ Signs
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                  Sassy Corp Intermediate CA                     │
│                     (ML-DSA-65 / Level 3)                       │
└─────────────────────────────────────────────────────────────────┘
                                │
                                │ Signs
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                    End-Entity Certificates                      │
│              (ML-DSA-44/65/87 / Levels 2-5)                     │
│                                                                 │
│  ┌──────────────┐ ┌──────────────┐ ┌──────────────┐             │
│  │   Server     │ │    User      │ │    OCSP      │             │
│  │ Certificates │ │ Certificates │ │  Responder   │             │
│  └──────────────┘ └──────────────┘ └──────────────┘             │
└─────────────────────────────────────────────────────────────────┘
```

### Component Details

| Component | Algorithm | Security Level | Purpose |
|-----------|-----------|---------------|---------|
| Root CA | ML-DSA-87 | Level 5 (256-bit) | Trust anchor, highest security |
| Intermediate CA | ML-DSA-65 | Level 3 (192-bit) | Day-to-day certificate issuance |
| Server Certificates | ML-DSA-65 | Level 3 (192-bit) | TLS server authentication |
| User Certificates | ML-DSA-44 | Level 2 (128-bit) | Client authentication, email signing |
| OCSP Responder | ML-DSA-65 | Level 3 (192-bit) | Real-time certificate status |

---

## Directory Structure

You will create the following directory structure:

```
/opt/sassycorp-pqc/
├── root-ca/
│   ├── private/          # Root CA private keys (chmod 700)
│   ├── certs/            # Root CA certificates
│   ├── crl/              # Certificate Revocation Lists
│   ├── newcerts/         # Issued certificate archive
│   └── openssl.cnf       # Root CA configuration
├── intermediate-ca/
│   ├── private/          # Intermediate CA private keys (chmod 700)
│   ├── certs/            # Intermediate CA certificates
│   ├── crl/              # Intermediate CRLs
│   ├── newcerts/         # Issued certificate archive
│   ├── csr/              # Certificate Signing Requests
│   └── openssl.cnf       # Intermediate CA configuration
└── ocsp/
    ├── private/          # OCSP responder private key
    ├── certs/            # OCSP responder certificate
    └── logs/             # OCSP server logs
```

---

## Security Practices

Throughout this lab, you will implement proper security practices:

### File Permissions

| Resource | Permission | Rationale |
|----------|------------|-----------|
| Private key directories | 700 | Only owner can access |
| Private keys | 400 | Read-only by owner |
| Certificates | 444 | Read by all (public data) |
| Configuration files | 644 | Read by all, write by owner |
| Database files | 644 | Read by all, write by owner |

### Separation of Duties

- Root CA operates offline (in production, use air-gapped systems)
- Intermediate CA handles day-to-day operations
- OCSP responder uses a dedicated signing certificate with limited scope

---

## Conventions Used

Throughout this lab:

- Commands to type are shown in code blocks:
  ```bash
  openssl version
  ```

- File contents are shown with syntax highlighting:
  ```ini
  [ca]
  default_ca = CA_default
  ```

- Important notes are highlighted:
  > **Note:** Pay attention to these callouts for critical information.

- Verification steps confirm your work:
  ```bash
  # Expected output shown after verification commands
  ```

---

## Ready to Begin

You now understand the purpose and structure of this lab. Prepare yourself and let's get PQCing!

---

**Next:** [Environment Setup →](01_fips_quantum_ca_environment.md)
