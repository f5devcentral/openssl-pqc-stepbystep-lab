# Module 02: FrodoKEM - Conservative Unstructured Lattice KEM

## Overview

FrodoKEM represents the most conservative approach to lattice-based cryptography. Unlike ML-KEM (which uses structured lattices with algebraic ring properties), FrodoKEM is based on the "plain" Learning With Errors (LWE) problem without additional mathematical structure.

This conservative design means FrodoKEM has stronger theoretical security guarantees but significantly larger key sizes and slower performance.

---

## Learning Objectives

After completing this module, you will be able to:

- Explain the difference between structured and unstructured lattice cryptography
- Configure TLS connections using FrodoKEM key exchange
- Measure FrodoKEM's performance characteristics
- Understand when FrodoKEM is the appropriate choice
- Compare FrodoKEM variants (AES vs SHAKE)

---

## Understanding FrodoKEM

### Structured vs Unstructured Lattices

| Property | ML-KEM (Structured) | FrodoKEM (Unstructured) |
|----------|---------------------|------------------------|
| Mathematical basis | Module-LWE | Plain LWE |
| Ring structure | Yes (polynomial rings) | No |
| Key sizes | Small (800-1,568 B) | Large (9,616-21,632 B) |
| Performance | Fast | Slow (~100x slower keygen) |
| Security margin | Good | Conservative (no structure to exploit) |

### Why FrodoKEM Matters

The algebraic structure in ML-KEM (and NTRU) enables efficiency but also provides attackers with additional mathematical properties to potentially exploit. FrodoKEM deliberately avoids this structure:

- **No ring attacks possible**: Algorithms that exploit polynomial ring structure cannot be applied
- **Worst-case to average-case reduction**: Stronger theoretical security proofs
- **European recommendation**: BSI (Germany) and ANSSI (France) recommend FrodoKEM for high-security applications

### FrodoKEM Variants

| Variant | Security Level | Public Key | Ciphertext | Use Case |
|---------|---------------|------------|------------|----------|
| frodo640aes | 1 (128-bit) | 9,616 B | 9,720 B | General high-security |
| frodo640shake | 1 (128-bit) | 9,616 B | 9,720 B | SHAKE-based PRF |
| frodo976aes | 3 (192-bit) | 15,632 B | 15,744 B | Enhanced security |
| frodo976shake | 3 (192-bit) | 15,632 B | 15,744 B | SHAKE-based PRF |
| frodo1344aes | 5 (256-bit) | 21,520 B | 21,632 B | Maximum security |
| frodo1344shake | 5 (256-bit) | 21,520 B | 21,632 B | SHAKE-based PRF |

**AES vs SHAKE:** The variants differ only in the pseudorandom function used internally. AES variants may be faster on hardware with AES-NI instructions; SHAKE variants align with NIST hash standards.

---

## Step 1: Verify FrodoKEM Availability

Confirm FrodoKEM algorithms are available via the OQS provider:

```bash
openssl list -kem-algorithms | grep -i frodo
```

**Expected output:**

```
frodo640aes @ oqsprovider
frodo640shake @ oqsprovider
frodo976aes @ oqsprovider
frodo976shake @ oqsprovider
frodo1344aes @ oqsprovider
frodo1344shake @ oqsprovider
```

---

## Step 2: List Available FrodoKEM TLS Groups

Check which FrodoKEM groups are available for TLS:

```bash
openssl list -tls1-groups | grep -i frodo
```

**Note:** Group availability depends on OQS provider version and configuration.

---

## Step 3: Start TLS Server with FrodoKEM

Navigate to your lab directory:

```bash
cd /opt/sassycorp-pqc-alt
```

Start a TLS server configured to use FrodoKEM for key exchange:

```bash
openssl s_server \
    -key keys/test.key \
    -cert certs/test.crt \
    -port 4433 \
    -tls1_3 \
    -groups frodo640aes \
    -www
```

**Options explained:**

| Option | Purpose |
|--------|---------|
| -tls1_3 | Force TLS 1.3 (required for PQC KEMs) |
| -groups frodo640aes | Use FrodoKEM-640 with AES |
| -www | Serve simple HTTP response |

The server is now listening. Open a **new terminal** for client testing.

---

## Step 4: Connect with FrodoKEM Key Exchange

In the new terminal, switch to the lab user:

```bash
sudo su - pqcadmin
```

Or if using the alternative user:

```bash
sudo su - pqcaltadmin
```

Navigate to the lab directory:

```bash
cd /opt/sassycorp-pqc-alt
```

Connect to the server:

```bash
openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups frodo640aes \
    -CAfile certs/test.crt
```

**Look for these indicators in the output:**

```
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Server Temp Key: frodo640aes
```

Or:

```
Negotiated TLS1.3 group: frodo640aes
```

The presence of `frodo640aes` confirms FrodoKEM was used for key exchange.

Type `GET /` to see the server response, then `QUIT` to exit.

---

## Step 5: Measure FrodoKEM Handshake Size

Understanding the bandwidth impact of FrodoKEM is critical for deployment decisions.

### Measure Handshake Bytes

With the server still running, measure the handshake size:

```bash
echo | openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups frodo640aes \
    -CAfile certs/test.crt 2>&1 | \
    grep "SSL handshake has read"
```

**Example output:**

```
SSL handshake has read 29847 bytes and written 10521 bytes
```

### Compare to ML-KEM (Native)

For comparison, test with native ML-KEM:

```bash
echo | openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups MLKEM768 \
    -CAfile certs/test.crt 2>&1 | \
    grep "SSL handshake has read"
```

**Note:** This may fail if the server doesn't offer MLKEM768. Start a new server with `-groups frodo640aes:MLKEM768` to test both.

### Handshake Size Comparison

| Algorithm | Client Writes | Server Reads | Total Overhead |
|-----------|---------------|--------------|----------------|
| X25519 (classical) | ~350 B | ~10,000 B | ~10,350 B |
| X25519MLKEM768 (hybrid) | ~1,500 B | ~11,000 B | ~12,500 B |
| frodo640aes | ~10,500 B | ~30,000 B | ~40,500 B |
| frodo976aes | ~16,500 B | ~48,000 B | ~64,500 B |

**Key insight:** FrodoKEM adds approximately 3-5x more handshake data compared to hybrid ML-KEM methods.

---

## Step 6: Test Higher Security Levels

Stop the current server (`Ctrl+C`) and start one with FrodoKEM-976:

```bash
openssl s_server \
    -key keys/test.key \
    -cert certs/test.crt \
    -port 4433 \
    -tls1_3 \
    -groups frodo976aes \
    -www
```

In the client terminal:

```bash
echo | openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups frodo976aes \
    -CAfile certs/test.crt 2>&1 | \
    grep -E "SSL handshake|group"
```

Note the increased handshake size with the higher security level.

---

## Step 7: Compare AES vs SHAKE Variants

The AES and SHAKE variants have identical security properties but may differ in performance based on your hardware.

### Test SHAKE Variant

Stop the server and start with SHAKE:

```bash
openssl s_server \
    -key keys/test.key \
    -cert certs/test.crt \
    -port 4433 \
    -tls1_3 \
    -groups frodo640shake \
    -www
```

Connect and measure:

```bash
echo | openssl s_client \
    -connect localhost:4433 \
    -tls1_3 \
    -groups frodo640shake \
    -CAfile certs/test.crt 2>&1 | \
    grep "SSL handshake has read"
```

The handshake sizes should be identical between AES and SHAKE variants—the difference is internal computation only.

---

## Step 8: Record Performance Metrics

Create a test results file:

```bash
cat > /opt/sassycorp-pqc-alt/tests/frodokem-results.txt << 'EOF'
FrodoKEM Performance Test Results
=================================
Date: $(date)
System: $(uname -a)

Algorithm         | Client Writes | Server Reads | Notes
------------------|---------------|--------------|-------
frodo640aes       |               |              |
frodo640shake     |               |              |
frodo976aes       |               |              |
frodo976shake     |               |              |
frodo1344aes      |               |              |
frodo1344shake    |               |              |

Fill in your measured values above.
EOF
```

---

## When to Use FrodoKEM

### Appropriate Use Cases

| Use Case | Why FrodoKEM |
|----------|--------------|
| **Government/Defense** | Maximum security assurance required |
| **Long-term secrets** | Data must remain secure for 50+ years |
| **European compliance** | BSI/ANSSI recommendations |
| **Defense in depth** | Backup to ML-KEM using different math |
| **HSM environments** | FPGA acceleration makes it practical |

### Inappropriate Use Cases

| Use Case | Why Not FrodoKEM |
|----------|------------------|
| **High-volume TLS** | Performance overhead too high |
| **IoT/embedded** | Key sizes exceed memory constraints |
| **Mobile apps** | Bandwidth costs for handshakes |
| **Latency-sensitive** | Handshake delay unacceptable |

### Decision Framework

```
Need maximum security assurance?
├── Yes → Can accept 4-5x larger handshakes?
│         ├── Yes → Use FrodoKEM
│         └── No → Use ML-KEM, accept structured lattice risk
└── No → Use ML-KEM (FIPS 203)
```

---

## FrodoKEM and FPGA Acceleration

For organizations where FrodoKEM's software performance is prohibitive, FPGA acceleration can achieve dramatic improvements:

| Platform | Operations/Second | Relative Speed |
|----------|-------------------|----------------|
| ARM Cortex-M4 (software) | ~20 | Baseline |
| Xilinx Artix-7 (FPGA) | ~840 | 42x faster |
| Xilinx Ultrascale+ (FPGA) | ~3,164 | 158x faster |

Matrix multiplication accounts for 97.5% of FrodoKEM's runtime—parallelization on FPGA dramatically reduces this bottleneck.

---

## Summary

FrodoKEM provides the most conservative lattice-based security at the cost of significant performance and bandwidth overhead:

| Metric | FrodoKEM-640 | ML-KEM-768 | Difference |
|--------|--------------|------------|------------|
| Public key | 9,616 B | 1,184 B | 8x larger |
| Ciphertext | 9,720 B | 1,088 B | 9x larger |
| Handshake total | ~40 KB | ~12 KB | 3x larger |
| Key generation | Very slow | Fast | ~100x slower |
| Security basis | Unstructured LWE | Module-LWE | More conservative |

**Bottom line:** FrodoKEM is for organizations that prioritize security assurance over performance and can accept the overhead. For most applications, ML-KEM provides excellent security with better performance.

---

## Next Steps

Proceed to **[Module 03: NTRU](03-NTRU.md)** to explore the most compact lattice-based KEM with 25+ years of security history.

---

**Module Navigation:**

| Previous | Current | Next |
|----------|---------|------|
| [01 - Environment Setup](01-ENVIRONMENT-SETUP.md) | **02 - FrodoKEM** | [03 - NTRU](03-NTRU.md) |
