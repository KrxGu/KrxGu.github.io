---
title: Mentor Meeting #1 – 16 May 2025
description: Notes from the first sync with Josh Morman and a quick primer on GNU Radio 3.x → 4.0 changes
date: 2025-05-16
description: First catch-up with mentor Josh Morman and a quick GR 3.x → 4.0 primer
layout: post
---

# 🎯 Kick-off call – 16 May 2025  
**Project:** GSoC 2025 – “Expanding the GNU Radio 4.0 Block Set”  
**Mentor:** Josh Morman • **Student:** Krish Gupta  

---

## What we agreed on

| # | Decision |
|---|----------|
| **1** | **`work()` → `processBulk()`** is the new normal. All ports I migrate must follow the 4.0 API and be SIMD-friendly from the outset. |
| **2** | **GCC 14** is our reference compiler (brings full C++23 plus `std::format`). CI will still test Clang ≥ 17, but optimisation happens on GCC. |
| **3** | I’ll develop in a **separate incubator repo** inside the GNU Radio workspace (`gr4-incubator`, or whatever name Ralph prefers). Only when everything is green will we open one polished PR against the main 4.0 tree. |
| **4** | **Unit tests first.** Aim for ~95 % coverage and make sure scalar *and* SIMD code paths run in CI (`-march=native -O3 -ftree-vectorize`). |
| **5** | **Licensing:**<br>— Brand-new code starts as **MIT** (Josh will drop a `LICENSE.txt`).<br>— Anything lifted from GR 3.x stays **GPL-3.0-or-later**. We can later re-license to LGPL if every copyright holder consents.<br>— Tiny snippets (≪ 10 LOC) can remain MIT with attribution. |

---

## Quick GR 3 → 4 cheat-sheet

| Area | 3.x | 4.0 | Win for us |
|------|-----|-----|-----------|
| Block API | `sync_block::work()` | `Block<>::processBulk()` / `processOne()` | simpler kernels, fewer virtual calls |
| Ports | hard-coded (`multiply_const_ff`) | `PortIn<T> / PortOut<T>` | one template covers every numeric type |
| SIMD | manual intrinsics | auto with `std::simd` | free vectorisation |
| Reflection | none; hand-written YAML | `GR_MAKE_REFLECTABLE` | GUI & Python glue generate themselves |
| Registration | factory + GRC YAML | `GR_REGISTER_BLOCK` | zero boilerplate |
| Build | CMake + Boost | Meson, no Boost core | faster compile, easier cross-compile |

---

## Next up

* Josh will scaffold the incubator repo and drop in the MIT licence.
* I’ll prototype a **tiny `AddConst` block** with a matching GoogleTest suite, exercising both scalar and SIMD paths.

Stay tuned!
