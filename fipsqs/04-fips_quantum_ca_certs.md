# Module 4: Building Certificates

## Overview

This module guides you through issuing quantum-resistant end-entity certificates from Sassy Corp's Intermediate CA. You will create server certificates for TLS authentication and user certificates for client authentication and email signing.

---

## Learning Objectives

After completing this module, you will be able to:

- Generate ML-DSA key pairs for different security levels
- Create Certificate Signing Requests with proper Subject Alternative Names
- Issue server and user certificates from the Intermediate CA
- Verify certificate chains and extensions
- Export certificates in various formats

---

## Certificate Types and Algorithms

| Certificate Type | Algorithm | Security Level | Validity | Use Case |
|-----------------|-----------|---------------|----------|----------|
| Server (TLS) | ML-DSA-65 | Level 3 | 2 years | Web servers, APIs |
| User/Client | ML-DSA-44 | Level 2 | 1 year | Authentication, email |
| High-Security Server | ML-DSA-87 | Level 5 | 2 years | Critical infrastructure |
| Code Signing | ML-DSA-65 | Level 3 | 1 year | Software distribution |

---

## Step 1: Ensure Correct User Context

Verify you are operating as the pqcadmin user:

```bash
whoami
```

If not, switch users:

```bash
sudo su - pqcadmin
```

Create a working directory for certificate requests:

```bash
mkdir -p /opt/sassycorp-pqc/requests
```

---

## Part A: Server Certificate (ML-DSA-65)

### Step 2: Generate Server Private Key

Generate an ML-DSA-65 key for the web server:

```bash
openssl genpkey -algorithm ML-DSA-65 -out /opt/sassycorp-pqc/requests/www.sassycorp.lab.key
```

Set permissions:

```bash
chmod 400 /opt/sassycorp-pqc/requests/www.sassycorp.lab.key
```

Verify the key algorithm:

```bash
openssl pkey -in /opt/sassycorp-pqc/requests/www.sassycorp.lab.key -noout -text | head -3
```

**Expected output:**

```
ML-DSA-65 Private-Key:
priv:
    <hexadecimal values>
```

---

### Step 3: Create Server Certificate Configuration

Create a configuration file for the server certificate:

```bash
vim /opt/sassycorp-pqc/requests/www.sassycorp.lab.cnf
```

Enter the following:

```ini
[ req ]
default_bits       = 4096
distinguished_name = req_distinguished_name
req_extensions     = v3_req
prompt             = no

[ req_distinguished_name ]
countryName            = US
stateOrProvinceName    = Washington
localityName           = Winthrop
organizationName       = Sassy Corp
organizationalUnitName = Web Services
commonName             = www.sassycorp.lab
emailAddress           = webadmin@sassycorp.internal

[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = www.sassycorp.lab
DNS.2 = sassycorp.lab
DNS.3 = api.sassycorp.lab
DNS.4 = portal.sassycorp.lab
IP.1 = 192.168.1.100
IP.2 = 10.0.0.100
```

Save and exit.

---

### Step 4: Create Server CSR

Generate the Certificate Signing Request:

```bash
openssl req -new \
    -config /opt/sassycorp-pqc/requests/www.sassycorp.lab.cnf \
    -key /opt/sassycorp-pqc/requests/www.sassycorp.lab.key \
    -out /opt/sassycorp-pqc/requests/www.sassycorp.lab.csr
```

Verify the CSR includes the Subject Alternative Names:

```bash
openssl req -in /opt/sassycorp-pqc/requests/www.sassycorp.lab.csr -noout -text | grep -A10 "Subject Alternative Name"
```

**Expected output:**

```
            X509v3 Subject Alternative Name:
                DNS:www.sassycorp.lab, DNS:sassycorp.lab, DNS:api.sassycorp.lab, DNS:portal.sassycorp.lab, IP Address:192.168.1.100, IP Address:10.0.0.100
```

---

### Step 5: Sign the Server Certificate

Sign the CSR with the Intermediate CA (valid for 2 years):

```bash
openssl ca -config /opt/sassycorp-pqc/intermediate-ca/openssl.cnf \
    -extensions server_cert \
    -days 730 \
    -notext \
    -in /opt/sassycorp-pqc/requests/www.sassycorp.lab.csr \
    -out /opt/sassycorp-pqc/requests/www.sassycorp.lab.crt
```

Confirm the signing when prompted:

```
Sign the certificate? [y/n]: y
1 out of 1 certificate requests certified, commit? [y/n]: y
```

Set permissions:

```bash
chmod 444 /opt/sassycorp-pqc/requests/www.sassycorp.lab.crt
```

---

### Step 6: Verify the Server Certificate

Check the certificate details:

```bash
openssl x509 -in /opt/sassycorp-pqc/requests/www.sassycorp.lab.crt -noout -text
```

Verify the issuer is the Intermediate CA:

```bash
openssl x509 -in /opt/sassycorp-pqc/requests/www.sassycorp.lab.crt -noout -issuer
```

**Expected output:**

```
issuer=C=US, ST=Washington, L=Winthrop, O=Sassy Corp, OU=PKI Operations, CN=Sassy Corp Intermediate CA, emailAddress=pki@sassycorp.internal
```

Verify the Subject Alternative Names:

```bash
openssl x509 -in /opt/sassycorp-pqc/requests/www.sassycorp.lab.crt -noout -text | grep -A2 "Subject Alternative Name"
```

Verify the signature algorithm (should show the Intermediate CA's algorithm):

```bash
openssl x509 -in /opt/sassycorp-pqc/requests/www.sassycorp.lab.crt -noout -text | grep "Signature Algorithm" | head -1
```

**Expected output:**

```
    Signature Algorithm: ML-DSA-65
```

Verify Extended Key Usage:

```bash
openssl x509 -in /opt/sassycorp-pqc/requests/www.sassycorp.lab.crt -noout -text | grep -A1 "Extended Key Usage"
```

**Expected output:**

```
            X509v3 Extended Key Usage:
                TLS Web Server Authentication
```

---

### Step 7: Verify Server Certificate Chain

Verify the certificate against the CA chain:

```bash
openssl verify -CAfile /opt/sassycorp-pqc/intermediate-ca/certs/ca-chain.crt /opt/sassycorp-pqc/requests/www.sassycorp.lab.crt
```

**Expected output:**

```
/opt/sassycorp-pqc/requests/www.sassycorp.lab.crt: OK
```

---

## Part B: User Certificate (ML-DSA-44)

### Step 8: Generate User Private Key

Generate an ML-DSA-44 key for the user (smaller, appropriate for client auth):

```bash
openssl genpkey -algorithm ML-DSA-44 -out /opt/sassycorp-pqc/requests/alice.smith.key
```

Set permissions:

```bash
chmod 400 /opt/sassycorp-pqc/requests/alice.smith.key
```

Verify the algorithm:

```bash
openssl pkey -in /opt/sassycorp-pqc/requests/alice.smith.key -noout -text | head -2
```

**Expected output:**

```
ML-DSA-44 Private-Key:
priv:
```

---

### Step 9: Create User Certificate Configuration

Create a configuration file:

```bash
vim /opt/sassycorp-pqc/requests/alice.smith.cnf
```

Enter the following:

```ini
[ req ]
default_bits       = 4096
distinguished_name = req_distinguished_name
req_extensions     = v3_req
prompt             = no

[ req_distinguished_name ]
countryName            = US
stateOrProvinceName    = Washington
localityName           = Winthrop
organizationName       = Sassy Corp
organizationalUnitName = Engineering
commonName             = Alice Smith
emailAddress           = alice.smith@sassycorp.internal

[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection
subjectAltName = @alt_names

[ alt_names ]
email.1 = alice.smith@sassycorp.internal
email.2 = alice@sassycorp.lab
URI.1 = https://directory.sassycorp.lab/users/alice.smith
```

Save and exit.

---

### Step 10: Create User CSR

Generate the CSR:

```bash
openssl req -new \
    -config /opt/sassycorp-pqc/requests/alice.smith.cnf \
    -key /opt/sassycorp-pqc/requests/alice.smith.key \
    -out /opt/sassycorp-pqc/requests/alice.smith.csr
```

Verify the CSR:

```bash
openssl req -in /opt/sassycorp-pqc/requests/alice.smith.csr -noout -text | head -20
```

---

### Step 11: Sign the User Certificate

Sign with the Intermediate CA (valid for 1 year):

```bash
openssl ca -config /opt/sassycorp-pqc/intermediate-ca/openssl.cnf \
    -extensions usr_cert \
    -days 365 \
    -notext \
    -in /opt/sassycorp-pqc/requests/alice.smith.csr \
    -out /opt/sassycorp-pqc/requests/alice.smith.crt
```

Confirm the signing when prompted.

Set permissions:

```bash
chmod 444 /opt/sassycorp-pqc/requests/alice.smith.crt
```

---

### Step 12: Verify User Certificate

Check extended key usage:

```bash
openssl x509 -in /opt/sassycorp-pqc/requests/alice.smith.crt -noout -text | grep -A2 "Extended Key Usage"
```

**Expected output:**

```
            X509v3 Extended Key Usage:
                TLS Web Client Authentication, E-mail Protection
```

Check Subject Alternative Names:

```bash
openssl x509 -in /opt/sassycorp-pqc/requests/alice.smith.crt -noout -text | grep -A3 "Subject Alternative Name"
```

Verify the chain:

```bash
openssl verify -CAfile /opt/sassycorp-pqc/intermediate-ca/certs/ca-chain.crt /opt/sassycorp-pqc/requests/alice.smith.crt
```

---

## Part C: High-Security Server Certificate (ML-DSA-87)

### Step 13: Generate High-Security Key

For critical infrastructure, use ML-DSA-87:

```bash
openssl genpkey -algorithm ML-DSA-87 -out /opt/sassycorp-pqc/requests/vault.sassycorp.lab.key
```

```bash
chmod 400 /opt/sassycorp-pqc/requests/vault.sassycorp.lab.key
```

---

### Step 14: Create High-Security Server Configuration

```bash
vim /opt/sassycorp-pqc/requests/vault.sassycorp.lab.cnf
```

Enter:

```ini
[ req ]
default_bits       = 4096
distinguished_name = req_distinguished_name
req_extensions     = v3_req
prompt             = no

[ req_distinguished_name ]
countryName            = US
stateOrProvinceName    = Washington
localityName           = Winthrop
organizationName       = Sassy Corp
organizationalUnitName = Security Infrastructure
commonName             = vault.sassycorp.lab
emailAddress           = security@sassycorp.internal

[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = vault.sassycorp.lab
DNS.2 = secrets.sassycorp.lab
DNS.3 = hsm.sassycorp.lab
IP.1 = 192.168.1.10
```

Save and exit.

---

### Step 15: Create and Sign High-Security Certificate

Create CSR:

```bash
openssl req -new \
    -config /opt/sassycorp-pqc/requests/vault.sassycorp.lab.cnf \
    -key /opt/sassycorp-pqc/requests/vault.sassycorp.lab.key \
    -out /opt/sassycorp-pqc/requests/vault.sassycorp.lab.csr
```

Sign the certificate:

```bash
openssl ca -config /opt/sassycorp-pqc/intermediate-ca/openssl.cnf \
    -extensions server_cert \
    -days 730 \
    -notext \
    -in /opt/sassycorp-pqc/requests/vault.sassycorp.lab.csr \
    -out /opt/sassycorp-pqc/requests/vault.sassycorp.lab.crt
```

```bash
chmod 444 /opt/sassycorp-pqc/requests/vault.sassycorp.lab.crt
```

Verify the certificate uses the ML-DSA-87 public key:

```bash
openssl x509 -in /opt/sassycorp-pqc/requests/vault.sassycorp.lab.crt -noout -text | grep "Public Key Algorithm"
```

**Expected output:**

```
            Public Key Algorithm: ML-DSA-87
```

---

## Part D: Certificate Export Formats

### Step 16: Export to PKCS#12 (PFX) Format

Create a PKCS#12 bundle for the server certificate (includes key, cert, and chain):

```bash
openssl pkcs12 -export \
    -inkey /opt/sassycorp-pqc/requests/www.sassycorp.lab.key \
    -in /opt/sassycorp-pqc/requests/www.sassycorp.lab.crt \
    -certfile /opt/sassycorp-pqc/intermediate-ca/certs/ca-chain.crt \
    -out /opt/sassycorp-pqc/requests/www.sassycorp.lab.p12 \
    -name "Sassy Corp Web Server"
```

You will be prompted to enter an export password. Use a strong password.

Set permissions:

```bash
chmod 400 /opt/sassycorp-pqc/requests/www.sassycorp.lab.p12
```

---

### Step 17: Export to DER Format

Create DER-encoded versions:

```bash
openssl x509 -in /opt/sassycorp-pqc/requests/www.sassycorp.lab.crt -outform DER -out /opt/sassycorp-pqc/requests/www.sassycorp.lab.der
```

```bash
chmod 444 /opt/sassycorp-pqc/requests/www.sassycorp.lab.der
```

---

### Step 18: Create Full Chain PEM File

Create a PEM file with the full certificate chain (useful for web servers):

```bash
cat /opt/sassycorp-pqc/requests/www.sassycorp.lab.crt /opt/sassycorp-pqc/intermediate-ca/certs/ca-chain.crt > /opt/sassycorp-pqc/requests/www.sassycorp.lab.fullchain.pem
```

```bash
chmod 444 /opt/sassycorp-pqc/requests/www.sassycorp.lab.fullchain.pem
```

---

## Step 19: View Certificate Database

Check the Intermediate CA database to see all issued certificates:

```bash
cat /opt/sassycorp-pqc/intermediate-ca/index.txt
```

**Expected output (entries will vary):**

```
V	<expiry>		2000	unknown	/C=US/ST=Washington/L=Winthrop/O=Sassy Corp/OU=PKI Operations/CN=Sassy Corp OCSP Responder/emailAddress=ocsp@sassycorp.internal
V	<expiry>		2001	unknown	/C=US/ST=Washington/L=Winthrop/O=Sassy Corp/OU=Web Services/CN=www.sassycorp.lab/emailAddress=webadmin@sassycorp.internal
V	<expiry>		2002	unknown	/C=US/ST=Washington/L=Winthrop/O=Sassy Corp/OU=Engineering/CN=Alice Smith/emailAddress=alice.smith@sassycorp.internal
V	<expiry>		2003	unknown	/C=US/ST=Washington/L=Winthrop/O=Sassy Corp/OU=Security Infrastructure/CN=vault.sassycorp.lab/emailAddress=security@sassycorp.internal
```

The `V` indicates Valid certificates.

---

## Step 20: Test Signature Operations

Test signing and verifying with the user certificate:

Create test data:

```bash
echo "This is a test message from Sassy Corp" > /tmp/testmessage.txt
```

Sign the message:

```bash
openssl dgst -sign /opt/sassycorp-pqc/requests/alice.smith.key -out /tmp/testmessage.sig /tmp/testmessage.txt
```

Extract the public key:

```bash
openssl x509 -in /opt/sassycorp-pqc/requests/alice.smith.crt -pubkey -noout > /tmp/alice.pub
```

Verify the signature:

```bash
openssl dgst -verify /tmp/alice.pub -signature /tmp/testmessage.sig /tmp/testmessage.txt
```

**Expected output:**

```
Verified OK
```

Clean up:

```bash
rm /tmp/testmessage.txt /tmp/testmessage.sig /tmp/alice.pub
```

---

## Module Summary

You have successfully created quantum-resistant certificates:

| Certificate | Algorithm | Subject | Validity |
|------------|-----------|---------|----------|
| www.sassycorp.lab | ML-DSA-65 | Web server with 4 DNS + 2 IP SANs | 2 years |
| alice.smith | ML-DSA-44 | User authentication and email | 1 year |
| vault.sassycorp.lab | ML-DSA-87 | High-security infrastructure | 2 years |

### Files Created

```
/opt/sassycorp-pqc/requests/
├── www.sassycorp.lab.key        # Server private key
├── www.sassycorp.lab.csr        # Server CSR
├── www.sassycorp.lab.crt        # Server certificate
├── www.sassycorp.lab.cnf        # Server config
├── www.sassycorp.lab.p12        # PKCS#12 bundle
├── www.sassycorp.lab.der        # DER format
├── www.sassycorp.lab.fullchain.pem  # Full chain
├── alice.smith.key              # User private key
├── alice.smith.csr              # User CSR
├── alice.smith.crt              # User certificate
├── alice.smith.cnf              # User config
├── vault.sassycorp.lab.key      # High-security key
├── vault.sassycorp.lab.csr      # High-security CSR
├── vault.sassycorp.lab.crt      # High-security certificate
└── vault.sassycorp.lab.cnf      # High-security config
```

---

## Troubleshooting

### "TXT_DB error number 2" Error

**Problem:** Duplicate certificate entry

**Solution:** This means a certificate with the same subject already exists. Either:
- Use a unique Common Name
- Revoke the existing certificate first
- Edit `index.txt.attr` to set `unique_subject = no` (not recommended)

### SAN Not Appearing in Certificate

**Problem:** Subject Alternative Names missing

**Solution:** Ensure `copy_extensions = copy` is in the CA config and the CSR includes the extensions:
```bash
openssl req -in <csr> -noout -text | grep -A5 "Subject Alternative Name"
```

### Wrong Algorithm in Certificate

**Problem:** Certificate shows wrong algorithm

**Solution:** The certificate's public key algorithm is determined by the key used to create the CSR, not the CA. Verify your key:
```bash
openssl pkey -in <key> -noout -text | head -1
```

---

## Security Checklist

Before proceeding, verify:

- [ ] All private keys have 400 permissions
- [ ] Server certificates have serverAuth extended key usage
- [ ] User certificates have clientAuth extended key usage
- [ ] All certificates verify against the CA chain
- [ ] PKCS#12 files have strong export passwords
- [ ] No private keys are world-readable

---

**Next:** [Revocation →](05_fips_quantum_ca_recovation.md)
