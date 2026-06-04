# Hi, I'm Shengyu Wu

I work on **SIMD optimization** and **ML inference** for **RISC-V** architecture.

## Open Source Contributions — [OpenCV](https://github.com/opencv/opencv)

### [PR #28938](https://github.com/opencv/opencv/pull/28938) — Score-first FAST-16 with RVV intrinsics (Merged, OpenCV 4.14.0)

Replaced the count-based RVV HAL implementation with a **score-first approach**, achieving **2.5–4x speedup** on RISC-V hardware.

| Benchmark | Before | After | Speedup |
|-----------|--------|-------|---------|
| FAST-20 NMS=true (orig.png) | 7.31 ms | 1.91 ms | **3.82x** |
| FAST-20 NMS=true (chess9) | 23.06 ms | 8.19 ms | **2.82x** |
| FAST_DEFAULT (s2.jpg) | 76.94 ms | 28.19 ms | **2.73x** |
| FAST_DEFAULT (a3.png) | 20.40 ms | 5.60 ms | **3.64x** |
| ORB_DEFAULT (chess9) | 64.46 ms | 25.55 ms | **2.52x** |

Tested on Muse Pi v3.0 (SpacemiT K1, RVV 1.0) by OpenCV maintainer.

**Key techniques:**
- Score-first: compute corner scores for all pixels, then threshold — eliminates the separate `cornerScore` pass
- `u8→i16` zero-extension for signed comparison, replacing the error-prone XOR-delta trick
- `vcompress` for batch corner extraction instead of per-lane bitmask scanning
- `vlen`-adaptive via `vsetvl_e16m1` — works on any RVV 1.0 implementation (128/256/512-bit)

**Impact:** The score-first pattern was adopted by OpenCV maintainers for other architectures:
- [AVX2 port (PR #29038)](https://github.com/opencv/opencv/pull/29038)
- [ARM NEON port (PR #29039)](https://github.com/opencv/opencv/pull/29039)

### [PR #29197](https://github.com/opencv/opencv/pull/29197) — Deferred u8mf2 widening for FAST pre-screen (Merged, OpenCV 4.14.0)

Follow-up micro-optimization: defer the `vzext` (u8→i16 widening) in the pre-screen phase until after the quick-reject check. Since ~70-80% of pixel-strips are rejected at the default threshold, this saves 5 unnecessary widen operations per rejected strip.

| Threshold | NMS | Before | After | Speedup |
|-----------|-----|--------|-------|---------|
| 20 | true (orig.png) | 1.88 ms | 1.79 ms | **1.05x** |
| 20 | true (chess9) | 8.50 ms | 7.93 ms | **1.07x** |
| 100 | false (orig.png) | 0.86 ms | 0.73 ms | **1.18x** |
| 100 | false (chess9) | 4.98 ms | 4.23 ms | **1.18x** |

Tested on Banana Pi BPI-F3 (SpacemiT K1, 8-core RVV 1.0).

## Tech Stack

`C/C++` `RISC-V (RVV 1.0)` `Python` `QEMU` `OpenCV` `LLVM`
