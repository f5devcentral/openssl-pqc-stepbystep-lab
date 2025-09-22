# Module 2: Building the Quantum-Resistant Root CA

## Overview

This module covers the creation of a CNSA 2.0 compliant Root Certificate Authority with a 10-year validity period using quantum-resistant algorithms. You'll type each command directly to understand the process.

## Step 1: Create Root CA Directory Structure

Switch to the CA admin user and navigate to the root CA directory:

```bash
sudo su - caadmin
cd /opt/sassycorp-ca/root-ca
```

Create the required subdirectories:

```bash
mkdir -p private certs crl newcerts csr
```

Set restrictive permissions on the private key directory:

```bash
chmod 700 private
```

Set appropriate permissions on other directories:

```bash
chmod 755 certs crl newcerts csr
```

Create the database files:

```bash
touch index.txt
echo 1000 > serial
echo 1000 > crlnumber
```

Set file permissions:

```bash
chmod 644 index.txt serial crlnumber
```

Verify your directory structure:

```bash
ls -la
```

## Step 2: Create Root CA OpenSSL Configuration

Create the OpenSSL configuration file for the Root CA. Use your preferred text editor:

```bash
nano /opt/sassycorp-ca/root-ca/openssl.cnf
```

Copy the following configuration into the file:

```ini
# OpenSSL Root CA Configuration - CNSA 2.0 Compliant
# Organization: SassyCorp

# Enable OQS Provider for quantum-resistant algorithms
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

####################################################################
[ ca ]
default_ca = CA_default

[ CA_default ]
# Directory and file locations
dir               = /opt/sassycorp-ca/root-ca
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand

# Root CA certificate and key
private_key       = $dir/private/root-ca.key
certificate       = $dir/certs/root-ca.crt

# Certificate revocation list
crlnumber         = $dir/crlnumber
crl               = $dir/crl/root-ca.crl
crl_extensions    = crl_ext
default_crl_days  = 30

# SHA-512 for CNSA 2.0 compliance
default_md        = sha512

name_opt          = ca_default
cert_opt          = ca_default
default_days      = 3650
preserve          = no
policy            = policy_strict

[ policy_strict ]
# Root CA should only sign intermediate CA certificates
countryName             = match
stateOrProvinceName     = match
organizationName        = match
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

[ policy_loose ]
# Allow intermediate CA to sign a wider range of certificates
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional

####################################################################
[ req ]
# Options for certificate requests
default_bits        = 4096
distinguished_name  = req_distinguished_name
string_mask         = utf8only
default_md          = sha512

# Extension to add when the -x509 option is used
x509_extensions     = v3_ca

# Quantum-resistant signature algorithm
# Using ML-DSA-87 (mldsa87) for CNSA 2.0 compliance
default_sigalg      = mldsa87

[ req_distinguished_name ]
# Prompts for distinguished name
countryName                     = Country Name (2 letter code)
countryName_default            = US
countryName_min                = 2
countryName_max                = 2

stateOrProvinceName            = State or Province Name
stateOrProvinceName_default    = Washington

localityName                   = Locality Name
localityName_default           = Glacier

0.organizationName             = Organization Name
0.organizationName_default     = SassyCorp

organizationalUnitName         = Organizational Unit Name
organizationalUnitName_default = IT Security Division

commonName                     = Common Name
commonName_max                 = 64

emailAddress                   = Email Address
emailAddress_max               = 64
emailAddress_default           = ca-admin@sassycorp.internal

[ v3_ca ]
# Extensions for a typical CA
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:TRUE
keyUsage = critical, digitalSignature, cRLSign, keyCertSign
subjectAltName = @alt_names_ca
issuerAltName = @alt_names_ca

[ alt_names_ca ]
DNS.1 = ca.sassycorp.lab
DNS.2 = root-ca.sassycorp.lab
email = ca-admin@sassycorp.internal
URI = https://ca.sassycorp.lab

[ v3_intermediate_ca ]
# Extensions for intermediate CA certificates
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:TRUE, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign
crlDistributionPoints = @crl_section
authorityInfoAccess = @ocsp_section
subjectAltName = @alt_names_intermediate

[ alt_names_intermediate ]
DNS.1 = intermediate-ca.sassycorp.lab
DNS.2 = ca.sassycorp.lab
email = intermediate-ca@sassycorp.internal
URI = https://intermediate-ca.sassycorp.lab

[ crl_section ]
URI.0 = http://crl.sassycorp.lab/root-ca.crl
URI.1 = http://ca.sassycorp.lab/crl/root-ca.crl

[ ocsp_section ]
caIssuers;URI.0 = http://ca.sassycorp.lab/certs/root-ca.crt
OCSP;URI.0 = http://ocsp.sassycorp.lab/

[ crl_ext ]
# Extension for CRLs
authorityKeyIdentifier=keyid:always

[ ocsp ]
# Extension for OCSP signing certificates
basicConstraints = CA:FALSE
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, digitalSignature
extendedKeyUsage = critical, OCSPSigning
```

Save the file and set appropriate permissions:

```bash
chmod 644 /opt/sassycorp-ca/root-ca/openssl.cnf
```

## Step 3: Generate Root CA Private Key

Navigate to the Root CA directory:

```bash
cd /opt/sassycorp-ca/root-ca
```

Generate an ML-DSA-87 (mldsa87) private key for highest CNSA 2.0 security level:

```bash
openssl genpkey \
    -provider oqsprovider \
    -provider default \
    -algorithm mldsa87 \
    -out private/root-ca.key
```

**Note**: We use ML-DSA-87 (mldsa87) for the Root CA as it provides the highest security level in CNSA 2.0.

Set restrictive permissions on the private key:

```bash
chmod 400 private/root-ca.key
```

Verify the key was created:

```bash
ls -la private/root-ca.key
```

## Step 4: Create Root CA Certificate

Create the Root CA certificate with 10-year validity (3650 days):

```bash
openssl req -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -key private/root-ca.key \
    -new -x509 \
    -sha512 \
    -extensions v3_ca \
    -days 3650 \
    -out certs/root-ca.crt \
    -subj "/C=US/ST=Washington/L=Glacier/O=SassyCorp/OU=IT Security Division/CN=SassyCorp Root CA/emailAddress=ca-admin@sassycorp.internal"
```

Set appropriate permissions:

```bash
chmod 444 certs/root-ca.crt
```

Verify the Root CA certificate:

```bash
openssl x509 -noout -text -in certs/root-ca.crt
```

## Step 5: Verify CNSA 2.0 Compliance

Check the signature algorithm (should show mldsa87 for ML-DSA-87):

```bash
openssl x509 -in certs/root-ca.crt -noout -text | grep "Signature Algorithm"
```

**Expected Output**: You should see `mldsa87` which is ML-DSA-87 in CNSA 2.0.

Check certificate validity (should be 10 years):

```bash
openssl x509 -in certs/root-ca.crt -noout -dates
```

Check key usage:

```bash
openssl x509 -in certs/root-ca.crt -noout -text | grep -A1 "Key Usage"
```

Check Subject Alternative Names:

```bash
openssl x509 -in certs/root-ca.crt -noout -text | grep -A5 "Subject Alternative Name"
```

Verify certificate hash with SHA-512:

```bash
openssl x509 -in certs/root-ca.crt -noout -fingerprint -sha512
```

## Step 6: Create Certificate Revocation List

Generate an initial empty CRL:

```bash
openssl ca -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -gencrl \
    -out crl/root-ca.crl
```

Set appropriate permissions:

```bash
chmod 644 crl/root-ca.crl
```

Verify the CRL:

```bash
openssl crl -in crl/root-ca.crl -noout -text
```

## Step 7: Create Root CA Bundle and Distribution Files

Create a PEM bundle for distribution:

```bash
cat certs/root-ca.crt > certs/root-ca-bundle.pem
chmod 444 certs/root-ca-bundle.pem
```

Create DER format for Windows compatibility:

```bash
openssl x509 -in certs/root-ca.crt -outform DER -out certs/root-ca.der
chmod 444 certs/root-ca.der
```

Create a certificate fingerprints file:

```bash
echo "Root CA Certificate Fingerprints" > certs/root-ca-fingerprints.txt
echo "=================================" >> certs/root-ca-fingerprints.txt
echo "" >> certs/root-ca-fingerprints.txt
echo "SHA256:" >> certs/root-ca-fingerprints.txt
openssl x509 -in certs/root-ca.crt -noout -fingerprint -sha256 >> certs/root-ca-fingerprints.txt
echo "" >> certs/root-ca-fingerprints.txt
echo "SHA512:" >> certs/root-ca-fingerprints.txt
openssl x509 -in certs/root-ca.crt -noout -fingerprint -sha512 >> certs/root-ca-fingerprints.txt
chmod 444 certs/root-ca-fingerprints.txt
```

Display the fingerprints:

```bash
cat certs/root-ca-fingerprints.txt
```

## Step 8: Backup Root CA

Create backup directory:

```bash
mkdir -p /opt/sassycorp-ca/backups/root-ca-$(date +%Y%m%d)
```

Create a backup archive:

```bash
tar -czf /opt/sassycorp-ca/backups/root-ca-$(date +%Y%m%d)/root-ca-backup.tar.gz \
    --exclude='newcerts/*' \
    -C /opt/sassycorp-ca/root-ca \
    private certs crl index.txt serial crlnumber openssl.cnf
```

Set restrictive permissions on the backup:

```bash
chmod 600 /opt/sassycorp-ca/backups/root-ca-$(date +%Y%m%d)/root-ca-backup.tar.gz
```

Generate backup verification hash:

```bash
sha512sum /opt/sassycorp-ca/backups/root-ca-$(date +%Y%m%d)/root-ca-backup.tar.gz > \
    /opt/sassycorp-ca/backups/root-ca-$(date +%Y%m%d)/root-ca-backup.sha512
```

Verify the backup was created:

```bash
ls -la /opt/sassycorp-ca/backups/root-ca-$(date +%Y%m%d)/
```

## Verification Commands

Run these commands to perform a complete verification of your Root CA:

Display certificate details:

```bash
openssl x509 -in /opt/sassycorp-ca/root-ca/certs/root-ca.crt -noout -subject -issuer -dates
```

Verify signature algorithm is ML-DSA-87 (mldsa87) for CNSA 2.0:

```bash
openssl x509 -in /opt/sassycorp-ca/root-ca/certs/root-ca.crt -noout -text | grep "Signature Algorithm" | head -1
```

**Expected Output**: Should show `mldsa87` confirming ML-DSA-87 CNSA 2.0 compliance.

Display key usage:

```bash
openssl x509 -in /opt/sassycorp-ca/root-ca/certs/root-ca.crt -noout -text | grep -A2 "Key Usage"
```

Display Subject Alternative Names:

```bash
openssl x509 -in /opt/sassycorp-ca/root-ca/certs/root-ca.crt -noout -text | grep -A5 "Subject Alternative Name"
```

Check file permissions:

```bash
ls -la /opt/sassycorp-ca/root-ca/private/root-ca.key
ls -la /opt/sassycorp-ca/root-ca/certs/root-ca.crt
```

Verify certificate chain (should show OK):

```bash
openssl verify -CAfile /opt/sassycorp-ca/root-ca/certs/root-ca.crt \
    /opt/sassycorp-ca/root-ca/certs/root-ca.crt
```

## Troubleshooting

### OQS Provider Not Found

If you get an error about the OQS provider, verify installation:

```bash
openssl list -providers -provider oqsprovider
```

### Permission Denied Errors

Ensure you're running as the caadmin user:

```bash
whoami  # Should show 'caadmin'
```

### ML-DSA-87 Algorithm Not Available

List available quantum-resistant algorithms:

```bash
openssl list -signature-algorithms -provider oqsprovider | grep -i mldsa
```

If you see `dilithium5` instead of `mldsa87`, you may be using an older version of the OQS provider. Use `dilithium5` in that case.

## Summary

You have successfully created a CNSA 2.0 compliant quantum-resistant Root CA that is:
- ✅ CNSA 2.0 compliant using ML-DSA-87 (mldsa87)
- ✅ Valid for 10 years as recommended for Root CAs
- ✅ Using SHA-512 for hashing (CNSA 2.0 requirement)
- ✅ Configured with proper SANs according to RFC specifications
- ✅ Protected with appropriate Unix file permissions
- ✅ Ready to sign intermediate CA certificates

---

**Next**: [Module 3 - Building the Intermediate CA →](03-intermediate-ca.md)