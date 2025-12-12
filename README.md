# ZXC: Asymmetric High-Performance Compression

[![Build & Release](https://github.com/hellobertrand/zxc/actions/workflows/build.yml/badge.svg)](https://github.com/hellobertrand/zxc/actions/workflows/build.yml)
[![Code Quality](https://github.com/hellobertrand/zxc/actions/workflows/quality.yml/badge.svg)](https://github.com/hellobertrand/zxc/actions/workflows/quality.yml)
[![Fuzzing](https://github.com/hellobertrand/zxc/actions/workflows/fuzzing.yml/badge.svg)](https://github.com/hellobertrand/zxc/actions/workflows/fuzzing.yml)
[![Benchmark](https://github.com/hellobertrand/zxc/actions/workflows/benchmark.yml/badge.svg)](https://github.com/hellobertrand/zxc/actions/workflows/benchmark.yml)

[![License](https://img.shields.io/badge/license-BSD--3--Clause-blue)](LICENSE)


**ZXC** is a asymmetric high-performance lossless compression library optimized for **Content Delivery** and **Embedded Systems**. 
It is designed to be *"Write Once, Read Many."* By spending more time compressing data during your build process, ZXC allows your application to decompress data significantly faster than LZ4 on ARM devices.

> **Key Result:** On ARM64 (Mobile/Embedded), ZXC decodes **30-40% faster** than LZ4 while producing smaller files.

## Performance Proof

To ensure consistent performance, benchmarks are automatically executed on every commit via GitHub Actions.
We monitor metrics on both **x86_64** (Linux) and **ARM64** (Apple Silicon M1/M2) runners to track compression speed, decompression speed, and ratios.

*(See the [latest benchmark logs](https://github.com/hellobertrand/zxc/actions/workflows/benchmark.yml))*


## The ZXC Advantage

Traditional codecs often force a trade-off between **symmetric speed** (LZ4) and **archival density** (Zstd). Even when using high-compression modes (like LZ4-HC), the decompression throughput remains limited by the format's legacy design.

ZXC breaks this ceiling through **Asymmetric Efficiency**.

Designed for the *"Write-Once, Read-Many"* reality of software distribution and game development, ZXC invests heavy computational power during the build process (on high-end x86 servers) to produce a stream that is mathematically optimal for ARM processors to decode.

ZXC utilizes a computationally intensive encoder to generate a bitstream specifically structured to **maximize decompression throughput**. By performing heavy analysis upfront, the encoder produces a layout optimized for the instruction pipelining and branch prediction capabilities of modern CPUs, particularly ARMv8, effectively offloading complexity from the decoder to the encoder.

*   **Build Time:** You generally compress only once (on CI/CD).
*   **Run Time:** You decompress millions of times (on every user's device). **ZXC respects this asymmetry.**

[**Read the full Technical Whitepaper**](./WHITEPAPER.md)

## Benchmarks

| Compressor | Decompression Speed (Ratio vs LZ4) | Compressed Size (Index LZ4=100) (Lower is Better) |
| :--- | :--- | :--- |
| **zxc 0.1.0 -2** | **1.65x** | **126.91** |
| **zxc 0.1.0 -3** | **1.46x** | **98.43** |
| **zxc 0.1.0 -4** | **1.31x** | **92.62** |
| **zxc 0.1.0 -5** | **1.17x** | **85.94** |
| lz4 1.10.0 --fast -17 | 1.17x | 130.57 |
| lz4 1.10.0 (Ref) | 1.00x | 100.00 |
| lz4hc 1.10.0 -12 | 0.99x | 76.59 |
| snappy 1.2.2 | 0.67x | 100.47 |
| zstd 1.5.7 -1 | 0.35x | 72.59 |

**Benchmark Graph ARM64** : Decompression Performance Comparison on ARM64 Architecture

![Benchmark Graph ARM64](docs/images/benchmark_arm64.png)

### Benchmark ARM64

Benchmarks were conducted using lzbench (from @inikep), compiled with Clang 17.0.0 on macOS Sequoia 15.7.2 (Build 24G325). The reference hardware is an Apple M2 processor (ARM64). All performance metrics reflect single-threaded execution on the standard Silesia Corpus.

| Compressor name         | Compression| Decompress.| Compr. size | Ratio | Filename |
| ---------------         | -----------| -----------| ----------- | ----- | -------- |
| memcpy                  | 50955 MB/s | 52187 MB/s |   211938580 |100.00 | 12 files|
| **zxc 0.1.0 -2**            |   **429 MB/s** |  **7568 MB/s** |   **128031177** | **60.41** | 12 files|
| **zxc 0.1.0 -3**            |   **197 MB/s** |  **6692 MB/s** |    **99295121** | **46.85** | 12 files|
| **zxc 0.1.0 -4**            |   **165 MB/s** |  **5995 MB/s** |    **93431082** | **44.08** | 12 files|
| **zxc 0.1.0 -5**            |  **70.2 MB/s** |  **5363 MB/s** |    **86696245** | **40.91** | 12 files|
| lz4 1.10.0              |   771 MB/s |  4591 MB/s |   100880147 | 47.60 | 12 files|
| lz4 1.10.0 --fast -17   |  1268 MB/s |  5354 MB/s |   131723524 | 62.15 | 12 files|
| lz4hc 1.10.0 -12        |  13.9 MB/s |  4533 MB/s |    77262399 | 36.46 | 12 files|
| zstd 1.5.7 -1           |   639 MB/s |  1614 MB/s |    73229468 | 34.55 | 12 files|
| snappy 1.2.2            |   833 MB/s |  3096 MB/s |   101352257 | 47.82 | 12 files|


### Benchmark x86_64

Benchmarks were conducted using lzbench (from @inikep), compiled with GCC 13.3.0 on Linux 64-bits Ubuntu 24.04. The reference hardware is an AMD EPYC 7B13  processor (x86_64). All performance metrics reflect single-threaded execution on the standard Silesia Corpus.

| Compressor name         | Compression| Decompress.| Compr. size | Ratio | Filename |
| ---------------         | -----------| -----------| ----------- | ----- | -------- |
| memcpy                  | 17788 MB/s | 17846 MB/s |   211938580 |100.00 | 12 files|
| **zxc 0.1.0 -2**            |   **287 MB/s** |  **3954 MB/s** |   **128031177** | **60.41** | 12 files|
| **zxc 0.1.0 -3**            |   **129 MB/s** |  **3257 MB/s** |    **99295121** | **46.85** | 12 files|
| **zxc 0.1.0 -4**            |   **114 MB/s** |  **3060 MB/s** |    **93431082** | **44.08** | 12 files|
| **zxc 0.1.0 -5**            |  **48.3 MB/s** |  **2852 MB/s** |    **86696245** | **40.91** | 12 files|
| lz4 1.10.0              |   535 MB/s |  3388 MB/s |   100880147 | 47.60 | 12 files|
| lz4 1.10.0 --fast -17   |   925 MB/s |  3794 MB/s |   131723524 | 62.15 | 12 files|
| lz4hc 1.10.0 -12        |  10.1 MB/s |  3322 MB/s |    77262399 | 36.46 | 12 files|
| zstd 1.5.7 -1           |   377 MB/s |  1168 MB/s |    73229468 | 34.55 | 12 files|
| snappy 1.2.2            |   531 MB/s |  1521 MB/s |   101464727 | 47.87 | 12 files|

---

## Installation

### Option 1: Download Release (GitHub)

1.  Go to the [Releases page](https://github.com/hellobertrand/zxc/releases).
2.  Download the binary matching your architecture:
    *   `zxc-macos-arm64` for Apple Silicon (M1/M2/M3).
    *   `zxc-linux-x86_64` for standard Linux servers.
    *   `zxc-windows-x86_64.exe` for Windows servers.
3.  Make the binary executable:
    ```bash
    chmod +x zxc-*
    mv zxc-* zxc
    ```

### Option 2: Building from Source

**Requirements:** CMake (3.10+), C Compiler (Clang/GCC C11), Make/Ninja.

```bash
git clone https://github.com/hellobertrand/zxc.git
cd zxc
mkdir build && cd build
cmake .. -DCMAKE_BUILD_TYPE=Release
make
# Binary usage:
./zxc --help
```

---

## Compression Levels

*   **Level 2 or 3 (Fast):** Optimized for real-time assets (Gaming, UI). ~40% faster loading than LZ4 with comparable compression (Level 3).
*   **Level 4 (Balanced):** A strong middle-ground offering efficient compression speed and a ratio superior to LZ4.
*   **Level 5 (Compact):** The best choice for Embedded, Firmware, or Archival. Better compression than LZ4 and significantly faster decoding than Zstd.

---

## Usage

### 1. Command Line Interface (CLI)

The CLI is perfect for benchmarking or manually compressing assets.

```bash
# Basic Compression (Level 3 is default)
zxc -z input_file output_file

# High Compression (Level 5)
zxc -z input_file output_file -l 5

# Decompression
zxc -d compressed_file output_file

# Benchmark Mode (Testing speed on your machine)
zxc -b input_file
```

### 2. C/C++ Integration

Interfacing with ZXC is simple. Include `zxc.h` and link against `libzxc`.

#### Single-Threaded API (Memory Buffers)
Ideal for small assets or simple integrations.

```c
#include "zxc.h"

void compress_asset(const void* src, size_t src_size, void* dst, size_t dst_cap) {
    // Single-shot compression (Level 3) with checksum enabled
    // Returns actual compressed size, or 0 on error
    size_t c_size = zxc_compress(src, dst, src_size, dst_cap, 3, 1);
}

void decompress_asset(const void* src, size_t c_size, void* dst, size_t dst_cap) {
    // Single-shot decompression  with checksum enabled
    // Returns decompressed size, or 0 on error
    size_t d_size = zxc_decompress(src, dst, c_size, dst_cap, 1);
}
```

#### Multi-Threaded API (File Streams)
For large files, use the streaming API to process data in parallel chunks.

```c
#include "zxc.h"

void compress_file_parallel(const char* input_path, const char* output_path) {
    FILE* f_in = fopen(input_path, "rb");
    FILE* f_out = fopen(output_path, "wb");

    // 0 = Auto-detect CPU cores
    // Level 3, Checksum enabled (1)
    zxc_stream_compress(f_in, f_out, 0, 3, 1);

    fclose(f_in);
    fclose(f_out);
}

void decompress_file_parallel(const char* input_path, const char* output_path) {
    FILE* f_in = fopen(input_path, "rb");
    FILE* f_out = fopen(output_path, "wb");

    // 0 = Auto-detect CPU cores
    // Checksum verification enabled (1)
    zxc_stream_decompress(f_in, f_out, 0, 1);

    fclose(f_in);
    fclose(f_out);
}
```

## License & Credits

**ZXC Codec** Copyright © 2025, Bertrand Lebonnois.
Licensed under the **BSD 3-Clause License**. See LICENSE for details.

**Third-Party Components:**
- **xxHash** by Yann Collet (BSD 2-Clause) - Used for high-speed checksums.