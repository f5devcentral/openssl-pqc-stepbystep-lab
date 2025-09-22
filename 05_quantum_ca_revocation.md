# Module 5: Certificate Revocation with OCSP and CRL

## Overview

This module covers implementing certificate revocation using both Certificate Revocation Lists (CRL) and Online Certificate Status Protocol (OCSP) for the quantum-resistant CA infrastructure. You'll learn to revoke certificates and manage revocation lists manually.

## Part 1: Certificate Revocation List (CRL)

### Step 1: Configure CRL Distribution

Switch to the CA admin user:

```bash
sudo su - caadmin
cd /opt/sassycorp-ca
```

Create CRL distribution directories:

```bash
mkdir -p /opt/sassycorp-ca/crl-dist/{root,intermediate}
chmod 755 /opt/sassycorp-ca/crl-dist
chmod 755 /opt/sassycorp-ca/crl-dist/{root,intermediate}
```

### Step 2: Update CRLs

Update the Root CA CRL:

```bash
cd /opt/sassycorp-ca/root-ca

openssl ca -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -gencrl \
    -out crl/root-ca.crl
```

Copy to distribution point:

```bash
cp crl/root-ca.crl /opt/sassycorp-ca/crl-dist/root/
chmod 644 /opt/sassycorp-ca/crl-dist/root/root-ca.crl
```

Convert to DER format for compatibility:

```bash
openssl crl -in crl/root-ca.crl -outform DER -out /opt/sassycorp-ca/crl-dist/root/root-ca.der
chmod 644 /opt/sassycorp-ca/crl-dist/root/root-ca.der
```

Update the Intermediate CA CRL:

```bash
cd /opt/sassycorp-ca/intermediate-ca

openssl ca -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -gencrl \
    -out crl/intermediate-ca.crl
```

Copy to distribution point:

```bash
cp crl/intermediate-ca.crl /opt/sassycorp-ca/crl-dist/intermediate/
chmod 644 /opt/sassycorp-ca/crl-dist/intermediate/intermediate-ca.crl
```

Convert to DER format:

```bash
openssl crl -in crl/intermediate-ca.crl -outform DER -out /opt/sassycorp-ca/crl-dist/intermediate/intermediate-ca.der
chmod 644 /opt/sassycorp-ca/crl-dist/intermediate/intermediate-ca.der
```

### Step 3: View CRL Information

Display Root CA CRL info:

```bash
openssl crl -in /opt/sassycorp-ca/crl-dist/root/root-ca.crl -noout -text | grep -E "Last Update|Next Update|Revoked Certificates" | head -3
```

Display Intermediate CA CRL info:

```bash
openssl crl -in /opt/sassycorp-ca/crl-dist/intermediate/intermediate-ca.crl -noout -text | grep -E "Last Update|Next Update|Revoked Certificates" | head -3
```

## Part 2: Revoking Certificates

### Step 4: Create a Test Certificate to Revoke

First, create a test certificate that we'll revoke:

```bash
cd /opt/sassycorp-ca/intermediate-ca
mkdir -p certs/test/revoke-test
cd certs/test/revoke-test
```

Generate a key using CNSA 2.0 compliant ML-DSA-65 (mldsa65):

```bash
openssl genpkey \
    -provider oqsprovider \
    -provider default \
    -algorithm mldsa65 \
    -out revoke-test.key

chmod 400 revoke-test.key
```

Create a simple configuration:

```bash
nano revoke-test.cnf
```

Add:

```ini
[ req ]
distinguished_name = req_distinguished_name
prompt = no

[ req_distinguished_name ]
C = US
ST = Washington
L = Glacier
O = SassyCorp
OU = Testing
CN = revoke-test.sassycorp.lab
emailAddress = test@sassycorp.internal
```

Save and create the CSR:

```bash
openssl req -config revoke-test.cnf \
    -provider oqsprovider \
    -provider default \
    -new \
    -key revoke-test.key \
    -out revoke-test.csr
```

Sign the certificate:

```bash
cd /opt/sassycorp-ca/intermediate-ca

openssl ca -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -extensions server_cert \
    -days 30 \
    -notext \
    -in certs/test/revoke-test/revoke-test.csr \
    -out certs/test/revoke-test/revoke-test.crt
```

Get the serial number of the certificate:

```bash
openssl x509 -in certs/test/revoke-test/revoke-test.crt -noout -serial
```

Note this serial number (e.g., 1004) for the next step.

### Step 5: Revoke the Certificate

Check the certificate status before revocation:

```bash
openssl ca -config openssl.cnf -status 1004
```

(Replace 1004 with your actual serial number)

Revoke the certificate with a reason:

```bash
cd /opt/sassycorp-ca/intermediate-ca

openssl ca -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -revoke newcerts/1004.pem \
    -crl_reason keyCompromise
```

Type 'y' when prompted to confirm revocation.

### Revocation Reasons

Valid revocation reasons are:
- `unspecified` - No specific reason (0)
- `keyCompromise` - Private key compromised (1)
- `CACompromise` - CA key compromised (2)
- `affiliationChanged` - Subject's affiliation changed (3)
- `superseded` - Certificate superseded (4)
- `cessationOfOperation` - Operation ceased (5)
- `certificateHold` - Certificate on hold (6)
- `removeFromCRL` - Remove from CRL (8)
- `privilegeWithdrawn` - Privilege withdrawn (9)
- `AACompromise` - AA compromise (10)

### Step 6: Update CRL After Revocation

Generate a new CRL with the revoked certificate:

```bash
openssl ca -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -gencrl \
    -out crl/intermediate-ca.crl
```

Copy to distribution point:

```bash
cp crl/intermediate-ca.crl /opt/sassycorp-ca/crl-dist/intermediate/
```

View the updated CRL to confirm revocation:

```bash
openssl crl -in crl/intermediate-ca.crl -noout -text | grep -A5 "Revoked Certificates"
```

### Step 7: Verify Certificate is Revoked

Test with CRL checking:

```bash
openssl verify -crl_check \
    -CAfile certs/ca-chain.crt \
    -CRLfile crl/intermediate-ca.crl \
    certs/test/revoke-test/revoke-test.crt
```

You should see an error indicating the certificate is revoked.

Check the certificate status:

```bash
openssl ca -config openssl.cnf -status 1004
```

The status should now show 'R' for revoked.

### Step 8: Create Revocation Log

Create a revocation log file:

```bash
touch /opt/sassycorp-ca/revocation.log
chmod 644 /opt/sassycorp-ca/revocation.log
```

Add a revocation entry:

```bash
echo "$(date +%Y%m%d_%H%M%S)|intermediate|1004|keyCompromise|$(whoami)" >> /opt/sassycorp-ca/revocation.log
```

View the log:

```bash
cat /opt/sassycorp-ca/revocation.log
```

## Part 3: OCSP Implementation

### Step 9: Configure OCSP Responder

Create OCSP directories:

```bash
mkdir -p /opt/sassycorp-ca/ocsp/{db,certs,logs}
chmod 755 /opt/sassycorp-ca/ocsp/{db,certs,logs}
```

Create OCSP responder configuration:

```bash
nano /opt/sassycorp-ca/ocsp/ocsp-responder.cnf
```

Add:

```ini
# OCSP Responder Configuration
# SassyCorp Quantum-Resistant CA

[ ocsp ]
default_ocsp = ocsp_config

[ ocsp_config ]
# Intermediate CA configuration
dir = /opt/sassycorp-ca/intermediate-ca
private_key = $dir/private/ocsp.key
certificate = $dir/certs/ocsp.crt
ca_certificate = $dir/certs/intermediate-ca.crt
other_certificates = $dir/certs/ca-chain.crt

# Database location
database = $dir/index.txt

# Response validity
ndays = 7
nmin = 5

# Response extensions
response_cert_ext = ocsp_cert_ext

[ ocsp_cert_ext ]
basicConstraints = CA:FALSE
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
extendedKeyUsage = OCSPSigning
```

Save and set permissions:

```bash
chmod 644 /opt/sassycorp-ca/ocsp/ocsp-responder.cnf
```

### Step 10: Start OCSP Responder

Check if the OCSP certificate exists:

```bash
ls -la /opt/sassycorp-ca/intermediate-ca/certs/ocsp.crt
```

Start the OCSP responder manually (run in a separate terminal or background):

```bash
cd /opt/sassycorp-ca/intermediate-ca

openssl ocsp \
    -provider oqsprovider \
    -provider default \
    -port 2560 \
    -index index.txt \
    -CA certs/ca-chain.crt \
    -rkey private/ocsp.key \
    -rsigner certs/ocsp.crt \
    -text
```

To run in background:

```bash
nohup openssl ocsp \
    -provider oqsprovider \
    -provider default \
    -port 2560 \
    -index index.txt \
    -CA certs/ca-chain.crt \
    -rkey private/ocsp.key \
    -rsigner certs/ocsp.crt \
    -text > /opt/sassycorp-ca/ocsp/logs/ocsp.log 2>&1 &
```

Get the process ID:

```bash
echo $!
```

### Step 11: Test OCSP Response

In another terminal, test the OCSP responder with a valid certificate:

```bash
cd /opt/sassycorp-ca/intermediate-ca

# Test with a valid certificate
openssl ocsp \
    -provider oqsprovider \
    -provider default \
    -CAfile certs/ca-chain.crt \
    -issuer certs/intermediate-ca.crt \
    -cert certs/server/webserver01/webserver01.crt \
    -url http://localhost:2560 \
    -resp_text
```

You should see "Response verify OK" and the certificate status as "good".

Test with the revoked certificate:

```bash
openssl ocsp \
    -provider oqsprovider \
    -provider default \
    -CAfile certs/ca-chain.crt \
    -issuer certs/intermediate-ca.crt \
    -cert certs/test/revoke-test/revoke-test.crt \
    -url http://localhost:2560 \
    -resp_text
```

You should see the certificate status as "revoked" with the revocation reason.

### Step 12: Create OCSP Request and Response Files

Create an OCSP request:

```bash
openssl ocsp \
    -provider oqsprovider \
    -provider default \
    -CAfile certs/ca-chain.crt \
    -issuer certs/intermediate-ca.crt \
    -cert certs/server/webserver01/webserver01.crt \
    -reqout /opt/sassycorp-ca/ocsp/requests/webserver01.req
```

Send the request and save the response:

```bash
openssl ocsp \
    -provider oqsprovider \
    -provider default \
    -reqin /opt/sassycorp-ca/ocsp/requests/webserver01.req \
    -url http://localhost:2560 \
    -respout /opt/sassycorp-ca/ocsp/requests/webserver01.resp
```

Verify the response:

```bash
openssl ocsp \
    -provider oqsprovider \
    -provider default \
    -respin /opt/sassycorp-ca/ocsp/requests/webserver01.resp \
    -CAfile certs/ca-chain.crt \
    -resp_text
```

## Part 4: OCSP Stapling Configuration Examples

### Step 13: Create OCSP Stapling Examples

Create an Nginx configuration example:

```bash
nano /opt/sassycorp-ca/ocsp-stapling-nginx.conf
```

Add:

```nginx
# Nginx OCSP Stapling Configuration Example
# For use with SassyCorp quantum-resistant certificates

server {
    listen 443 ssl http2;
    server_name example.sassycorp.lab;

    # Certificate and key paths
    ssl_certificate /path/to/certificate.crt;
    ssl_certificate_key /path/to/private.key;
    
    # Intermediate certificates for chain
    ssl_trusted_certificate /opt/sassycorp-ca/intermediate-ca/certs/ca-chain.crt;

    # Enable OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;
    
    # OCSP responder URL (override if needed)
    # ssl_stapling_responder http://ocsp.sassycorp.lab:2560/;
    
    # DNS resolver for OCSP queries
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;

    # Other SSL settings
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_protocols TLSv1.3;
    ssl_prefer_server_ciphers off;
}
```

Save and create an Apache example:

```bash
nano /opt/sassycorp-ca/ocsp-stapling-apache.conf
```

Add:

```apache
# Apache OCSP Stapling Configuration Example
# For use with SassyCorp quantum-resistant certificates

<VirtualHost *:443>
    ServerName example.sassycorp.lab
    
    # Certificate configuration
    SSLCertificateFile /path/to/certificate.crt
    SSLCertificateKeyFile /path/to/private.key
    SSLCertificateChainFile /opt/sassycorp-ca/intermediate-ca/certs/ca-chain.crt
    
    # Enable OCSP stapling
    SSLUseStapling On
    SSLStaplingCache shmcb:/tmp/stapling_cache(128000)
    
    # OCSP responder URL (if not in certificate)
    # SSLStaplingResponderURI http://ocsp.sassycorp.lab:2560/
    
    # OCSP response max age (1 hour)
    SSLStaplingResponseMaxAge 3600
    
    # OCSP response time limit
    SSLStaplingResponseTimeSkew 300
    
    # Return OCSP response errors to client
    SSLStaplingReturnResponderErrors On
    
    # Other SSL settings
    SSLEngine On
    SSLProtocol -all +TLSv1.3
</VirtualHost>
```

Save both files with appropriate permissions:

```bash
chmod 644 /opt/sassycorp-ca/ocsp-stapling-*.conf
```

## Part 5: Testing and Validation

### Step 14: Comprehensive Revocation Testing

Test CRL accessibility:

```bash
ls -la /opt/sassycorp-ca/crl-dist/root/root-ca.crl
ls -la /opt/sassycorp-ca/crl-dist/intermediate/intermediate-ca.crl
```

Check CRL validity:

```bash
# Check Root CA CRL
openssl crl -in /opt/sassycorp-ca/crl-dist/root/root-ca.crl -noout -nextupdate

# Check Intermediate CA CRL
openssl crl -in /opt/sassycorp-ca/crl-dist/intermediate/intermediate-ca.crl -noout -nextupdate
```

Verify OCSP responder is running:

```bash
ps aux | grep "openssl ocsp"
```

Check if port 2560 is listening:

```bash
netstat -tln | grep 2560
```

### Step 15: Test Multiple Certificate Revocations

Create another test certificate:

```bash
cd /opt/sassycorp-ca/intermediate-ca
mkdir -p certs/test/revoke-test2
cd certs/test/revoke-test2

# Generate key using CNSA 2.0 compliant ML-DSA-65 (mldsa65)
openssl genpkey \
    -provider oqsprovider \
    -provider default \
    -algorithm mldsa65 \
    -out revoke-test2.key

chmod 400 revoke-test2.key

# Create CSR with simple subject
openssl req \
    -provider oqsprovider \
    -provider default \
    -new \
    -key revoke-test2.key \
    -out revoke-test2.csr \
    -subj "/C=US/ST=Washington/L=Glacier/O=SassyCorp/OU=Testing/CN=revoke-test2.sassycorp.lab"

# Sign certificate
cd /opt/sassycorp-ca/intermediate-ca

openssl ca -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -extensions server_cert \
    -days 30 \
    -notext \
    -in certs/test/revoke-test2/revoke-test2.csr \
    -out certs/test/revoke-test2/revoke-test2.crt
```

Get the serial number:

```bash
openssl x509 -in certs/test/revoke-test2/revoke-test2.crt -noout -serial
```

Revoke with a different reason:

```bash
openssl ca -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -revoke newcerts/1005.pem \
    -crl_reason superseded
```

(Replace 1005 with your actual serial number)

Update the CRL:

```bash
openssl ca -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -gencrl \
    -out crl/intermediate-ca.crl
```

### Step 16: View Revocation Database

Check the certificate database to see revocation status:

```bash
cd /opt/sassycorp-ca/intermediate-ca
cat index.txt
```

The format is:
- First column: V (Valid) or R (Revoked)
- Second column: Expiry date
- Third column: Revocation date (if revoked)
- Fourth column: Serial number
- Fifth column: Unknown
- Sixth column: Subject DN

Count certificates by status:

```bash
echo "Valid certificates: $(grep -c '^V' index.txt)"
echo "Revoked certificates: $(grep -c '^R' index.txt)"
```

View only revoked certificates:

```bash
grep '^R' index.txt | awk -F'\t' '{print "Serial: "$4", Revoked: "$3", Subject: "$6}'
```

### Step 17: Generate Revocation Report

Create a comprehensive revocation report:

```bash
cd /opt/sassycorp-ca

echo "=== SassyCorp CA Revocation Report ===" > revocation-report.txt
echo "Generated: $(date)" >> revocation-report.txt
echo "" >> revocation-report.txt

echo "=== Root CA CRL Status ===" >> revocation-report.txt
openssl crl -in root-ca/crl/root-ca.crl -noout -text | \
    grep -E "Last Update|Next Update" >> revocation-report.txt
echo "" >> revocation-report.txt

echo "=== Intermediate CA CRL Status ===" >> revocation-report.txt
openssl crl -in intermediate-ca/crl/intermediate-ca.crl -noout -text | \
    grep -E "Last Update|Next Update" >> revocation-report.txt
echo "" >> revocation-report.txt

echo "=== Revoked Certificates ===" >> revocation-report.txt
if [ -f intermediate-ca/index.txt ]; then
    grep "^R" intermediate-ca/index.txt | \
        awk -F'\t' '{print "Serial: "$4" | Revoked: "$3" | Reason: "$6}' >> revocation-report.txt
fi
echo "" >> revocation-report.txt

echo "=== Statistics ===" >> revocation-report.txt
TOTAL_ISSUED=$(wc -l < intermediate-ca/index.txt)
TOTAL_VALID=$(grep -c "^V" intermediate-ca/index.txt || echo "0")
TOTAL_REVOKED=$(grep -c "^R" intermediate-ca/index.txt || echo "0")

echo "Total Issued: $TOTAL_ISSUED" >> revocation-report.txt
echo "Currently Valid: $TOTAL_VALID" >> revocation-report.txt
echo "Total Revoked: $TOTAL_REVOKED" >> revocation-report.txt

cat revocation-report.txt
```

### Step 18: Test OCSP with Different Certificates

Test OCSP for each type of certificate you created:

```bash
cd /opt/sassycorp-ca/intermediate-ca

# Test web server certificate
echo "Testing webserver01 certificate:"
openssl ocsp \
    -provider oqsprovider \
    -provider default \
    -CAfile certs/ca-chain.crt \
    -issuer certs/intermediate-ca.crt \
    -cert certs/server/webserver01/webserver01.crt \
    -url http://localhost:2560 | grep "webserver01"

# Test user certificate
echo "Testing jane.smith certificate:"
openssl ocsp \
    -provider oqsprovider \
    -provider default \
    -CAfile certs/ca-chain.crt \
    -issuer certs/intermediate-ca.crt \
    -cert certs/users/jane.smith/jane.smith.crt \
    -url http://localhost:2560 | grep "jane.smith"

# Test revoked certificate
echo "Testing revoked certificate:"
openssl ocsp \
    -provider oqsprovider \
    -provider default \
    -CAfile certs/ca-chain.crt \
    -issuer certs/intermediate-ca.crt \
    -cert certs/test/revoke-test/revoke-test.crt \
    -url http://localhost:2560 | grep "revoke-test"
```

### Step 19: Clean Up Test Certificates

Remove test certificates if desired:

```bash
cd /opt/sassycorp-ca/intermediate-ca
rm -rf certs/test/
```

Note: The revocation information remains in the database even after removing the certificate files.

### Step 20: Stop OCSP Responder

If you started the OCSP responder in the background, find and stop it:

```bash
# Find the process
ps aux | grep "openssl ocsp" | grep -v grep

# Kill the process (replace PID with actual process ID)
kill [PID]
```

## Security Checklist

Verify the following for revocation infrastructure:

- [ ] CRLs are generated and accessible
- [ ] CRLs have appropriate validity periods (30 days)
- [ ] OCSP responder certificate is properly configured
- [ ] OCSP responder has OCSPSigning extended key usage
- [ ] Revocation reasons are properly recorded
- [ ] CRL distribution points are accessible
- [ ] OCSP responder responds correctly to queries
- [ ] Revocation logs are maintained
- [ ] Database reflects accurate certificate status
- [ ] Revoked certificates fail validation

Verify permissions:

```bash
ls -la /opt/sassycorp-ca/crl-dist/
ls -la /opt/sassycorp-ca/intermediate-ca/crl/
ls -la /opt/sassycorp-ca/intermediate-ca/private/ocsp.key
ls -la /opt/sassycorp-ca/intermediate-ca/certs/ocsp.crt
```

## Troubleshooting

### OCSP Responder Won't Start

Check for port conflicts:

```bash
sudo netstat -tlnp | grep 2560
```

Verify OCSP certificate exists and has correct extension:

```bash
ls -la /opt/sassycorp-ca/intermediate-ca/certs/ocsp.crt
openssl x509 -in /opt/sassycorp-ca/intermediate-ca/certs/ocsp.crt -noout -text | grep -A1 "Extended Key Usage"
```

### CRL Verification Fails

Verify CRL is properly signed:

```bash
openssl crl -in /opt/sassycorp-ca/intermediate-ca/crl/intermediate-ca.crl -noout -verify \
    -CAfile /opt/sassycorp-ca/intermediate-ca/certs/ca-chain.crt
```

### OCSP Response Invalid

Check that the OCSP responder can access the database:

```bash
ls -la /opt/sassycorp-ca/intermediate-ca/index.txt
cat /opt/sassycorp-ca/intermediate-ca/index.txt
```

Check OCSP responder logs:

```bash
tail -f /opt/sassycorp-ca/ocsp/logs/ocsp.log
```

### Certificate Still Shows as Valid After Revocation

Ensure you've updated the CRL after revocation:

```bash
cd /opt/sassycorp-ca/intermediate-ca
openssl ca -config openssl.cnf \
    -provider oqsprovider \
    -provider default \
    -gencrl \
    -out crl/intermediate-ca.crl
```

Verify the certificate appears in the CRL:

```bash
openssl crl -in crl/intermediate-ca.crl -noout -text | grep -A20 "Revoked Certificates"
```

## Manual Revocation Process Summary

The complete manual process for revoking a certificate:

1. Identify the certificate serial number:
   ```bash
   openssl x509 -in [certificate.crt] -noout -serial
   ```

2. Revoke the certificate:
   ```bash
   openssl ca -config openssl.cnf \
       -provider oqsprovider \
       -provider default \
       -revoke newcerts/[SERIAL].pem \
       -crl_reason [reason]
   ```

3. Update the CRL:
   ```bash
   openssl ca -config openssl.cnf \
       -provider oqsprovider \
       -provider default \
       -gencrl \
       -out crl/intermediate-ca.crl
   ```

4. Copy CRL to distribution point:
   ```bash
   cp crl/intermediate-ca.crl /opt/sassycorp-ca/crl-dist/intermediate/
   ```

5. Log the revocation:
   ```bash
   echo "$(date +%Y%m%d_%H%M%S)|intermediate|[SERIAL]|[reason]|$(whoami)" >> /opt/sassycorp-ca/revocation.log
   ```

6. Verify revocation with CRL:
   ```bash
   openssl verify -crl_check \
       -CAfile certs/ca-chain.crt \
       -CRLfile crl/intermediate-ca.crl \
       [certificate.crt]
   ```

7. Verify with OCSP (if running):
   ```bash
   openssl ocsp \
       -provider oqsprovider \
       -provider default \
       -CAfile certs/ca-chain.crt \
       -issuer certs/intermediate-ca.crt \
       -cert [certificate.crt] \
       -url http://localhost:2560
   ```

## Summary

You have successfully implemented:
- ✅ Certificate Revocation Lists (CRL) with distribution points
- ✅ Manual certificate revocation with reason codes
- ✅ Online Certificate Status Protocol (OCSP) responder
- ✅ CRL generation and updates
- ✅ OCSP testing and verification
- ✅ Revocation logging and reporting
- ✅ OCSP stapling configuration examples
- ✅ Complete quantum-resistant CA with revocation features

## Complete Lab Summary

Congratulations! You have built a complete CNSA 2.0 compliant quantum-resistant Certificate Authority infrastructure that includes:

1. **Root CA** - 10-year validity with ML-DSA-87 (mldsa87) - CNSA 2.0 compliant
2. **Intermediate CA** - 5-year validity with ML-DSA-65 (mldsa65) - CNSA 2.0 compliant
3. **End-Entity Certificates** - Using ML-DSA-65 (mldsa65) and ML-DSA-87 (mldsa87) - CNSA 2.0 compliant
4. **Revocation Infrastructure** - Both CRL and OCSP with quantum-resistant signatures
5. **Security Best Practices** - Proper Unix permissions throughout
6. **RFC Compliance** - Proper SAN configuration
7. **Manual Processes** - Direct command-line control for learning

### CNSA 2.0 Compliance Achievement

Your PKI infrastructure is fully compliant with NSA's Commercial National Security Algorithm Suite 2.0:
- ✅ Uses only ML-DSA-65 (mldsa65) and ML-DSA-87 (mldsa87) for signatures
- ✅ Implements SHA-512 for all hashing operations
- ✅ Ready for post-quantum cryptography requirements
- ✅ Suitable for protecting classified information up to TOP SECRET level

### Key Commands Learned

You've mastered these essential OpenSSL commands:
- `openssl genpkey` - Generate quantum-resistant private keys
- `openssl req` - Create certificate signing requests
- `openssl ca` - Sign certificates and manage CA operations
- `openssl x509` - Examine and verify certificates
- `openssl crl` - Manage certificate revocation lists
- `openssl ocsp` - Query and respond to OCSP requests
- `openssl verify` - Validate certificate chains

### What You've Accomplished

- Created a complete PKI hierarchy with quantum-resistant algorithms
- Configured proper Subject Alternative Names (DNS, IP, email, URI)
- Implemented both CRL and OCSP revocation mechanisms
- Applied secure Unix file permissions throughout
- Learned to manually manage all aspects of a CA

This lab environment is ready for internal testing of quantum-resistant PKI infrastructure in preparation for the post-quantum cryptography era.

---

**[← Return to Introduction](01_quantum_ca_intro.md)**