# Module 2: Building the Root CA

## Overview

This module guides you through creating a quantum-resistant Root Certificate Authority for Sassy Corp using ML-DSA-87 (FIPS 204). The Root CA is the trust anchor of your PKI—it signs the Intermediate CA certificate and establishes the chain of trust.

---

## Learning Objectives

After completing this module, you will be able to:

- Generate an ML-DSA-87 key pair for the Root CA
- Create an OpenSSL configuration file for CA operations
- Create a self-signed Root CA certificate
- Generate an initial Certificate Revocation List (CRL)
- Verify the Root CA certificate structure

---

## Root CA Design

| Attribute | Value |
|-----------|-------|
| Algorithm | ML-DSA-87 (NIST Level 5 / 256-bit security) |
| Validity | 10 years |
| Key Usage | Certificate Sign, CRL Sign |
| Path Length | 1 (can sign only one level of sub-CAs) |
| Hash Function | SHA-512 |

**Why ML-DSA-87?**

The Root CA requires the highest security level because:
- It is the ultimate trust anchor
- Compromise would affect the entire PKI (but it's a lab so c'est la vie)
- Long validity period (10 years) requires future-proof security
- ML-DSA-87 provides NIST Level 5 security (equivalent to 256-bit classical)

---

## Step 1: Ensure Correct User Context

Verify you are operating as the pqcadmin user:

```bash
whoami
```

If not, switch to the pqcadmin user:

```bash
sudo su - pqcadmin
```

Navigate to the Root CA directory:

```bash
cd /opt/sassycorp-pqc/root-ca
```

---

## Step 2: Create Root CA OpenSSL Configuration

Create the OpenSSL configuration file for the Root CA:

```bash
vim /opt/sassycorp-pqc/root-ca/openssl.cnf
```

Enter the following configuration:

```ini
# Sassy Corp Root CA - OpenSSL Configuration
# FIPS 204 (ML-DSA-87) Compliant

####################################################################
[ ca ]
default_ca = CA_default

[ CA_default ]
# Directory and file locations
dir               = /opt/sassycorp-pqc/root-ca
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand

# Root CA certificate and private key
private_key       = $dir/private/root-ca.key
certificate       = $dir/certs/root-ca.crt

# Certificate Revocation Lists
crlnumber         = $dir/crlnumber
crl               = $dir/crl/root-ca.crl
crl_extensions    = crl_ext
default_crl_days  = 30

# Certificate defaults
default_md        = sha512
name_opt          = ca_default
cert_opt          = ca_default
default_days      = 1825
preserve          = no
policy            = policy_strict

# Extensions
x509_extensions   = v3_intermediate_ca
copy_extensions   = copy

####################################################################
[ policy_strict ]
# Root CA only signs Intermediate CAs
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_loose ]
# Intermediate CA can sign certificates with different attributes
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

####################################################################
[ req ]
default_bits        = 4096
distinguished_name  = req_distinguished_name
string_mask         = utf8only
default_md          = sha512
x509_extensions     = v3_ca
prompt              = no

[ req_distinguished_name ]
countryName                     = US
stateOrProvinceName             = Washington
localityName                    = Winthrop
organizationName                = Sassy Corp
organizationalUnitName          = PKI Operations
commonName                      = Sassy Corp Root CA
emailAddress                    = pki@sassycorp.internal

####################################################################
[ v3_ca ]
# Root CA certificate extensions
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:1
keyUsage = critical, digitalSignature, cRLSign, keyCertSign
subjectAltName = @alt_names_ca

[ v3_intermediate_ca ]
# Intermediate CA certificate extensions
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign
authorityInfoAccess = caIssuers;URI:http://pki.sassycorp.lab/root-ca.crt
crlDistributionPoints = URI:http://pki.sassycorp.lab/root-ca.crl
subjectAltName = @alt_names_intermediate

[ crl_ext ]
# CRL extensions
authorityKeyIdentifier = keyid:always

####################################################################
[ alt_names_ca ]
DNS.1 = sassycorp.lab
DNS.2 = pki.sassycorp.lab
email.1 = pki@sassycorp.internal

[ alt_names_intermediate ]
DNS.1 = intermediate.sassycorp.lab
email.1 = pki@sassycorp.internal
```

Save and exit (Ctrl+X, Y, Enter in vim).

Set appropriate permissions on the configuration file:

```bash
chmod 644 /opt/sassycorp-pqc/root-ca/openssl.cnf
```

---

## Step 3: Generate Root CA Private Key

Generate the ML-DSA-87 private key:

```bash
openssl genpkey -algorithm ML-DSA-87 -out /opt/sassycorp-pqc/root-ca/private/root-ca.key
```

Set restrictive permissions on the private key:

```bash
chmod 400 /opt/sassycorp-pqc/root-ca/private/root-ca.key
```

Verify the key was created:

```bash
ls -la /opt/sassycorp-pqc/root-ca/private/
```

**Expected output:**

```
-r-------- 1 pqcadmin pqcadmin <size> <date> root-ca.key
```

---

## Step 4: Examine the Private Key

View the private key details (without exposing the actual key material):

```bash
openssl pkey -in /opt/sassycorp-pqc/root-ca/private/root-ca.key -noout -text | head -20
```

**Expected output:**

```
ML-DSA-87 Private-Key:
priv:
    <hexadecimal values representing the private key>
```

Extract and view the public key component:

```bash
openssl pkey -in /opt/sassycorp-pqc/root-ca/private/root-ca.key -pubout -out /opt/sassycorp-pqc/root-ca/certs/root-ca.pub
```

```bash
openssl pkey -pubin -in /opt/sassycorp-pqc/root-ca/certs/root-ca.pub -noout -text | head -10
```

---

## Step 5: Create Self-Signed Root CA Certificate

Generate the self-signed Root CA certificate valid for 10 years (3650 days):

```bash
openssl req -config /opt/sassycorp-pqc/root-ca/openssl.cnf \
    -key /opt/sassycorp-pqc/root-ca/private/root-ca.key \
    -new -x509 -days 3650 \
    -extensions v3_ca \
    -out /opt/sassycorp-pqc/root-ca/certs/root-ca.crt
```

**Command breakdown:**

| Option | Purpose |
|--------|---------|
| `-config` | Use the Root CA configuration file |
| `-key` | Private key to sign the certificate |
| `-new` | Create a new certificate request |
| `-x509` | Output a self-signed certificate |
| `-days 3650` | Certificate validity (10 years) |
| `-extensions v3_ca` | Use v3_ca extension section |
| `-out` | Output file for the certificate |

Set permissions on the certificate:

```bash
chmod 444 /opt/sassycorp-pqc/root-ca/certs/root-ca.crt
```

---

## Step 6: Verify the Root CA Certificate

View the complete certificate:

```bash
openssl x509 -in /opt/sassycorp-pqc/root-ca/certs/root-ca.crt -noout -text
```

### Verify Key Fields

Check the issuer and subject (should be identical for self-signed):

```bash
openssl x509 -in /opt/sassycorp-pqc/root-ca/certs/root-ca.crt -noout -issuer -subject
```

**Expected output:**

```
issuer=C=US, ST=Washington, L=Winthrop, O=Sassy Corp, OU=PKI Operations, CN=Sassy Corp Root CA, emailAddress=pki@sassycorp.internal
subject=C=US, ST=Washington, L=Winthrop, O=Sassy Corp, OU=PKI Operations, CN=Sassy Corp Root CA, emailAddress=pki@sassycorp.internal
```

Check the validity dates:

```bash
openssl x509 -in /opt/sassycorp-pqc/root-ca/certs/root-ca.crt -noout -dates
```

**Expected output (dates will vary):**

```
notBefore=<current date>
notAfter=<date 10 years from now>
```

Check the signature algorithm:

```bash
openssl x509 -in /opt/sassycorp-pqc/root-ca/certs/root-ca.crt -noout -text | grep "Signature Algorithm"
```

**Expected output:**

```
    Signature Algorithm: ML-DSA-87
    Signature Algorithm: ML-DSA-87
```

Verify the Basic Constraints extension:

```bash
openssl x509 -in /opt/sassycorp-pqc/root-ca/certs/root-ca.crt -noout -text | grep -A1 "Basic Constraints"
```

**Expected output:**

```
            X509v3 Basic Constraints: critical
                CA:TRUE, pathlen:1
```

Verify the Key Usage extension:

```bash
openssl x509 -in /opt/sassycorp-pqc/root-ca/certs/root-ca.crt -noout -text | grep -A1 "Key Usage"
```

**Expected output:**

```
            X509v3 Key Usage: critical
                Digital Signature, Certificate Sign, CRL Sign
```

---

## Step 7: Verify Certificate Self-Signature

Verify the certificate signature is valid:

```bash
openssl verify -CAfile /opt/sassycorp-pqc/root-ca/certs/root-ca.crt /opt/sassycorp-pqc/root-ca/certs/root-ca.crt
```

**Expected output:**

```
/opt/sassycorp-pqc/root-ca/certs/root-ca.crt: OK
```

---

## Step 8: Generate Initial CRL

Create the initial Certificate Revocation List:

```bash
openssl ca -config /opt/sassycorp-pqc/root-ca/openssl.cnf \
    -gencrl \
    -out /opt/sassycorp-pqc/root-ca/crl/root-ca.crl
```

Set permissions on the CRL:

```bash
chmod 444 /opt/sassycorp-pqc/root-ca/crl/root-ca.crl
```

Verify the CRL:

```bash
openssl crl -in /opt/sassycorp-pqc/root-ca/crl/root-ca.crl -noout -text
```

**Expected output:**

```
Certificate Revocation List (CRL):
        Version 2 (0x1)
        Signature Algorithm: ML-DSA-87
        Issuer: C=US, ST=Washington, L=Winthrop, O=Sassy Corp, OU=PKI Operations, CN=Sassy Corp Root CA, emailAddress=pki@sassycorp.internal
        Last Update: <date>
        Next Update: <date + 30 days>
        CRL extensions:
            X509v3 Authority Key Identifier:
                <key identifier>
No Revoked Certificates.
```

---

## Step 9: Create Certificate Fingerprints

Generate fingerprints for certificate verification:

SHA-256 fingerprint:

```bash
openssl x509 -in /opt/sassycorp-pqc/root-ca/certs/root-ca.crt -noout -fingerprint -sha256
```

SHA-512 fingerprint:

```bash
openssl x509 -in /opt/sassycorp-pqc/root-ca/certs/root-ca.crt -noout -fingerprint -sha512
```

Record these fingerprints securely—they can be used to verify certificate authenticity during distribution.

---

## Step 10: Create DER Format Certificate

Create a DER-encoded version for systems that require it:

```bash
openssl x509 -in /opt/sassycorp-pqc/root-ca/certs/root-ca.crt -outform DER -out /opt/sassycorp-pqc/root-ca/certs/root-ca.der
```

```bash
chmod 444 /opt/sassycorp-pqc/root-ca/certs/root-ca.der
```

---

## Step 11: Backup the Root CA

Create a secure backup of the Root CA (in production, this would go to offline storage):

```bash
mkdir -p /opt/sassycorp-pqc/backups
```

Create an encrypted backup archive:

```bash
tar -czf - /opt/sassycorp-pqc/root-ca/private/root-ca.key | openssl enc -aes-256-cbc -pbkdf2 -out /opt/sassycorp-pqc/backups/root-ca-key-backup.tar.gz.enc
```

You will be prompted for a password. Use a strong password and store it securely.

> **Warning:** In a production environment, the Root CA private key should be stored on an air-gapped system or Hardware Security Module (HSM). Never leave the private key on an internet-connected system.

---

## Step 12: Verify Final Directory State

List all files in the Root CA directory:

```bash
find /opt/sassycorp-pqc/root-ca -type f -exec ls -la {} \;
```

**Expected files and permissions:**

```
-rw-r--r-- 1 pqcadmin pqcadmin <size> openssl.cnf
-r-------- 1 pqcadmin pqcadmin <size> private/root-ca.key
-r--r--r-- 1 pqcadmin pqcadmin <size> certs/root-ca.crt
-r--r--r-- 1 pqcadmin pqcadmin <size> certs/root-ca.der
-r--r--r-- 1 pqcadmin pqcadmin <size> certs/root-ca.pub
-r--r--r-- 1 pqcadmin pqcadmin <size> crl/root-ca.crl
-rw-r--r-- 1 pqcadmin pqcadmin <size> index.txt
-rw-r--r-- 1 pqcadmin pqcadmin <size> index.txt.attr
-rw-r--r-- 1 pqcadmin pqcadmin <size> serial
-rw-r--r-- 1 pqcadmin pqcadmin <size> crlnumber
```

---

## Module Summary

You have successfully created a quantum-resistant Root CA for Sassy Corp:

| Component | Details |
|-----------|---------|
| Algorithm | ML-DSA-87 (FIPS 204, Level 5) |
| Certificate | `/opt/sassycorp-pqc/root-ca/certs/root-ca.crt` |
| Private Key | `/opt/sassycorp-pqc/root-ca/private/root-ca.key` |
| Public Key | `/opt/sassycorp-pqc/root-ca/certs/root-ca.pub` |
| CRL | `/opt/sassycorp-pqc/root-ca/crl/root-ca.crl` |
| Validity | 10 years |
| Path Length | 1 (one sub-CA level) |

---

## Troubleshooting

### "Algorithm not found" Error

**Problem:** OpenSSL doesn't recognize ML-DSA-87

**Solution:** Verify you're using the correct OpenSSL binary:
```bash
openssl version
```

### Permission Denied on Private Key

**Problem:** Cannot read private key

**Solution:** Ensure correct ownership and permissions:
```bash
sudo chown pqcadmin:pqcadmin /opt/sassycorp-pqc/root-ca/private/root-ca.key
chmod 400 /opt/sassycorp-pqc/root-ca/private/root-ca.key
```

### Certificate Verification Fails

**Problem:** Self-signature verification fails

**Solution:** Ensure the certificate was created with the correct key:
```bash
openssl x509 -in /opt/sassycorp-pqc/root-ca/certs/root-ca.crt -noout -pubkey > /tmp/cert-pub.pem
openssl pkey -in /opt/sassycorp-pqc/root-ca/private/root-ca.key -pubout > /tmp/key-pub.pem
diff /tmp/cert-pub.pem /tmp/key-pub.pem
```

If there's a difference, regenerate the certificate.

---

## Security Checklist

Before proceeding, verify:

- [ ] Root CA private key has 400 permissions
- [ ] Private key directory has 700 permissions
- [ ] Certificate signature algorithm is ML-DSA-87
- [ ] Basic Constraints shows CA:TRUE with pathlen:1
- [ ] Key Usage includes keyCertSign and cRLSign
- [ ] Self-signature verification succeeds
- [ ] Backup of private key is created and secured

---

**Next:** [Intermediate CA →](03_fips_quantum_ca_intermediate.md)
