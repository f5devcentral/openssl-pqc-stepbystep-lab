# Module 05: BIKE and HQC - Code-Based KEM Alternatives

## Overview

BIKE (Bit Flipping Key Encapsulation) and HQC (Hamming Quasi-Cyclic) are code-based KEMs that provide alternatives to lattice-based cryptography. In March 2025, NIST selected HQC as the backup standard to ML-KEM, with a draft standard expected in 2026 and finalization in 2027.

Both algorithms are based on different mathematical problems than ML-KEM, providing algorithm diversity for defense-in-depth strategies.

---

## Learning Objectives

After completing this module, you will be able to:

- Explain code-based cryptography fundamentals
- Distinguish between BIKE and HQC approaches
- Configure TLS connections using BIKE and HQC
- Understand why HQC was selected as the ML-KEM backup
- Recognize appropriate deployment scenarios for code-based KEMs

---

## Understanding Code-Based Cryptography

### Mathematical Foundation

Code-based cryptography relies on the difficulty of decoding random linear codes:

| Problem | Description | Used By |
|---------|-------------|---------|
| Syndrome Decoding | Decode arbitrary linear code | Classic McEliece |
| MDPC Decoding | Decode moderate-density parity-check codes | BIKE |
| Quasi-Cyclic Decoding | Decode quasi-cyclic codes | HQC |

### Why Code-Based Matters

If a breakthrough occurs in lattice cryptanalysis, code-based algorithms provide a fallback:

| Family | Mathematical Problem | Primary Standard | Backup Option |
|--------|---------------------|------------------|---------------|
| Lattice | Module-LWE | ML-KEM | FrodoKEM, NTRU |
| Code-based | Syndrome Decoding | **HQC (2027)** | BIKE, Classic McEliece |
| Hash-based | Hash Function Security | SLH-DSA | LMS, XMSS |

---

## BIKE Overview

### What is BIKE?

BIKE uses QC-MDPC (Quasi-Cyclic Moderate Density Parity-Check) codes for key encapsulation.

| Variant | Security Level | Public Key | Secret Key | Ciphertext |
|---------|---------------|------------|------------|------------|
| bike1l1fo | 1 | 1,541 B | 3,114 B | 1,573 B |
| bike1l3fo | 3 | 3,083 B | 6,198 B | 3,115 B |
| bike1l5fo | 5 | 5,122 B | 10,276 B | 5,154 B |

### BIKE Characteristics

| Aspect | BIKE | Notes |
|--------|------|-------|
| Key sizes | Medium | Between NTRU and FrodoKEM |
| Performance | Good | Competitive with lattice schemes |
| Decryption failures | Very rare | ~2^-128 probability |
| History | Since 2017 | NIST Round 4 candidate |

### Historical Concerns

BIKE has faced some implementation challenges:

- **2022**: Timing attack vulnerabilities discovered
- **2023**: Patches and mitigations implemented
- **Current**: Secure implementations available

---

## HQC Overview

### What is HQC?

HQC uses quasi-cyclic codes with a different construction than BIKE. NIST selected HQC as the backup to ML-KEM in March 2025.

| Variant | Security Level | Public Key | Secret Key | Ciphertext |
|---------|---------------|------------|------------|------------|
| hqc128 | 1 | 2,249 B | 2,289 B | 4,497 B |
| hqc192 | 3 | 4,522 B | 4,562 B | 9,042 B |
| hqc256 | 5 | 7,245 B | 7,285 B | 14,485 B |

### HQC Characteristics

| Aspect | HQC | Notes |
|--------|-----|-------|
| Key sizes | Medium-large | Larger ciphertexts than BIKE |
| Performance | Good | Slightly better than BIKE |
| Decryption failures | Very rare | Addressed in recent versions |
| Status | **NIST Selected** | Standard expected 2027 |

### Why NIST Selected HQC

NIST's selection rationale:

1. **Different mathematical basis**: True backup to lattice-based ML-KEM
2. **Conservative security**: Well-analyzed code construction
3. **Reasonable performance**: Practical for most applications
4. **Implementation maturity**: Years of refinement through NIST process

---

## Step 1: Verify BIKE and HQC Availability

Confirm both algorithms are available:

```bash
openssl list -kem-algorithms | grep -i bike
```

**Expected output:**

```
bike1l1fo @ oqsprovider
bike1l3fo @ oqsprovider
bike1l5fo @ oqsprovider
```

```bash
openssl list -kem-algorithms | grep -i hqc
```

**Expected output:**

```
hqc128 @ oqsprovider
hqc192 @ oqsprovider
hqc256 @ oqsprovider
```

---

## Step 2: Start TLS Server with BIKE

Navigate to your lab directory:

```bash
cd /opt/sassycorp-pqc-alt
```

Start a TLS server with BIKE:

```bash
openssl s_server \
    -key keys/test.key \
    -cert certs/test.crt \
    -port 4433 \
    -tls1_3 \
    -groups bike1l3fo \
    -www
```

Open a **new terminal** for client testing.

---

## Step 3: Connect with BIKE Key Exchange

```bash
cd /opt/sassycorp-pqc-alt
```

```bash
openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups bike1l3fo \
    -CAfile certs/test.crt
```

**Look for:**

```
Server Temp Key: bike1l3fo
```

Or:

```
Negotiated TLS1.3 group: bike1l3fo
```

Type `GET /` then `QUIT` to exit.

---

## Step 4: Measure BIKE Handshake Size

```bash
echo | openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups bike1l3fo \
    -CAfile certs/test.crt 2>&1 | \
    grep "SSL handshake has read"
```

**Example output:**

```
SSL handshake has read 13547 bytes and written 4021 bytes
```

---

## Step 5: Start TLS Server with HQC

Stop the BIKE server (`Ctrl+C`) and start HQC:

```bash
openssl s_server \
    -key keys/test.key \
    -cert certs/test.crt \
    -port 4433 \
    -tls1_3 \
    -groups hqc192 \
    -www
```

---

## Step 6: Connect with HQC Key Exchange

```bash
openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups hqc192 \
    -CAfile certs/test.crt
```

**Look for:**

```
Server Temp Key: hqc192
```

Type `GET /` then `QUIT` to exit.

---

## Step 7: Measure HQC Handshake Size

```bash
echo | openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups hqc192 \
    -CAfile certs/test.crt 2>&1 | \
    grep "SSL handshake has read"
```

**Example output:**

```
SSL handshake has read 19876 bytes and written 5432 bytes
```

---

## Step 8: Compare BIKE and HQC

### Size Comparison

| Algorithm | Public Key | Ciphertext | Total Overhead |
|-----------|------------|------------|----------------|
| bike1l1fo | 1,541 B | 1,573 B | 3,114 B |
| bike1l3fo | 3,083 B | 3,115 B | 6,198 B |
| hqc128 | 2,249 B | 4,497 B | 6,746 B |
| hqc192 | 4,522 B | 9,042 B | 13,564 B |
| MLKEM768 | 1,184 B | 1,088 B | 2,272 B |

### Observations

| Metric | BIKE | HQC | Notes |
|--------|------|-----|-------|
| Public key size | Smaller | Larger | BIKE ~30% smaller |
| Ciphertext size | Smaller | Larger | HQC has larger ciphertexts |
| Total overhead | Lower | Higher | BIKE more compact |
| NIST status | Round 4 | **Selected** | HQC will be standardized |

---

## Step 9: Test Different Security Levels

### BIKE Security Levels

Test all BIKE security levels:

**Level 1:**

```bash
openssl s_server \
    -key keys/test.key \
    -cert certs/test.crt \
    -port 4433 \
    -tls1_3 \
    -groups bike1l1fo \
    -www
```

```bash
echo | openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups bike1l1fo \
    -CAfile certs/test.crt 2>&1 | \
    grep "SSL handshake has read"
```

**Level 5:**

```bash
openssl s_server \
    -key keys/test.key \
    -cert certs/test.crt \
    -port 4433 \
    -tls1_3 \
    -groups bike1l5fo \
    -www
```

```bash
echo | openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups bike1l5fo \
    -CAfile certs/test.crt 2>&1 | \
    grep "SSL handshake has read"
```

### HQC Security Levels

Test all HQC security levels:

**Level 1 (hqc128):**

```bash
openssl s_server \
    -key keys/test.key \
    -cert certs/test.crt \
    -port 4433 \
    -tls1_3 \
    -groups hqc128 \
    -www
```

```bash
echo | openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups hqc128 \
    -CAfile certs/test.crt 2>&1 | \
    grep "SSL handshake has read"
```

**Level 5 (hqc256):**

```bash
openssl s_server \
    -key keys/test.key \
    -cert certs/test.crt \
    -port 4433 \
    -tls1_3 \
    -groups hqc256 \
    -www
```

```bash
echo | openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups hqc256 \
    -CAfile certs/test.crt 2>&1 | \
    grep "SSL handshake has read"
```

---

## Step 10: Record Performance Metrics

Update your test results file:

```bash
cat >> /opt/sassycorp-pqc-alt/tests/bike-hqc-results.txt << 'EOF'
BIKE and HQC Performance Test Results
=====================================
Date: $(date)
System: $(uname -a)

BIKE Results:
Algorithm     | Public Key | Ciphertext | Handshake Read | Handshake Write
--------------|------------|------------|----------------|----------------
bike1l1fo     | 1,541 B    | 1,573 B    |                |
bike1l3fo     | 3,083 B    | 3,115 B    |                |
bike1l5fo     | 5,122 B    | 5,154 B    |                |

HQC Results:
Algorithm     | Public Key | Ciphertext | Handshake Read | Handshake Write
--------------|------------|------------|----------------|----------------
hqc128        | 2,249 B    | 4,497 B    |                |
hqc192        | 4,522 B    | 9,042 B    |                |
hqc256        | 7,245 B    | 14,485 B   |                |

Fill in your measured values above.
EOF
```

---

## BIKE vs HQC: When to Use Each

### BIKE Considerations

| Factor | Assessment |
|--------|------------|
| **Key sizes** | Smaller than HQC |
| **Ciphertext sizes** | Smaller than HQC |
| **Standardization** | NIST Round 4 (not selected) |
| **Implementation history** | Timing attack concerns (patched) |
| **Future** | May see adoption outside NIST |

### HQC Considerations

| Factor | Assessment |
|--------|------------|
| **Key sizes** | Larger than BIKE |
| **Ciphertext sizes** | Larger than BIKE |
| **Standardization** | **NIST selected (2027)** |
| **Implementation history** | Clean security record |
| **Future** | Will be FIPS standard |

### Decision Framework

```
Need code-based KEM for algorithm diversity?
├── Yes → Need NIST standardized algorithm?
│         ├── Yes → Use HQC (wait for 2027 standard)
│         └── No → Which matters more?
│                   ├── Smaller sizes → Use BIKE
│                   └── Standards compliance → Use HQC
└── No → Use ML-KEM (FIPS 203)
```

---

## HQC Standardization Timeline

| Date | Milestone |
|------|-----------|
| March 2025 | NIST selects HQC as backup |
| Early 2026 | Draft FIPS standard expected |
| 2027 | Final FIPS standard expected |
| 2027+ | Production deployments begin |

Organizations planning to use HQC should:
1. Monitor draft standard development
2. Test with current OQS implementation
3. Plan migration from ML-KEM when diversity needed

---

## Summary

BIKE and HQC provide code-based alternatives to lattice cryptography:

| Metric | BIKE Level 3 | HQC Level 3 | ML-KEM-768 |
|--------|--------------|-------------|------------|
| Public key | 3,083 B | 4,522 B | 1,184 B |
| Ciphertext | 3,115 B | 9,042 B | 1,088 B |
| Total overhead | 6,198 B | 13,564 B | 2,272 B |
| Standardization | Round 4 | **NIST selected** | FIPS 203 |
| Mathematical basis | QC-MDPC | Quasi-cyclic | Module-LWE |

**Bottom line:** HQC will be the standardized code-based backup to ML-KEM. Organizations needing algorithm diversity should plan for HQC adoption when the standard finalizes in 2027. BIKE remains viable for non-NIST deployments where smaller sizes matter.

---

## Next Steps

Proceed to **[Module 06: International PQC](06-INTERNATIONAL-PQC.md)** to learn about post-quantum cryptography standards from South Korea, China, and European agencies.

---

**Module Navigation:**

| Previous | Current | Next |
|----------|---------|------|
| [04 - Classic McEliece](04-CLASSIC-MCELIECE.md) | **05 - BIKE and HQC** | [06 - International PQC](06-INTERNATIONAL-PQC.md) |
