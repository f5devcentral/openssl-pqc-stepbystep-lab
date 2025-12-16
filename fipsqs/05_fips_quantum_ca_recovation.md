# Module 5: Certificate Revocation

## Overview

This module guides you through implementing certificate revocation for Sassy Corp's PKI using both Certificate Revocation Lists (CRLs) and the Online Certificate Status Protocol (OCSP). These mechanisms allow you to invalidate certificates before their expiration date when needed.

---

## Learning Objectives

After completing this module, you will be able to:

- Revoke certificates and update CRLs
- Configure and run an OCSP responder
- Query certificate status using both CRL and OCSP
- Understand the differences between revocation mechanisms
- Implement revocation checking in verification workflows

---

## Revocation Mechanisms Compared

| Feature | CRL | OCSP |
|---------|-----|------|
| Update Frequency | Periodic (hours/days) | Real-time |
| Data Transfer | Download full list | Query single certificate |
| Privacy | Higher (no query sent) | Lower (server sees queries) |
| Scalability | Less scalable | More scalable |
| Offline Support | Yes (cached CRL) | Requires network |
| Freshness | May be stale | Always current |

---

## Part A: Certificate Revocation Lists (CRLs)

### Step 1: Ensure Correct User Context

```bash
sudo su - pqcadmin
```

**Navigate to the Intermediate CA directory:**

```bash
cd /opt/sassycorp-pqc/intermediate-ca
```

---

### Step 2: Create a Test Certificate to Revoke

**Generate a test certificate that we will revoke:**

```bash
openssl genpkey -algorithm ML-DSA-44 -out /opt/sassycorp-pqc/requests/revoke-test.key
```

```bash
chmod 400 /opt/sassycorp-pqc/requests/revoke-test.key
```

**Create a CSR:**

```bash
openssl req -new \
    -key /opt/sassycorp-pqc/requests/revoke-test.key \
    -subj "/C=US/ST=Washington/L=Winthrop/O=Sassy Corp/OU=Testing/CN=Revocation Test Certificate/emailAddress=test@sassycorp.internal" \
    -out /opt/sassycorp-pqc/requests/revoke-test.csr
```

**Sign the certificate:**

```bash
openssl ca -config /opt/sassycorp-pqc/intermediate-ca/openssl.cnf \
    -extensions usr_cert \
    -days 365 \
    -notext \
    -in /opt/sassycorp-pqc/requests/revoke-test.csr \
    -out /opt/sassycorp-pqc/requests/revoke-test.crt
```

Confirm the signing when prompted.

**Get the serial number:**

```bash
openssl x509 -in /opt/sassycorp-pqc/requests/revoke-test.crt -noout -serial
```

Note this serial number for later verification.

---

### Step 3: Verify Certificate Before Revocation

Verify the certificate is currently valid:

```bash
openssl verify -CAfile /opt/sassycorp-pqc/intermediate-ca/certs/ca-chain.crt /opt/sassycorp-pqc/requests/revoke-test.crt
```

**Expected output:**

```
/opt/sassycorp-pqc/requests/revoke-test.crt: OK
```

Check the certificate database:

```bash
grep "Revocation Test" /opt/sassycorp-pqc/intermediate-ca/index.txt
```

**Expected output (V = Valid):**

```
V	<expiry>		<serial>	unknown	/C=US/ST=Washington/L=Winthrop/O=Sassy Corp/OU=Testing/CN=Revocation Test Certificate/emailAddress=test@sassycorp.internal
```

---

### Step 4: Revoke the Certificate

**Revoke the certificate:**

```bash
openssl ca -config /opt/sassycorp-pqc/intermediate-ca/openssl.cnf \
    -revoke /opt/sassycorp-pqc/requests/revoke-test.crt \
    -crl_reason keyCompromise
```

**Command options:**

| Option | Purpose |
|--------|---------|
| `-revoke` | Certificate file to revoke |
| `-crl_reason` | Reason for revocation |

**Available CRL reasons:**
- `unspecified` - No reason given
- `keyCompromise` - Private key compromised
- `CACompromise` - CA key compromised
- `affiliationChanged` - Subject information changed
- `superseded` - Replaced by new certificate
- `cessationOfOperation` - No longer needed
- `certificateHold` - Temporarily suspended
- `removeFromCRL` - Remove hold status

Confirm the revocation when prompted.

---

### Step 5: Verify Database Update

**Check the database shows revoked status:**

```bash
grep "Revocation Test" /opt/sassycorp-pqc/intermediate-ca/index.txt
```

**Expected output (R = Revoked):**

```
R	<expiry>	<revocation date>	<serial>	unknown	/C=US/ST=Washington/L=Winthrop/O=Sassy Corp/OU=Testing/CN=Revocation Test Certificate/emailAddress=test@sassycorp.internal
```

---

### Step 6: Generate Updated CRL

**Generate a new CRL containing the revoked certificate:**

```bash
openssl ca -config /opt/sassycorp-pqc/intermediate-ca/openssl.cnf \
    -gencrl \
    -out /opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.crl
```

**Set permissions:**

```bash
chmod 644 /opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.crl
```

---

### Step 7: Verify the CRL

**View the CRL contents:**

```bash
openssl crl -in /opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.crl -noout -text
```

**Expected output includes:**

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
Revoked Certificates:
    Serial Number: <serial>
        Revocation Date: <date>
        CRL entry extensions:
            X509v3 CRL Reason Code:
                Key Compromise
```

---

### Step 8: Verify Certificate Against CRL

**Create a combined file with CA chain and CRL for verification:**

```bash
cat /opt/sassycorp-pqc/intermediate-ca/certs/ca-chain.crt > /tmp/verify-bundle.pem
```

**Verify the revoked certificate with CRL checking:**

```bash
openssl verify -crl_check -CAfile /opt/sassycorp-pqc/intermediate-ca/certs/ca-chain.crt -CRLfile /opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.crl /opt/sassycorp-pqc/requests/revoke-test.crt
```

**Expected output:**

```
C=US, ST=Washington, L=Winthrop, O=Sassy Corp, OU=Testing, CN=Revocation Test Certificate, emailAddress=test@sassycorp.internal
error 23 at 0 depth lookup: certificate revoked
error /opt/sassycorp-pqc/requests/revoke-test.crt: verification failed
```

**Verify a valid certificate still works:**

```bash
openssl verify -crl_check -CAfile /opt/sassycorp-pqc/intermediate-ca/certs/ca-chain.crt -CRLfile /opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.crl /opt/sassycorp-pqc/requests/www.sassycorp.lab.crt
```

**Expected output:**

```
/opt/sassycorp-pqc/requests/www.sassycorp.lab.crt: OK
```

---

### Step 9: Convert CRL to DER Format

**Some systems require DER-encoded CRLs:**

```bash
openssl crl -in /opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.crl -outform DER -out /opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.der.crl
```

```bash
chmod 444 /opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.der.crl
```

---

## Part B: OCSP Responder

### Step 10: Verify OCSP Certificate

**Verify the OCSP responder certificate was created in Module 03:**

```bash
openssl x509 -in /opt/sassycorp-pqc/ocsp/certs/ocsp.crt -noout -text | grep -A2 "Extended Key Usage"
```

**Expected output:**

```
            X509v3 Extended Key Usage: critical
                OCSP Signing
```

---

### Step 11: Create OCSP Index File

**The OCSP responder needs access to the certificate database:**

```bash
cp /opt/sassycorp-pqc/intermediate-ca/index.txt /opt/sassycorp-pqc/ocsp/index.txt
```

> **Note:** In production, you would either share the database or implement a synchronization mechanism.

---

### Step 12: Start the OCSP Responder

**Start the OCSP responder in the foreground (for testing):**

```bash
openssl ocsp -index /opt/sassycorp-pqc/ocsp/index.txt \
    -port 8888 \
    -rsigner /opt/sassycorp-pqc/ocsp/certs/ocsp.crt \
    -rkey /opt/sassycorp-pqc/ocsp/private/ocsp.key \
    -CA /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt \
    -text \
    -out /opt/sassycorp-pqc/ocsp/logs/ocsp.log
```

**Command options:**

| Option | Purpose |
|--------|---------|
| `-index` | Certificate database file |
| `-port` | TCP port to listen on |
| `-rsigner` | OCSP responder certificate |
| `-rkey` | OCSP responder private key |
| `-CA` | CA certificate for verification |
| `-text` | Output text format responses |
| `-out` | Log file location |

The responder will now wait for connections. *Open a **new terminal** for the next steps.*

---

### Step 13: Query OCSP for Valid Certificate

**In a new terminal, switch to pqcadmin:**

```bash
sudo su - pqcadmin
```

**Query the OCSP responder for the server certificate:**

```bash
openssl ocsp -CAfile /opt/sassycorp-pqc/intermediate-ca/certs/ca-chain.crt \
    -issuer /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt \
    -cert /opt/sassycorp-pqc/requests/www.sassycorp.lab.crt \
    -url http://127.0.0.1:8888 \
    -resp_text
```

**Expected output includes:**

```
Response verify OK
/opt/sassycorp-pqc/requests/www.sassycorp.lab.crt: good
        This Update: <date>
```

---

### Step 14: Query OCSP for Revoked Certificate

**Query for the revoked certificate:**

```bash
openssl ocsp -CAfile /opt/sassycorp-pqc/intermediate-ca/certs/ca-chain.crt \
    -issuer /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt \
    -cert /opt/sassycorp-pqc/requests/revoke-test.crt \
    -url http://127.0.0.1:8888 \
    -resp_text
```

**Expected output includes:**

```
Response verify OK
/opt/sassycorp-pqc/requests/revoke-test.crt: revoked
        This Update: <date>
        Reason: keyCompromise
        Revocation Time: <date>
```

---

### Step 15: Stop the OCSP Responder

Return to the original terminal and press `Ctrl+C` to stop the OCSP responder.

---

### Step 16: Create OCSP Responder Systemd Service (Optional)

**For production use, create a systemd service file:**
**Note:** *Exit pqcadmin and return to root to create the service*

```bash
sudo vim /etc/systemd/system/sassycorp-ocsp.service
```

Enter the following:

```ini
[Unit]
Description=Sassy Corp OCSP Responder
After=network.target

[Service]
Type=simple
User=pqcadmin
Group=pqcadmin
ExecStart=/opt/openssl-3.5.3/bin/openssl ocsp \
    -index /opt/sassycorp-pqc/ocsp/index.txt \
    -port 8888 \
    -rsigner /opt/sassycorp-pqc/ocsp/certs/ocsp.crt \
    -rkey /opt/sassycorp-pqc/ocsp/private/ocsp.key \
    -CA /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt \
    -nrequest 100
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Save and exit.

**Enable and start the service:**

```bash
sudo systemctl daemon-reload
```

```bash
sudo systemctl enable sassycorp-ocsp
```

```bash
sudo systemctl start sassycorp-ocsp
```

Check status:

```bash
sudo systemctl status sassycorp-ocsp
```

---

## Part C: Additional Revocation Operations

### Step 17: View All Revoked Certificates

**List all revoked certificates:**

```bash
grep "^R" /opt/sassycorp-pqc/intermediate-ca/index.txt
```

---

### Step 18: Place Certificate on Hold

Certificate hold is a temporary revocation that can be reversed:

**First, create another test certificate:**

```bash
openssl genpkey -algorithm ML-DSA-44 -out /opt/sassycorp-pqc/requests/hold-test.key
```

```bash
chmod 400 /opt/sassycorp-pqc/requests/hold-test.key
```

```bash
openssl req -new \
    -key /opt/sassycorp-pqc/requests/hold-test.key \
    -subj "/C=US/ST=Washington/L=Winthrop/O=Sassy Corp/OU=Testing/CN=Certificate Hold Test/emailAddress=hold@sassycorp.internal" \
    -out /opt/sassycorp-pqc/requests/hold-test.csr
```

```bash
openssl ca -config /opt/sassycorp-pqc/intermediate-ca/openssl.cnf \
    -extensions usr_cert \
    -days 365 \
    -notext \
    -in /opt/sassycorp-pqc/requests/hold-test.csr \
    -out /opt/sassycorp-pqc/requests/hold-test.crt
```

**Place it on hold:**

```bash
openssl ca -config /opt/sassycorp-pqc/intermediate-ca/openssl.cnf \
    -revoke /opt/sassycorp-pqc/requests/hold-test.crt \
    -crl_reason certificateHold
```

**Check the database:**

```bash
grep "Hold Test" /opt/sassycorp-pqc/intermediate-ca/index.txt
```

---

### Step 19: Update CRL Number

**Check the current CRL number:**

```bash
cat /opt/sassycorp-pqc/intermediate-ca/crlnumber
```

**Generate a new CRL (this increments the CRL number):**

```bash
openssl ca -config /opt/sassycorp-pqc/intermediate-ca/openssl.cnf -gencrl -out /opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.crl
```

**Check the CRL number was incremented:**

```bash
cat /opt/sassycorp-pqc/intermediate-ca/crlnumber
```

---

### Step 20: Verify CRL Signature

**Verify the CRL is properly signed:**

```bash
openssl crl -in /opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.crl -CAfile /opt/sassycorp-pqc/intermediate-ca/certs/intermediate-ca.crt -verify -noout
```

**Expected output:**

```
verify OK
```

---

### Step 21: Get CRL Information

**Extract CRL metadata:**

```bash
openssl crl -in /opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.crl -noout -lastupdate -nextupdate
```

**Expected output:**

```
lastUpdate=<date>
nextUpdate=<date + 7 days>
```

**Count revoked certificates:**

```bash
openssl crl -in /opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.crl -noout -text | grep -c "Serial Number"
```

---

## Step 22: Synchronize OCSP Index

**If you made changes to the certificate database, update the OCSP index:**

```bash
cp /opt/sassycorp-pqc/intermediate-ca/index.txt /opt/sassycorp-pqc/ocsp/index.txt
```

**If the OCSP service is running, restart it:**
**Note:** *Exit pqcadmin to restart the service*

```bash
sudo systemctl restart sassycorp-ocsp
```

---

## Module Summary

You have successfully implemented certificate revocation:

| Mechanism | Status | Location |
|-----------|--------|----------|
| CRL | Operational | `/opt/sassycorp-pqc/intermediate-ca/crl/intermediate-ca.crl` |
| OCSP | Configured | Port 8888 |

### Revoked Certificates

| Certificate | Reason | Status |
|------------|--------|--------|
| Revocation Test Certificate | keyCompromise | Permanently revoked |
| Certificate Hold Test | certificateHold | Temporarily suspended |

### Key Commands Reference

| Operation | Command |
|-----------|---------|
| Revoke certificate | `openssl ca -revoke <cert> -crl_reason <reason>` |
| Generate CRL | `openssl ca -gencrl -out <crl>` |
| Verify with CRL | `openssl verify -crl_check -CRLfile <crl>` |
| Start OCSP | `openssl ocsp -index <db> -port <port> -rsigner <cert> -rkey <key>` |
| Query OCSP | `openssl ocsp -issuer <ca> -cert <cert> -url <url>` |

---

## Troubleshooting

### OCSP "unable to load certificate" Error

**Problem:** OCSP responder can't start

**Solution:** Verify certificate and key match:
```bash
openssl x509 -in /opt/sassycorp-pqc/ocsp/certs/ocsp.crt -noout -pubkey > /tmp/cert.pub
openssl pkey -in /opt/sassycorp-pqc/ocsp/private/ocsp.key -pubout > /tmp/key.pub
diff /tmp/cert.pub /tmp/key.pub
```

### CRL Verification Fails

**Problem:** CRL signature verification fails

**Solution:** Ensure you're using the correct CA certificate:
```bash
openssl crl -in <crl> -noout -text | grep Issuer
openssl x509 -in <ca-cert> -noout -subject
```

### OCSP "unknown certificate" Response

**Problem:** OCSP returns "unknown" for valid certificate

**Solution:** The certificate may not be in the OCSP index. Update it:
```bash
cp /opt/sassycorp-pqc/intermediate-ca/index.txt /opt/sassycorp-pqc/ocsp/index.txt
```

---

## Security Checklist

Before proceeding, verify:

- [ ] CRL is signed with correct algorithm (ML-DSA-65)
- [ ] CRL Next Update is set appropriately
- [ ] OCSP responder certificate has OCSPSigning EKU
- [ ] OCSP private key has 400 permissions
- [ ] Revoked certificates show proper reason codes
- [ ] Valid certificates still verify successfully

---

**Next:** [Hybrid Methods →](06_fips_quantum_ca_hybrid_methods.md)
