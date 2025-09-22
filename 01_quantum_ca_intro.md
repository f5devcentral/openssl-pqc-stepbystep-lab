# Quantum-Resistant Certificate Authority Lab Guide

## Introduction

This lab guide provides a comprehensive walkthrough for building a quantum-resistant Certificate Authority (CA) infrastructure using OpenSSL 3.0+ with compliance to NSA's Commercial National Security Algorithm Suite 2.0 (CNSA 2.0) standards.

### Overview

This hands-on guide will teach you how to:
- Build a quantum-resistant Root CA with 10-year validity
- Create CNSA 2.0 compliant Intermediate CAs  
- Generate quantum-resistant certificates with proper SAN configuration
- Implement OCSP certificate revocation
- Apply proper Unix file permissions for security

### Important Note on Lab Approach

This guide presents commands for you to type directly rather than using automated scripts. This hands-on approach ensures you understand each step of the process and learn the OpenSSL command syntax. Take time to read the output of each command and understand what it's doing.

### CNSA 2.0 Requirements

The Commercial National Security Algorithm Suite 2.0 specifies the following quantum-resistant algorithms:

| Algorithm Type | CNSA 2.0 Requirement | OpenSSL Algorithm Name | Legacy Name | Use Case |
|---------------|---------------------|------------------------|-------------|----------|
| Digital Signatures | ML-DSA-65 (FIPS 204) | mldsa65 | dilithium3 | Standard security certificate signing |
| Digital Signatures | ML-DSA-87 (FIPS 204) | mldsa87 | dilithium5 | Highest security certificate signing |
| Key Establishment | ML-KEM-768 (FIPS 203) | mlkem768 | kyber768 | Standard security key exchange |
| Key Establishment | ML-KEM-1024 (FIPS 203) | mlkem1024 | kyber1024 | Highest security key exchange |
| Hash Functions | SHA-384 or SHA-512 | sha384, sha512 | - | Integrity verification |

**Important Note**: Only ML-DSA-65 (mldsa65) and ML-DSA-87 (mldsa87) are CNSA 2.0 compliant for digital signatures. Other algorithms like ML-DSA-44, Falcon-512, and Falcon-1024 are NOT part of CNSA 2.0.

**Algorithm Naming**: Newer versions of the OQS provider use NIST standardized names (mldsa65, mldsa87) instead of the legacy names (dilithium3, dilithium5).

### Prerequisites

1. **OpenSSL 3.2+** with OQS provider support
2. **Operating System**: Linux (Ubuntu 22.04+ or RHEL 9+ recommended)
3. **Permissions**: Root or sudo access for initial setup
4. **Storage**: At least 1GB free space for certificates and keys
5. **Text Editor**: Familiarity with vi, nano, or your preferred editor

### Installing OpenSSL with Quantum Support

First, install the required dependencies:

```bash
sudo apt-get update
sudo apt-get install -y build-essential cmake gcc libtool libssl-dev make ninja-build git pkg-config
```

Clone and build liboqs:

```bash
git clone -b main https://github.com/open-quantum-safe/liboqs.git
cd liboqs
mkdir build && cd build
cmake -GNinja -DCMAKE_INSTALL_PREFIX=/usr/local ..
ninja
sudo ninja install
```

Clone and build the OQS-OpenSSL Provider:

```bash
cd ~/
git clone https://github.com/open-quantum-safe/oqs-provider.git
cd oqs-provider
mkdir build && cd build
cmake -DOPENSSL_ROOT_DIR=/usr/ -DCMAKE_INSTALL_PREFIX=/usr/ ..
make
sudo make install
```

Verify the installation:

```bash
openssl list -providers -provider oqsprovider
```

You should see the OQS provider listed in the output.

### Directory Structure

Our lab will use the following directory structure:

```
/opt/sassycorp-ca/
├── root-ca/
│   ├── private/          # Root CA private keys (700)
│   ├── certs/            # Root CA certificates (755)
│   ├── crl/              # Certificate Revocation Lists (755)
│   ├── newcerts/         # Newly issued certificates (755)
│   ├── csr/              # Certificate Signing Requests (755)
│   ├── index.txt         # Certificate database (644)
│   ├── serial            # Serial number counter (644)
│   └── openssl.cnf       # Root CA configuration (644)
├── intermediate-ca/
│   ├── private/          # Intermediate CA private keys (700)
│   ├── certs/            # Intermediate CA certificates (755)
│   ├── crl/              # Certificate Revocation Lists (755)
│   ├── newcerts/         # Newly issued certificates (755)
│   ├── csr/              # Certificate Signing Requests (755)
│   ├── index.txt         # Certificate database (644)
│   ├── serial            # Serial number counter (644)
│   ├── crlnumber         # CRL number counter (644)
│   └── openssl.cnf       # Intermediate CA configuration (644)
└── ocsp/
    ├── private/          # OCSP responder keys (700)
    ├── certs/            # OCSP certificates (755)
    └── requests/         # OCSP requests (755)
```

### Security Considerations

**⚠️ IMPORTANT**: This lab guide is for **internal testing and educational purposes only**. In a production environment:

1. **Hardware Security Modules (HSMs)** should be used to protect private keys
2. **Air-gapped systems** should be used for Root CA operations
3. **Multi-person control** should be implemented for Root CA access
4. **Audit logging** must be enabled and monitored
5. **Regular security assessments** should be conducted

### Organization Details

Throughout this lab, we'll use the following organizational details:

- **Organization**: SassyCorp
- **Country**: US
- **State**: Washington
- **Locality**: Glacier
- **Email Domain**: sassycorp.internal
- **DNS Domain**: sassycorp.lab

### Lab Modules

This guide is organized into the following modules:

1. **Introduction** (this document) - Overview and prerequisites
2. **Building the Root CA** - Creating a CNSA 2.0 compliant Root CA with ML-DSA-87 (mldsa87)
3. **Building the Intermediate CA** - Establishing the intermediate tier with ML-DSA-65 (mldsa65)
4. **Building Certificates** - Generating end-entity certificates with CNSA 2.0 algorithms
5. **Certificate Revocation** - Implementing OCSP and CRL with quantum-resistant signatures

**CNSA 2.0 Compliance Note**: This lab uses only ML-DSA-65 (mldsa65) and ML-DSA-87 (mldsa87) algorithms throughout, ensuring full compliance with NSA's quantum-resistant cryptography requirements.

### Working with Configuration Files

Throughout this lab, you'll create several configuration files. When you see a configuration file in the guide:

1. Create the file using your preferred text editor
2. Copy the configuration content carefully
3. Save the file with the specified name and path
4. Verify the file permissions match what's specified

Example of creating a configuration file:

```bash
# Create and edit a file
nano /path/to/config.cnf

# After saving, set appropriate permissions
chmod 644 /path/to/config.cnf

# Verify the file was created correctly
ls -la /path/to/config.cnf
```

### Compliance Notes

This lab implements the following standards:
- **CNSA 2.0** - NSA's quantum-resistant algorithm requirements
- **RFC 5280** - Internet X.509 Public Key Infrastructure
- **RFC 6960** - Online Certificate Status Protocol (OCSP)
- **RFC 8446** - TLS 1.3 specifications
- **FIPS 204** - Module-Lattice-Based Digital Signature Standard
- **FIPS 203** - Module-Lattice-Based Key-Encapsulation Mechanism

### Getting Started

Before proceeding to Module 2, complete these setup steps:

1. Create a dedicated user for CA operations:

```bash
sudo useradd -r -s /bin/bash -m -d /opt/sassycorp-ca caadmin
sudo usermod -aG sudo caadmin
```

2. Switch to the CA admin user:

```bash
sudo su - caadmin
```

3. Create the base directory structure:

```bash
mkdir -p /opt/sassycorp-ca/{root-ca,intermediate-ca,ocsp}
```

4. Verify you have the correct user and directories:

```bash
whoami  # Should show: caadmin
ls -la /opt/sassycorp-ca/  # Should show three directories
```

### Validation Checklist

Before proceeding to Module 2, ensure you have:

- ✅ Installed OpenSSL 3.2+ with OQS provider
- ✅ Created the caadmin user
- ✅ Created the base directory structure
- ✅ Verified the OQS provider is available
- ✅ Understood the security considerations

### Algorithm Naming and Version Compatibility

The OQS provider algorithm names have evolved to match NIST standards:

| NIST Standard | New Name (Current) | Legacy Name (Older versions) |
|--------------|-------------------|------------------------------|
| ML-DSA-65 | mldsa65 | dilithium3 |
| ML-DSA-87 | mldsa87 | dilithium5 |

**Version Check**: After installing the OQS provider, check which names are available:

```bash
# Check for new names
openssl list -signature-algorithms -provider oqsprovider | grep -i mldsa

# If no results, check for legacy names
openssl list -signature-algorithms -provider oqsprovider | grep -i dilithium
```

---

**Next**: [Module 2 - Building the Root CA →](02_quantum_ca_root.md)