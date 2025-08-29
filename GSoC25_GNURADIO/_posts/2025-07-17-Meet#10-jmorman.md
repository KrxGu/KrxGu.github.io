---
title: "Meet#10 jmorman"
date: 2025-07-17
description: "Weekly progress update for GNU Radio 4.0 Block Set Expansion project."
layout: post
---

# GSoC 2025 · GNU Radio 4.0 Block-Set Expansion  
### Week-10 check-in (mentor — **Josh Morman**)

| | |
|---|---|
| **Date** | 17 July 2025, 17:00 IST |
| **Duration** | 50 min |
| **Participants** | Josh Morman · Krish Gupta |

---

## 1 · Progress update <!--──────────────────────────-->

* **Digital PR Submitted**  
  * Created PR [gnuradio/gnuradio4#6](https://github.com/gnuradio/gnuradio4/pull/6) with Digital block structure and core implementations
  * Includes full directory structure and essential components
  * CI configuration optimized for focused testing

* **Timing Recovery Implementation**  
  * Started implementation of key timing recovery blocks:
    * `MuellerMullerClockRecovery.hpp` - M&M timing recovery
    * `SymbolSync.hpp` - Generic symbol synchronization
    * `CostasLoop.hpp` - Phase-locked loop for carrier recovery

* **CI Improvements**  
  * Successfully resolved memory issues with optimized test matrices
  * Improved build times with focused component testing
  * Added comprehensive documentation to aid reviewers

---

## 2 · Digital PR Highlights <!--──────────────────────────-->

The Digital PR includes:

* **Complete directory structure** organized by functionality
* **Core components:**
  * LFSR, CRC, Scrambler/Descrambler
  * Bits manipulation utilities
  * Constellation representation
* **Mapping utilities:**
  * Chunks to symbols conversion
  * Differential coding
  * Slicers

Josh's initial feedback was positive:

> "The directory structure looks clean and logical. Breaking it down by functionality rather than by block type will make navigation much more intuitive for users."

---

## 3 · Mueller & Muller Implementation <!--──────────────────────────-->

Implemented the Mueller & Muller clock recovery algorithm with modern C++ approaches:

```cpp
template <typename T>
class MuellerMullerClockRecovery : public Block<MuellerMullerClockRecovery<T>> {
private:
    float omega_{1.0f};      // Samples per symbol
    float gain_omega_{0.25f}; // Loop gain for frequency
    float gain_mu_{0.175f};   // Loop gain for phase
    float mu_{0.5f};          // Fractional sample position
    
    // Minimal state with clear lifecycle
    std::complex<T> p_0{0, 0};
    std::complex<T> p_1{0, 0};
    std::complex<T> c_0{0, 0};
    std::complex<T> c_1{0, 0};

public:
    GR_MAKE_REFLECTABLE(MuellerMullerClockRecovery, 
                        omega_, gain_omega_, gain_mu_);
    
    void start() {
        mu_ = 0.5f;
        p_0 = p_1 = c_0 = c_1 = {0, 0};
    }
    
    template <typename InputIt, typename OutputIt>
    void processBulk(InputIt in, size_t n, OutputIt out, size_t& produced) {
        // Implementation of M&M algorithm
        // ...
    }
};

GR_REGISTER_BLOCK(MuellerMullerClockRecovery<float>);
GR_REGISTER_BLOCK(MuellerMullerClockRecovery<double>);
```

This implementation:
* Uses minimal state
* Provides clear parameter configuration
* Handles fractional timing efficiently
* Follows the GR4 lifecycle pattern

---

## 4 · CI Optimization Success <!--────────────────────────────-->

Our CI optimization efforts have paid off:

| Metric | Before | After | Improvement |
|--------|--------|-------|-------------|
| **Memory usage** | ~7GB (OOM errors) | ~4GB (stable) | ~40% reduction |
| **Build time** | ~15 minutes | ~5 minutes | ~67% faster |
| **Test failures** | Intermittent | Stable | Reliable CI |

Key optimizations:
* Focused test matrices with strategic type coverage
* Component-based testing instead of monolithic builds
* Improved template instantiation control
* Better handling of complex types

---

## 5 · Porting Pattern Refinements <!--────────────────────────────-->

Refined our porting patterns based on implementation experience:

| GR3 Pattern | GR4 Pattern | Benefits |
|-------------|-------------|----------|
| **Virtual `work()`** | CRTP with `processOne()/processBulk()` | No virtual overhead, better inlining |
| **Fixed types** | Templates with type lists | One implementation for all types |
| **Manual history** | Minimal state with clear lifecycle | Better optimization, cleaner code |
| **Global functions** | Class static methods | Better encapsulation, namespace control |

These patterns are now well-documented in the porting guide.

---

## 6 · Action items <!--──────────────────────────────────-->

| Owner | Task |
|-------|------|
| **Krish** | • Complete timing recovery implementations<br>• Address initial feedback on Digital PR<br>• Begin work on OFDM components |
| **Josh** | • Coordinate final approval of Analog PR<br>• Review Digital PR core components<br>• Provide guidance on timing recovery implementation |

---

>**Next sync**: July 24th to review timing recovery implementations and Digital PR feedback.

---

## 7 · Strategic Pivot Reflection <!--────────────────────────────-->

Josh and I discussed the strategic pivot from Audio I/O to Math, Analog, and Digital blocks:

> "The pivot to core DSP blocks was definitely the right call. We've unblocked the most critical functionality for signal processing workflows, which will have a much bigger impact on GR4 adoption than Audio I/O alone would have had."

The current implementation roadmap aligns well with community priorities:
1. **Math blocks** - Basic operations that everything else builds on
2. **Analog blocks** - Essential modulation/demodulation for radio workflows
3. **Digital blocks** - Advanced processing for digital communications

This prioritization maximizes the impact of the GSoC project.
