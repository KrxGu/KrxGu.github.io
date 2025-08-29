---
title: "Meet#4 jmorman"
date: 2025-06-05
description: "Weekly progress update for GNU Radio 4.0 Block Set Expansion project."
layout: post
---

# GSoC 2025 · GNU Radio 4.0 Block-Set Expansion  
### Week-4 check-in (mentor — **Josh Morman**)

| | |
|---|---|
| **Date** | 5 June 2025, 17:00 IST |
| **Duration** | 45 min |
| **Participants** | Josh Morman · Krish Gupta |

---

## 1 · Progress update <!--──────────────────────────-->

* **Math blocks**  
  * **100% of MATH headers** now ported with successful compilation on both GCC 14 and Clang 17
  * Created PR [fair-acc/gnuradio4#595](https://github.com/fair-acc/gnuradio4/pull/595) for Math blocks
  * Completed `Conjugate` SIMD implementation with proper specializations for complex types
  * Finished `Log10` block with edge case handling for negative inputs and NaN/Inf

* **CI workflow**  
  * Discovered memory constraints in GitHub Actions runners (~7GB RAM)
  * Large template instantiation matrices causing OOM errors in Docker builds

---

## 2 · CI Memory Constraints <!--──────────────────────────-->

Identified root causes of CI memory issues:

```cpp
// Original approach causing memory issues
using TestTypes = std::tuple<
    int8_t, int16_t, int32_t, int64_t,
    uint8_t, uint16_t, uint32_t, uint64_t,
    float, double, 
    std::complex<float>, std::complex<double>,
    std::complex<int16_t>, std::complex<int32_t>
>;
```

Solution:

```cpp
// More focused test matrix to reduce memory pressure
using PriorityTypes = std::tuple<
    float, double,
    std::complex<float>, std::complex<double>,
    int32_t, int16_t // Most common integer types
>;
```

* Trimmed type matrices to focus on most common types
* Added selective testing for integer variants
* Improved compiler diagnostic messages by using explicit template instantiations

---

## 3 · Analog Block Planning <!--────────────────────────────-->

Started design for Analog blocks with key considerations:

| Block Family | Design Approach | Implementation Status |
|--------------|-----------------|------------------------|
| **Modulators** | CRTP pattern with `processOne()` | Planning stage |
| **Demodulators** | State management via `start()/stop()` | Planning stage |
| **AGC** | IIR-based approach with optimized gain calculation | Initial research |

Josh suggested we pivot from the original Audio I/O focus to prioritize core DSP blocks:

> "Analog and Digital blocks will unblock more users than Audio I/O. Let's focus there first, as they're the building blocks of most flowgraphs."

---

## 4 · Porting Guide Progress <!--────────────────────────────-->

Started work on a comprehensive porting guide:

* Documented the templating pattern used in Math blocks
* Created sections for reflection macro usage
* Added example transformation of GR3 → GR4 API
* Planning to make it scrapable for future contributors and LLMs

---

## 5 · Action items <!--──────────────────────────────────-->

| Owner | Task |
|-------|------|
| **Krish** | • Address CI feedback on Math PR<br>• Begin implementation of `FrequencyMod` and `PhaseModulator`<br>• Continue work on porting guide |
| **Josh** | • Review Math PR<br>• Coordinate with maintainers on CI memory issues<br>• Provide sample code for AGC block patterns |

---

>**Next sync**: June 12th to review Analog block progress and discuss Digital block directory structure.

---

## 6 · Technical Notes <!--────────────────────────────────-->

Key learnings from this week:

* Template instantiation can quickly explode memory usage in C++23
* CI Docker environments add overhead that further constrains available memory
* CRTP (Curiously Recurring Template Pattern) provides static polymorphism without virtual overhead
* Type matrices need careful pruning to ensure CI stability without sacrificing coverage
