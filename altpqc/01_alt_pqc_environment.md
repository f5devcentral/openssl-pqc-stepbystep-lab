# Module 1: Environment Setup for Alternative PQC Algorithms

## Overview

This module guides you through configuring Ubuntu 25.10 with OpenSSL 3.5.x and the Open Quantum Safe (OQS) provider to access alternative post-quantum algorithms not available in native OpenSSL. While OpenSSL 3.5.x includes native support for NIST FIPS algorithms (ML-KEM, ML-DSA, SLH-DSA), algorithms like FrodoKEM, BIKE, and HQC require the OQS provider.

<br>

## Learning Objectives

After completing this module, you will be able to:

- Verify Ubuntu 25.10 and OpenSSL 3.5.x installation
- Build and install liboqs from source
- Build and install the OQS provider for OpenSSL
- Configure OpenSSL to load the OQS provider
- Verify access to alternative PQC algorithms
- Create the lab structure

<br>

## Prerequisites

- Fresh Ubuntu 25.10 installation (or upgraded system), you do you
- Root or sudo access
- Connectivity to github for downloading source code

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

Ubuntu 25.10 includes OpenSSL 3.5.x by default. To enable alternate algorithms, first compile the liboqs and oqsproviders found in the [Updating Openssl PQC Addendum](../addendum_updating_openssl_pqc.md)).  It's rather annoying but you'll need it.

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

If these didn't show up, did you complete the addedum? No?  The link is right above you. The addendum's oqsprovider will add FrodoKEM, BIKE, and HQC.  I'll wait.

<br>

## Step 3: Create CA Administrator Account

**Create a dedicated user account for CA administration:**

```bash
sudo useradd -r -m -d /opt/sassycorp-pqc-alt -s /bin/bash pqcaltadmin
```

**Flags explained:**

| Flag | Purpose |
|------|---------|
| `-r` | System account |
| `-m` | Create home directory |
| `-d /opt/sassycorp-pqc-alt` | Home directory location |
| `-s /bin/bash` | Login shell |

**Set a password for the account:**

```bash
sudo passwd pqcaltadmin
```

**Add your user to the pqcadmin group (optional, for easier administration):**

```bash
sudo usermod -aG pqcaaltdmin $USER
```

---

## Step 1: Create Lab Directory Structure

**Switch to the pqcadmin user:**

```bash
sudo su - pqcaltadmin
```

Create the working directory for alternative algorithm testing:

```bash
mkdir -p /opt/sassycorp-pqc-alt/{certs,keys,tests}
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
| OQS provider active | `openssl list -providers` | oqsprovider: active |
| FrodoKEM available | `openssl list -kem-algorithms \| grep frodo` | frodo640aes, etc. |
| BIKE available | `openssl list -kem-algorithms \| grep bike` | bike1l1fo, etc. |
| HQC available | `openssl list -kem-algorithms \| grep hqc` | hqc128, etc. |

---

## Troubleshooting

### Provider Not Loading

**Symptom:** `openssl list -providers` doesn't show oqsprovider

**Solutions:**

1. Go back to the [OpenSSL oqsprovider addendum](../addendum_updating_openssl_pqc.md) and validate the troubleshooting steps (don't run this under your pqcaltadmin)

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

## Next Steps

Your environment is now configured with access to alternative PQC algorithms. Proceed to **[Module 02: FrodoKEM](02_alt_pqc_frodokem.md)** to begin working with the conservative unstructured lattice KEM.

<br>

**Module Navigation:**

| Previous | Current | Next |
| ---------- | --------- | ------ |
| [00 - Introduction](00_alt_pqc_introduction.md) | **01 - Environment Setup** | [02 - FrodoKEM](02_alt_pqc_frodokem.md) |
