# Addendum: PQC Environment Setup

This addendum provides instructions for building and installing the Open Quantum Safe (OQS) provider to access alternative post-quantum algorithms (FrodoKEM, BIKE, HQC) alongside OpenSSL 3.5's native NIST algorithms.

<br>

## Installing the OQS Provider on Ubuntu 25.10 with OpenSSL 3.5.x

### What You Get

**Native OpenSSL 3.5.x algorithms (no OQS required):**

| Algorithm | Type | Standard |
| ----------- | ------ | ---------- |
| ML-KEM-512/768/1024 | KEM | FIPS 203 |
| ML-DSA-44/65/87 | Signature | FIPS 204 |
| SLH-DSA (all variants) | Signature | FIPS 205 |
| X25519MLKEM768 | Hybrid KEM | Native |

**OQS provider adds:**

| Algorithm | Type | Notes |
| ----------- | ------ | ------- |
| FrodoKEM | KEM | Conservative unstructured lattice |
| BIKE | KEM | Code-based, compact |
| HQC | KEM | NIST Round 4 (expected 2027), requires explicit enable |

**Note:** *Support for NTRU and Classic McEliece is currently not enabled in the current oqs-provider releases since they were removed from the initial NIST competition. If you require these algorithms, you'll need to use an older oqs-provider version or access liboqs directly. This is not currently in scope for the supporting lab modules.  For now.*

<br>

## Prerequisites

### Verify Ubuntu Version

```bash
lsb_release -d
```

**Expected output:**

```bash
Description:    Ubuntu 25.10
```

### Verify OpenSSL Version

```bash
openssl version
```

**Expected output:**

```bash
OpenSSL 3.5.x <date>
```

### Verify Native PQC Support

Confirm ML-KEM is available natively:

```bash
openssl list -kem-algorithms | grep -i mlkem
```

**Expected output:**

```
MLKEM512 @ default
MLKEM768 @ default
MLKEM1024 @ default
X25519MLKEM768 @ default
```

<br>

## Step 1: Install Build Dependencies

Update package lists:

```bash
sudo apt update
```

Install required build tools and libraries:

```bash
sudo apt install -y build-essential cmake ninja-build git libssl-dev pkg-config python3 python3-pip python3-pytest-xdist python3-tabulate astyle
```

**Package purposes:**

| Package | Purpose |
|---------|---------|
| build-essential | GCC, make, standard build tools |
| cmake | Build system for liboqs and oqs-provider |
| ninja-build | Fast build system |
| git | Source code retrieval |
| libssl-dev | OpenSSL development headers |
| pkg-config | Library detection (required for liboqs) |
| python3 | Build script requirements |
| python3-pip | Python package manager |
| python3-pytest-xdist | Required for parallel test execution |
| python3-tabulate | Required for oqs-provider code generation |
| astyle | Code formatting |

<br>

## Step 2: Clone and Build liboqs

liboqs is the underlying library that implements the alternative PQC algorithms.

### Clone the Repository

```bash
cd ~ && git clone --depth 1 https://github.com/open-quantum-safe/liboqs.git && cd liboqs
```

### Create Build Directory and Configure

```bash
mkdir build && cd build
```

```bash
cmake -GNinja -DCMAKE_INSTALL_PREFIX=/usr/local -DBUILD_SHARED_LIBS=ON -DOQS_ENABLE_KEM_HQC=ON ..
```

**Configuration options:**

| Option | Purpose |
| -------- | --------- |
| -GNinja | Use Ninja build system (faster) |
| -DCMAKE_INSTALL_PREFIX=/usr/local | Install to standard system location |
| -DBUILD_SHARED_LIBS=ON | Build shared libraries |
| -DOQS_ENABLE_KEM_HQC=ON | Enable HQC (disabled by default in liboqs) |

**Algorithm defaults:** FrodoKEM and BIKE are enabled by default. HQC is disabled for key encapsulation by default in liboqs and must be explicitly enabled with the cmake flag above (which we do). It was disabled by default in oqsprovider v0.9.0 until a [bug can be fixed](https://groups.google.com/a/list.nist.gov/g/pqc-forum/c/Wiu4ZQo3fP8). To verify which algorithms are enabled:

```bash
grep OQS_ENABLE_KEM CMakeCache.txt | grep -v "^//"
```

**Note:** NTRU and Classic McEliece are no longer supported in oqs-provider 0.8.0+, even though they exist in liboqs.

### Build liboqs

```bash
ninja
```

Build time varies by system (typically 2-5 minutes).

### Run Tests (Optional but Recommended)

```bash
ninja run_tests
```

All tests should pass. Minor failures in specific algorithm tests are usually acceptable.

### Install liboqs

```bash
sudo ninja install
```

### Update Library Cache

```bash
sudo ldconfig
```

### Verify Installation

Check that liboqs was installed:

```bash
ls -la /usr/local/lib/liboqs* 2>/dev/null || ls -la /usr/local/lib64/liboqs* 2>/dev/null || echo "Checking alternate locations..."
```

If no files are found, check if it installed elsewhere:

```bash
sudo find /usr -name "liboqs*" -type f 2>/dev/null
```

**Note:** On some systems, liboqs installs to `/usr/local/lib64/` instead of `/usr/local/lib/`. If you find it there, update the library path:

```bash
echo "/usr/local/lib64" | sudo tee /etc/ld.so.conf.d/liboqs.conf
sudo ldconfig
```

**Alternative:** If no shared library files are found but the build completed successfully, the OQS provider may statically link liboqs. This is fine—proceed to Step 3 and verify the provider works in Step 5.

<br>

## Step 3: Clone and Build OQS Provider

### Clone the Repository

```bash
cd ~ && git clone --depth 1 https://github.com/open-quantum-safe/oqs-provider.git && cd oqs-provider
```

### Enable HQC Algorithm

By default, HQC is only enabled for TLS key exchange, not as a standalone KEM. To enable HQC as a KEM algorithm, edit the generate.yml template:

```bash
vim oqs-template/generate.yml
```

Find the HQC entries (search for `hqc128`, `hqc192`, `hqc256`) and change `enable_kem: false` to `enable_kem: true` for each:

```yaml
# Before:
    enable_kem: false
    enable_tls: true

# After:
    enable_kem: true
    enable_tls: true
```

Save and exit (`:wq` in vim).

### Regenerate Provider Code

Set the liboqs source directory and regenerate:

```bash
export LIBOQS_SRC_DIR=~/liboqs
```

```bash
python3 oqs-template/generate.py
```

**Expected output:**

```
All files generated
```

### Create Build Directory and Configure

```bash
mkdir build && cd build
```

```bash
cmake -GNinja -DCMAKE_INSTALL_PREFIX=/usr/local -Dliboqs_DIR=/usr/local ..
```

### Build the Provider

```bash
ninja
```

### Verify the Provider Built Successfully

```bash
ls -la lib/oqsprovider.so
```

**Expected output:**

```
-rwxr-xr-x 1 <user> <group> <size> <date> lib/oqsprovider.so
```

<br>

## Step 4: Install the OQS Provider

### Determine Your Architecture

Check your system architecture:

```bash
dpkg --print-architecture
```

- **amd64** → Use `/usr/lib/x86_64-linux-gnu/ossl-modules/`
- **arm64** → Use `/usr/lib/aarch64-linux-gnu/ossl-modules/`

### Locate OpenSSL Providers Directory

Find where OpenSSL expects providers:

```bash
openssl version -d
```

Note the OPENSSLDIR path.

List existing providers:

For x86_64 (amd64):

```bash
ls /usr/lib/x86_64-linux-gnu/ossl-modules/
```

For ARM64 (aarch64):

```bash
ls /usr/lib/aarch64-linux-gnu/ossl-modules/
```

### Copy Provider to System Location

**For x86_64 (amd64):**

```bash
sudo cp ~/oqs-provider/build/lib/oqsprovider.so /usr/lib/x86_64-linux-gnu/ossl-modules/
```

**For ARM64 (aarch64):**

```bash
sudo cp ~/oqs-provider/build/lib/oqsprovider.so /usr/lib/aarch64-linux-gnu/ossl-modules/
```

### Set Permissions

**For x86_64:**

```bash
sudo chmod 644 /usr/lib/x86_64-linux-gnu/ossl-modules/oqsprovider.so
```

**For ARM64:**

```bash
sudo chmod 644 /usr/lib/aarch64-linux-gnu/ossl-modules/oqsprovider.so
```

### Verify Installation

**For x86_64:**

```bash
ls -la /usr/lib/x86_64-linux-gnu/ossl-modules/oqsprovider.so
```

**For ARM64:**

```bash
ls -la /usr/lib/aarch64-linux-gnu/ossl-modules/oqsprovider.so
```

---

## Step 5: Test Provider Before Configuring System-Wide

**CRITICAL:** Before modifying `/etc/ssl/openssl.cnf`, test that the provider loads correctly. A broken openssl.cnf will break SSH and other OpenSSL-dependent services.

### Test Provider Loading Manually

```bash
openssl list -providers -provider oqsprovider -provider default
```

**Expected output:**

```
Providers:
  default
    name: OpenSSL Default Provider
    version: 3.5.x
    status: active
  oqsprovider
    name: OpenSSL OQS Provider
    version: 0.x.x
    status: active
```

If this command fails, **DO NOT proceed to Step 6**. Troubleshoot the provider installation first.

### Test Algorithm Availability

```bash
openssl list -kem-algorithms -provider oqsprovider -provider default | grep -i frodo
```

You should see FrodoKEM variants listed. If not, the provider is not loading correctly.

<br>

## Step 6: Configure OpenSSL to Load OQS Provider System-Wide

Only proceed if Step 5 was successful.

### Locate the Correct Configuration File

On Ubuntu, check where OpenSSL looks for its configuration:

```bash
openssl version -d
```

This typically shows `/usr/lib/ssl`. However, `/usr/lib/ssl/openssl.cnf` is usually a **symlink** to `/etc/ssl/openssl.cnf`:

```bash
ls -la /usr/lib/ssl/openssl.cnf
```

**Expected output:**

```
lrwxrwxrwx 1 root root 20 <date> /usr/lib/ssl/openssl.cnf -> /etc/ssl/openssl.cnf
```

**Important:** Always edit the actual file (`/etc/ssl/openssl.cnf`), not the symlink. The symlink cannot be edited directly.

### Backup the Original Configuration

```bash
sudo cp /etc/ssl/openssl.cnf /etc/ssl/openssl.cnf.backup
```

### Edit OpenSSL Configuration

```bash
sudo vim /etc/ssl/openssl.cnf
```

The default Ubuntu openssl.cnf already has a `[provider_sect]` section. You only need to add two things:

**1. Find the `[provider_sect]` section and add the `oqsprovider` line:**

```ini
[provider_sect]
default = default_sect
oqsprovider = oqsprovider_sect
```

**2. Add the `[oqsprovider_sect]` section after `[default_sect]`:**

```ini
[default_sect]
activate = 1

[oqsprovider_sect]
activate = 1
```

**Complete example of the modified sections:**

```ini
# List of providers to load
[provider_sect]
default = default_sect
oqsprovider = oqsprovider_sect
# The fips section name should match the section name inside the
# included fipsmodule.cnf.
# fips = fips_sect

# If no providers are activated explicitly, the default one is activated implicitly.
# See man 7 OSSL_PROVIDER-default for more details.
#
# If you add a section explicitly activating any other provider(s), you most
# probably need to explicitly activate the default provider, otherwise it
# becomes unavailable in openssl.  As a consequence applications depending on
# OpenSSL may not work correctly which could lead to significant system
# problems including inability to remotely access the system.
[default_sect]
activate = 1

[oqsprovider_sect]
activate = 1
```

Save and exit (`:wq` in vim).

### Reload the Library Cache

After modifying openssl.cnf, update the shared library cache:

```bash
sudo ldconfig
```

### Verify Configuration Immediately

Test OpenSSL immediately after saving:

```bash
openssl list -providers
```

**Expected output:**

```
Providers:
  default
    name: OpenSSL Default Provider
    version: 3.5.x
    status: active
  oqsprovider
    name: OpenSSL OQS Provider
    version: 0.x.x
    status: active
```

If this fails, immediately restore the backup:

```bash
sudo cp /etc/ssl/openssl.cnf.backup /etc/ssl/openssl.cnf
```

<br>

## Step 7: Verify Alternative Algorithm Access

### List FrodoKEM Algorithms

```bash
openssl list -kem-algorithms | grep -i frodo
```

**Expected output (partial):**

```
frodo640aes @ oqsprovider
frodo640shake @ oqsprovider
frodo976aes @ oqsprovider
frodo976shake @ oqsprovider
frodo1344aes @ oqsprovider
frodo1344shake @ oqsprovider
```

### List BIKE Algorithms

```bash
openssl list -kem-algorithms | grep -i bike
```

**Expected output:**

```
bikel1 @ oqsprovider
p256_bikel1 @ oqsprovider
x25519_bikel1 @ oqsprovider
bikel3 @ oqsprovider
p384_bikel3 @ oqsprovider
x448_bikel3 @ oqsprovider
bikel5 @ oqsprovider
p521_bikel5 @ oqsprovider
```

### List HQC Algorithms

```bash
openssl list -kem-algorithms | grep -i hqc
```

**Expected output:**

```
hqc128 @ oqsprovider
hqc192 @ oqsprovider
hqc256 @ oqsprovider
```

**Note:** If HQC algorithms are missing, verify that you edited `generate.yml` to set `enable_kem: true` for all HQC entries and ran `python3 oqs-template/generate.py` before building.

<br>

## Troubleshooting

### Provider Not Loading

**Problem:** `openssl list -providers` doesn't show oqsprovider

**Solutions:**

1. Verify the .so file exists and has correct permissions:
   ```bash
   # x86_64:
   ls -la /usr/lib/x86_64-linux-gnu/ossl-modules/oqsprovider.so
   
   # ARM64:
   ls -la /usr/lib/aarch64-linux-gnu/ossl-modules/oqsprovider.so
   ```

2. Check for library loading errors:
   ```bash
   openssl list -providers -provider oqsprovider -provider default 2>&1
   ```

3. Verify liboqs is installed and library cache is updated:
   ```bash
   sudo ldconfig -p | grep oqs
   ```

### OpenSSL Broken After Editing openssl.cnf

**Problem:** SSH or other OpenSSL-dependent services fail after editing openssl.cnf (ask me how I found this out...)

**Solutions:**

1. If you can still access the system (console, recovery mode):
   ```bash
   sudo cp /etc/ssl/openssl.cnf.backup /etc/ssl/openssl.cnf
   sudo ldconfig
   ```

2. Common causes:
   - Provider .so file doesn't exist at the specified path
   - Syntax errors in the configuration (missing brackets, typos)
   - Wrong architecture path (x86_64 vs aarch64)
   - Missing `[oqsprovider_sect]` section

3. Check for syntax errors:

   ```bash
   openssl version 2>&1
   ```
  
   This will output any configuration parsing errors.

### liboqs Not Found

**Problem:** Provider fails to load with library errors

**Solution:** Verify liboqs installation and update library cache:

```bash
ls -la /usr/local/lib/liboqs*
sudo ldconfig
sudo ldconfig -p | grep oqs
```

### Algorithm Not Available

**Problem:** Specific algorithm not showing in list

**Solutions:**

1. Check OQS provider version supports the algorithm
2. Some algorithms may be disabled at compile time
3. Rebuild liboqs with specific algorithm enabled:

   ```bash
   cmake -GNinja -DCMAKE_INSTALL_PREFIX=/usr/local -DBUILD_SHARED_LIBS=ON -DOQS_ENABLE_KEM_FRODOKEM=ON ..
   ```

### Tests Fail with pytest Error

**Problem:** `ninja run_tests` fails with pytest-related errors

**Solution:** Ensure pytest-xdist is installed:

```bash
sudo apt install -y python3-pytest-xdist
```

<br>

## Cleanup (Optional)

After successful installation, you can remove the build directories to save space:

```bash
rm -rf ~/liboqs ~/oqs-provider
```

<br>

## OpenSSL and OQS Compatibility Matrix

### OpenSSL Version Support

| OpenSSL Version | OQS Provider Support | Native NIST PQC | TLS 1.3 Signatures | Notes |
|-----------------|---------------------|-----------------|-------------------|-------|
| 3.0.0 - 3.0.13 | ✅ Yes | ❌ No | ❌ No | Groups limit bug present |
| 3.0.14+ | ✅ Yes | ❌ No | ❌ No | Groups limit bug fixed |
| 3.1.0 - 3.1.5 | ✅ Yes | ❌ No | ❌ No | Groups limit bug present |
| 3.1.6+ | ✅ Yes | ❌ No | ❌ No | Groups limit bug fixed |
| 3.2.0 - 3.2.1 | ✅ Yes | ❌ No | ✅ Yes | Groups limit bug present |
| 3.2.2+ | ✅ Yes | ❌ No | ✅ Yes | Groups limit bug fixed |
| 3.3.0+ | ✅ Yes | ❌ No | ✅ Yes | Stable |
| 3.4.x | ✅ Yes | ❌ No | ✅ Yes | Stable |
| 3.5.x | ✅ Yes | ✅ Yes | ✅ Yes | Native ML-KEM, ML-DSA, SLH-DSA |

### OQS Provider and liboqs Version Alignment

**Note** *At time of writing, liboqs was v0.15 and oqsprovider was rolled "back" from v0.11rc1 to .012.0-dev (release still 0.11). We'll revisit this addedum as OpenSSL, liboqs, and oqsproviders update over time HOPEFULLY adding support for more submissions into NIST.*

| OQS Provider | liboqs | Release Date | Key Features |
| -------------- | -------- | -------------- | -------------- |
| 0.10.0 | 0.14.0 | July 2025 | Current stable; composite signatures removed |
| 0.9.0 | 0.13.0 | May 2025 | OpenSSL 3.5 native algorithm detection |
| 0.8.0 | 0.12.0 | December 2024 | FIPS algorithm names (ML-KEM, ML-DSA) |
| 0.7.0 | 0.11.0 | October 2024 | Algorithm updates |
| 0.6.1 | 0.10.1 | June 2024 | Bug fixes |
| 0.6.0 | 0.10.0 | April 2024 | First standardized PQ algorithms |

### Important Notes

**OpenSSL 3.5+ with OQS Provider:**
- oquprovider version 0.9.0+ automatically disables ML-KEM and ML-DSA when it detects OpenSSL 3.5+ native support
- This prevents algorithm conflicts and ensures you use native implementations for NIST algorithms
- oqsprovider continues to provide FrodoKEM, BIKE, and HQC (with HQC requiring explicit enabling)

**Algorithm Availability:**
- NTRU and Classic McEliece have been removed from oqsprovider
- HQC requires editing `generate.yml` to set `enable_kem: true` before building (ask me how we figured that garbage out)
- FrodoKEM and BIKE are available by default

**Groups Limit Bug:**
- OpenSSL versions before 3.0.14, 3.1.6, 3.2.2, and 3.3.0 have a limit of 44 default TLS groups
- Enabling too many KEMs via OQS provider can cause crashes on affected versions (LETS GO!)
- This is resolved in newer OpenSSL releases (BOOOO!)

**Minimum Requirements:**
- OQS provider requires OpenSSL 3.0.0 or later
- TLS 1.3 signature functionality requires OpenSSL 3.2.0 or later
- For best experience, use OpenSSL 3.5.x with OQS provider 0.10.0+

<br>

## Next Steps

Your environment is now configured with access to alternative PQC algorithms. Continue with one of the learning paths:

**Learning Path 1: FIPS 203/204/205**

For commercial compliance using native OpenSSL 3.5.x algorithms.

**Start here:** [FIPS Path - Module 00: Introduction](fips/00_introduction.md)

<br>

**Learning Path 2: CNSA 2.0**

For government/defense compliance focusing on ML-DSA-65/87 and ML-KEM-768/1024.

**Start here:** [CNSA Path - Module 00: Introduction](CNSA-Path/00-INTRODUCTION.md)

<br>

**Learning Path 3: Alternative PQC Algorithms**

For researchers and organizations requiring algorithm diversity (FrodoKEM, BIKE, HQC).

**Start here:** [Alt Path - Module 00: Introduction](Alt-Path/00-INTRODUCTION.md)

<br>

## Additional Resources

- [Open Quantum Safe Project](https://openquantumsafe.org/)
- [OQS Provider GitHub](https://github.com/open-quantum-safe/oqs-provider)
- [liboqs GitHub](https://github.com/open-quantum-safe/liboqs)
- [OpenSSL 3.5 Documentation](https://www.openssl.org/docs/)
- [NIST Post-Quantum Cryptography](https://csrc.nist.gov/projects/post-quantum-cryptography)

<br>

**Return to:** [README](README.md)
