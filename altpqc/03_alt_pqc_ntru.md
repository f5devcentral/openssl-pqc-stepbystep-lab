# Module 03: NTRU - Compact Lattice-Based KEM

## Overview

NTRU is one of the oldest and most studied lattice-based cryptographic systems, with a security history spanning over 25 years. It offers the smallest key sizes among all post-quantum KEMs while maintaining strong security guarantees.

NTRU was standardized by ANSI in 2010 (X9.98-2010, updated 2017) and by IEEE (P1363.1), making it one of the few PQC algorithms with industry standardization predating the NIST competition.

---

## Learning Objectives

After completing this module, you will be able to:

- Explain NTRU's mathematical foundation and security history
- Configure TLS connections using NTRU key exchange
- Measure NTRU's compact key sizes and performance
- Understand NTRU's relationship to FALCON/FN-DSA
- Identify appropriate NTRU deployment scenarios

---

## Understanding NTRU

### History and Standardization

| Year | Milestone |
|------|-----------|
| 1996 | NTRU introduced by Hoffstein, Pipher, and Silverman |
| 1998 | First public analysis and refinements |
| 2010 | ANSI X9.98-2010 standardization |
| 2017 | ANSI X9.98-2017 update |
| 2020 | NIST Round 3 candidate (not selected) |
| 2024+ | Continues as widely-deployed alternative |

### Why NTRU Wasn't Selected by NIST

NIST chose ML-KEM (Kyber) over NTRU for the primary standard because:

- ML-KEM offered slightly better performance
- NIST wanted to standardize Module-LWE as the primary lattice approach
- NTRU's decryption failures (though extremely rare) were a concern

However, NTRU remains valuable because:
- **Smallest keys** among all PQC KEMs
- **25+ years of cryptanalysis** without practical attacks
- **Different structure** than ML-KEM (diversity benefit)
- **FALCON/FN-DSA foundation** (NTRU lattices used in the upcoming signature standard)

### NTRU Variants

| Variant | Security Level | Public Key | Secret Key | Ciphertext |
|---------|---------------|------------|------------|------------|
| ntru_hps2048509 | 1 | 699 B | 935 B | 699 B |
| ntru_hps2048677 | 3 | 930 B | 1,234 B | 930 B |
| ntru_hps4096821 | 5 | 1,230 B | 1,590 B | 1,230 B |
| ntru_hrss701 | 3 | 1,138 B | 1,450 B | 1,138 B |

**Note:** HPS (Hoffstein-Pipher-Silverman) and HRSS (NTRU-HRSS) are different parameter sets with slightly different properties.

---

## Step 1: Verify NTRU Availability

Confirm NTRU algorithms are available via the OQS provider:

```bash
openssl list -kem-algorithms | grep -i ntru
```

**Expected output:**

```
ntru_hps2048509 @ oqsprovider
ntru_hps2048677 @ oqsprovider
ntru_hps4096821 @ oqsprovider
ntru_hrss701 @ oqsprovider
```

---

## Step 2: Start TLS Server with NTRU

Navigate to your lab directory:

```bash
cd /opt/sassycorp-pqc-alt
```

Start a TLS server configured to use NTRU for key exchange:

```bash
openssl s_server \
    -key keys/test.key \
    -cert certs/test.crt \
    -port 4433 \
    -tls1_3 \
    -groups ntru_hps2048677 \
    -www
```

**Options explained:**

| Option | Purpose |
|--------|---------|
| -tls1_3 | Force TLS 1.3 (required for PQC KEMs) |
| -groups ntru_hps2048677 | Use NTRU-HPS at Security Level 3 |
| -www | Serve simple HTTP response |

The server is now listening. Open a **new terminal** for client testing.

---

## Step 3: Connect with NTRU Key Exchange

In the new terminal:

```bash
cd /opt/sassycorp-pqc-alt
```

Connect to the server:

```bash
openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups ntru_hps2048677 \
    -CAfile certs/test.crt
```

**Look for these indicators:**

```
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Server Temp Key: ntru_hps2048677
```

Or:

```
Negotiated TLS1.3 group: ntru_hps2048677
```

Type `GET /` to see the server response, then `QUIT` to exit.

---

## Step 4: Measure NTRU Handshake Size

NTRU's primary advantage is compact key sizes. Let's measure this.

### Measure Handshake Bytes

```bash
echo | openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups ntru_hps2048677 \
    -CAfile certs/test.crt 2>&1 | \
    grep "SSL handshake has read"
```

**Example output:**

```
SSL handshake has read 11543 bytes and written 1823 bytes
```

### Compare Across NTRU Variants

Test each NTRU variant to see the size progression.

**Level 1 (ntru_hps2048509):**

Stop the server and restart with Level 1:

```bash
openssl s_server \
    -key keys/test.key \
    -cert certs/test.crt \
    -port 4433 \
    -tls1_3 \
    -groups ntru_hps2048509 \
    -www
```

Measure:

```bash
echo | openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups ntru_hps2048509 \
    -CAfile certs/test.crt 2>&1 | \
    grep "SSL handshake has read"
```

**Level 5 (ntru_hps4096821):**

```bash
openssl s_server \
    -key keys/test.key \
    -cert certs/test.crt \
    -port 4433 \
    -tls1_3 \
    -groups ntru_hps4096821 \
    -www
```

```bash
echo | openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups ntru_hps4096821 \
    -CAfile certs/test.crt 2>&1 | \
    grep "SSL handshake has read"
```

---

## Step 5: Compare NTRU to Other KEMs

### Size Comparison Table

| Algorithm | Public Key | Ciphertext | Total KEM Overhead |
|-----------|------------|------------|-------------------|
| X25519 (classical) | 32 B | 32 B | 64 B |
| **ntru_hps2048509** | **699 B** | **699 B** | **1,398 B** |
| **ntru_hps2048677** | **930 B** | **930 B** | **1,860 B** |
| MLKEM768 | 1,184 B | 1,088 B | 2,272 B |
| frodo640aes | 9,616 B | 9,720 B | 19,336 B |

**Key insight:** NTRU offers the smallest post-quantum keys while providing comparable security to ML-KEM.

---

## Step 6: Test NTRU-HRSS Variant

NTRU-HRSS is an alternative parameter set with different trade-offs:

```bash
openssl s_server \
    -key keys/test.key \
    -cert certs/test.crt \
    -port 4433 \
    -tls1_3 \
    -groups ntru_hrss701 \
    -www
```

```bash
echo | openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups ntru_hrss701 \
    -CAfile certs/test.crt 2>&1 | \
    grep -E "SSL handshake|group"
```

### HPS vs HRSS Comparison

| Property | NTRU-HPS | NTRU-HRSS |
|----------|----------|-----------|
| Key sizes | Slightly smaller | Slightly larger |
| Decryption failures | Possible (very rare) | Zero |
| Performance | Similar | Similar |
| Security margin | Conservative | Conservative |

NTRU-HRSS eliminates decryption failures entirely, at the cost of slightly larger keys.

---

## Step 7: Understand NTRU's Key Generation Characteristics

NTRU has an interesting performance profile:

| Operation | NTRU | ML-KEM | Notes |
|-----------|------|--------|-------|
| Key Generation | Slow | Fast | NTRU ~2-3x slower |
| Encapsulation | Fast | Fast | Comparable |
| Decapsulation | Fast | Fast | Comparable |

**Implication:** NTRU is ideal for applications where:
- Keys are generated infrequently
- Keys are reused across many sessions
- Static key configurations are acceptable

---

## Step 8: Record Performance Metrics

Update your test results file:

```bash
cat >> /opt/sassycorp-pqc-alt/tests/ntru-results.txt << 'EOF'
NTRU Performance Test Results
=============================
Date: $(date)
System: $(uname -a)

Algorithm          | Public Key | Ciphertext | Client Writes | Server Reads
-------------------|------------|------------|---------------|-------------
ntru_hps2048509    | 699 B      | 699 B      |               |
ntru_hps2048677    | 930 B      | 930 B      |               |
ntru_hps4096821    | 1,230 B    | 1,230 B    |               |
ntru_hrss701       | 1,138 B    | 1,138 B    |               |

Fill in your measured handshake values above.
EOF
```

---

## NTRU and FN-DSA (FALCON)

NTRU lattices form the mathematical foundation for FN-DSA (FALCON), the upcoming NIST FIPS 206 signature standard:

| Algorithm | Type | Based On | Status |
|-----------|------|----------|--------|
| NTRU | KEM | NTRU lattices | ANSI/IEEE standardized |
| FN-DSA (FALCON) | Signature | NTRU lattices | FIPS 206 draft |

Understanding NTRU provides insight into how FN-DSA signatures will work.

---

## When to Use NTRU

### Appropriate Use Cases

| Use Case | Why NTRU |
|----------|----------|
| **IoT/Embedded** | Smallest keys fit in constrained memory |
| **Mobile applications** | Minimal bandwidth overhead |
| **Static key configurations** | Slow keygen acceptable if keys are reused |
| **Algorithm diversity** | Different math than ML-KEM |
| **Legacy PQC systems** | Long deployment history |

### Inappropriate Use Cases

| Use Case | Why Not NTRU |
|----------|--------------|
| **Ephemeral-only keys** | Slow keygen becomes bottleneck |
| **High-volume key rotation** | Keygen performance matters |
| **NIST compliance required** | Not a FIPS standard |

### Decision Framework

```
Need smallest possible keys?
├── Yes → Slow key generation acceptable?
│         ├── Yes → Use NTRU
│         └── No → Use ML-KEM (better keygen performance)
└── No → Use ML-KEM (FIPS 203)
```

---

## Security Considerations

### 25+ Years of Cryptanalysis

NTRU has survived extensive academic scrutiny:

| Year | Attack/Analysis | Result |
|------|-----------------|--------|
| 1996-2000 | Initial lattice attacks | Parameters adjusted |
| 2001-2010 | Hybrid attacks | Parameters refined |
| 2016 | Kirchner-Fouque attack | Parameters increased |
| 2020+ | Ongoing analysis | Remains secure |

### Known Considerations

- **Decryption failures**: NTRU-HPS has extremely rare decryption failures (~2^-150 probability)
- **Implementation attacks**: Side-channel resistance requires careful implementation
- **Parameter selection**: Use standardized parameters only

---

## Summary

NTRU provides the most compact post-quantum key exchange with extensive security history:

| Metric | NTRU-HPS-2048677 | ML-KEM-768 | Difference |
|--------|------------------|------------|------------|
| Public key | 930 B | 1,184 B | 21% smaller |
| Ciphertext | 930 B | 1,088 B | 15% smaller |
| Security level | 3 | 3 | Equivalent |
| Key generation | Slow | Fast | Slower |
| Standardization | ANSI/IEEE | NIST FIPS | Different bodies |

**Bottom line:** NTRU is ideal for bandwidth-constrained applications where key generation happens infrequently. Its 25-year security track record provides confidence for conservative deployments.

---

## Next Steps

Proceed to **[Module 04: Classic McEliece](04-CLASSIC-MCELIECE.md)** to explore the most conservative code-based KEM with 40+ years of cryptanalysis history.

---

**Module Navigation:**

| Previous | Current | Next |
|----------|---------|------|
| [02 - FrodoKEM](02-FRODOKEM.md) | **03 - NTRU** | [04 - Classic McEliece](04-CLASSIC-MCELIECE.md) |
