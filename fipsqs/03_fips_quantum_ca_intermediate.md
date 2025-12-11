# Module 3: Building the Intermediate CA

## Overview

This module guides you through creating a quantum-resistant Intermediate Certificate Authority for Sassy Corp using ML-DSA-65 (FIPS 204). The Intermediate CA handles day-to-day certificate issuance while the Root CA remains offline.

---

## Learning Objectives

After completing this module, you will be able to:

- Generate an ML-DSA-65 key pair for the Intermediate CA
- Create a Certificate Signing Request (CSR)
- Sign the Intermediate CA certificate using the Root CA
- Configure the Intermediate CA for certificate issuance
- Create the certificate chain bundle

---

## Intermediate CA Design

| Attribute | Value |
|-----------|-------|
| Algorithm | ML-DSA-65 (NIST Level 3 / 192-bit security) |
| Validity | 5 years |
| Key Usage | Certificate Sign, CRL Sign, Digital Signature |
| Path Length | 0 (cannot sign additional CAs) |
| Hash Function | SHA-512 |

**Why ML-DSA-65?**

The Intermediate CA uses ML-DSA-65 because:
- Provides NIST Level 3 security (192-bit equivalent)
- Smaller signatures and keys than ML-DSA-87 (better performance)
- Appropriate for operational CA with 5-year validity
- Balance between security and operational efficiency

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

Navigate to the Intermediate CA directory:

```bash
cd /opt/sassycorp-pqc/intermediate-ca
```

---

## Step 2: Create Intermediate CA OpenSSL Configuration

Create the OpenSSL configuration file:

```bash
vim /opt/sassycorp-pqc/intermediate-ca/openssl.cnf
```

Enter the following configuration:

```ini
# Sassy Corp Intermediate CA - OpenSSL Configuration
# FIPS 204 (ML-DSA-65) Compliant

####################################################################
[ ca ]
default_ca = CA_default

[ CA_default ]
# Directory and file locations
dir               = /opt/sassycorp-pqc/intermediate-ca
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand

# Intermediate CA certificate and private key
private_key       = $dir/private/intermediate-ca.key
certificate       = $dir/certs/intermediate-ca.crt

# Certificate Revocation Lists
crlnumber         = $dir/crlnumber
crl               = $dir/crl/intermediate-ca.crl
crl_extensions    = crl_ext
default_crl_days  = 7

# Certificate defaults
default_md        = sha512
name_opt          = ca_default
cert_opt          = ca_default
default_days      = 730
preserve          = no
policy            = policy_loose

# Extensions for different certificate types
x509_extensions   = usr_cert
copy_extensions   = copy

####################################################################
[ policy_loose ]
# Intermediate CA can sign certificates with varied attributes
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
x509_extensions     = v3_intermediate_ca
prompt              = no

[ req_distinguished_name ]
countryName                     = US
stateOrProvinceName             = Washington
localityName                    = Winthrop
organizationName                = Sassy Corp
organizationalUnitName          = PKI Operations
commonName                      = Sassy Corp Intermediate CA
emailAddress                    = pki@sassycorp.internal

####################################################################
[ v3_intermediate_ca ]
# Intermediate CA certificate extensions (used when creating CSR)
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:true, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign

####################################################################
[ usr_cert ]
# User/Client certificate extensions
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "Sassy Corp User Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection
authorityInfoAccess = OCSP;URI:http://ocsp.sassycorp.lab,caIssuers;URI:http://pki.sassycorp.lab/intermediate-ca.crt
crlDistributionPoints = URI:http://pki.sassycorp.lab/intermediate-ca.crl

[ server_cert ]
# Server certificate extensions
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "Sassy Corp Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
authorityInfoAccess = OCSP;URI:http://ocsp.sassycorp.lab,caIssuers;URI:http://pki.sassycorp.lab/intermediate-ca.crt
crlDistributionPoints = URI:http://pki.sassycorp.lab/intermediate-ca.crl

[ ocsp_cert ]
# OCSP Responder certificate extensions
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, digitalSignature
extendedKeyUsage = critical, OCSPSigning
noCheck = ignored

[ crl_ext ]
# CRL extensions
authorityKeyIdentifier = keyid:always
```

Save and exit (Ctrl+X, Y, Enter).

Set permissions:

```bash
chmod 644 /opt/sassycorp-pqc/intermediate-ca/openssl.cnf
```

---

## Step 3: Generate Intermediate CA Private Key

Generate the ML-DSA-65 private key:

```bash
openssl genpkey -algorithm ML-DSA-65 -out /opt/sassycorp-pqc/intermediate-ca/private/intermediate-ca.key
```

Set restrictive permissions:

```bash
chmod 400 /opt/sassycorp-pqc/intermediate-ca/private/intermediate-ca.key
```

Verify the key:

```bash
openssl pkey -in /opt/sassycorp-pqc/intermediate-ca/private/intermediate-ca.key -noout -text | head -5
```

**Expected output:**

```
ML-DSA-65 Private-Key:
priv:
    <hexadecimal values>
```

---

## Step 4: Create Certificate Signing Request

Create a CSR for the Intermediate CA:

```bash
openssl req -config /opt/sassycorp-pqc/intermediate-ca/openssl.cnf \
    -new \
    -key /opt/sassycorp-pqc/intermediate-ca/private/intermediate-ca.key \
    -out /opt/sassycorp-pqc/intermediate-ca/csr/intermediate-ca.csr
```

Verify the CSR:

```bash
openssl req -in /opt/sassycorp-pqc/intermediate-ca/csr/intermediate-ca.csr -noout -text
```

**Key fields to verify:**

```
Certificate Request:
    Data:
        Version: 1 (0x0)
        Subject: C=US, ST=Washington, L=Winthrop, O=Sassy Corp, OU=PKI Operations, CN=Sassy Corp Intermediate CA, emailAddress=pki@sassycorp.internal
        Subject Public Key Info:
            Public Key Algorithm: ML-DSA-65
```

Verify the CSR signature:

```bash
openssl req -in /opt/sassycorp-pqc/intermediate-ca/csr/intermediate-ca.csr -noout -verify
```

**Expected output:**

```
Certificate request self-signature verify OK
```

---

## Step 5: Sign the Intermediate CA with Root CA

Sign the CSR using the Root CA (valid for 5 years / 1825 days):

```bash
openssl ca -config /opt/sassycorp-pqc/root-ca/openssl.cnf \
    -extensions v3_intermediate_ca \
    -days 1825 \
    -notext \
    -md sha512 \
    -in /opt/sassycorp-pqc/intermediate-ca/csr/intermediate-ca.csr \
    -out /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt
```

When prompted, confirm the signing:

```
Sign the certificate? [y/n]: y
1 out of 1 certificate requests certified, commit? [y/n]: y
```

**Command breakdown:**

| Option | Purpose |
|--------|---------|
| `-config` | Use Root CA configuration |
| `-extensions v3_intermediate_ca` | Apply intermediate CA extensions |
| `-days 1825` | 5-year validity |
| `-notext` | Don't include text representation in cert |
| `-md sha512` | Use SHA-512 for any hashing operations |
| `-in` | Input CSR file |
| `-out` | Output certificate file |

Set permissions on the certificate:

```bash
chmod 444 /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt
```

---

## Step 6: Verify the Intermediate CA Certificate

View the certificate:

```bash
openssl x509 -in /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt -noout -text
```

### Check Critical Fields

Verify the issuer (should be Root CA):

```bash
openssl x509 -in /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt -noout -issuer
```

**Expected output:**

```
issuer=C=US, ST=Washington, L=Winthrop, O=Sassy Corp, OU=PKI Operations, CN=Sassy Corp Root CA, emailAddress=pki@sassycorp.internal
```

Verify the subject:

```bash
openssl x509 -in /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt -noout -subject
```

**Expected output:**

```
subject=C=US, ST=Washington, L=Winthrop, O=Sassy Corp, OU=PKI Operations, CN=Sassy Corp Intermediate CA, emailAddress=pki@sassycorp.internal
```

Verify the signature algorithm:

```bash
openssl x509 -in /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt -noout -text | grep "Signature Algorithm"
```

**Expected output:**

```
    Signature Algorithm: ML-DSA-87
    Signature Algorithm: ML-DSA-87
```

> Note: The signature is ML-DSA-87 because the Root CA signed the certificate.

Verify Basic Constraints:

```bash
openssl x509 -in /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt -noout -text | grep -A1 "Basic Constraints"
```

**Expected output:**

```
            X509v3 Basic Constraints: critical
                CA:TRUE, pathlen:0
```

---

## Step 7: Verify Certificate Chain

Verify the Intermediate CA certificate against the Root CA:

```bash
openssl verify -CAfile /opt/sassycorp-pqc/root-ca/certs/root-ca.crt /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt
```

**Expected output:**

```
/opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt: OK
```

---

## Step 8: Create Certificate Chain Bundle

Create a certificate chain file (Intermediate + Root):

```bash
cat /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt /opt/sassycorp-pqc/root-ca/certs/root-ca.crt > /opt/sassycorp-pqc/intermediate-ca/certs/ca-chain.crt
```

Set permissions:

```bash
chmod 444 /opt/sassycorp-pqc/intermediate-ca/certs/ca-chain.crt
```

Verify the chain bundle contains both certificates:

```bash
openssl crl2pkcs7 -nocrl -certfile /opt/sassycorp-pqc/intermediate-ca/certs/ca-chain.crt | openssl pkcs7 -print_certs -noout
```

**Expected output shows both certificates:**

```
subject=C=US, ST=Washington, L=Winthrop, O=Sassy Corp, OU=PKI Operations, CN=Sassy Corp Intermediate CA, emailAddress=pki@sassycorp.internal
issuer=C=US, ST=Washington, L=Winthrop, O=Sassy Corp, OU=PKI Operations, CN=Sassy Corp Root CA, emailAddress=pki@sassycorp.internal

subject=C=US, ST=Washington, L=Winthrop, O=Sassy Corp, OU=PKI Operations, CN=Sassy Corp Root CA, emailAddress=pki@sassycorp.internal
issuer=C=US, ST=Washington, L=Winthrop, O=Sassy Corp, OU=PKI Operations, CN=Sassy Corp Root CA, emailAddress=pki@sassycorp.internal
```

---

## Step 9: Generate Initial Intermediate CA CRL

Create the initial CRL:

```bash
openssl ca -config /opt/sassycorp-pqc/intermediate-ca/openssl.cnf \
    -gencrl \
    -out /opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.crl
```

Set permissions:

```bash
chmod 444 /opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.crl
```

Verify the CRL:

```bash
openssl crl -in /opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.crl -noout -text
```

**Expected output:**

```
Certificate Revocation List (CRL):
        Version 2 (0x1)
        Signature Algorithm: ML-DSA-65
        Issuer: C=US, ST=Washington, L=Winthrop, O=Sassy Corp, OU=PKI Operations, CN=Sassy Corp Intermediate CA, emailAddress=pki@sassycorp.internal
        Last Update: <date>
        Next Update: <date + 7 days>
        CRL extensions:
            X509v3 Authority Key Identifier:
                <key identifier>
No Revoked Certificates.
```

---

## Step 10: Create OCSP Signing Certificate

Generate a key for the OCSP responder:

```bash
openssl genpkey -algorithm ML-DSA-65 -out /opt/sassycorp-pqc/ocsp/private/ocsp.key
```

Set permissions:

```bash
chmod 400 /opt/sassycorp-pqc/ocsp/private/ocsp.key
```

Create a CSR for the OCSP responder:

```bash
openssl req -new \
    -key /opt/sassycorp-pqc/ocsp/private/ocsp.key \
    -subj "/C=US/ST=Washington/L=Winthrop/O=Sassy Corp/OU=PKI Operations/CN=Sassy Corp OCSP Responder/emailAddress=ocsp@sassycorp.internal" \
    -out /opt/sassycorp-pqc/ocsp/certs/ocsp.csr
```

Sign the OCSP certificate (valid for 1 year):

```bash
openssl ca -config /opt/sassycorp-pqc/intermediate-ca/openssl.cnf \
    -extensions ocsp_cert \
    -days 365 \
    -notext \
    -in /opt/sassycorp-pqc/ocsp/certs/ocsp.csr \
    -out /opt/sassycorp-pqc/ocsp/certs/ocsp.crt
```

Confirm the signing when prompted.

Set permissions:

```bash
chmod 444 /opt/sassycorp-pqc/ocsp/certs/ocsp.crt
```

Verify the OCSP certificate:

```bash
openssl x509 -in /opt/sassycorp-pqc/ocsp/certs/ocsp.crt -noout -text | grep -A1 "Extended Key Usage"
```

**Expected output:**

```
            X509v3 Extended Key Usage: critical
                OCSP Signing
```

---

## Step 11: Export Public Key

Extract and save the Intermediate CA public key:

```bash
openssl x509 -in /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt -noout -pubkey > /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.pub
```

```bash
chmod 444 /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.pub
```

---

## Step 12: Verify Database Update

Check that the Root CA database was updated:

```bash
cat /opt/sassycorp-pqc/root-ca/index.txt
```

**Expected output:**

```
V	<expiry date>		1000	unknown	/C=US/ST=Washington/L=Winthrop/O=Sassy Corp/OU=PKI Operations/CN=Sassy Corp Intermediate CA/emailAddress=pki@sassycorp.internal
```

Check the serial number:

```bash
cat /opt/sassycorp-pqc/root-ca/serial
```

**Expected output:**

```
1001
```

Check the Intermediate CA database for the OCSP certificate:

```bash
cat /opt/sassycorp-pqc/intermediate-ca/index.txt
```

---

## Step 13: Verify Final Directory State

List all Intermediate CA files:

```bash
find /opt/sassycorp-pqc/intermediate-ca -type f | sort
```

**Expected files:**

```
/opt/sassycorp-pqc/intermediate-ca/certs/ca-chain.crt
/opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt
/opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.pub
/opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.crl
/opt/sassycorp-pqc/intermediate-ca/csr/intermediate-ca.csr
/opt/sassycorp-pqc/intermediate-ca/index.txt
/opt/sassycorp-pqc/intermediate-ca/index.txt.attr
/opt/sassycorp-pqc/intermediate-ca/openssl.cnf
/opt/sassycorp-pqc/intermediate-ca/private/intermediate-ca.key
/opt/sassycorp-pqc/intermediate-ca/serial
...
```

---

## Module Summary

You have successfully created the Intermediate CA for Sassy Corp:

| Component | Details |
|-----------|---------|
| Algorithm | ML-DSA-65 (FIPS 204, Level 3) |
| Signed By | Sassy Corp Root CA (ML-DSA-87) |
| Certificate | `/opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt` |
| Private Key | `/opt/sassycorp-pqc/intermediate-ca/private/intermediate-ca.key` |
| Chain Bundle | `/opt/sassycorp-pqc/intermediate-ca/certs/ca-chain.crt` |
| CRL | `/opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.crl` |
| OCSP Cert | `/opt/sassycorp-pqc/ocsp/certs/ocsp.crt` |
| Validity | 5 years |
| Path Length | 0 (end-entity only) |

### Certificate Chain

```
Sassy Corp Root CA (ML-DSA-87)
    └── Sassy Corp Intermediate CA (ML-DSA-65)
            └── End-Entity Certificates
            └── OCSP Responder Certificate
```

---

## Troubleshooting

### "Different organization name" Error

**Problem:** Root CA refuses to sign due to policy_strict

**Solution:** Ensure organization name matches exactly:

```bash
openssl req -in /opt/sassycorp-pqc/intermediate-ca/csr/intermediate-ca.csr -noout -subject
```

Compare with Root CA:

```bash
openssl x509 -in /opt/sassycorp-pqc/root-ca/certs/root-ca.crt -noout -subject
```

### Chain Verification Fails

**Problem:** Intermediate CA doesn't verify against Root

**Solution:** Ensure the correct certificates are being compared:

```bash
openssl x509 -in /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt -noout -issuer_hash
openssl x509 -in /opt/sassycorp-pqc/root-ca/certs/root-ca.crt -noout -subject_hash
```

These hashes should match.

### "Unable to load CA certificate" Error

**Problem:** OpenSSL can't find the Root CA certificate

**Solution:** Verify the path in the configuration file:

```bash
grep "certificate" /opt/sassycorp-pqc/root-ca/openssl.cnf
```

---

## Security Checklist

Before proceeding, verify:

- [ ] Intermediate CA private key has 400 permissions
- [ ] Private key directory has 700 permissions
- [ ] Certificate is signed by Root CA (ML-DSA-87)
- [ ] Basic Constraints shows CA:TRUE with pathlen:0
- [ ] Chain verification succeeds
- [ ] CRL is generated
- [ ] OCSP responder certificate is created

---

**Next:** [Certificates →](04-fips_quantum_ca_certs.md)
