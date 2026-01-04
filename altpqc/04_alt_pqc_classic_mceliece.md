# Module 04: Classic McEliece - Conservative Code-Based KEM

## Overview

Classic McEliece is based on the McEliece cryptosystem, originally proposed in 1978—making it the oldest public-key encryption system still considered secure. With over 40 years of cryptanalysis without practical attacks, Classic McEliece represents the most conservative approach to post-quantum key encapsulation.

The trade-off for this conservative security is significant: Classic McEliece has the largest public keys of any PQC algorithm, ranging from 261 KB to over 1 MB.

---

## Learning Objectives

After completing this module, you will be able to:

- Explain Classic McEliece's code-based security foundation
- Understand why 40+ years of cryptanalysis matters
- Configure TLS connections using Classic McEliece
- Recognize the key size vs security trade-off
- Identify appropriate deployment scenarios for massive keys

---

## Understanding Classic McEliece

### Historical Significance

| Year | Milestone |
|------|-----------|
| 1978 | McEliece proposes original cryptosystem |
| 1980-2000 | Various attack attempts, all unsuccessful |
| 2008 | Modern Classic McEliece specification |
| 2017 | Submitted to NIST PQC competition |
| 2020 | NIST Round 3 finalist |
| 2022 | NIST Round 4 (delayed for ISO process) |
| 2024+ | ISO standardization in progress |

### Why Such Large Keys?

Classic McEliece security is based on the hardness of decoding random linear codes (the "Syndrome Decoding Problem"). The public key is essentially a large matrix, and security increases with matrix size.

| Parameter Set | Security Level | Public Key | Private Key | Ciphertext |
|--------------|---------------|------------|-------------|------------|
| mceliece348864 | 1 | 261,120 B (255 KB) | 6,492 B | 196 B |
| mceliece460896 | 3 | 524,160 B (512 KB) | 13,608 B | 156 B |
| mceliece6688128 | 5 | 1,044,992 B (1 MB) | 13,932 B | 208 B |
| mceliece6960119 | 5 | 1,047,319 B (1 MB) | 13,948 B | 194 B |
| mceliece8192128 | 5 | 1,357,824 B (1.3 MB) | 14,120 B | 240 B |

### The Paradox: Huge Keys, Tiny Ciphertexts

Notice the ciphertext sizes: 156-240 bytes—smaller than any other PQC KEM. This creates an interesting deployment model:

- **Initial key exchange**: Very expensive (megabyte of data)
- **Subsequent operations**: Very cheap (under 250 bytes)

This makes Classic McEliece ideal for **static key** scenarios where:
1. Public keys are distributed out-of-band (certificates, DNS, etc.)
2. Many encryption operations use the same key
3. Per-message overhead matters more than initial setup

---

## Step 1: Verify Classic McEliece Availability

Confirm Classic McEliece algorithms are available:

```bash
openssl list -kem-algorithms | grep -i mceliece
```

**Expected output:**

```
mceliece348864 @ oqsprovider
mceliece460896 @ oqsprovider
mceliece6688128 @ oqsprovider
mceliece6960119 @ oqsprovider
mceliece8192128 @ oqsprovider
```

---

## Step 2: Understand the Memory Requirements

Before testing, understand the memory implications:

| Variant | Memory for Public Key | Typical Use Case |
|---------|----------------------|------------------|
| mceliece348864 | ~256 KB | Minimum viable |
| mceliece460896 | ~512 KB | Balanced security |
| mceliece6688128 | ~1 MB | High security |
| mceliece8192128 | ~1.3 MB | Maximum security |

**Warning:** Testing with larger variants requires adequate system memory and may cause significant delays during key operations.

---

## Step 3: Start TLS Server with Classic McEliece

Navigate to your lab directory:

```bash
cd /opt/sassycorp-pqc-alt
```

Start a TLS server with Classic McEliece (using the smallest variant for demonstration):

```bash
openssl s_server \
    -key keys/test.key \
    -cert certs/test.crt \
    -port 4433 \
    -tls1_3 \
    -groups mceliece348864 \
    -www
```

**Note:** Server startup may take longer than with other algorithms due to key generation overhead.

The server is now listening. Open a **new terminal** for client testing.

---

## Step 4: Connect with Classic McEliece Key Exchange

In the new terminal:

```bash
cd /opt/sassycorp-pqc-alt
```

Connect to the server:

```bash
openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups mceliece348864 \
    -CAfile certs/test.crt
```

**Observe:** The connection establishment takes noticeably longer than with other algorithms due to:
1. Large key transmission
2. Computationally intensive operations

**Look for:**

```
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Server Temp Key: mceliece348864
```

Type `GET /` then `QUIT` to exit.

---

## Step 5: Measure Classic McEliece Handshake Size

This is where Classic McEliece's trade-off becomes clear.

### Measure Handshake Bytes

```bash
echo | openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups mceliece348864 \
    -CAfile certs/test.crt 2>&1 | \
    grep "SSL handshake has read"
```

**Example output:**

```
SSL handshake has read 271543 bytes and written 1823 bytes
```

**Key observation:** The server sends approximately 256 KB just for the public key!

### Handshake Size Comparison

| Algorithm | Public Key | Total Handshake | Relative Size |
|-----------|------------|-----------------|---------------|
| X25519 | 32 B | ~10 KB | 1x (baseline) |
| MLKEM768 | 1,184 B | ~12 KB | 1.2x |
| ntru_hps2048677 | 930 B | ~11 KB | 1.1x |
| frodo640aes | 9,616 B | ~30 KB | 3x |
| **mceliece348864** | **261,120 B** | **~270 KB** | **27x** |
| **mceliece6688128** | **1,044,992 B** | **~1.1 MB** | **110x** |

---

## Step 6: Test Higher Security Variants (Optional)

**Warning:** Higher security variants will cause significant delays and transmit megabytes of data.

If you want to test mceliece460896:

```bash
openssl s_server \
    -key keys/test.key \
    -cert certs/test.crt \
    -port 4433 \
    -tls1_3 \
    -groups mceliece460896 \
    -www
```

```bash
time echo | openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups mceliece460896 \
    -CAfile certs/test.crt 2>&1 | \
    grep "SSL handshake has read"
```

The `time` command will show you the elapsed time for the handshake.

---

## Step 7: Understand the Static Key Model

Classic McEliece is designed for a different deployment model than ephemeral key exchange:

### Traditional TLS Model (Ephemeral Keys)

```
Client                        Server
   │                             │
   │──── ClientHello ──────────►│
   │◄─── ServerHello + Key ─────│ (New key each connection)
   │──── Key + Finished ───────►│
   │◄─── Finished ──────────────│
```

### Classic McEliece Model (Static Keys)

```
Certificate/DNS                Server
     │                           │
     │ Public Key (1 MB)         │
     │ distributed offline       │
     ▼                           │
  Client                         │
     │                           │
     │──── Ciphertext (240 B) ──►│
     │◄─── Response ─────────────│
```

When public keys are distributed out-of-band:
- Each subsequent message requires only ~200 bytes of KEM overhead
- No large key transmission during session establishment
- Ideal for systems with expensive initial setup but cheap ongoing operations

---

## Step 8: Compare Ciphertext Sizes

While Classic McEliece has huge public keys, its ciphertexts are remarkably small:

| Algorithm | Ciphertext Size | Relative Size |
|-----------|-----------------|---------------|
| **mceliece348864** | **196 B** | **Smallest PQC** |
| **mceliece6688128** | **208 B** | Very small |
| ntru_hps2048677 | 930 B | 5x larger |
| MLKEM768 | 1,088 B | 5.5x larger |
| frodo640aes | 9,720 B | 50x larger |

This makes Classic McEliece attractive for:
- Satellite communications (expensive per-byte transmission)
- Constrained IoT devices (after initial provisioning)
- Systems with pre-shared public keys

---

## Step 9: Record Performance Metrics

Update your test results file:

```bash
cat >> /opt/sassycorp-pqc-alt/tests/mceliece-results.txt << 'EOF'
Classic McEliece Performance Test Results
=========================================
Date: $(date)
System: $(uname -a)

Algorithm          | Public Key    | Ciphertext | Handshake Read | Handshake Write | Time
-------------------|---------------|------------|----------------|-----------------|------
mceliece348864     | 261,120 B     | 196 B      |                |                 |
mceliece460896     | 524,160 B     | 156 B      |                |                 |
mceliece6688128    | 1,044,992 B   | 208 B      |                |                 |

Fill in your measured values above.
EOF
```

---

## When to Use Classic McEliece

### Appropriate Use Cases

| Use Case | Why Classic McEliece |
|----------|----------------------|
| **Maximum security assurance** | 40+ years of cryptanalysis |
| **Static key infrastructure** | Keys distributed offline |
| **Satellite/space systems** | Tiny ciphertexts after setup |
| **Long-term key storage** | Conservative security guarantees |
| **European recommendations** | BSI endorsement |

### Inappropriate Use Cases

| Use Case | Why Not Classic McEliece |
|----------|--------------------------|
| **Dynamic TLS** | Megabyte handshakes unacceptable |
| **Mobile/IoT** | Key storage requirements too large |
| **High-volume connections** | Key generation too slow |
| **Bandwidth-constrained initial setup** | Public key transmission |

### Decision Framework

```
Need maximum conservative security?
├── Yes → Can distribute public keys out-of-band?
│         ├── Yes → Static key model acceptable?
│         │         ├── Yes → Use Classic McEliece
│         │         └── No → Use FrodoKEM
│         └── No → Key transmission acceptable?
│                   ├── Yes (> 1 MB OK) → Use Classic McEliece
│                   └── No → Use FrodoKEM or ML-KEM
└── No → Use ML-KEM (FIPS 203)
```

---

## Security Considerations

### 40+ Years of Cryptanalysis

The McEliece cryptosystem has survived longer than any other public-key encryption:

| Era | Attack Attempts | Result |
|-----|-----------------|--------|
| 1978-1990 | Information set decoding | Parameters scaled |
| 1990-2000 | Generalized decoding attacks | Parameters scaled |
| 2000-2010 | Algebraic attacks on Goppa codes | Goppa codes remain secure |
| 2010-2020 | Quantum algorithm analysis | Grover's algorithm partially helps |
| 2020+ | Ongoing analysis | No practical attacks |

### Why Goppa Codes?

Classic McEliece specifically uses binary Goppa codes because:
- Well-understood algebraic structure
- Efficient decoding algorithms exist
- No structural weaknesses discovered in 45+ years

---

## Classic McEliece and ISO Standardization

NIST delayed Classic McEliece standardization to coordinate with ISO:

| Organization | Status | Expected Timeline |
|--------------|--------|-------------------|
| NIST | Round 4 | Waiting for ISO |
| ISO/IEC | Active standardization | In progress |
| European agencies | Recommending | Already endorsed |

This dual-track standardization reflects Classic McEliece's importance despite its practical limitations.

---

## Summary

Classic McEliece represents the most conservative PQC approach with extreme trade-offs:

| Metric | mceliece348864 | ML-KEM-768 | Difference |
|--------|----------------|------------|------------|
| Public key | 261,120 B | 1,184 B | 220x larger |
| Ciphertext | 196 B | 1,088 B | 5.5x smaller |
| Security history | 40+ years | 5+ years | Much longer |
| Key generation | Very slow | Fast | Much slower |
| Standardization | ISO pending | NIST FIPS | Different tracks |

**Bottom line:** Classic McEliece is for organizations that prioritize maximum security assurance and can work with the static key model. Its tiny ciphertexts make it attractive for specific use cases despite the massive public keys.

---

## Next Steps

Proceed to **[Module 05: BIKE and HQC](05-BIKE-HQC.md)** to explore code-based alternatives including HQC, the NIST-selected backup to ML-KEM.

---

**Module Navigation:**

| Previous | Current | Next |
|----------|---------|------|
| [03 - NTRU](03-NTRU.md) | **04 - Classic McEliece** | [05 - BIKE and HQC](05-BIKE-HQC.md) |
