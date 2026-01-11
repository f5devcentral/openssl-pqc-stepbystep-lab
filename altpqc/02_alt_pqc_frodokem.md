# Module 2: FrodoKEM - Conservative Unstructured Lattice KEM

## Overview

FrodoKEM represents the most conservative approach to lattice-based cryptography. Unlike ML-KEM (which uses structured lattices with algebraic ring properties), FrodoKEM is based on [Learning With Errors (LWE)](https://cims.nyu.edu/~regev/papers/lwesurvey.pdf).

This conservative design means FrodoKEM has stronger theoretical security guarantees but significantly larger key sizes and slower performance.

## Learning Objectives

After completing this module, you will be able to:

- Configure TLS connections using FrodoKEM key exchange
- Measure FrodoKEM's performance characteristics
- Understand when FrodoKEM is the appropriate choice
- Compare FrodoKEM variants (AES vs SHAKE)

<br>

## Understanding FrodoKEM

### Structured vs Unstructured Lattices

| Property | ML-KEM (Structured) | FrodoKEM (Unstructured) |
| ---------- | --------------------- | ------------------------ |
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
| --------- | --------------- | ------------ | ------------ | ---------- |
| frodo640aes | 1 (128-bit) | 9,616 B | 9,720 B | General high-security |
| frodo640shake | 1 (128-bit) | 9,616 B | 9,720 B | SHAKE-based PRF |
| frodo976aes | 3 (192-bit) | 15,632 B | 15,744 B | Enhanced security |
| frodo976shake | 3 (192-bit) | 15,632 B | 15,744 B | SHAKE-based PRF |
| frodo1344aes | 5 (256-bit) | 21,520 B | 21,632 B | Maximum security |
| frodo1344shake | 5 (256-bit) | 21,520 B | 21,632 B | SHAKE-based PRF |

**AES vs SHAKE:** The variants differ only in the pseudorandom function used internally. AES variants may be faster on hardware with AES-NI instructions; SHAKE variants align with NIST hash standards.

<br>

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

<br>

## Step 2: List Available FrodoKEM TLS Groups

Check which FrodoKEM groups are available for TLS:

```bash
openssl list -tls-groups | grep -i frodo
```

**Note:** Group availability depends on OQS provider version and configuration.

<br>

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
| -------- | --------- |
| -tls1_3 | Force TLS 1.3 (required for PQC KEMs) |
| -groups frodo640aes | Use FrodoKEM-640 with AES |
| -www | Serve simple HTTP response |

The server is now listening. Open a **new terminal** for client testing.

<br>

## Step 4: Connect with FrodoKEM Key Exchange

In the new terminal, switch to the CA admin:

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
Peer signature type: mldsa65
Negotiated TLS1.3 group: frodo640aes
---
SSL handshake has read 18983 bytes and written 9978 bytes
Verification: OK
```

The presence of `frodo640aes` confirms FrodoKEM was used for key exchange.

Type `GET /` to see the server response, this will also close the connection.

<br>

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
SSL handshake has read 18983 bytes and written 9978 bytes
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

<br>

## Step 8: Record Performance Metrics (optional)

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

<br>

## When to Use FrodoKEM

### Appropriate Use Cases

| Use Case | Why FrodoKEM |
| ---------- | -------------- |
| **Government/Defense** | Maximum security assurance required |
| **Long-term secrets** | Data must remain secure for 50+ years |
| **European compliance** | BSI/ANSSI recommendations |
| **Defense in depth** | Backup to ML-KEM using different math |
| **HSM environments** | FPGA acceleration makes it practical |

### Inappropriate Use Cases

| Use Case | Why Not FrodoKEM |
| ---------- | ------------------ |
| **High-volume TLS** | Performance overhead too high currenty (without FPGA/ASIC acceleration) |
| **IoT/embedded** | Key sizes exceed memory constraints |
| **Mobile apps** | Bandwidth costs for handshakes |
| **Latency-sensitive** | Handshake delay unacceptable |

<br>

## Next Steps

Proceed to **[Module 03: BIKE and HQC](03_alt_pqc_bike_hqc.md)**

---

**Module Navigation:**

| Previous | Current | Next |
|----------|---------|------|
| [01 - Environment Setup](01_alt_pqc_environment.md) | **02 - FrodoKEM** | [03 - Bike and HQC](03_alt_pqc_bike_hqc.mdd) |
