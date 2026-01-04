# Module 01: Environment Setup for Alternative PQC Algorithms

## Overview

This module guides you through configuring Ubuntu 25.10 with OpenSSL 3.5.x and the Open Quantum Safe (OQS) provider to access alternative post-quantum algorithms not available in native OpenSSL.

While OpenSSL 3.5.x includes native support for NIST FIPS algorithms (ML-KEM, ML-DSA, SLH-DSA), algorithms like FrodoKEM, NTRU, Classic McEliece, BIKE, and HQC require the OQS provider.

---

## Learning Objectives

After completing this module, you will be able to:

- Verify Ubuntu 25.10 and OpenSSL 3.5.x installation
- Build and install liboqs from source
- Build and install the OQS provider for OpenSSL
- Configure OpenSSL to load the OQS provider
- Verify access to alternative PQC algorithms
- Create the lab directory structure

---

## Prerequisites

- Fresh Ubuntu 25.10 installation (or upgraded system)
- Root or sudo access
- Internet connection for downloading source code
- Approximately 2GB free disk space for build process

---

## Step 1: Verify System Requirements

### Check Ubuntu Version

```bash
lsb_release -d
```

**Expected output:**

```
Description:    Ubuntu 25.10
```

### Check OpenSSL Version

```bash
openssl version
```

**Expected output:**

```
OpenSSL 3.5.x <date>
```

Ubuntu 25.10 includes OpenSSL 3.5.x by default. If your version is older, you may need to upgrade your system or compile OpenSSL from source (see the [Environment Setup Addendum](../ADDENDUM-PQC-ENVIRONMENT-SETUP.md)).

### Check Available Native PQC Algorithms

Verify native ML-KEM support:

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

These are the NIST-standardized algorithms. The OQS provider will add FrodoKEM, NTRU, Classic McEliece, BIKE, and HQC.

---

## Step 2: Install Build Dependencies

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
    python3 \
    python3-pip \
    astyle \
    pkg-config
```

**Package purposes:**

| Package | Purpose |
|---------|---------|
| build-essential | GCC, make, and standard build tools |
| cmake | Build system for liboqs and oqs-provider |
| ninja-build | Fast build system (used by liboqs) |
| git | Source code retrieval |
| libssl-dev | OpenSSL development headers |
| python3 | Required for some build scripts |
| astyle | Code formatting (optional but recommended) |
| pkg-config | Library detection |

---

## Step 3: Create Build Directory

Create a working directory for the build process:

```bash
mkdir -p ~/pqc-alt-build
```

```bash
cd ~/pqc-alt-build
```

---

## Step 4: Clone and Build liboqs

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

This compiles all supported algorithms. Build time varies by system (typically 2-5 minutes).

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
-rw-r--r-- 1 root root 12345678 <date> /usr/local/lib/liboqs.so
-rw-r--r-- 1 root root 12345678 <date> /usr/local/lib/liboqs.so.0
-rw-r--r-- 1 root root 12345678 <date> /usr/local/lib/liboqs.so.0.x.x
```

---

## Step 5: Clone and Build OQS Provider

Return to the build directory:

```bash
cd ~/pqc-alt-build
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

## Step 6: Install the OQS Provider

### Locate OpenSSL Providers Directory

Find where OpenSSL expects providers:

```bash
openssl version -d
```

Note the OPENSSLDIR path.

```bash
ls /usr/lib/x86_64-linux-gnu/ossl-modules/
```

This shows existing providers (typically `legacy.so` and `fips.so` if installed).

### Copy Provider to System Location

```bash
sudo cp ~/pqc-alt-build/oqs-provider/build/lib/oqsprovider.so \
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

## Step 7: Configure OpenSSL to Load OQS Provider

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

**Important:** Ensure there are no duplicate `openssl_conf` lines in the file. If one exists, modify it rather than adding a new one.

Save and exit (`:wq` in vim).

---

## Step 8: Verify OQS Provider Activation

### List Active Providers

```bash
openssl list -providers
```

**Expected output:**

```
Providers:
  default
    name: OpenSSL Default Provider
    version: 3.5.x
    status: active
  oqsprovider
    name: OpenSSL OQS Provider
    version: 0.x.x
    status: active
```

Both `default` and `oqsprovider` should show `status: active`.

---

## Step 9: Verify Alternative Algorithm Availability

### List FrodoKEM Algorithms

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

### List NTRU Algorithms

```bash
openssl list -kem-algorithms | grep -i ntru
```

**Expected output (may vary by OQS version):**

```
ntru_hps2048509 @ oqsprovider
ntru_hps2048677 @ oqsprovider
ntru_hps4096821 @ oqsprovider
ntru_hrss701 @ oqsprovider
```

### List Classic McEliece Algorithms

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

### List BIKE Algorithms

```bash
openssl list -kem-algorithms | grep -i bike
```

**Expected output:**

```
bike1l1fo @ oqsprovider
bike1l3fo @ oqsprovider
bike1l5fo @ oqsprovider
```

### List HQC Algorithms

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

## Step 10: Create Lab Directory Structure

Create the working directory for alternative algorithm testing:

```bash
sudo mkdir -p /opt/sassycorp-pqc-alt
```

```bash
sudo mkdir -p /opt/sassycorp-pqc-alt/certs
```

```bash
sudo mkdir -p /opt/sassycorp-pqc-alt/keys
```

```bash
sudo mkdir -p /opt/sassycorp-pqc-alt/tests
```

### Create Lab User (Optional)

If you haven't already created the pqcadmin user from previous learning paths:

```bash
sudo useradd -r -m -d /opt/sassycorp-pqc-alt -s /bin/bash pqcaltadmin
```

```bash
sudo chown -R pqcaltadmin:pqcaltadmin /opt/sassycorp-pqc-alt
```

Or reuse the existing pqcadmin user:

```bash
sudo chown -R pqcadmin:pqcadmin /opt/sassycorp-pqc-alt
```

---

## Step 11: Create Test Certificate (If Needed)

If you don't have certificates from the FIPS or CNSA paths, create a simple ML-DSA test certificate for TLS demonstrations:

### Generate Key

```bash
openssl genpkey -algorithm mldsa65 -out /opt/sassycorp-pqc-alt/keys/test.key
```

### Set Key Permissions

```bash
chmod 400 /opt/sassycorp-pqc-alt/keys/test.key
```

### Generate Self-Signed Certificate

```bash
openssl req -new -x509 \
    -key /opt/sassycorp-pqc-alt/keys/test.key \
    -out /opt/sassycorp-pqc-alt/certs/test.crt \
    -days 365 \
    -subj "/C=US/ST=California/L=San Francisco/O=Sassy Corp/CN=test.sassycorp.lab"
```

### Verify Certificate

```bash
openssl x509 -in /opt/sassycorp-pqc-alt/certs/test.crt -noout -text | head -20
```

---

## Verification Summary

Before proceeding to the next module, verify:

| Check | Command | Expected Result |
|-------|---------|-----------------|
| Ubuntu version | `lsb_release -d` | Ubuntu 25.10 |
| OpenSSL version | `openssl version` | 3.5.x |
| OQS provider active | `openssl list -providers` | oqsprovider: active |
| FrodoKEM available | `openssl list -kem-algorithms \| grep frodo` | frodo640aes, etc. |
| NTRU available | `openssl list -kem-algorithms \| grep ntru` | ntru_hps2048509, etc. |
| Classic McEliece available | `openssl list -kem-algorithms \| grep mceliece` | mceliece348864, etc. |
| BIKE available | `openssl list -kem-algorithms \| grep bike` | bike1l1fo, etc. |
| HQC available | `openssl list -kem-algorithms \| grep hqc` | hqc128, etc. |

---

## Troubleshooting

### Provider Not Loading

**Symptom:** `openssl list -providers` doesn't show oqsprovider

**Solutions:**

1. Check provider file exists:
   ```bash
   ls -la /usr/lib/x86_64-linux-gnu/ossl-modules/oqsprovider.so
   ```

2. Verify configuration syntax:
   ```bash
   openssl version -a
   ```
   Look for configuration errors.

3. Check for duplicate `openssl_conf` lines in `/etc/ssl/openssl.cnf`

### Library Loading Errors

**Symptom:** `error while loading shared libraries: liboqs.so`

**Solution:**

```bash
sudo ldconfig
```

If still failing, check library path:

```bash
sudo ldconfig -p | grep oqs
```

### Algorithm Not Found

**Symptom:** `grep` returns no results for expected algorithm

**Solutions:**

1. Check OQS provider version supports the algorithm
2. Rebuild liboqs with algorithm enabled:
   ```bash
   cmake -DOQS_ENABLE_KEM_FRODOKEM=ON ...
   ```

---

## Cleanup (Optional)

After successful installation, you can remove the build directories to save space:

```bash
rm -rf ~/pqc-alt-build
```

---

## Next Steps

Your environment is now configured with access to alternative PQC algorithms. Proceed to **[Module 02: FrodoKEM](02-FRODOKEM.md)** to begin working with the conservative unstructured lattice KEM.

---

**Module Navigation:**

| Previous | Current | Next |
|----------|---------|------|
| [00 - Introduction](00-INTRODUCTION.md) | **01 - Environment Setup** | [02 - FrodoKEM](02-FRODOKEM.md) |
