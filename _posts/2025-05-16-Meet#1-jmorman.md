---
title: Mentor Meeting #1 – 16 May 2025
description: Notes from the first sync with Josh Morman and a quick primer on GNU Radio 3.x → 4.0 changes
date: 2025-05-16
layout: post
---

# 🚀 GSoC 2025 – GNU Radio 4.0 Block-Set Expansion  
### _Kick-off call recap (Mentor: **Josh Morman**)_  

|   |  |
|---|---|
| **Date** | 16 May 2025 |
| **Duration** | 45 min |
| **Participants** | Josh Morman (mentor) · Krish Gupta (GSoC student) |
| **Goal** | Align on development environment, licensing, workflow, and testing strategy for porting math blocks to GNU Radio 4.0 |

---

## ✅ Action items & decisions

1. **`work()` ➜ `processBulk()` migration**  
   * All legacy blocks use `work()`; new blocks must implement `processBulk()` (or `processOne()` for scalar paths).  
   * Keep vectorisation (`std::simd`) in mind from day 1; design kernels to operate on spans, not individual samples.

2. **Compiler toolchain**  
   * **GCC 14** will be the reference compiler for the whole summer (has complete C++23 _and_ `std::format` support).  
   * CI matrix will still build with Clang ≥ 17, but primary optimisation work happens on GCC 14.

3. **Repository workflow**  
   * I’ll hack in **`gr4-incubator(or whatever ralph likes)`** (a fresh repo living **inside** the overall GNU Radio workspace).  
   * Feature branches stay there until the final “merge week”; then we create one polished PR against the main GR 4.0 tree.  
   * Benefit: avoids noisy intermediate PRs and keeps review focused.

4. **Unit-test philosophy**  
   * Tests must exercise **scalar**, **SIMD**, and **mixed-width** paths.  
   * Minimum target: 95 % line coverage for each new block.  
   * All CI runs under `-march=native -O3 -ftree-vectorize`.

5. **Licensing plan**  
   | Code origin | Licence during development | Final licence |
   |-------------|----------------------------|---------------|
   | Brand-new code | **MIT** (Josh will drop `LICENSE.txt` scaffold) | TBD – can stay MIT |
   | Code copied/adapted from GR 3.x | Must stay **GPL-3.0-or-later** | May re-licence to LGPL if all copyright holders agree |
   | Small snippets (≪ 10 LOC) | MIT, attribute original file | n/a |

---

## 🔄  Quick reference – What actually changed from GR 3.x to 4.0?

| Category | GNU Radio 3.x | GNU Radio 4.0 | Why it helps us |
|----------|---------------|---------------|-----------------|
| **Block API** | `sync_block::work()` | `Block<>::processBulk()` / `processOne()` | Cleaner templates, simpler signatures |
| **Port system** | Fixed, typed by class name (`multiply_const_ff`) | `PortIn<T> / PortOut<T>`, type = template param | One template covers all numeric types |
| **SIMD** | Manual intrinsics or helper (`fast_cc_multiply`) | Automatic via `std::simd` – compile-time dispatch | Free vectorisation on AVX/NEON |
| **Reflection** | None → hand-written setters & XML | `GR_MAKE_REFLECTABLE` exposes ports/props | GUI & Python wrappers auto-generate |
| **Registration** | Factory + separate GRC XML | `GR_REGISTER_BLOCK` one-liner | Zero boilerplate for new blocks |
| **Build system** | CMake + Boost | Meson + no Boost core | Faster compilation, easier cross-compile |
| **Licensing norm** | GPL-3.0-or-later everywhere | Project can mix MIT / LGPL / GPL | Allows permissive licensing for new code |

---

> _“Focus first on exhaustive tests and clean SIMD-friendly kernels; polish GUIs later.”_  
> — Josh Morman, 16 May 2025

---

## Changelog

| Date | Change |
|------|--------|
| 2025-05-16 | Initial meeting notes added |

---

**Next up:** josh will set up my seprate repo under gnu radio workspace and will set mit licences meanwhile i will play around basic blocks
