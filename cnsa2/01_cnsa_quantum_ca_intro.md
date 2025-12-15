# Module 1: CNSA 2.0 Post-Quantum Certificate Authority Lab Guide

## Introduction

This lab guide provides a  walkthrough for building a quantum-resistant Certificate Authority (CA) infrastructure using OpenSSL 3.0+ with compliance to NSA's Commercial National Security Algorithm Suite 2.0 (CNSA 2.0) standards.

### Important Note on Lab Approach

This guide presents commands for you to type directly rather than using automated scripts. This hands-on approach ensures you understand each step of the process and learn the OpenSSL command syntax. Take time to read the output of each command and understand what it's doing... or don't. I'm breezy.

## CNSA 2.0 Requirements

The Commercial National Security Algorithm Suite 2.0 specifies the following quantum-resistant algorithms:

| Algorithm Type | CNSA 2.0 Requirement | OpenSSL Algorithm Name | Legacy Name | Use Case |
|---------------|---------------------|------------------------|-------------|----------|
| Digital Signatures | ML-DSA-65 (FIPS 204) | mldsa65 | dilithium3 | Standard security certificate signing |
| Digital Signatures | ML-DSA-87 (FIPS 204) | mldsa87 | dilithium5 | Highest security certificate signing |
| Key Establishment | ML-KEM-768 (FIPS 203) | mlkem768 | kyber768 | Standard security key exchange |
| Key Establishment | ML-KEM-1024 (FIPS 203) | mlkem1024 | kyber1024 | Highest security key exchange |
| Hash Functions | SHA-384 or SHA-512 | sha384, sha512 | - | Integrity verification |

**Important Note**: *Only ML-DSA-65 (mldsa65) and ML-DSA-87 (mldsa87) are CNSA 2.0 compliant for digital signatures. Other algorithms like ML-DSA-44, Falcon-512, and Falcon-1024 are NOT part of CNSA 2.0.*

### Prerequisites

1. **OpenSSL 3.2+** with OQS provider support installed
2. **Operating System**: Ubuntu 25.04 but you do you. Installing liboqs might differ
3. **Permissions**: Root or sudo access for initial setup
4. **Text Editor**: Familiarity with vi, vim, or your preferred editor

## Installing OpenSSL with Quantum Support

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
cmake -GNinja -DCMAKE_INSTALL_PREFIX=/usr ..
ninja
sudo ninja install
```

Clone and build the OQS-OpenSSL Provider:

```bash
cd ~/
git clone https://github.com/open-quantum-safe/oqs-provider.git
cd oqs-provider
mkdir build && cd build
cmake -DOPENSSL_ROOT_DIR=/usr -DCMAKE_INSTALL_PREFIX=/usr ..
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

```bash
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

## Getting Started

Before proceeding to Module 2, install the prerequisite packages and compile the liboqs libraries:

1. Create a dedicated user for CA operations:

```bash
sudo useradd -r -s /bin/bash -m -d /opt/sassycorp-ca caadmin
sudo usermod -aG sudo caadmin
sudo passwd caadmin
```

*Note: The last command will provide an interactive password prompt.*

2.Switch to the CA admin user:

```bash
sudo su - caadmin
```

3.Create the base directory structure:

```bash
mkdir -p /opt/sassycorp-ca/{root-ca,intermediate-ca,ocsp}
```

4.Verify you have the correct user and directories:

```bash
whoami  # Should show: caadmin
ls -la /opt/sassycorp-ca/  # Should show three directories
```

### Validation Checklist

Before proceeding to Module 2, ensure you have:

- Installed OpenSSL 3.2+ with OQS provider
- Created the caadmin user
- Created the base directory structure
- Verified the OQS provider is available

### Algorithm Naming and Version Compatibility

**Version Check**: After installing the OQS provider, check which names are available:

```bash
# Check for new names
openssl list -signature-algorithms -provider oqsprovider | grep -i mldsa

# If no results, check for legacy names
openssl list -signature-algorithms -provider oqsprovider | grep -i dilithium
```

If you see dilithium, you snagged an older version of oqs and MIGHT want to go back and check the liboqs build.  Otherwise you'll be substituting names all lab long.  That would be sad.

---

**Next**: [Building the Root CA →](02_cnsa_quantum_ca_root.md)
