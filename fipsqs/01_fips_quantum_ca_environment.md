# Module 1: Environment Setup

This module guides you through verifying your system's OpenSSL installation has native post-quantum cryptography support and preparing the directory structure for Sassy Corp's quantum-resistant PKI.

For FIPS PQC we're going to rely on OpenSSL 3.5 which has NIST standardized algorithms already installed making our lives much easier. OpenSSL 3.5 is the first version to include built-in support for FIPS 203, 204, and 205 algorithms—no external providers required. For this we lab we used Ubuntu 25.10 but any linux variant that has OpenSSL 3.5.x will suffice. Basic instructions shouldn't deviate.

---

## Learning Objectives

After completing this module, you will be able to:

- Verify your system has OpenSSL 3.5.x with PQC support
- Confirm PQC algorithm availability
- Create the CA directory structure with proper permissions
- Configure the dedicated CA administrator account

---

## Step 1: Verify OpenSSL Version

Check your system's OpenSSL version:

```bash
openssl version
```

**Expected output:**

```
OpenSSL 3.5.x <date>
```

If your version is below 3.5.0, you will need to upgrade your operating system or [install a newer OpenSSL package](/addendum_updating_openssl_pqc.md) before continuing.

---

## Step 2: Verify PQC Algorithm Availability

### Check Available Signature Algorithms

**List available ML-DSA signature algorithms:**

```bash
openssl list -signature-algorithms | grep -i ml-dsa
```

**Expected output:**

```
  ML-DSA-44 @ default
  ML-DSA-65 @ default
  ML-DSA-87 @ default
```

**List available SLH-DSA signature algorithms:**

```bash
openssl list -signature-algorithms | grep -i slh-dsa
```

**Expected output (partial):**

```
  SLH-DSA-SHA2-128f @ default
  SLH-DSA-SHA2-128s @ default
  SLH-DSA-SHA2-192f @ default
  SLH-DSA-SHA2-192s @ default
  SLH-DSA-SHA2-256f @ default
  SLH-DSA-SHA2-256s @ default
  SLH-DSA-SHAKE-128f @ default
  ...
```

### Check Available Key Encapsulation Mechanisms

**List available ML-KEM algorithms:**

```bash
openssl list -kem-algorithms | grep -i mlkem
```

**Expected output (partial):**

```
  MLKEM512 @ default
  MLKEM768 @ default
  MLKEM1024 @ default
  ...
```

**List hybrid KEM algorithms:**

```bash
openssl list -kem-algorithms | grep -i x25519
```

**Expected output:**

```
  X25519MLKEM768 @ default
```

> **Note:** If any of these algorithms are missing, your OpenSSL installation may not have PQC support enabled. Verify you are running OpenSSL 3.5.0 or later.

---

## Step 3: Create CA Administrator Account

**Create a dedicated user account for CA administration:**

```bash
sudo useradd -r -m -d /opt/sassycorp-pqc -s /bin/bash pqcadmin
```

**Flags explained:**

| Flag | Purpose |
|------|---------|
| `-r` | System account |
| `-m` | Create home directory |
| `-d /opt/sassycorp-pqc` | Home directory location |
| `-s /bin/bash` | Login shell |

**Set a password for the account:**

```bash
sudo passwd pqcadmin
```

**Add your user to the pqcadmin group (optional, for easier administration):**

```bash
sudo usermod -aG pqcadmin $USER
```

---

## Step 4: Create Directory Structure

**Switch to the pqcadmin user:**

```bash
sudo su - pqcadmin
```

**Create the Root CA directory structure:**

```bash
mkdir -p /opt/sassycorp-pqc/root-ca/{private,certs,crl,newcerts}
```

**Create the Intermediate CA directory structure:**

```bash
mkdir -p /opt/sassycorp-pqc/intermediate-ca/{private,certs,crl,newcerts,csr}
```

**Create the OCSP responder directory structure:**

```bash
mkdir -p /opt/sassycorp-pqc/ocsp/{private,certs,logs}
```

---

## Step 5: Set Directory Permissions

**Set restrictive permissions on private key directories:**

```bash
chmod 700 /opt/sassycorp-pqc/root-ca/private
```

```bash
chmod 700 /opt/sassycorp-pqc/intermediate-ca/private
```

```bash
chmod 700 /opt/sassycorp-pqc/ocsp/private
```

**Set appropriate permissions on other directories:**

```bash
chmod 755 /opt/sassycorp-pqc/root-ca/{certs,crl,newcerts}
```

```bash
chmod 755 /opt/sassycorp-pqc/intermediate-ca/{certs,crl,newcerts,csr}
```

```bash
chmod 755 /opt/sassycorp-pqc/ocsp/{certs,logs}
```

---

## Step 6: Initialize CA Database Files

### Root CA Database

**Create the certificate database** (these files allow the CA to track issuing and revocation information and other such stuff):

```bash
touch /opt/sassycorp-pqc/root-ca/index.txt
```

**Create the database attributes file (and forces unique subjects for certs):**

```bash
echo 'unique_subject = yes' > /opt/sassycorp-pqc/root-ca/index.txt.attr
```

**Create the serial number file (starting at 1000):**

```bash
echo 1000 > /opt/sassycorp-pqc/root-ca/serial
```

**Create the CRL number file:**

```bash
echo 1000 > /opt/sassycorp-pqc/root-ca/crlnumber
```

### Intermediate CA Database

```bash
touch /opt/sassycorp-pqc/intermediate-ca/index.txt
```

```bash
echo 'unique_subject = yes' > /opt/sassycorp-pqc/intermediate-ca/index.txt.attr
```

```bash
echo 2000 > /opt/sassycorp-pqc/intermediate-ca/serial
```

```bash
echo 2000 > /opt/sassycorp-pqc/intermediate-ca/crlnumber
```

**Set permissions on database files:**

```bash
chmod 644 /opt/sassycorp-pqc/root-ca/{index.txt,index.txt.attr,serial,crlnumber}
```

```bash
chmod 644 /opt/sassycorp-pqc/intermediate-ca/{index.txt,index.txt.attr,serial,crlnumber}
```

---

## Step 7: Verify Directory Structure

**Display the complete directory structure:**

```bash
find /opt/sassycorp-pqc -type d | sort
```

**Expected output:**

```
/opt/sassycorp-pqc
/opt/sassycorp-pqc/intermediate-ca
/opt/sassycorp-pqc/intermediate-ca/certs
/opt/sassycorp-pqc/intermediate-ca/crl
/opt/sassycorp-pqc/intermediate-ca/csr
/opt/sassycorp-pqc/intermediate-ca/newcerts
/opt/sassycorp-pqc/intermediate-ca/private
/opt/sassycorp-pqc/ocsp
/opt/sassycorp-pqc/ocsp/certs
/opt/sassycorp-pqc/ocsp/logs
/opt/sassycorp-pqc/ocsp/private
/opt/sassycorp-pqc/root-ca
/opt/sassycorp-pqc/root-ca/certs
/opt/sassycorp-pqc/root-ca/crl
/opt/sassycorp-pqc/root-ca/newcerts
/opt/sassycorp-pqc/root-ca/private
```

**Verify permissions on private directories:**

```bash
ls -la /opt/sassycorp-pqc/root-ca/private
```

**Expected output:**

```
drwx------ 2 pqcadmin pqcadmin 4096 <date> .
```

---

## Step 8: Test PQC Key Generation

**Test ML-DSA key generation to ensure everything works:**

```bash
openssl genpkey -algorithm ML-DSA-65 -out /tmp/test-ml-dsa-65.key
```

**Verify the key was created:**

```bash
openssl pkey -in /tmp/test-ml-dsa-65.key -noout -text | head -5
```

**Expected output:**

```
ML-DSA-65 Private-Key:
priv:
    <hex values>
```

**Clean up the test key:**

```bash
rm /tmp/test-ml-dsa-65.key
```

**Test ML-KEM key generation:**

```bash
openssl genpkey -algorithm MLKEM768 -out /tmp/test-ml-kem-768.key
```

**Verify the key:**

```bash
openssl pkey -in /tmp/test-ml-kem-768.key -noout -text | head -5
```

**Clean up:**

```bash
rm /tmp/test-ml-kem-768.key
```

---

## Module Summary

You have completed the environment setup. Your system now has:

| Component | Status |
|-----------|--------|
| OpenSSL 3.5.x | Verified native PQC support |
| ML-DSA support | Verified (ML-DSA-44, ML-DSA-65, ML-DSA-87) |
| SLH-DSA support | Verified (multiple variants) |
| ML-KEM support | Verified (MLKEM512, MLKEM768, MLKEM1024) |
| Hybrid KEM support | Verified (X25519MLKEM768) |
| CA admin account | `pqcadmin` created |
| Directory structure | `/opt/sassycorp-pqc` ready |
| Database files | Initialized |

---

## Troubleshooting

### PQC Algorithms Not Listed

**Problem:** ML-DSA or ML-KEM algorithms not shown

**Solution:** Verify you're using OpenSSL 3.5.x:
```bash
openssl version
```

If using an older version, upgrade your operating system or OpenSSL package.

### Permission Denied Errors

**Problem:** Cannot write to `/opt/sassycorp-pqc`

**Solution:** Ensure you're operating as the `pqcadmin` user:
```bash
whoami  # Should show: pqcadmin
```

### OpenSSL Version Too Old

**Problem:** System has OpenSSL 3.0.x or earlier

**Solution:** Upgrade! Either install OpenSSL 3.5.X (complicated) or run a current linux distro with with updated packages.

---

## Security Checklist

**Before proceeding, verify:**

- [ ] OpenSSL version is 3.5.0 or later
- [ ] PQC algorithms are available
- [ ] Private key directories have 700 permissions
- [ ] Database files have 644 permissions
- [ ] CA admin account has a strong password
- [ ] Test keys have been removed

---

**Next:** [Building a Root CA →](02_fips_quantum_ca_root.md)
