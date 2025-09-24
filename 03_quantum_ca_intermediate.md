# Module 3: Building the Quantum-Resistant Intermediate CA

## Overview

This module covers creating a CNSA 2.0 compliant Intermediate Certificate Authority that will be signed by the Root CA created in Module 2. Continue typing each command to understand the complete PKI hierarchy. And to develop powerful typing skills.

## Step 1: Create Intermediate CA Directory Structure

Ensure you're working as the caadmin user:

```bash
sudo su - caadmin
cd /opt/sassycorp-ca/intermediate-ca
```

Create the directory structure:

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

Create database files:

```bash
touch index.txt
touch index.txt.attr
echo 1000 > serial
echo 1000 > crlnumber
```

Set file permissions:

```bash
chmod 644 index.txt index.txt.attr serial crlnumber
```

Configure unique subject enforcement:

```bash
echo 'unique_subject = yes' > index.txt.attr
```

Verify your directory structure:

```bash
ls -la
```

## Step 2: Create Intermediate CA OpenSSL Configuration

Create the OpenSSL configuration file for the Intermediate CA:

```bash
vim /opt/sassycorp-ca/intermediate-ca/openssl.cnf
```

Copy the following configuration into the file:

```ini
# OpenSSL Intermediate CA Configuration - CNSA 2.0 Compliant
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
dir               = /opt/sassycorp-ca/intermediate-ca
certs             = $dir/certs
crl_dir           = $dir/crl
new_certs_dir     = $dir/newcerts
database          = $dir/index.txt
serial            = $dir/serial
RANDFILE          = $dir/private/.rand

# Intermediate CA certificate and key
private_key       = $dir/private/intermediate-ca.key
certificate       = $dir/certs/intermediate-ca.crt

# Certificate revocation list
crlnumber         = $dir/crlnumber
crl               = $dir/crl/intermediate-ca.crl
crl_extensions    = crl_ext
default_crl_days  = 30

# SHA-512 for CNSA 2.0 compliance
default_md        = sha512

name_opt          = ca_default
cert_opt          = ca_default
default_days      = 365
preserve          = no
policy            = policy_loose
copy_extensions   = copy

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
x509_extensions     = v3_intermediate_ca

# Quantum-resistant signature algorithm
# Using ML-DSA-65 (mldsa65) for intermediate CA
default_sigalg      = mldsa65

[ req_distinguished_name ]
countryName                     = Country Name (2 letter code)
countryName_default            = US

stateOrProvinceName            = State or Province Name
stateOrProvinceName_default    = Washington

localityName                   = Locality Name
localityName_default           = Glacier

0.organizationName             = Organization Name
0.organizationName_default     = SassyCorp

organizationalUnitName         = Organizational Unit Name
organizationalUnitName_default = IT Security Division - Intermediate CA

commonName                     = Common Name
commonName_max                 = 64

emailAddress                   = Email Address
emailAddress_max               = 64
emailAddress_default           = intermediate-ca@sassycorp.internal

[ v3_intermediate_ca ]
# Extensions for intermediate CA certificates
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid:always,issuer
basicConstraints = critical, CA:TRUE, pathlen:0
keyUsage = critical, digitalSignature, cRLSign, keyCertSign
crlDistributionPoints = @crl_section
authorityInfoAccess = @ocsp_section
subjectAltName = @alt_names_intermediate
issuerAltName = issuer:copy

[ alt_names_intermediate ]
DNS.1 = intermediate-ca.sassycorp.lab
DNS.2 = issuing-ca.sassycorp.lab
DNS.3 = ca.sassycorp.lab
email = intermediate-ca@sassycorp.internal
URI = https://intermediate-ca.sassycorp.lab

[ usr_cert ]
# Extensions for end entity certificates
basicConstraints = CA:FALSE
nsCertType = client, email
nsComment = "SassyCorp Intermediate CA Issued Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection
crlDistributionPoints = @crl_section
authorityInfoAccess = @ocsp_section

[ server_cert ]
# Extensions for server certificates
basicConstraints = CA:FALSE
nsCertType = server
nsComment = "SassyCorp Intermediate CA Issued Server Certificate"
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer:always
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
crlDistributionPoints = @crl_section
authorityInfoAccess = @ocsp_section

[ crl_section ]
URI.0 = http://crl.sassycorp.lab/intermediate-ca.crl
URI.1 = http://intermediate-ca.sassycorp.lab/crl/intermediate-ca.crl

[ ocsp_section ]
caIssuers;URI.0 = http://ca.sassycorp.lab/certs/intermediate-ca.crt
OCSP;URI.0 = http://ocsp.sassycorp.lab/

[ crl_ext ]
# Extension for CRLs
authorityKeyIdentifier=keyid:always
issuerAltName=issuer:copy

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
chmod 644 /opt/sassycorp-ca/intermediate-ca/openssl.cnf
```

## Step 3: Generate Intermediate CA Private Key

Navigate to the Intermediate CA directory (if you went wandering around):

```bash
cd /opt/sassycorp-ca/intermediate-ca
```

Generate an ML-DSA-65 (mldsa65) private key for standard CNSA 2.0 security level:

```bash
openssl genpkey \
    -provider oqsprovider \
    -provider default \
    -algorithm mldsa65 \
    -out private/intermediate-ca.key
```

**Note**: We use ML-DSA-65 (mldsa65) for the Intermediate CA as it provides Level 3 security, which is suitable for most operational certificates while being CNSA 2.0 compliant.

Set restrictive permissions on the private key:

```bash
chmod 400 private/intermediate-ca.key
```

Verify the key was created with the correct algorithm:

```bash
openssl pkey -in private/intermediate-ca.key -text -noout -provider oqsprovider -provider default | head -5
```

## Step 4: Create Intermediate CA Certificate Signing Request

Create a CSR for the intermediate CA:

```bash
openssl req -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -new \
    -sha512 \
    -key private/intermediate-ca.key \
    -out csr/intermediate-ca.csr \
    -subj "/C=US/ST=Washington/L=Glacier/O=SassyCorp/OU=IT Security Division - Intermediate CA/CN=SassyCorp Intermediate CA/emailAddress=intermediate-ca@sassycorp.internal"
```

Set appropriate permissions:

```bash
chmod 444 csr/intermediate-ca.csr
```

Verify the CSR:

```bash
openssl req -in csr/intermediate-ca.csr -noout -text -provider oqsprovider -provider default
```

## Step 5: Sign Intermediate CA Certificate with Root CA

Switch to the Root CA directory:

```bash
cd /opt/sassycorp-ca/root-ca
```

Sign the intermediate CA certificate with the Root CA (5-year validity):

```bash
openssl ca -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -extensions v3_intermediate_ca \
    -days 1825 \
    -notext \
    -batch \
    -in /opt/sassycorp-ca/intermediate-ca/csr/intermediate-ca.csr \
    -out /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt
```

Set appropriate permissions:

```bash
chmod 444 /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt
```

Verify the intermediate certificate:

```bash
openssl x509 -provider oqsprovider -provider default -noout -text -in /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt
```

Verify the certificate chain:

```bash
openssl verify -provider oqsprovider -provider default -CAfile /opt/sassycorp-ca/root-ca/certs/root-ca.crt \
    /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt
```

You should see "OK" indicating the chain is valid.

## Step 6: Create Certificate Chain File

Navigate back to the intermediate CA directory:

```bash
cd /opt/sassycorp-ca/intermediate-ca
```

Create the complete certificate chain:

```bash
cat certs/intermediate-ca.crt \
    /opt/sassycorp-ca/root-ca/certs/root-ca.crt > certs/ca-chain.crt
```

Set appropriate permissions:

```bash
chmod 444 certs/ca-chain.crt
```

Create a PEM bundle for distribution:

```bash
cp certs/ca-chain.crt certs/intermediate-ca-bundle.pem
chmod 444 certs/intermediate-ca-bundle.pem
```

Create DER format for Windows compatibility:

```bash
openssl x509 -in certs/intermediate-ca.crt -outform DER -out certs/intermediate-ca.der
chmod 444 certs/intermediate-ca.der
```

## Step 7: Generate Intermediate CA CRL

Generate an initial empty CRL:

```bash
cd /opt/sassycorp-ca/intermediate-ca

openssl ca -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -gencrl \
    -out crl/intermediate-ca.crl
```

Set appropriate permissions:

```bash
chmod 644 crl/intermediate-ca.crl
```

Verify the CRL:

```bash
openssl crl -provider oqsprovider -provider default -in crl/intermediate-ca.crl -noout -text
```

## Step 8: Create OCSP Signing Certificate

Generate an OCSP signing key using ML-DSA-65 (mldsa65) for CNSA 2.0 compliance:

```bash
openssl genpkey \
    -provider oqsprovider \
    -provider default \
    -algorithm mldsa65 \
    -out private/ocsp.key
```

**Note**: The OCSP responder certificate also uses ML-DSA-65 (mldsa65) to maintain CNSA 2.0 compliance throughout the infrastructure.

Set restrictive permissions:

```bash
chmod 400 private/ocsp.key
```

Create an OCSP certificate request:

```bash
openssl req -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -new \
    -sha512 \
    -key private/ocsp.key \
    -out csr/ocsp.csr \
    -subj "/C=US/ST=Washington/L=Glacier/O=SassyCorp/OU=IT Security Division - OCSP/CN=SassyCorp OCSP Responder/emailAddress=ocsp@sassycorp.internal"
```

Sign the OCSP certificate with the intermediate CA:

```bash
openssl ca -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -extensions ocsp \
    -days 365 \
    -notext \
    -batch \
    -in csr/ocsp.csr \
    -out certs/ocsp.crt
```

Set appropriate permissions:

```bash
chmod 444 certs/ocsp.crt
```

Verify the OCSP certificate:

```bash
openssl x509 -provider oqsprovider -provider default -noout -text -in certs/ocsp.crt
```

Check that it has the OCSPSigning extended key usage:

```bash
openssl x509 -in certs/ocsp.crt -noout -text | grep -A1 "Extended Key Usage"
```

## Step 9: Verify CNSA 2.0 Compliance

Run the following verification commands to ensure your Intermediate CA meets CNSA 2.0 requirements:

Display certificate details:

```bash
openssl x509 -provider oqsprovider -provider default -in /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt -noout -subject -issuer -dates
```

Verify signature algorithm is CNSA 2.0 compliant (should show mldsa65 or mldsa87):

```bash
openssl x509 -provider oqsprovider -provider default -in /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt -noout -text | grep "Signature Algorithm" | head -1
```

**Expected Output**: The signature algorithm should show `mldsa65` (ML-DSA-65) or `mldsa87` (ML-DSA-87) for CNSA 2.0 compliance.

Check key usage:

```bash
openssl x509 -provider oqsprovider -provider default -in /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt -noout -text | grep -A2 "Key Usage"
```

Verify basic constraints (pathlen should be 0):

```bash
openssl x509 -in /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt -noout -text | grep -A1 "Basic Constraints"
```

Display Subject Alternative Names:

```bash
openssl x509 -provider oqsprovider -provider default -in /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt -noout -text | grep -A5 "Subject Alternative Name"
```

Check CRL Distribution Points:

```bash
openssl x509 -in /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt -noout -text | grep -A3 "CRL Distribution"
```

Check OCSP information:

```bash
openssl x509 -in /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt -noout -text | grep -A3 "Authority Information"
```

Verify certificate chain:

```bash
openssl verify -provider oqsprovider -provider default -CAfile /opt/sassycorp-ca/intermediate-ca/certs/ca-chain.crt \
    /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt
```

Check file permissions:

```bash
ls -la /opt/sassycorp-ca/intermediate-ca/private/intermediate-ca.key
ls -la /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt
ls -la /opt/sassycorp-ca/intermediate-ca/certs/ca-chain.crt
```

## Step 10: Backup Intermediate CA

Create a backup directory:

```bash
mkdir -p /opt/sassycorp-ca/backups/intermediate-ca-$(date +%Y%m%d)
```

Create a backup archive:

```bash
tar -czf /opt/sassycorp-ca/backups/intermediate-ca-$(date +%Y%m%d)/intermediate-ca-backup.tar.gz \
    --exclude='newcerts/*' \
    -C /opt/sassycorp-ca/intermediate-ca \
    private certs crl csr index.txt serial crlnumber openssl.cnf
```

Set restrictive permissions on the backup:

```bash
chmod 600 /opt/sassycorp-ca/backups/intermediate-ca-$(date +%Y%m%d)/intermediate-ca-backup.tar.gz
```

Generate backup verification hash:

```bash
sha512sum /opt/sassycorp-ca/backups/intermediate-ca-$(date +%Y%m%d)/intermediate-ca-backup.tar.gz > \
    /opt/sassycorp-ca/backups/intermediate-ca-$(date +%Y%m%d)/intermediate-ca-backup.sha512
```

Verify the backup was created:

```bash
ls -la /opt/sassycorp-ca/backups/intermediate-ca-$(date +%Y%m%d)/
```

## Step 11: Configure OCSP Responder (Optional)

Create OCSP directories:

```bash
mkdir -p /opt/sassycorp-ca/ocsp/{requests,responses}
chmod 755 /opt/sassycorp-ca/ocsp/{requests,responses}
```

Create an OCSP responder configuration file:

```bash
vim /opt/sassycorp-ca/ocsp/ocsp-responder.conf
```

Add the following configuration:

```ini
# OCSP Responder Configuration
[ ocsp ]
default_ocsp = ocsp_section

[ ocsp_section ]
dir = /opt/sassycorp-ca/intermediate-ca
private_key = $dir/private/ocsp.key
certificate = $dir/certs/ocsp.crt
database = $dir/index.txt
ca_certificate = $dir/certs/intermediate-ca.crt
other_certificates = $dir/certs/ca-chain.crt
response_cert_ext = ocsp_cert_ext
ndays = 7
nmin = 5

[ ocsp_cert_ext ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = OCSPSigning
```

Save and set permissions:

```bash
chmod 644 /opt/sassycorp-ca/ocsp/ocsp-responder.conf
```

## Testing Commands

Test certificate chain validation:

```bash
echo "Testing certificate chain..."
openssl verify -CAfile /opt/sassycorp-ca/root-ca/certs/root-ca.crt \
    -untrusted /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt \
    /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt
```

Test OCSP certificate:

```bash
openssl verify -provider oqsprovider -provider default -CAfile /opt/sassycorp-ca/intermediate-ca/certs/ca-chain.crt \
    /opt/sassycorp-ca/intermediate-ca/certs/ocsp.crt
```

Display certificate fingerprints:

```bash
openssl x509 -in /opt/sassycorp-ca/root-ca/certs/root-ca.crt -noout -fingerprint -sha512

echo "Intermediate CA SHA-512 fingerprint:"
openssl x509 -in /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt -noout -fingerprint -sha512
```

## Troubleshooting

### Certificate Chain Validation Fails

Ensure both certificates are in the chain file:

```bash
cat /opt/sassycorp-ca/intermediate-ca/certs/ca-chain.crt
```

### OCSP Certificate Missing Extended Key Usage

Verify OCSP certificate has OCSPSigning:

```bash
openssl x509 -in certs/ocsp.crt -noout -text | grep -A1 "Extended Key Usage"
```

### Path Length Constraint Issues

Verify pathlen is 0 for intermediate CA:

```bash
openssl x509 -in certs/intermediate-ca.crt -noout -text | grep -A1 "Basic Constraints"
```

## Summary

You have successfully created a CNSA 2.0 compliant quantum-resistant Intermediate CA that is:

- ✅ CNSA 2.0 compliant using ML-DSA-65 (mldsa65)
- ✅ Signed by the Root CA with 5-year validity
- ✅ Using SHA-512 for hashing (CNSA 2.0 requirement)
- ✅ Includes CRL Distribution Points and OCSP configuration
- ✅ Has OCSP responder certificate ready (also using ML-DSA-65)

---

**Next**: [Module 4 - Building Certificates →](04_quantum_ca_certificates.md)