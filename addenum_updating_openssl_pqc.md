# Addendum: PQC Environment Setup

## Overview

This addendum provides instructions for installing the Open Quantum Safe (OQS) provider on Ubuntu 25.10 systems with OpenSSL 3.5.3. The OQS provider extends OpenSSL's native PQC support with additional algorithms not included in the NIST FIPS standards.

| Component | Version | Purpose |
|-----------|---------|---------|
| Ubuntu | 25.10 | Operating system |
| OpenSSL | 3.5.3 (native) | Cryptographic library with FIPS PQC |
| liboqs | Latest | OQS algorithm implementations |
| oqs-provider | Latest | OpenSSL provider for OQS algorithms |

---

## What You Get

### Native OpenSSL 3.5.3 Algorithms (No OQS Required)

Ubuntu 25.10 includes OpenSSL 3.5.3 with native support for NIST FIPS algorithms:

| Algorithm | Type | Standard |
|-----------|------|----------|
| ML-KEM-512/768/1024 | KEM | FIPS 203 |
| ML-DSA-44/65/87 | Signature | FIPS 204 |
| SLH-DSA variants | Signature | FIPS 205 |
| X25519MLKEM768 | Hybrid KEM | TLS 1.3 |

### Additional Algorithms via OQS Provider

The OQS provider adds algorithms not in the NIST FIPS standards:

| Algorithm | Type | Use Case |
|-----------|------|----------|
| FrodoKEM | KEM | Conservative unstructured lattice |
| NTRU | KEM | Compact keys, long security history |
| Classic McEliece | KEM | Maximum conservative security |
| BIKE | KEM | Code-based alternative |
| HQC | KEM | NIST-selected backup (2027 standard) |

---

## Prerequisites

### Verify Ubuntu Version

```bash
lsb_release -d
```

**Expected output:**

```
Description:    Ubuntu 25.10
```

### Verify OpenSSL Version

```bash
openssl version
```

**Expected output:**

```
OpenSSL 3.5.3 <date>
```

### Verify Native PQC Support

Confirm ML-KEM is available natively:

```bash
openssl list -kem-algorithms | grep -i mlkem
```

**Expected output:**

```
MLKEM512 @ default
MLKEM768 @ default
MLKEM1024 @ default
X25519MLKEM768 @ default
```

---

## Step 1: Install Build Dependencies

Update package lists:

```bash
sudo apt update
```

Install required build tools and libraries:

```bash
sudo apt install -y \
    build-essential \
    cmake \
    ninja-build \
    git \
    libssl-dev \
    pkg-config \
    python3 \
    python3-pip \
    astyle
```

**Package purposes:**

| Package | Purpose |
|---------|---------|
| build-essential | GCC, make, standard build tools |
| cmake | Build system for liboqs and oqs-provider |
| ninja-build | Fast build system |
| git | Source code retrieval |
| libssl-dev | OpenSSL development headers |
| pkg-config | Library detection (required for liboqs) |
| python3 | Build script requirements |
| astyle | Code formatting |

---

## Step 2: Create Build Directory

Create a working directory for the build process:

```bash
mkdir -p ~/oqs-build
```

```bash
cd ~/oqs-build
```

---

## Step 3: Clone and Build liboqs

liboqs is the underlying library that implements the alternative PQC algorithms.

### Clone the Repository

```bash
git clone --depth 1 https://github.com/open-quantum-safe/liboqs.git
```

```bash
cd liboqs
```

### Create Build Directory

```bash
mkdir build
```

```bash
cd build
```

### Configure the Build

```bash
cmake -GNinja \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    -DBUILD_SHARED_LIBS=ON \
    ..
```

**Configuration options:**

| Option | Purpose |
|--------|---------|
| -GNinja | Use Ninja build system (faster) |
| -DCMAKE_INSTALL_PREFIX=/usr/local | Install to standard system location |
| -DBUILD_SHARED_LIBS=ON | Build shared libraries |

### Build liboqs

```bash
ninja
```

Build time varies by system (typically 2-5 minutes).

### Run Tests (Optional but Recommended)

```bash
ninja run_tests
```

All tests should pass. Minor failures in specific algorithm tests are usually acceptable.

### Install liboqs

```bash
sudo ninja install
```

### Update Library Cache

```bash
sudo ldconfig
```

### Verify Installation

```bash
ls -la /usr/local/lib/liboqs*
```

**Expected output (similar to):**

```
-rw-r--r-- 1 root root <size> <date> /usr/local/lib/liboqs.so
-rw-r--r-- 1 root root <size> <date> /usr/local/lib/liboqs.so.0
-rw-r--r-- 1 root root <size> <date> /usr/local/lib/liboqs.so.0.x.x
```

---

## Step 4: Clone and Build OQS Provider

Return to the build directory:

```bash
cd ~/oqs-build
```

### Clone the Repository

```bash
git clone --depth 1 https://github.com/open-quantum-safe/oqs-provider.git
```

```bash
cd oqs-provider
```

### Create Build Directory

```bash
mkdir build
```

```bash
cd build
```

### Configure the Build

```bash
cmake -GNinja \
    -DCMAKE_INSTALL_PREFIX=/usr/local \
    -Dliboqs_DIR=/usr/local \
    ..
```

### Build the Provider

```bash
ninja
```

### Verify the Provider Built Successfully

```bash
ls -la lib/oqsprovider.so
```

**Expected output:**

```
-rwxr-xr-x 1 <user> <group> <size> <date> lib/oqsprovider.so
```

---

## Step 5: Install the OQS Provider

### Locate OpenSSL Providers Directory

Find where OpenSSL expects providers:

```bash
openssl version -d
```

Note the OPENSSLDIR path.

```bash
ls /usr/lib/x86_64-linux-gnu/ossl-modules/
```

This shows existing providers.

### Copy Provider to System Location

```bash
sudo cp ~/oqs-build/oqs-provider/build/lib/oqsprovider.so \
    /usr/lib/x86_64-linux-gnu/ossl-modules/
```

### Set Permissions

```bash
sudo chmod 644 /usr/lib/x86_64-linux-gnu/ossl-modules/oqsprovider.so
```

### Verify Installation

```bash
ls -la /usr/lib/x86_64-linux-gnu/ossl-modules/oqsprovider.so
```

---

## Step 6: Configure OpenSSL to Load OQS Provider

### Edit OpenSSL Configuration

```bash
sudo vim /etc/ssl/openssl.cnf
```

Add the following at the **beginning** of the file, before any other sections:

```ini
# OpenSSL configuration with OQS Provider
openssl_conf = openssl_init

[openssl_init]
providers = provider_sect

[provider_sect]
default = default_sect
oqsprovider = oqsprovider_sect

[default_sect]
activate = 1

[oqsprovider_sect]
activate = 1
```

Save and exit (`:wq` in vim).

**Important:** Ensure there are no duplicate `openssl_conf` lines in the file. If one exists, modify it rather than adding a new one.

---

## Step 7: Verify OQS Provider Activation

### List Active Providers

```bash
openssl list -providers
```

**Expected output:**

```
Providers:
  default
    name: OpenSSL Default Provider
    version: 3.5.3
    status: active
  oqsprovider
    name: OpenSSL OQS Provider
    version: 0.x.x
    status: active
```

Both `default` and `oqsprovider` should show `status: active`.

---

## Step 8: Verify Algorithm Availability

### Native NIST Algorithms (default provider)

```bash
openssl list -kem-algorithms | grep -i mlkem
```

**Expected output:**

```
MLKEM512 @ default
MLKEM768 @ default
MLKEM1024 @ default
X25519MLKEM768 @ default
```

### OQS Alternative Algorithms

```bash
openssl list -kem-algorithms | grep -i frodo
```

**Expected output:**

```
frodo640aes @ oqsprovider
frodo640shake @ oqsprovider
frodo976aes @ oqsprovider
frodo976shake @ oqsprovider
frodo1344aes @ oqsprovider
frodo1344shake @ oqsprovider
```

```bash
openssl list -kem-algorithms | grep -i ntru
```

**Expected output:**

```
ntru_hps2048509 @ oqsprovider
ntru_hps2048677 @ oqsprovider
ntru_hps4096821 @ oqsprovider
ntru_hrss701 @ oqsprovider
```

```bash
openssl list -kem-algorithms | grep -i mceliece
```

**Expected output:**

```
mceliece348864 @ oqsprovider
mceliece460896 @ oqsprovider
mceliece6688128 @ oqsprovider
mceliece6960119 @ oqsprovider
mceliece8192128 @ oqsprovider
```

```bash
openssl list -kem-algorithms | grep -i bike
```

**Expected output:**

```
bike1l1fo @ oqsprovider
bike1l3fo @ oqsprovider
bike1l5fo @ oqsprovider
```

```bash
openssl list -kem-algorithms | grep -i hqc
```

**Expected output:**

```
hqc128 @ oqsprovider
hqc192 @ oqsprovider
hqc256 @ oqsprovider
```

---

## Cleanup (Optional)

After successful installation, remove the build directories to save space:

```bash
rm -rf ~/oqs-build
```

---

## Troubleshooting

### Provider Not Loading

**Symptom:** `openssl list -providers` doesn't show oqsprovider

**Solutions:**

1. Check provider file exists:
   ```bash
   ls -la /usr/lib/x86_64-linux-gnu/ossl-modules/oqsprovider.so
   ```

2. Verify configuration syntax in `/etc/ssl/openssl.cnf`

3. Check for duplicate `openssl_conf` lines

### Library Loading Errors

**Symptom:** `error while loading shared libraries: liboqs.so`

**Solution:**

```bash
sudo ldconfig
```

If still failing:

```bash
sudo ldconfig -p | grep oqs
```

### pkg-config Not Found During Build

**Symptom:** cmake fails with pkg-config errors

**Solution:**

```bash
sudo apt install -y pkg-config
```

Then re-run cmake.

### Algorithm Not Found

**Symptom:** grep returns no results for expected algorithm

**Solutions:**

1. Verify OQS provider is active:
   ```bash
   openssl list -providers
   ```

2. Check OQS provider version supports the algorithm

3. Rebuild with algorithm enabled if necessary

---

## Algorithm Quick Reference

### Using Native Algorithms (No Provider Flags Needed)

```bash
openssl genpkey -algorithm mldsa65 -out key.pem

openssl req -new -x509 -key key.pem -out cert.crt -days 365 \
    -subj "/CN=test.example.com"

openssl s_server -key key.pem -cert cert.crt -port 4433 \
    -tls1_3 -groups X25519MLKEM768 -www
```

### Using OQS Algorithms (Automatic via Provider)

With the OQS provider configured system-wide, OQS algorithms are available automatically:

```bash
openssl s_server -key key.pem -cert cert.crt -port 4433 \
    -tls1_3 -groups frodo640aes -www

openssl s_client -connect localhost:4433 \
    -tls1_3 -groups ntru_hps2048677
```

---

## Next Steps

Your Ubuntu 25.10 system now has full PQC support with both native NIST algorithms and OQS alternative algorithms. Continue to one of the learning paths:

### Learning Path 1: NIST FIPS 203/204/205

**For commercial organizations implementing quantum-resistant cryptography using NIST standards.**

Uses native OpenSSL 3.5.x PQC support for ML-KEM, ML-DSA, and SLH-DSA.

**Start here:** [FIPS Path - Module 00: Introduction](FIPS-Path/00-INTRODUCTION.md)

---

### Learning Path 2: NSA CNSA 2.0

**For government contractors and organizations requiring CNSA 2.0 compliance.**

Focuses on ML-DSA-65/87 and ML-KEM-768/1024 for classified system requirements.

**Start here:** [CNSA Path - Module 00: Introduction](CNSA-Path/00-INTRODUCTION.md)

---

### Learning Path 3: Alternative PQC Algorithms

**For researchers, organizations requiring algorithm diversity, and international compliance.**

Explores FrodoKEM, NTRU, Classic McEliece, BIKE, and HQC using the OQS provider you just installed.

**Start here:** [Alt Path - Module 00: Introduction](Alt-Path/00-INTRODUCTION.md)

---

---

## OpenSSL and OQS Compatibility Matrix

### OpenSSL Version Support

| OpenSSL Version | OQS Provider Support | Native NIST PQC | TLS 1.3 Signatures | Notes |
|-----------------|---------------------|-----------------|-------------------|-------|
| 3.0.0 - 3.0.13 | ✅ Yes | ❌ No | ❌ No | Groups limit bug present |
| 3.0.14+ | ✅ Yes | ❌ No | ❌ No | Groups limit fixed |
| 3.1.0 - 3.1.5 | ✅ Yes | ❌ No | ❌ No | Groups limit bug present |
| 3.1.6+ | ✅ Yes | ❌ No | ❌ No | Groups limit fixed |
| 3.2.0 - 3.2.1 | ✅ Yes | ❌ No | ✅ Yes | TLS sig support added; groups limit bug |
| 3.2.2+ | ✅ Yes | ❌ No | ✅ Yes | Groups limit fixed |
| 3.3.0+ | ✅ Yes | ❌ No | ✅ Yes | Full support |
| 3.4.x | ✅ Yes | ❌ No | ✅ Yes | Full support |
| **3.5.x** | ✅ Yes | ✅ Yes | ✅ Yes | Native ML-KEM/ML-DSA; OQS auto-disables duplicates |

### OQS Provider and liboqs Version Alignment

| OQS Provider | liboqs | Release Date | Key Features |
|--------------|--------|--------------|--------------|
| 0.10.0 | 0.14.0 | July 2025 | Current stable; composite signatures removed |
| 0.9.0 | 0.13.0 | May 2025 | OpenSSL 3.5 native algorithm detection |
| 0.8.0 | 0.12.0 | December 2024 | FIPS algorithm names (ML-KEM, ML-DSA) |
| 0.7.0 | 0.11.0 | October 2024 | Algorithm updates |
| 0.6.1 | 0.10.1 | June 2024 | Bug fixes |
| 0.6.0 | 0.10.0 | April 2024 | First standardized PQ algorithms |

### Important Notes

**OpenSSL 3.5+ with OQS Provider:**
- OQS provider version 0.9.0+ automatically disables ML-KEM and ML-DSA when it detects OpenSSL 3.5+ native support
- This prevents algorithm conflicts and ensures you use native implementations for NIST algorithms
- OQS provider continues to provide FrodoKEM, NTRU, Classic McEliece, BIKE, and other non-NIST algorithms

**Groups Limit Bug:**
- OpenSSL versions before 3.0.14, 3.1.6, 3.2.2, and 3.3.0 have a limit of 44 default TLS groups
- Enabling too many KEMs via OQS provider can cause crashes on affected versions
- This is resolved in newer OpenSSL releases

**Minimum Requirements:**
- OQS provider requires OpenSSL 3.0.0 or later
- TLS 1.3 signature functionality requires OpenSSL 3.2.0 or later
- For best experience, use OpenSSL 3.5.x with OQS provider 0.9.0+

---

## Additional Resources

- [Open Quantum Safe Project](https://openquantumsafe.org/)
- [OQS Provider GitHub](https://github.com/open-quantum-safe/oqs-provider)
- [liboqs GitHub](https://github.com/open-quantum-safe/liboqs)
- [OpenSSL 3.5 Documentation](https://www.openssl.org/docs/)
- [NIST Post-Quantum Cryptography](https://csrc.nist.gov/projects/post-quantum-cryptography)

---

**Return to:** [Main README](README.md)
