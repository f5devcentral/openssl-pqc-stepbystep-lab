# Module 6: IETF Hybrid PQC Methods *(addendum/bonus module)*

## Overview

This fancy addendum explores hybrid post-quantum cryptographic approaches being standardized by the IETF. Hybrid schemes combine traditional (classical) algorithms with post-quantum algorithms, providing security even if one algorithm is broken. This defense-in-depth strategy is recommended during the transition period to post-quantum cryptography. There's also good arguments for keeping hybrid methods around longer but at the cost of larger payloads. But that's for another argument in another forum.

---

## Learning Objectives

After completing this module, you will be able to:

- Understand the purpose and benefits of hybrid PQC schemes
- Understand IETF standardization efforts for hybrid cryptography
- Configure hybrid key exchange in TLS 1.3
- Describe composite signature approaches
- Evaluate when to use pure PQC vs. hybrid approaches

---

## Why Hybrid Cryptography?

### The Uncertainty Problem

During the transition to post-quantum cryptography, organizations face two uncertainties:

1. **Classical algorithms may fall to quantum attacks**: RSA, ECDSA, and ECDH will be broken when sufficiently powerful quantum computers exist.

2. **Post-quantum algorithms are new**: ML-KEM, ML-DSA, and SLH-DSA have undergone extensive analysis, but they haven't been deployed at scale for decades like RSA.

3. **Half of Round 1 Submissions to NIST were broken pretty quick**: Take a look at table 6 per [Quantifying risks in cryptographic selection processes](https://cr.yp.to/papers/qrcsp-20231202.pdf). Given the infancy of our current PQC standards, a risk adverse position would be to keep hybrid around while the smarter people at the table keep working on improving what we have and developing faster and more robust solutions down the road.  Yet another need for cryptographic agility practices.

### The Hybrid Solution

Hybrid schemes combine both algorithm types so that:

- If quantum computers break classical algorithms, the PQC component provides security
- If a classical attack is found against PQC algorithms, the classical component provides security
- Security is maintained as long as **at least one** algorithm remains secure

---

## IETF Hybrid Standardization

### Key IETF Documents

| Document | Status | Purpose |
|----------|--------|---------|
| RFC 9794 | Published | Terminology for PQ/T hybrid schemes |
| draft-ietf-tls-hybrid-design | Active | Hybrid key exchange in TLS 1.3 |
| draft-ietf-lamps-pq-composite-sigs | Active | Composite signatures for X.509 |
| draft-ietf-lamps-pq-composite-kem | Active | Composite KEM for X.509 and CMS |
| draft-ietf-pquip-pqc-engineers | Active | PQC guidance for engineers |

### Terminology (RFC 9794)

| Term | Definition |
|------|------------|
| **PQ/T Hybrid** | Post-Quantum/Traditional hybrid combining both algorithm types |
| **Composite** | Multiple algorithms treated as a single atomic unit |
| **Non-composite** | Multiple algorithms used separately at the protocol level |
| **Hybrid Confidentiality** | Security against both classical and quantum adversaries |
| **Hybrid Authentication** | Authentication using both algorithm types |

---

## Part A: Hybrid Key Exchange in TLS 1.3

OpenSSL 3.5.x supports hybrid key exchange groups that combine X25519 with ML-KEM.

### Step 1: List Available Hybrid Groups

Check available KEM algorithms:

```bash
openssl list -kem-algorithms | grep -i mlkem
```

**Expected output includes:**

```
MLKEM512 @ default
MLKEM768 @ default
MLKEM1024 @ default
X25519MLKEM768 @ default
```

The `X25519MLKEM768` is a hybrid that combines:
- **X25519**: Elliptic curve Diffie-Hellman (classical)
- **ML-KEM-768**: Module-Lattice Key Encapsulation (post-quantum)

---

### Step 2: Generate a Hybrid KEM Key

Generate an X25519MLKEM768 key pair:

```bash
openssl genpkey -algorithm X25519MLKEM768 -out /tmp/hybrid-kem.key
```

View the key details:

```bash
openssl pkey -in /tmp/hybrid-kem.key -noout -text | head -20
```

Extract the public key:

```bash
openssl pkey -in /tmp/hybrid-kem.key -pubout -out /tmp/hybrid-kem.pub
```

Clean up:

```bash
rm /tmp/hybrid-kem.key /tmp/hybrid-kem.pub
```

---

### Step 3: Test TLS 1.3 with Hybrid Key Exchange

Create a test server certificate (using our existing server cert):

Start a TLS server with hybrid key exchange:

```bash
openssl s_server \
    -key /opt/sassycorp-pqc/requests/www.sassycorp.lab.key \
    -cert /opt/sassycorp-pqc/requests/www.sassycorp.lab.fullchain.pem \
    -port 4433 \
    -tls1_3 \
    -groups X25519MLKEM768:X25519 \
    -www
```

**Options explained:**

| Option | Purpose |
|--------|---------|
| `-tls1_3` | Force TLS 1.3 |
| `-groups X25519MLKEM768:X25519` | Prefer hybrid, fallback to X25519 |
| `-www` | Serve a simple HTML response |

Open a **new terminal** for the client test.

---

### Step 4: Connect with Hybrid Key Exchange

In the new terminal:

```bash
sudo su - pqcadmin
```

Connect to the server requesting hybrid key exchange:

```bash
openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups X25519MLKEM768 \
    -CAfile /opt/sassycorp-pqc/intermediate-ca/certs/ca-chain.crt
```

After connecting, look for:

```
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
...
Server Temp Key: X25519MLKEM768
```

Type `GET /` to see the server response, then `QUIT` to exit.

Return to the original terminal and press `Ctrl+C` to stop the server.

---

### Step 5: Verify Negotiated Group

To see more details about the key exchange, use:

```bash
openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups X25519MLKEM768 \
    -CAfile /opt/sassycorp-pqc/intermediate-ca/certs/ca-chain.crt \
    -msg 2>&1 | grep -i "server temp key\|group"
```

---

## Part B: Understanding Composite Signatures

### Composite vs. Non-Composite Approaches

**Non-Composite (Protocol-Level Hybrid):**
- Multiple separate signatures in the protocol
- Each algorithm produces its own signature
- Verifier checks all signatures
- Example: Multiple signatures in a document

**Composite (Atomic Hybrid):**
- Single signature value containing both algorithms
- Appears as one algorithm to the protocol
- Cannot be split or verified separately
- Example: Composite ML-DSA + ECDSA signature

### IETF Composite Signature Draft

The draft `draft-ietf-lamps-pq-composite-sigs` defines combinations like:

| Composite Algorithm | Components |
|--------------------|------------|
| MLDSA44-ECDSA-P256 | ML-DSA-44 + ECDSA P-256 |
| MLDSA65-ECDSA-P384 | ML-DSA-65 + ECDSA P-384 |
| MLDSA87-ECDSA-P384 | ML-DSA-87 + ECDSA P-384 |
| MLDSA65-Ed25519 | ML-DSA-65 + Ed25519 |

> **Note:** As of OpenSSL 3.5.3, [composite signatures are not yet implemented natively](https://github.com/openssl/openssl/issues/26121). They require either the OQS provider or future OpenSSL updates.

---

### Step 6: Simulating Non-Composite Hybrid Signatures

While composite signatures aren't yet available, you can implement non-composite hybrid signatures at the application level:

Create test data:

```bash
echo "This document requires hybrid signature verification" > /tmp/hybrid-doc.txt
```

Create an ML-DSA signature:

```bash
openssl dgst -sign /opt/sassycorp-pqc/requests/alice.smith.key -out /tmp/hybrid-doc.mldsa.sig /tmp/hybrid-doc.txt
```

If you also had an ECDSA key, you would create a second signature:

```bash
# Example (requires ECDSA key):
# openssl dgst -sha256 -sign ecdsa.key -out /tmp/hybrid-doc.ecdsa.sig /tmp/hybrid-doc.txt
```

In a non-composite hybrid system, both signatures would be verified.

Clean up:

```bash
rm /tmp/hybrid-doc.txt /tmp/hybrid-doc.mldsa.sig
```

---

## Part C: Hybrid Certificate Approaches

### Certificate with Hybrid Signature

A certificate signed with a composite algorithm would use both ML-DSA and a classical algorithm (like ECDSA) to create the signature. The signature value contains both component signatures.

**Benefits:**
- Single certificate works with both PQC and legacy systems
- Backward compatibility during migration

**Challenges:**
- Larger certificate sizes
- Requires updated software for full verification
- Standards still evolving

### Dual Certificate Approach

An alternative is issuing two separate certificates:

| Certificate | Algorithm | Use Case |
|------------|-----------|----------|
| Primary | ML-DSA-65 | PQC-aware systems |
| Fallback | ECDSA P-384 | Legacy systems |

Servers can present different certificates based on client capabilities.

---

## Part D: Recommendations for Hybrid Deployment

### When to Use Hybrid

| Scenario | Recommendation |
|----------|---------------|
| Long-lived encrypted data | Use hybrid KEM now |
| TLS connections | Enable hybrid key exchange |
| Internal PKI | Pure PQC may be sufficient |
| Public-facing services | Hybrid for compatibility |

### Implementation Priority

1. **Start with hybrid key exchange (TLS)**: Protects against "harvest now, decrypt later"
2. **Then hybrid signatures (where needed)**: Provides defense-in-depth
3. **Evaluate pure PQC**: When algorithms mature and compatibility isn't needed

---

## Step 7: Configure Default Hybrid Groups

To make hybrid key exchange the default for OpenSSL:

Create or edit the OpenSSL configuration:

```bash
vim ~/.openssl.cnf
```

Add:

```ini
[openssl_init]
ssl_conf = ssl_section

[ssl_section]
system_default = ssl_default_section

[ssl_default_section]
Groups = X25519MLKEM768:X25519:secp384r1
```

Set the environment variable:

```bash
echo 'export OPENSSL_CONF=~/.openssl.cnf' >> ~/.bashrc
source ~/.bashrc
```

Now hybrid key exchange will be preferred by default.

---

### Size Comparison

| Component | X25519 | ML-KEM-768 | X25519MLKEM768 (Hybrid) |
|-----------|--------|------------|-------------------------|
| Public Key | 32 bytes | 1,184 bytes | 1,216 bytes |
| Ciphertext/Shared | 32 bytes | 1,088 bytes | 1,120 bytes |

| Component | ECDSA P-256 | ML-DSA-65 | Composite (theoretical) |
|-----------|-------------|-----------|------------------------|
| Public Key | 64 bytes | 1,952 bytes | ~2,016 bytes |
| Signature | 64 bytes | 3,293 bytes | ~3,357 bytes |

---
<br>

## Module Summary

You have learned about hybrid post-quantum cryptography approaches. It's a hot topic so we'll keep paying attention and make updates to this guide as needed:

| Topic | Key Takeaway |
|-------|-------------|
| Why Hybrid | Defense-in-depth during algorithm transition |
| TLS Hybrid | X25519MLKEM768 available in OpenSSL 3.5.x |
| Composite Signatures | Standards evolving, not yet in OpenSSL |
| Deployment Priority | Start with key exchange, then signatures |
| Performance | Minimal overhead for hybrid key exchange |

### Key IETF Resources

- [RFC 9794: Hybrid Terminology](https://datatracker.ietf.org/doc/rfc9794/)
- [TLS Hybrid Design](https://datatracker.ietf.org/doc/draft-ietf-tls-hybrid-design/)
- [Composite Signatures](https://datatracker.ietf.org/doc/draft-ietf-lamps-pq-composite-sigs/)
- [PQC for Engineers](https://datatracker.ietf.org/doc/draft-ietf-pquip-pqc-engineers/)

---

## Lab Completion

Congratulations! You have completed the FIPS 203/204/205 Post-Quantum Cryptography Learning Path.

### What You Built

```
Sassy Corp Quantum-Resistant PKI
├── Root CA (ML-DSA-87)
│   └── 10-year validity, Level 5 security
├── Intermediate CA (ML-DSA-65)
│   └── 5-year validity, Level 3 security
├── End-Entity Certificates
│   ├── Server certs (ML-DSA-65)
│   ├── User certs (ML-DSA-44)
│   └── High-security certs (ML-DSA-87)
├── Revocation Infrastructure
│   ├── CRL distribution
│   └── OCSP responder
└── Hybrid TLS Support
    └── X25519MLKEM768 key exchange
```

### Skills Acquired

- Installing and configuring OpenSSL 3.5.x with native PQC
- Creating quantum-resistant CA hierarchies
- Issuing certificates with proper SAN configuration
- Implementing CRL and OCSP revocation
- Understanding IETF hybrid approaches
- Evaluating algorithm choices for different use cases

### Next Steps

1. **Apply to your environment**: Adapt these techniques for your organization
2. **Monitor standards**: Stay updated on IETF and NIST developments
3. **Plan migration**: Develop a cryptographic transition roadmap
4. **Build expertise**: Practice with additional certificate types and scenarios
5. **Consider CNSA 2.0**: If you have government/defense requirements, review the CNSA 2.0 learning path

---

## Resources

### Official Documentation

- [NIST Post-Quantum Cryptography](https://csrc.nist.gov/projects/post-quantum-cryptography)
- [OpenSSL Documentation](https://www.openssl.org/docs/)
- [IETF PQUIP Working Group](https://datatracker.ietf.org/wg/pquip/about/)

### Further Learning

- [Post-Quantum Cryptography for Engineers (IETF Draft)](https://datatracker.ietf.org/doc/draft-ietf-pquip-pqc-engineers/)
- [NIST SP 800-208: Recommendation for Stateful Hash-Based Signatures](https://csrc.nist.gov/publications/detail/sp/800-208/final)
- [Open Quantum Safe Project](https://openquantumsafe.org/)

---
## 🤝 Contributing

This lab was made because we wanted to learn this ourselves and cover popular use cases for PQC (beyon CNSA 2.0). We encourage you to help us expand this lab so we can cover more quantum-resistant content; or maybe cover more operating systems.  Or we might have inadvertently broken something.  Who knows... either way, characters welcome. Check out our [Contributing.md](/contributing.md) guidelines and fill in the gaps we probably missed.

Your lab is ready for internal testing of quantum-resistant PKI infrastructure in preparation for the post-quantum cryptography era. Nice work! Go take the rest of the day off.

---