# Module 05: Performance Analysis and Algorithm Selection

## Overview

We can't end without undersanding WHY we have so much more math to deal with now. Let's take a look at performance comparisons across algorithms covered in this learning path (and a few extra used internationally). Understanding these trade-offs is essential for selecting the right algorithm for your specific requirements.

We analyze key sizes, SSL handshake impacts, data transmission overhead, computational performance, and latency implications—then provide guidance on when each algorithm is most appropriate.

## Learning Objectives

After completing this module, you will be able to:

- Compare key sizes across all alternative PQC algorithms
- Measure SSL handshake overhead for each algorithm
- Understand data transmission size increases
- Identify latency impacts for high-latency networks
- Select the optimal algorithm for specific use cases

<br>

## Comprehensive Key Size Comparison

### Key Encapsulation Mechanisms (KEMs)

| Algorithm | Security Level | Public Key | Secret Key | Ciphertext | Total KEM Overhead |
|-----------|---------------|------------|------------|------------|-------------------|
| **Classical Reference** |
| X25519 | ~128-bit | 32 B | 32 B | 32 B | 64 B |
| **NIST Standards** |
| MLKEM512 | 1 | 800 B | 1,632 B | 768 B | 1,568 B |
| MLKEM768 | 3 | 1,184 B | 2,400 B | 1,088 B | 2,272 B |
| MLKEM1024 | 5 | 1,568 B | 3,168 B | 1,568 B | 3,136 B |
| X25519MLKEM768 | 3 (hybrid) | 1,216 B | 2,432 B | 1,120 B | 2,336 B |
| **FrodoKEM** |
| frodo640aes | 1 | 9,616 B | 19,888 B | 9,720 B | 19,336 B |
| frodo976aes | 3 | 15,632 B | 31,296 B | 15,744 B | 31,376 B |
| frodo1344aes | 5 | 21,520 B | 43,088 B | 21,632 B | 43,152 B |
| **NTRU** |
| ntru_hps2048509 | 1 | 699 B | 935 B | 699 B | 1,398 B |
| ntru_hps2048677 | 3 | 930 B | 1,234 B | 930 B | 1,860 B |
| ntru_hps4096821 | 5 | 1,230 B | 1,590 B | 1,230 B | 2,460 B |
| **Classic McEliece** |
| mceliece348864 | 1 | 261,120 B | 6,492 B | 196 B | 261,316 B |
| mceliece460896 | 3 | 524,160 B | 13,608 B | 156 B | 524,316 B |
| mceliece6688128 | 5 | 1,044,992 B | 13,932 B | 208 B | 1,045,200 B |
| **BIKE** |
| bike1l1fo | 1 | 1,541 B | 3,114 B | 1,573 B | 3,114 B |
| bike1l3fo | 3 | 3,083 B | 6,198 B | 3,115 B | 6,198 B |
| bike1l5fo | 5 | 5,122 B | 10,276 B | 5,154 B | 10,276 B |
| **HQC** |
| hqc128 | 1 | 2,249 B | 2,289 B | 4,497 B | 6,746 B |
| hqc192 | 3 | 4,522 B | 4,562 B | 9,042 B | 13,564 B |
| hqc256 | 5 | 7,245 B | 7,285 B | 14,485 B | 21,730 B |

<br>

## SSL Handshake Impact Analysis

### Measured Handshake Sizes (Security Level 3)

The TLS 1.3 handshake transmits KEM public keys and ciphertexts. Additional overhead comes from certificates, extensions, and protocol messages.

| Algorithm | Client Writes (approx) | Server Reads (approx) | Total Handshake |
|-----------|------------------------|----------------------|-----------------|
| X25519 (classical) | ~350 B | ~10,000 B | ~10,350 B |
| X25519MLKEM768 | ~1,500 B | ~11,000 B | ~12,500 B |
| MLKEM768 | ~1,200 B | ~11,500 B | ~12,700 B |
| ntru_hps2048677 | ~1,100 B | ~11,200 B | ~12,300 B |
| bike1l3fo | ~3,400 B | ~13,500 B | ~16,900 B |
| hqc192 | ~5,000 B | ~19,500 B | ~24,500 B |
| frodo976aes | ~16,500 B | ~48,000 B | ~64,500 B |
| mceliece460896 | ~1,500 B | ~536,000 B | ~537,500 B |

### Handshake Size Multiplier (vs Classical X25519)

| Algorithm | Size Multiplier | Practical Impact |
|-----------|-----------------|------------------|
| X25519MLKEM768 | 1.2x | Negligible |
| MLKEM768 | 1.2x | Negligible |
| ntru_hps2048677 | 1.2x | Negligible |
| bike1l3fo | 1.6x | Minor |
| hqc192 | 2.4x | Moderate |
| frodo976aes | 6.2x | Significant |
| mceliece460896 | 52x | Severe |

<br>

## Latency Impact for High-Latency Networks

### RTT (Round-Trip Time) Considerations

For networks with high latency (satellite, mobile, international), handshake size directly impacts connection establishment time.

**Assumptions:**
- Satellite link: ~600ms RTT, ~1 Mbps bandwidth
- Mobile 3G: ~300ms RTT, ~1 Mbps bandwidth
- Home broadband: ~30ms RTT, ~50 Mbps bandwidth
- Data center: ~1ms RTT, ~10 Gbps bandwidth

### Connection Establishment Time Impact

| Algorithm | Handshake Size | Satellite (600ms RTT) | Mobile 3G (300ms RTT) | Home Broadband |
|-----------|---------------|----------------------|----------------------|----------------|
| X25519 | ~10 KB | 600ms + 80ms = 680ms | 300ms + 80ms = 380ms | 30ms + 2ms |
| MLKEM768 | ~13 KB | 600ms + 104ms = 704ms | 300ms + 104ms = 404ms | 30ms + 2ms |
| ntru_hps2048677 | ~12 KB | 600ms + 96ms = 696ms | 300ms + 96ms = 396ms | 30ms + 2ms |
| bike1l3fo | ~17 KB | 600ms + 136ms = 736ms | 300ms + 136ms = 436ms | 30ms + 3ms |
| hqc192 | ~25 KB | 600ms + 200ms = 800ms | 300ms + 200ms = 500ms | 30ms + 4ms |
| frodo976aes | ~65 KB | 600ms + 520ms = 1,120ms | 300ms + 520ms = 820ms | 30ms + 10ms |
| mceliece460896 | ~538 KB | 600ms + 4,304ms = 4,904ms | 300ms + 4,304ms = 4,604ms | 30ms + 86ms |

### Key Observations

1. **For most applications**, MLKEM768 and NTRU add negligible latency
2. **For satellite/constrained links**, FrodoKEM adds ~0.5 second overhead
3. **Classic McEliece** adds 4+ seconds on constrained links—impractical for dynamic TLS
4. **BIKE and HQC** fall in the middle—acceptable for most applications

<br>

## Computational Performance

### Operations Per Second (Software Implementation)

| Algorithm | Key Generation | Encapsulation | Decapsulation |
|-----------|----------------|---------------|---------------|
| X25519 | ~50,000 | ~50,000 | ~50,000 |
| MLKEM768 | ~15,000 | ~20,000 | ~18,000 |
| ntru_hps2048677 | ~3,000 | ~15,000 | ~15,000 |
| frodo640aes | ~150 | ~200 | ~200 |
| bike1l3fo | ~5,000 | ~8,000 | ~6,000 |
| hqc192 | ~6,000 | ~9,000 | ~7,000 |
| mceliece348864 | ~5 | ~10,000 | ~8,000 |

### Key Generation Performance Issues

| Algorithm | Key Generation Speed | Implication |
|-----------|---------------------|-------------|
| MLKEM768 | Fast | Ephemeral keys practical |
| ntru_hps2048677 | Moderate | Static keys preferred |
| frodo640aes | Slow | Static keys strongly preferred |
| mceliece348864 | **Very slow** | Out-of-band key distribution required |

<br>

## Data Transmission Impact

### Per-Connection Overhead

For each TLS connection established, the KEM overhead is paid once:

| Algorithm | Per-Connection Overhead | 1,000 Connections/day | 1M Connections/day |
|-----------|------------------------|----------------------|-------------------|
| MLKEM768 | 2.3 KB | 2.3 MB | 2.3 GB |
| ntru_hps2048677 | 1.9 KB | 1.9 MB | 1.9 GB |
| bike1l3fo | 6.2 KB | 6.2 MB | 6.2 GB |
| hqc192 | 13.6 KB | 13.6 MB | 13.6 GB |
| frodo976aes | 31.4 KB | 31.4 MB | 31.4 GB |
| mceliece460896 | 524 KB | 524 MB | 524 GB |

<br>

## Algorithm Selection Guide

### Decision Matrix by Use Case

| Use Case | Recommended | Alternative | Avoid |
|----------|-------------|-------------|-------|
| **General TLS** | MLKEM768, X25519MLKEM768 | NTRU, BIKE | FrodoKEM, McEliece |
| **IoT/Embedded** | NTRU | MLKEM512 | FrodoKEM, McEliece |
| **Mobile Apps** | MLKEM768, NTRU | BIKE | FrodoKEM, McEliece |
| **High-Security** | FrodoKEM | Classic McEliece (static) | - |
| **Algorithm Diversity** | HQC + MLKEM | BIKE + MLKEM | - |
| **European Compliance** | FrodoKEM | Classic McEliece | - |
| **Satellite/Space** | Classic McEliece (static) | NTRU | FrodoKEM |
| **Long-Term Storage** | Classic McEliece | FrodoKEM | BIKE, HQC |

### When to Use Each Algorithm

#### NTRU: Best for Compact Keys

```
Use NTRU when:
✓ Key size is critical constraint
✓ Keys are generated infrequently
✓ Bandwidth is limited
✓ Algorithm diversity from ML-KEM desired
```

#### FrodoKEM: Best for Conservative Security

```
Use FrodoKEM when:
✓ Maximum security assurance required
✓ Performance overhead acceptable
✓ European recommendations apply
✓ Unstructured lattice preference
```

#### Classic McEliece: Best for Static Keys

```
Use Classic McEliece when:
✓ Keys distributed out-of-band
✓ Many operations per key
✓ Per-message size critical
✓ Maximum conservative security required
```

#### BIKE: Best Code-Based (Non-NIST)

```
Use BIKE when:
✓ Code-based diversity needed
✓ NIST standard not required
✓ Smaller sizes than HQC preferred
```

#### HQC: Best Code-Based (NIST Standard)

```
Use HQC when:
✓ Code-based diversity needed
✓ NIST standardization required (2027)
✓ Willing to wait for standard
```

<br>

## Performance Summary Table

### Security Level 3 Comparison

| Algorithm | Public Key | Ciphertext | Keygen Speed | Enc/Dec Speed | Best For |
|-----------|------------|------------|--------------|---------------|----------|
| **MLKEM768** | 1,184 B | 1,088 B | Fast | Fast | General use |
| **ntru_hps2048677** | 930 B | 930 B | Moderate | Fast | Compact keys |
| **frodo976aes** | 15,632 B | 15,744 B | Slow | Slow | Max security |
| **mceliece460896** | 524,160 B | 156 B | Very slow | Fast | Static keys |
| **bike1l3fo** | 3,083 B | 3,115 B | Moderate | Moderate | Code diversity |
| **hqc192** | 4,522 B | 9,042 B | Moderate | Moderate | NIST code backup |

### Visual Size Comparison

```
Public Key Sizes (Security Level 3):

NTRU        ████ 930 B
MLKEM768    █████ 1,184 B
BIKE        ██████████████ 3,083 B
HQC         ████████████████████ 4,522 B
FrodoKEM    █████████████████████████████████████████████████████████████████████ 15,632 B
McEliece    ████████████████████████████████████████... (524,160 B -> It's smaller to download Lord of the Rings Extended Edition!)
```

<br>

## Recommendations by Deployment Scenario

### Scenario 1: Enterprise Web Application

**Requirements:** Good security, reasonable performance, NIST compliance

**Recommendation:**
```
Primary: MLKEM768 or X25519MLKEM768 (hybrid)
Backup: HQC (when standardized)
Reason: Best balance of security, performance, and compliance
```

### Scenario 2: High-Security Government System

**Requirements:** Maximum security assurance, compliance flexibility

**Recommendation:**
```
Primary: FrodoKEM (conservative lattice)
Backup: Classic McEliece (different math basis)
Reason: Conservative algorithms with extensive analysis
```

### Scenario 3: IoT Device Fleet

**Requirements:** Minimal bandwidth, constrained memory

**Recommendation:**
```
Primary: NTRU (smallest keys)
Backup: MLKEM512 (NIST standard)
Reason: Compact sizes fit constrained environments
```

### Scenario 4: Algorithm Diversity Strategy

**Requirements:** Defense against single-algorithm compromise

**Recommendation:**
```
Lattice: MLKEM768 (primary) + FrodoKEM (backup)
Code-based: HQC (when standardized)
Reason: Multiple mathematical foundations
```

### Scenario 5: Satellite Communications

**Requirements:** Minimal per-message overhead, static keys acceptable

**Recommendation:**
```
Primary: Classic McEliece (static keys, tiny ciphertexts)
Alternative: NTRU (if dynamic keys needed)
Reason: After initial key distribution, per-message cost is minimal
```

<br>

## Test Your Own Environment

### Performance Testing Script

Create a comprehensive test:

```bash
cat > /opt/sassycorp-pqc-alt/tests/run-all-tests.sh << 'SCRIPT'
#!/bin/bash

echo "PQC Algorithm Performance Comparison"
echo "====================================="
echo "Date: $(date)"
echo "System: $(uname -a)"
echo ""

ALGORITHMS="MLKEM768 ntru_hps2048677 frodo640aes bike1l3fo hqc192"
CERT="/opt/sassycorp-pqc-alt/certs/test.crt"
KEY="/opt/sassycorp-pqc-alt/keys/test.key"

for ALG in $ALGORITHMS; do
    echo "Testing: $ALG"
    
    # Start server in background
    openssl s_server -key $KEY -cert $CERT -port 4433 -tls1_3 -groups $ALG -www &
    SERVER_PID=$!
    sleep 2
    
    # Test connection and measure
    RESULT=$(echo | openssl s_client -connect localhost:4433 -tls1_3 -groups $ALG -CAfile $CERT 2>&1 | grep "SSL handshake has read")
    
    echo "$ALG: $RESULT"
    
    # Stop server
    kill $SERVER_PID 2>/dev/null
    sleep 1
done
SCRIPT
```

**Note:** Run each algorithm test manually as shown in earlier modules for accurate results. The script above is for reference.

<br>

## Summary

### Key Takeaways

1. **MLKEM768 and NTRU** add minimal overhead—suitable for most applications
2. **BIKE and HQC** provide code-based diversity with moderate overhead
3. **FrodoKEM** offers conservative security at significant performance cost
4. **Classic McEliece** is only practical with static key distribution models
5. **Latency-sensitive applications** should avoid FrodoKEM and Classic McEliece
6. **High-volume servers** should consider bandwidth costs at scale

### Final Algorithm Rankings

| Priority | Criteria | Top Choices |
|----------|----------|-------------|
| 1 | General Purpose | MLKEM768, X25519MLKEM768 |
| 2 | Compact Keys | NTRU |
| 3 | Code-Based Diversity | HQC (2027), BIKE |
| 4 | Conservative Security | FrodoKEM |
| 5 | Maximum Security | Classic McEliece (static keys only) |

<br>

## Completion

Congratulations! You have completed the Alternative PQC Algorithms learning path.

You now understand:
- The mathematical foundations of non-NIST PQC algorithms
- Performance trade-offs for each algorithm
- When each algorithm is appropriate
- International PQC standardization landscape
- How to select algorithms for specific requirements

<br>

**Return to:** [Main README](../README.md)

**Other Learning Paths:**
- [FIPS 203/204/205 Path](../FIPS-Path/00-INTRODUCTION.md)
- [CNSA 2.0 Path](../CNSA-Path/00-INTRODUCTION.md)

**Module Navigation:**

| Previous | Current | Next |
|----------|---------|------|
| [04 - International PQC](04_alt_pqc_interational.md) | **05 - Performance Analysis** | [Main README](../README.md) |
