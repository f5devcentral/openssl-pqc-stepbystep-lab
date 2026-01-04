# 🔐 Updating OpenSSL for PQC Environments

## 📋 Overview

This addendum provides detailed instructions for enabling post-quantum cryptography support on Ubuntu systems. Choose the path that matches your Ubuntu version and requirements. Yea, we know not everyone is using Ubuntu but this is based on our lab guides and that's what we used. What it will do is give you a better idea of what you might want to do internally with our installations, either updating OpenSSL fully or "simply" adding OQS support as needed.

And [contributions](/contributing.md) welcome! Add your linux flavor or unique use case we're probably forgetting.

| Ubuntu Version | Default OpenSSL | Recommended Path | Result |
|----------------|-----------------|------------------|--------|
| 24.04 LTS (Noble) 🦫 | 3.0.x | Path A: OQS Provider | Adds PQC to system OpenSSL |
| 25.04 (Plucky) 🐧 | 3.4.x | Path B: OpenSSL 3.5 | Parallel install with `openssl-pqc` command |

<br>

## 🧭 Decision Guide

###  Choose Path A (OQS Provider) if:

- ✅ You are running older linux versions and can't just willy nilly upgrade OSs.
- ✅ You want PQC algorithms available system-wide
- ✅ You love compiling libraries into existing packages
- ✅ You prefer using the Open Quantum Safe project's implementation (needed for alternate algorithms)

###  Choose Path B (OpenSSL 3.5) if:

- ✅ You need FIPS/CNSA 2.0 algorithm suites (including ML-DSA-44, SLH-DSA) but don't care about life outside of NIST
- ✅ You want to preserve system OpenSSL while adding PQC capability

<br>

## Path A: OQS Provider for Ubuntu 24.04 LTS

This path installs the Open Quantum Safe (OQS) provider alongside your existing OpenSSL 3.0.x installation, enabling post-quantum algorithms through the provider mechanism.

> 📝 **Note:** The OQS provider now uses the official NIST FIPS standard names (ML-KEM, ML-DSA) rather than the draft competition names (Kyber, Dilithium). Ensure you are using OQS provider version 0.8.0 or later.

## 📋 Prerequisites

Verify your Ubuntu and OpenSSL versions:

```bash
lsb_release -d
```

**Expected output:**

```
Description:    Ubuntu 24.04.x LTS
```

```bash
openssl version
```

**Expected output:**

```
OpenSSL 3.0.x <date>
```

---

## 🛠️ Step A1: Install Build Dependencies

Update package lists:

```bash
sudo apt update
```

Install required build tools and libraries:

```bash
sudo apt install -y build-essential cmake ninja-build git libssl-dev python3 python3-pip astyle pkg-config
```

---

## 📦 Step A2: Clone and Build liboqs

Create a working directory:

```bash
mkdir -p ~/pqc-build
```

```bash
cd ~/pqc-build
```

Clone the liboqs repository:

```bash
git clone --depth 1 https://github.com/open-quantum-safe/liboqs.git
```

```bash
cd liboqs
```

Create build directory and configure:

```bash
mkdir build
```

```bash
cd build
```

```bash
cmake -GNinja -DCMAKE_INSTALL_PREFIX=/usr/local ..
```

Build liboqs: 🔨

```bash
ninja
```

Run tests to verify the build: 🧪

```bash
ninja run_tests
```

Install liboqs:

```bash
sudo ninja install
```

Update the shared library cache:

```bash
sudo ldconfig
```

---

## 📦 Step A3: Clone and Build OQS Provider

Return to the build directory:

```bash
cd ~/pqc-build
```

Clone the OQS provider repository:

```bash
git clone --depth 1 https://github.com/open-quantum-safe/oqs-provider.git
```

```bash
cd oqs-provider
```

Create build directory and configure:

```bash
mkdir build
```

```bash
cd build
```

```bash
cmake -GNinja -DCMAKE_INSTALL_PREFIX=/usr/local ..
```

Build the provider: 🔨

```bash
ninja
```

---

## 📂 Step A4: Install the OQS Provider

Determine the OpenSSL providers directory:

```bash
openssl version -d
```

Note the OPENSSLDIR path (typically `/usr/lib/ssl`).

Find the providers directory:

```bash
ls /usr/lib/x86_64-linux-gnu/ossl-modules/
```

Copy the provider to the correct location:

```bash
sudo cp ~/pqc-build/oqs-provider/build/lib/oqsprovider.so /usr/lib/x86_64-linux-gnu/ossl-modules/
```

Set permissions:

```bash
sudo chmod 644 /usr/lib/x86_64-linux-gnu/ossl-modules/oqsprovider.so
```

---

## ⚙️ Step A5: Configure OpenSSL to Load OQS Provider

Edit the OpenSSL configuration file:

```bash
sudo vim /etc/ssl/openssl.cnf
```

Add the following at the **beginning** of the file, before any other sections:

```ini
# OpenSSL configuration with OQS Provider
openssl_conf = openssl_init

[openssl_init]
providers = provider_sect

[provider_sect]
default = default_sect
oqsprovider = oqsprovider_sect

[default_sect]
activate = 1

[oqsprovider_sect]
activate = 1
```

Save and exit (Ctrl+X, Y, Enter).

> ⚠️ **Important:** Ensure there are no duplicate `openssl_conf` lines in the file. If one exists, modify it rather than adding a new one.

---

## ✅ Step A6: Verify OQS Provider Installation

List available providers:

```bash
openssl list -providers
```

**Expected output:**

```
Providers:
  default
    name: OpenSSL Default Provider
    version: 3.0.x
    status: active
  oqsprovider
    name: OpenSSL OQS Provider
    version: 0.8.x
    status: active
```

List available PQC signature algorithms: ✍️

```bash
openssl list -signature-algorithms -provider oqsprovider | grep -i ml-dsa
```

**Expected output:**

```
  mldsa44 @ oqsprovider
  mldsa65 @ oqsprovider
  mldsa87 @ oqsprovider
  ...
```

List available PQC KEM algorithms: 🔑

```bash
openssl list -kem-algorithms -provider oqsprovider | grep -i mlkem
```

**Expected output:**

```
  mlkem512 @ oqsprovider
  mlkem768 @ oqsprovider
  mlkem1024 @ oqsprovider
  ...
```

---

## 🧪 Step A7: Test PQC Key Generation

Test ML-DSA key generation:

```bash
openssl genpkey -provider oqsprovider -provider default -algorithm mldsa65 -out /tmp/test-mldsa65.key
```

Verify the key:

```bash
openssl pkey -provider oqsprovider -provider default -in /tmp/test-mldsa65.key -noout -text | head -5
```

Clean up: 🧹

```bash
rm /tmp/test-mldsa65.key
```

---

## 📊 Step A8: Algorithm Name Reference

The OQS provider uses the NIST FIPS standard algorithm names:

| FIPS Standard | OQS Provider Name | Security Level | CNSA 2.0 Approved |
|---------------|-------------------|----------------|-------------------|
| ML-DSA-44 (FIPS 204) | mldsa44 | Level 2 | ❌ No |
| ML-DSA-65 (FIPS 204) | mldsa65 | Level 3 | ✅ Yes |
| ML-DSA-87 (FIPS 204) | mldsa87 | Level 5 | ✅ Yes |
| ML-KEM-512 (FIPS 203) | mlkem512 | Level 1 | ❌ No |
| ML-KEM-768 (FIPS 203) | mlkem768 | Level 3 | ✅ Yes |
| ML-KEM-1024 (FIPS 203) | mlkem1024 | Level 5 | ✅ Yes |

**🔀 Hybrid algorithms also available:**
- X25519MLKEM768
- X448MLKEM1024
- SecP256r1MLKEM768
- SecP384r1MLKEM1024

> 🏛️ **Note for CNSA 2.0 compliance:** Only use mldsa65 (ML-DSA-65), mldsa87 (ML-DSA-87), mlkem768 (ML-KEM-768), and mlkem1024 (ML-KEM-1024).

---

## 🎉 Path A Complete!

Your Ubuntu 24.04 LTS system now has PQC support via the OQS provider. When using OpenSSL commands with PQC algorithms, include the provider flags:

```bash
openssl genpkey -provider oqsprovider -provider default -algorithm dilithium3 -out key.pem
```

**Continue to:** [Return to your learning path documentation] ➡️

---

# 🅱️ Path B: OpenSSL 3.5 for Ubuntu 25.04

This path compiles OpenSSL 3.5.x from source and installs it alongside your system OpenSSL, preserving system stability while providing access to native PQC algorithms via the `openssl-pqc` command.

## 📋 Prerequisites

Verify your Ubuntu and OpenSSL versions:

```bash
lsb_release -d
```

**Expected output:**

```
Description:    Ubuntu 25.04
```

```bash
openssl version
```

**Expected output:**

```
OpenSSL 3.4.x <date>
```

---

## 🛠️ Step B1: Install Build Dependencies

Update package lists:

```bash
sudo apt update
```

Install required build tools:

```bash
sudo apt install -y build-essential checkinstall zlib1g-dev libffi-dev perl wget
```

---

## 📥 Step B2: Download OpenSSL 3.5

Create a working directory:

```bash
mkdir -p ~/openssl-build
```

```bash
cd ~/openssl-build
```

Download OpenSSL 3.5.3 source code:

```bash
wget https://github.com/openssl/openssl/releases/download/openssl-3.5.3/openssl-3.5.3.tar.gz
```

Verify the download with SHA256 checksum: 🔍

```bash
sha256sum openssl-3.5.3.tar.gz
```

Compare the output against the checksum published on the [OpenSSL releases page](https://github.com/openssl/openssl/releases).

Extract the archive:

```bash
tar -xzf openssl-3.5.3.tar.gz
```

```bash
cd openssl-3.5.3
```

---

## ⚙️ Step B3: Configure the Build

Configure OpenSSL to install in a separate directory (not replacing system OpenSSL):

```bash
./Configure --prefix=/opt/openssl-3.5.3 --openssldir=/opt/openssl-3.5.3/ssl shared
```

**Configuration options explained:**

| Option | Purpose |
|--------|---------|
| `--prefix=/opt/openssl-3.5.3` | 📂 Installation directory (separate from system) |
| `--openssldir=/opt/openssl-3.5.3/ssl` | 📂 Configuration and certificate directory |
| `shared` | 📚 Build shared libraries |

Review the configuration:

```bash
perl configdata.pm --dump | head -30
```

---

## 🔨 Step B4: Build OpenSSL

Compile OpenSSL (this takes a few minutes depending on your system): ☕

```bash
make -j$(nproc)
```

The `-j$(nproc)` flag parallelizes the build using all available CPU cores.

---

## 🧪 Step B5: Run Tests (Optional but Recommended)

Run the test suite to verify the build:

```bash
make test
```

All tests should pass. Minor failures in network-related tests are acceptable if you're behind a firewall.

---

## 📦 Step B6: Install OpenSSL 3.5

Install to the prefix directory:

```bash
sudo make install
```

Verify installation: ✅

```bash
/opt/openssl-3.5.3/bin/openssl version
```

**Expected output:**

```
OpenSSL 3.5.3 <date>
```

---

## 📚 Step B7: Configure Library Path

Create a library configuration file:

```bash
sudo vim /etc/ld.so.conf.d/openssl-3.5.3.conf
```

Add the following line:

```
/opt/openssl-3.5.3/lib64
```

Save and exit.

Update the shared library cache:

```bash
sudo ldconfig
```

---

## 🔗 Step B8: Create the openssl-pqc Command

Create a symbolic link for easy access:

```bash
sudo ln -sf /opt/openssl-3.5.3/bin/openssl /usr/local/bin/openssl-pqc
```

Verify the command works: ✅

```bash
openssl-pqc version
```

**Expected output:**

```
OpenSSL 3.5.3 <date>
```

Verify system OpenSSL is unchanged: 🔒

```bash
openssl version
```

**Expected output:**

```
OpenSSL 3.4.x <date>
```

---

## 💻 Step B9: Create Shell Alias (Optional)

For convenience, you can add an alias to your shell configuration:

```bash
echo 'alias openssl-pqc="/opt/openssl-3.5.3/bin/openssl"' >> ~/.bashrc
```

```bash
source ~/.bashrc
```

---

## ✅ Step B10: Verify PQC Algorithm Availability

List available ML-DSA signature algorithms: ✍️

```bash
openssl-pqc list -signature-algorithms | grep -i ml-dsa
```

**Expected output:**

```
  ML-DSA-44 @ default
  ML-DSA-65 @ default
  ML-DSA-87 @ default
```

List available SLH-DSA signature algorithms: 🌳

```bash
openssl-pqc list -signature-algorithms | grep -i slh-dsa
```

**Expected output (partial):**

```
  SLH-DSA-SHA2-128f @ default
  SLH-DSA-SHA2-128s @ default
  SLH-DSA-SHA2-192f @ default
  ...
```

List available ML-KEM algorithms: 🔑

```bash
openssl-pqc list -kem-algorithms | grep -i mlkem
```

**Expected output:**

```
  MLKEM512 @ default
  MLKEM768 @ default
  MLKEM1024 @ default
```

List hybrid KEM algorithms: 🔀

```bash
openssl-pqc list -kem-algorithms | grep -i x25519mlkem
```

**Expected output:**

```
  X25519MLKEM768 @ default
```

---

## 🧪 Step B11: Test PQC Key Generation

Test ML-DSA key generation:

```bash
openssl-pqc genpkey -algorithm ML-DSA-65 -out /tmp/test-ml-dsa-65.key
```

Verify the key:

```bash
openssl-pqc pkey -in /tmp/test-ml-dsa-65.key -noout -text | head -5
```

**Expected output:**

```
ML-DSA-65 Private-Key:
priv:
    <hex values>
```

Clean up: 🧹

```bash
rm /tmp/test-ml-dsa-65.key
```

Test ML-KEM key generation:

```bash
openssl-pqc genpkey -algorithm MLKEM768 -out /tmp/test-mlkem768.key
```

Verify and clean up:

```bash
openssl-pqc pkey -in /tmp/test-mlkem768.key -noout -text | head -5
```

```bash
rm /tmp/test-mlkem768.key
```

---

## 🎉 Path B Complete!

Your Ubuntu 25.04 system now has OpenSSL 3.5.3 with native PQC support available via the `openssl-pqc` command. Your system OpenSSL remains unchanged.

**Usage Pattern:**

| Task | Command |
|------|---------|
| 🖥️ System operations | `openssl` (uses 3.4.x) |
| 🔐 PQC operations | `openssl-pqc` (uses 3.5.3) |

**Continue to:** [Return to your learning path documentation] ➡️

---

# 📖 Command Reference Summary

## Path A: OQS Provider Commands

When using the OQS provider on Ubuntu 24.04, include provider flags:

```bash
# 🔑 Generate an ML-DSA-65 key
openssl genpkey -provider oqsprovider -provider default -algorithm mldsa65 -out key.pem

# 📝 Create a CSR
openssl req -provider oqsprovider -provider default -new -key key.pem -out request.csr

# ✍️ Sign a certificate
openssl ca -provider oqsprovider -provider default -in request.csr -out certificate.crt

# 👁️ View a certificate
openssl x509 -provider oqsprovider -provider default -in certificate.crt -noout -text
```

## 🅱️ Path B: OpenSSL 3.5 Commands

When using OpenSSL 3.5 on Ubuntu 25.04, use the `openssl-pqc` command:

```bash
# 🔑 Generate an ML-DSA-65 key
openssl-pqc genpkey -algorithm ML-DSA-65 -out key.pem

# 📝 Create a CSR
openssl-pqc req -new -key key.pem -out request.csr

# ✍️ Sign a certificate
openssl-pqc ca -in request.csr -out certificate.crt

# 👁️ View a certificate
openssl-pqc x509 -in certificate.crt -noout -text
```

---

# 🔄 Algorithm Name Comparison

Both paths use the official NIST FIPS standard names, with slight formatting differences:

| FIPS Standard | OQS Provider (Path A) | OpenSSL 3.5 (Path B) |
|---------------|----------------------|----------------------|
| ML-DSA-44 | mldsa44 | ML-DSA-44 |
| ML-DSA-65 | mldsa65 | ML-DSA-65 |
| ML-DSA-87 | mldsa87 | ML-DSA-87 |
| ML-KEM-512 | mlkem512 | MLKEM512 |
| ML-KEM-768 | mlkem768 | MLKEM768 |
| ML-KEM-1024 | mlkem1024 | MLKEM1024 |
| SLH-DSA-SHA2-128f | slhdsa128f | SLH-DSA-SHA2-128f |

> 📝 **Note:** OpenSSL 3.5 uses hyphenated names (ML-DSA-65), while OQS provider uses lowercase concatenated names (mldsa65). Both refer to the same FIPS 204 standard algorithm.

---

# 🔧 Troubleshooting

## 🅰️ Path A Issues

### ❌ Provider Not Found

**Problem:** `oqsprovider` not listed in providers

**Solution:** Verify the provider file location:
```bash
ls -la /usr/lib/x86_64-linux-gnu/ossl-modules/oqsprovider.so
```

If missing, re-copy from build directory:
```bash
sudo cp ~/pqc-build/oqs-provider/build/lib/oqsprovider.so /usr/lib/x86_64-linux-gnu/ossl-modules/
```

### ❌ Algorithm Not Available

**Problem:** PQC algorithms not listed

**Solution:** Check OpenSSL configuration:
```bash
grep -A 10 "openssl_init" /etc/ssl/openssl.cnf
```

Ensure provider sections are at the top of the file.

### ❌ liboqs Not Found

**Problem:** Provider fails to load with library errors

**Solution:** Verify liboqs installation:
```bash
ls -la /usr/local/lib/liboqs*
sudo ldconfig -p | grep oqs
```

---

## 🅱️ Path B Issues

### ❌ openssl-pqc Command Not Found

**Problem:** `openssl-pqc: command not found`

**Solution:** Verify symbolic link:
```bash
ls -la /usr/local/bin/openssl-pqc
```

Recreate if necessary:
```bash
sudo ln -sf /opt/openssl-3.5.3/bin/openssl /usr/local/bin/openssl-pqc
```

### ❌ Library Loading Errors

**Problem:** `error while loading shared libraries`

**Solution:** Update library cache:
```bash
cat /etc/ld.so.conf.d/openssl-3.5.3.conf
sudo ldconfig
```

### ❌ Build Failures

**Problem:** Compilation errors during `make`

**Solution:** Ensure all dependencies are installed:
```bash
sudo apt install -y build-essential zlib1g-dev libffi-dev perl
```

Clean and reconfigure:
```bash
make clean
./Configure --prefix=/opt/openssl-3.5.3 --openssldir=/opt/openssl-3.5.3/ssl shared
make -j$(nproc)
```

---

# 🧹 Cleanup (Optional)

## 🗑️ Remove Build Directories

After successful installation, you can remove the build directories to save space:

**Path A:**
```bash
rm -rf ~/pqc-build
```

**Path B:**
```bash
rm -rf ~/openssl-build
```

---

# 🚀 Next Steps

After completing your chosen path, return to your learning path documentation:

- [**FIPS 203/204/205 Path:**](/fipsqs/00_fips_quantum_ca_intro.md) Continue with the FIPS Lab
- [**CNSA 2.0 Path:**](/cnsa2/01_cnsa_quantum_ca_intro.md) Continue with the CNSA 2.0 Lab
- [**Alternate PQC Path:**](/altpqc/00_alt_pqc_introduction.md) Continue wit Altnerate PQC Algorithm Lab

Remember to use the appropriate command syntax:
- 🅰️ **Path A (OQS):** Include `-provider oqsprovider -provider default` flags
- 🅱️ **Path B (OpenSSL 3.5):** Use `openssl-pqc` instead of `openssl`
