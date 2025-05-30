---
title: Mentor Meeting #3 – jmorman
date: 2025-05-29
description: Week-3 sync – progress on math-block porting, CMake sketch for the OOT module, and next steps toward SIMD and documentation.
layout: post
---

# GSoC 2025 · GNU Radio 4.0 Block-Set Expansion  
### Week-3 check-in (mentor — **Josh Morman**)

| | |
|---|---|
| **Date** | 29 May 2025, 17:00 IST |
| **Duration** | 30 min |
| **Participants** | Josh Morman · Krish Gupta |

---

## 1 · Progress update <!--──────────────────────────-->

* **Math block**  
  * **75% of MATH headers** were ported and are now compiling/building under GR-4 (`AddConst`, `SubtractConst`,  `MultiplyConst`,  `DivideConst`,  `Add`,  `Subtract`, `Multiply`, `Divide`, `Max`, `Min`,  `And`, `Or`, `Xor`, `Negate`, `Not`, `Abs`, `Integrate`, `Argmax`.
).
  * **`Conjugate` & `Log`** raised type/`std::simd` edge cases ⇒ plan is to split out a dedicated `Conjugate.hpp` that wraps both the scalar and SIMD/complex specialisations.
  * **Unit tests** (`qa_math.cpp`) extended to cover zero-input, NaN/Inf, and extreme-range values.

* **Structure goal**  
  * Keep **one umbrella header** (`math.hpp`) where practical; spin off a separate file only when template specialisation gets messy (e.g., `Conjugate`).

---

## 2 · OOT module bootstrap <!--──────────────────────-->

We clarified how the **`gr-incubator`** out-of-tree module will be wired into the GNURadio workspace:

```cmake
# CMakeLists.txt (top of gr-incubator)
cmake_minimum_required(VERSION 3.20)
project(gr4_incubator LANGUAGES CXX)

# 1. Prevent GR4 from building its own tests/benchmarks
set(BUILD_TESTING OFF CACHE BOOL "" FORCE)
set(GR4_ENABLE_BENCHMARKS OFF CACHE BOOL "" FORCE)

# 2. Bring GR4 headers in, but don't compile its sources
add_subdirectory(gnuradio4 EXCLUDE_FROM_ALL)

# 3. Header-only interface target for includes
add_library(gr4_headers INTERFACE)
target_include_directories(gr4_headers INTERFACE
    ${CMAKE_CURRENT_SOURCE_DIR}/gnuradio4/include
    ${CMAKE_CURRENT_SOURCE_DIR}/gnuradio4           # for <gnuradio4/...>
)

# 4. Your actual blocks live in /lib
add_subdirectory(lib)
Use `target_link_libraries(my_block PRIVATE gr4_headers)` inside `lib/CMakeLists.txt`.
```
Josh will adjust the skeleton so this drops in cleanly.

---

## 3 · Naming & build notes <!--────────────────────────-->

* **Block names** stay **PascalCase** (`Conjugate`, `Log10`).
* **File names** stay underscore-free – `Conjugate.hpp`, not `conjugate.hpp`.
* Ralph’s _multi-header-in-one-file_ style is fine **unless** readability suffers;  
  `Conjugate.hpp` will be the first stand-alone exception.
* Add **SIMD guards** to keep older toolchains happy:  
  `#if defined(__cpp_lib_simd) … #endif`.

---

## 4 · Action items <!--──────────────────────────────-->

| Owner | Task |
|-------|------|
| **Krish** | • Finish SIMD kernel for **Conjugate** block.<br>• Extend edge-case tests for **Log**.<br>• Draft **“Porting 101”** doc for newcomers. |
| **Josh** | • Commit initial **gr-incubator** CMake scaffold.<br>• Ping Ralph for doc-progress confirmation. |

---
>**Next sync**: early next week, once the CMake skeleton lands and SIMD builds succeed on **GCC 14** & **Clang 17**.
