# Module 4: Building End-Entity Certificates

## Overview

This module demonstrates creating various types of end-entity certificates using the CNSA 2.0 approved quantum-resistant signature algorithm. All end-entity certificates in a CNSA 2.0 PKI use ML-DSA-87 per draft-jenkins-cnsa2-pkix-profile §4.

## CNSA 2.0 Compliant Algorithms

For strict CNSA 2.0 compliance, we use only these quantum-resistant signature algorithms:

| Algorithm | NIST Designation | Security Level | OpenSSL Name | Legacy Name | Use Case |
|-----------|-----------------|----------------|--------------|-------------|----------|
| ML-DSA-87 | FIPS 204 | Level 5 | mldsa87 | dilithium5 | Highest security applications |

**Note**: *The newer OQS provider uses the NIST standardized name (mldsa87) instead of the legacy name (dilithium5). Other post-quantum algorithms — including ML-DSA-44, ML-DSA-65, SLH-DSA, and Falcon — are NOT CNSA 2.0 compliant, even though several are FIPS-approved.*

## Step 1: Create Certificate Configuration Templates

Navigate to the intermediate CA directory:

```bash
sudo su - caadmin
cd /opt/sassycorp-ca/intermediate-ca
```

Create a templates directory:

```bash
mkdir -p templates
chmod 755 templates
```

### Create Server Certificate Configuration

Create a server certificate configuration file:

```bash
vim templates/server-cert.cnf
```

Add the following configuration:

```ini
# Server Certificate Configuration - CNSA 2.0 Compliant
[ req ]
default_bits = 4096
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = yes
string_mask = utf8only
default_md = sha512

[ req_distinguished_name ]
countryName                 = Country Name (2 letter code)
countryName_default         = US
countryName_min            = 2
countryName_max            = 2

stateOrProvinceName        = State or Province Name
stateOrProvinceName_default = Washington

localityName               = Locality Name
localityName_default       = Glacier

organizationName           = Organization Name
organizationName_default   = SassyCorp

organizationalUnitName     = Organizational Unit Name
organizationalUnitName_default = IT Infrastructure

commonName                 = Common Name
commonName_max             = 64

emailAddress               = Email Address
emailAddress_max           = 64
emailAddress_default       = admin@sassycorp.internal

[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = localhost
DNS.2 = *.sassycorp.lab
DNS.3 = *.sassycorp.internal
IP.1 = 127.0.0.1
IP.2 = 192.168.1.100
email = admin@sassycorp.internal
URI = https://www.sassycorp.lab
```

Save and set permissions:

```bash
chmod 644 templates/server-cert.cnf
```

### Create User Certificate Configuration

Create a user certificate configuration file:

```bash
vim templates/user-cert.cnf
```

Add the following configuration:

```ini
# User Certificate Configuration - CNSA 2.0 Compliant
[ req ]
default_bits = 4096
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = yes
string_mask = utf8only
default_md = sha512

[ req_distinguished_name ]
countryName                 = Country Name (2 letter code)
countryName_default         = US

stateOrProvinceName        = State or Province Name
stateOrProvinceName_default = Washington

localityName               = Locality Name
localityName_default       = Glacier

organizationName           = Organization Name
organizationName_default   = SassyCorp

organizationalUnitName     = Organizational Unit Name
organizationalUnitName_default = Users

commonName                 = Common Name (User Full Name)
commonName_max             = 64

emailAddress               = Email Address
emailAddress_max           = 64

[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = critical, nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = clientAuth, emailProtection
subjectAltName = @alt_names

[ alt_names ]
email = copy
DNS = user.sassycorp.internal
```

Save and set permissions:

```bash
chmod 644 templates/user-cert.cnf
```

## Step 2: Generate a Web Server Certificate (ML-DSA-87 / mldsa87)

Create a directory for the server certificate:

```bash
mkdir -p certs/server/webserver01
chmod 755 certs/server/webserver01
cd certs/server/webserver01
```

Generate an ML-DSA-87 (mldsa87 private key for CNSA 2.0 compliance:

```bash
openssl genpkey \
    -provider oqsprovider \
    -provider default \
    -algorithm mldsa87 \
    -out webserver01.key
```

Set restrictive permissions:

```bash
chmod 400 webserver01.key
```

Create a certificate signing request:

```bash
openssl req -config /opt/sassycorp-ca/intermediate-ca/templates/server-cert.cnf \
    -provider oqsprovider \
    -provider default \
    -new \
    -sha512 \
    -key webserver01.key \
    -out webserver01.csr
```

When prompted, enter:

- Country Name: `US`
- State: `Washington`
- Locality: `Glacier`
- Organization: `SassyCorp`
- Organizational Unit: `Web Services`
- Common Name: `webserver01.sassycorp.lab`
- Email: `webadmin@sassycorp.internal`

For the challenge password and optional company name, just press Enter to skip.

Sign the certificate with the intermediate CA:

```bash
cd /opt/sassycorp-ca/intermediate-ca

openssl ca -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -extensions server_cert \
    -days 730 \
    -notext \
    -in certs/server/webserver01/webserver01.csr \
    -out certs/server/webserver01/webserver01.crt
```

Type 'y' to sign and 'y' to commit when prompted.

Set appropriate permissions:

```bash
chmod 444 certs/server/webserver01/webserver01.crt
```

Verify the certificate:

```bash
openssl x509 -noout -text -provider oqsprovider -provider default -in certs/server/webserver01/webserver01.crt
```

Check the Subject Alternative Names:

```bash
openssl x509 -provider oqsprovider -provider -in certs/server/webserver01/webserver01.crt -noout -text | grep -A10 "Subject Alternative Name"
```

## Step 3: Generate a User Certificate (ML-DSA-87 / mldsa87)

Create a directory for user certificates:

```bash
mkdir -p certs/users/sassy.molassy
chmod 755 certs/users/sassy.molassy
cd certs/users/sassy.molassy
```

Generate an ML-DSA-87 (mldsa87) private key:

```bash
openssl genpkey \
    -provider oqsprovider \
    -provider default \
    -algorithm mldsa87 \
    -out sassy.molassy.key
```

Set restrictive permissions:

```bash
chmod 400 sassy.molassy.key
```

Create a certificate signing request:

```bash
openssl req -config /opt/sassycorp-ca/intermediate-ca/templates/user-cert.cnf \
    -provider oqsprovider \
    -provider default \
    -new \
    -sha512 \
    -key sassy.molassy.key \
    -out sassy.molassy.csr
```

When prompted, enter:

- Country Name: `US`
- State: `Washington`
- Locality: `Glacier`
- Organization: `SassyCorp`
- Organizational Unit: `Human Resources`
- Common Name: `Sassy Molassy`
- Email: `sassy.molassy@sassycorp.internal`

Sign the certificate:

```bash
cd /opt/sassycorp-ca/intermediate-ca

openssl ca -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -extensions usr_cert \
    -days 365 \
    -notext \
    -in certs/users/sassy.molassy/sassy.molassy.csr \
    -out certs/users/sassy.molassy/sassy.molassy.crt
```

Set permissions:

```bash
chmod 444 certs/users/sassy.molassy/sassy.molassy.crt
```

## Step 4: Generate a High-Security Database Server Certificate (ML-DSA-87 / mldsa87)

For critical infrastructure requiring the highest level of quantum resistance, ML-DSA-87 (mldsa87):

```bash
mkdir -p certs/server/db-primary01
chmod 755 certs/server/db-primary01
cd certs/server/db-primary01
```

Generate an ML-DSA-87 (mldsa87) private key for highest CNSA 2.0 security:

```bash
openssl genpkey \
    -provider oqsprovider \
    -provider default \
    -algorithm mldsa87 \
    -out db-primary01.key
```

Set restrictive permissions:

```bash
chmod 400 db-primary01.key
```

Create a custom configuration for the database server with specific SANs:

```bash
vim db-primary01.cnf
```

Add the following configuration:

```ini
[ req ]
default_bits = 4096
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no
string_mask = utf8only
default_md = sha512

[ req_distinguished_name ]
C = US
ST = Washington
L = Glacier
O = SassyCorp
OU = Database Administration
CN = db-primary01.sassycorp.lab
emailAddress = dba@sassycorp.internal

[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = db-primary01.sassycorp.lab
DNS.2 = db-primary01.sassycorp.internal
DNS.3 = database.sassycorp.lab
DNS.4 = postgres.sassycorp.lab
IP.1 = 10.0.0.10
IP.2 = 127.0.0.1
email = dba@sassycorp.internal
URI = https://db-primary01.sassycorp.lab:5432
```

Save and set permissions:

```bash
chmod 644 db-primary01.cnf
```

Create the certificate signing request:

```bash
openssl req -config db-primary01.cnf \
    -provider oqsprovider \
    -provider default \
    -new \
    -sha512 \
    -key db-primary01.key \
    -out db-primary01.csr
```

Sign the certificate:

```bash
cd /opt/sassycorp-ca/intermediate-ca

openssl ca -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -extensions server_cert \
    -days 365 \
    -notext \
    -in certs/server/db-primary01/db-primary01.csr \
    -out certs/server/db-primary01/db-primary01.crt
```

Set permissions:

```bash
chmod 444 certs/server/db-primary01/db-primary01.crt
```

## Step 5: Generate an API Server Certificate (ML-DSA-87 / mldsa87)

Create another server certificate:

```bash
mkdir -p certs/server/api-server01
chmod 755 certs/server/api-server01
cd certs/server/api-server01
```

Generate an ML-DSA87 (mldsa87) private key:

```bash
openssl genpkey \
    -provider oqsprovider \
    -provider default \
    -algorithm mldsa87 \
    -out api-server01.key
```

Set restrictive permissions:

```bash
chmod 400 api-server01.key
```

Create API server configuration:

```bash
vim api-server01.cnf
```

Add the following:

```ini
[ req ]
distinguished_name = req_distinguished_name
req_extensions = v3_req
prompt = no
string_mask = utf8only
default_md = sha512

[ req_distinguished_name ]
C = US
ST = Washington
L = Glacier
O = SassyCorp
OU = API Services
CN = api.sassycorp.lab
emailAddress = api-admin@sassycorp.internal

[ v3_req ]
basicConstraints = CA:FALSE
keyUsage = critical, digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names

[ alt_names ]
DNS.1 = api.sassycorp.lab
DNS.2 = api.sassycorp.internal
DNS.3 = api-server01.sassycorp.lab
DNS.4 = api-v1.sassycorp.lab
DNS.5 = api-v2.sassycorp.lab
IP.1 = 10.0.2.100
email = api-admin@sassycorp.internal
URI = https://api.sassycorp.lab
```

Save and set permissions:

```bash
chmod 644 api-server01.cnf
```

Create the CSR:

```bash
openssl req -config api-server01.cnf \
    -provider oqsprovider \
    -provider default \
    -new \
    -sha512 \
    -key api-server01.key \
    -out api-server01.csr
```

Sign the certificate:

```bash
cd /opt/sassycorp-ca/intermediate-ca

openssl ca -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -extensions server_cert \
    -days 365 \
    -notext \
    -in certs/server/api-server01/api-server01.csr \
    -out certs/server/api-server01/api-server01.crt
```

Set permissions:

```bash
chmod 444 certs/server/api-server01/api-server01.crt
```

## Step 6: Create Certificate Bundles

For each certificate, create a bundle with the certificate chain:

### Web Server Bundle

```bash
cd /opt/sassycorp-ca/intermediate-ca/certs/server/webserver01

cat webserver01.crt \
    /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt \
    /opt/sassycorp-ca/root-ca/certs/root-ca.crt > webserver01-bundle.pem

chmod 444 webserver01-bundle.pem
```

### User Certificate Bundle

```bash
cd /opt/sassycorp-ca/intermediate-ca/certs/users/sassy.molassy

cat sassy.molassy.crt \
    /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt \
    /opt/sassycorp-ca/root-ca/certs/root-ca.crt > sassy.molassy-bundle.pem

chmod 444 sassy.molassy-bundle.pem
```

### Database Server Bundle

```bash
cd /opt/sassycorp-ca/intermediate-ca/certs/server/db-primary01

cat db-primary01.crt \
    /opt/sassycorp-ca/intermediate-ca/certs/intermediate-ca.crt \
    /opt/sassycorp-ca/root-ca/certs/root-ca.crt > db-primary01-bundle.pem

chmod 444 db-primary01-bundle.pem
```

## Step 7: Create PKCS#12 Files for Easy Import

### Create PKCS#12 for the web server

```bash
cd /opt/sassycorp-ca/intermediate-ca/certs/server/webserver01

openssl pkcs12 \
    -provider oqsprovider \
    -provider default \
    -export \
    -out webserver01.p12 \
    -inkey webserver01.key \
    -in webserver01.crt \
    -certfile /opt/sassycorp-ca/intermediate-ca/certs/ca-chain.crt
```

Set the password for the pkcs12 package with your favorite password. :-)

Set restrictive permissions:

```bash
chmod 400 webserver01.p12
```

### Create PKCS#12 for the user

```bash
cd /opt/sassycorp-ca/intermediate-ca/certs/users/sassy.molassy

openssl pkcs12 \
    -provider oqsprovider \
    -provider default \
    -export \
    -out sassy.molassy.p12 \
    -inkey sassy.molassy.key \
    -in sassy.molassy.crt \
    -certfile /opt/sassycorp-ca/intermediate-ca/certs/ca-chain.crt
```

Set restrictive permissions:

```bash
chmod 400 sassy.molassy.p12
```

## Step 8: Verify Certificates

### Verify all certificates

```bash
cd /opt/sassycorp-ca/intermediate-ca

# Verify web server certificate
openssl verify -provider oqsprovider -provider default -CAfile certs/ca-chain.crt \
    certs/server/webserver01/webserver01.crt

# Verify user certificate
openssl verify -provider oqsprovider -provider default -CAfile certs/ca-chain.crt \
    certs/users/sassy.molassy/sassy.molassy.crt

# Verify database server certificate
openssl verify -provider oqsprovider -provider default -CAfile certs/ca-chain.crt \
    certs/server/db-primary01/db-primary01.crt

# Verify API server certificate
openssl verify -provider oqsprovider -provider default -CAfile certs/ca-chain.crt \
    certs/server/api-server01/api-server01.crt
```

## Step 9: Compare CNSA 2.0 Algorithm Key Sizes

Compare the key sizes of the two CNSA 2.0 compliant algorithms:

```bash
cd /opt/sassycorp-ca/intermediate-ca

echo "=== CNSA 2.0 Algorithm Key Size Comparison ==="
echo "=== ML-DSA-87 Key Sizes Across Certificates ==="
ls -lh certs/server/webserver01/webserver01.key
ls -lh certs/users/sassy.molassy/sassy.molassy.key
ls -lh certs/server/api-server01/api-server01.key
ls -lh certs/server/db-primary01/db-primary01.key
```

**Note** *All CNSA 2.0 certificates use the same ~4.9KB ML-DSA-87 private key — roughly 15× the size of a P-256 ECDSA key. This size inflation is a core operational consideration for CNSA 2.0 migration and something to measure against your MTU, cert chain, and handshake budgets.*

## Step 10: View Certificate Details

### Display  certificate information

For the web server certificate:

```bash
openssl x509 -in certs/server/webserver01/webserver01.crt -provider oqsprovider -provider default -noout -text | less
```

Check specific fields:

```bash
# Subject
openssl x509 -in certs/server/webserver01/webserver01.crt -provider oqsprovider -provider default -noout -subject

# Issuer
openssl x509 -in certs/server/webserver01/webserver01.crt -provider oqsprovider -provider default -noout -issuer

# Validity dates
openssl x509 -in certs/server/webserver01/webserver01.crt -provider oqsprovider -provider default -noout -dates

# Serial number
openssl x509 -in certs/server/webserver01/webserver01.crt -provider oqsprovider -provider default -noout -serial

# Signature algorithm (should show mldsa87)
openssl x509 -in certs/server/webserver01/webserver01.crt -provider oqsprovider -provider default -noout -text | grep "Signature Algorithm" | head -1

# Subject Alternative Names
openssl x509 -in certs/server/webserver01/webserver01.crt -provider oqsprovider -provider default -noout -text | grep -A10 "Subject Alternative Name"

# Key Usage
openssl x509 -in certs/server/webserver01/webserver01.crt -provider oqsprovider -provider default -noout -text | grep -A2 "Key Usage"

# Extended Key Usage
openssl x509 -in certs/server/webserver01/webserver01.crt -provider oqsprovider -provider default -noout -text | grep -A2 "Extended Key Usage"
```

## Step 11: Export Certificates for Different Platforms

### Export to DER format (for Windows)

```bash
cd /opt/sassycorp-ca/intermediate-ca/certs/server/webserver01

openssl x509 -in webserver01.crt -outform DER -out webserver01.der
chmod 444 webserver01.der
```

### Export public key only

```bash
openssl x509 -in webserver01.crt -provider oqsprovider -provider default -pubkey -noout > webserver01-pubkey.pem
chmod 444 webserver01-pubkey.pem
```

## Step 12: Create Certificate Inventory

View all issued certificates:

```bash
cd /opt/sassycorp-ca/intermediate-ca

# Display certificate database
cat index.txt
```

Count certificates by status:

```bash
# Count valid certificates
grep -c "^V" index.txt

# Display certificate details from database
awk -F'\t' '{print "Status: "$1", Serial: "$4", Subject: "$6}' index.txt
```

Create a summary of issued certificates:

```bash
echo "=== CNSA 2.0 Compliant Certificate Inventory ===" > certificate-inventory.txt
echo "Generated: $(date)" >> certificate-inventory.txt
echo "" >> certificate-inventory.txt
echo "Valid Certificates:" >> certificate-inventory.txt
grep "^V" index.txt | awk -F'\t' '{print "  Serial: "$4" - "$6}' >> certificate-inventory.txt
echo "" >> certificate-inventory.txt
echo "Total issued: $(wc -l < index.txt)" >> certificate-inventory.txt

cat certificate-inventory.txt
```

## Testing and Validation

### Verify all certificates validate against the chain

```bash
cd /opt/sassycorp-ca/intermediate-ca

for cert in $(find certs -name "*.crt" -type f); do
    echo -n "Verifying $(basename $cert): "
    if openssl verify -provider oqsprovider -provider default -CAfile certs/ca-chain.crt "$cert" > /dev/null 2>&1; then
        echo "✓ Valid"
    else
        echo "✗ Invalid"
    fi
done
```

### Verify CNSA 2.0 compliant algorithms are used

```bash
echo "=== CNSA 2.0 Algorithm Verification ==="
for cert in $(find certs -name "*.crt" -type f); do
    ALGO=$(openssl x509 -in "$cert" -provider oqsprovider -provider default -noout -text | grep "Signature Algorithm" | head -1 | awk '{print $3}')
    echo -n "$(basename $cert): $ALGO"
    if [[ "$ALGO" == "mldsa87" ]]; then
        echo " ✓ CNSA 2.0 Compliant"
    else
        echo " ✗ Not CNSA 2.0 Compliant"
    fi
done
```

### Verify SANs are properly configured

```bash
for cert in $(find certs -name "*.crt" -type f | head -5); do
    echo "=== $(basename $cert) ==="
    openssl x509 -in "$cert" -provider oqsprovider -provider default -noout -text | grep -A5 "Subject Alternative Name" | grep -E "DNS:|email:|IP:|URI:"
    echo ""
done
```

Check permissions:

```bash
find /opt/sassycorp-ca/intermediate-ca/certs -name "*.key" -exec ls -la {} \;
find /opt/sassycorp-ca/intermediate-ca/certs -name "*.crt" -exec ls -la {} \;
```

## CNSA 2.0 Compliance Summary

All certificates created in this module use:

- ✅ **ML-DSA-87 (mldsa87)** for all certificates in the hierarchy per draft-jenkins-cnsa2-pkix-profile §4
- ✅ **ML-DSA-87 (mldsa87)** for highest security requirements  
- ✅ **SHA-512** for all hashing operations
- ✅ **Proper Subject Alternative Names** per RFC specifications
- ✅ **Appropriate key usage extensions**

These certificates are fully compliant with NSA's CNSA 2.0 requirements for quantum-resistant cryptography.

## Troubleshooting

### Certificate Validation Fails

Check the certificate chain:

```bash
openssl verify -provider oqsprovider -provider default -verbose -CAfile certs/ca-chain.crt [certificate.crt]
```

### Wrong Algorithm Error

Verify OQS provider is loaded:

```bash
openssl list -providers -provider oqsprovider
openssl list -signature-algorithms -provider oqsprovider | grep -E "mldsa87"
```

### Subject Alternative Names Not Showing

Ensure the configuration file includes the v3_req extension:

```bash
grep -A5 "v3_req" [config-file.cnf]
```

## Summary

You have successfully learned how to:

- Generate CNSA 2.0 compliant certificates using ML-DSA-87 (mldsa87)
- Create certificates for different purposes (server, user, critical infrastructure)
- Understand why CNSA 2.0 mandates a single parameter set across the entire PKI hierarchy
- Export certificates in multiple formats for different platforms
- Verify certificate compliance and validity
- Manage certificate inventory and tracking

Each certificate you created includes:

- CNSA 2.0 compliant quantum-resistant signature algorithms
- Appropriate key usage extensions
- CRL distribution points
- OCSP responder information

---

**Next**: [Certificate Revocation →](05_quantum_ca_revocation.md)
