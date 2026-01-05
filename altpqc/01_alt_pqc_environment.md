# Module 01: Environment Setup for Alternative PQC Algorithms

## Overview

This module guides you through configuring Ubuntu 25.10 with OpenSSL 3.5.x and the Open Quantum Safe (OQS) provider to access alternative post-quantum algorithms not available in native OpenSSL.

While OpenSSL 3.5.x includes native support for NIST FIPS algorithms (ML-KEM, ML-DSA, SLH-DSA), algorithms like FrodoKEM, BIKE, and HQC require the OQS provider.

<br>

## Learning Objectives

After completing this module, you will be able to:

- Verify Ubuntu 25.10 and OpenSSL 3.5.x installation
- Build and install liboqs from source
- Build and install the OQS provider for OpenSSL
- Configure OpenSSL to load the OQS provider
- Verify access to alternative PQC algorithms
- Create the lab directory structure

<br>

## Prerequisites

- Fresh Ubuntu 25.10 installation (or upgraded system)
- Root or sudo access
- Internet connection for downloading source code
- Approximately 2GB free disk space for build process

<br>

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
<br>

## Step 2: Update and Build liboqs and oqsprivders for OpenSSL

Ubuntu 25.10 includes OpenSSL 3.5.x by default. To enable alternate algorithms, first compile the liboqs and oqsproviders found in the [Updating Openssl PQC Addendum](../addenum_updating_openssl_pqc.md)).  It's rather annoying but you'll need it.

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

These are the NIST-standardized algorithms. The addendum's oqsprovider will add FrodoKEM, BIKE, and HQC.

<br>

## Step 1: Create Lab Directory Structure

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
| ------- | --------- | ----------------- |
| Ubuntu version | `lsb_release -d` | Ubuntu 25.10 |
| OpenSSL version | `openssl version` | 3.5.x |
| OQS provider active | `openssl list -providers` | oqsprovider: active |
| FrodoKEM available | `openssl list -kem-algorithms \| grep frodo` | frodo640aes, etc. |
| BIKE available | `openssl list -kem-algorithms \| grep bike` | bike1l1fo, etc. |
| HQC available | `openssl list -kem-algorithms \| grep hqc` | hqc128, etc. |

---

## Troubleshooting

### Provider Not Loading

**Symptom:** `openssl list -providers` doesn't show oqsprovider

**Solutions:**

1. Go back to the [OpenSSL oqsprovider addendum](../addenum_updating_openssl_pqc.md) and validate the troubleshooting steps

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

<br>

## Cleanup (Optional)

After successful installation, you can remove the build directories to save space:

```bash
rm -rf ~/pqc-alt-build
```

<br>

## Next Steps

Your environment is now configured with access to alternative PQC algorithms. Proceed to **[Module 02: FrodoKEM](02_alt_pqc_frodokem.md)** to begin working with the conservative unstructured lattice KEM.

<br>

**Module Navigation:**

| Previous | Current | Next |
| ---------- | --------- | ------ |
| [00 - Introduction](00_alt_pqc_introduction.md) | **01 - Environment Setup** | [02 - FrodoKEM](02_alt_pqc_frodokem.md) |
