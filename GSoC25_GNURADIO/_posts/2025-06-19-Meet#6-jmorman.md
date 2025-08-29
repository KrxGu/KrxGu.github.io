---
title: "Meet#6 jmorman"
date: 2025-06-19
description: "Weekly progress update for GNU Radio 4.0 Block Set Expansion project."
layout: post
---

# GSoC 2025 · GNU Radio 4.0 Block-Set Expansion  
### Week-6 check-in (mentor — **Josh Morman**)

| | |
|---|---|
| **Date** | 19 June 2025, 17:00 IST |
| **Duration** | 50 min |
| **Participants** | Josh Morman · Krish Gupta |

---

## 1 · Progress update <!--──────────────────────────-->

* **Analog Blocks**  
  * **Completed demodulator implementations:**
    * `QuadratureDemod.hpp` for FM demodulation
    * `FmDet.hpp` (slope detector) as an alternative implementation
    * `AmDemod.hpp` with configurable IIR smoothing
  * **Implemented AGC family:**
    * `Agc.hpp` - basic automatic gain control
    * `Agc2.hpp` - dual-rate AGC with separate attack/decay rates
  * All implementations thoroughly tested with Boost.UT

* **Testing Approach**  
  * Created end-to-end tests that verify mod→demod chains
  * Added edge case testing for extreme signal levels
  * Verified correct behavior with rapidly changing signals

---

## 2 · Implementation Highlights <!--──────────────────────────-->

### AmDemod Implementation

```cpp
template <typename T>
class AmDemod : public Block<AmDemod<T>> {
private:
    T alpha_{0.001}; // Smoothing factor
    T last_env_{0};  // Minimal state

public:
    // Configuration via reflection
    GR_MAKE_REFLECTABLE(AmDemod, alpha_);
    
    void start() { last_env_ = 0; }
    
    void processOne(const std::complex<T>& in, T& out) {
        // Envelope detection with IIR smoothing
        T env = std::abs(in);
        last_env_ = alpha_ * env + (1.0 - alpha_) * last_env_;
        out = last_env_;
    }
};

GR_REGISTER_BLOCK(AmDemod<float>);
GR_REGISTER_BLOCK(AmDemod<double>);
```

### Agc2 Implementation

Implemented dual-rate AGC with separate attack/decay rates for better performance in varying signal conditions.

---

## 3 · Challenges Resolved <!--──────────────────────────-->

| Challenge | Solution |
|-----------|----------|
| **State management** | Used minimal state variables with clear lifecycle through `start()/stop()` |
| **Phase unwrapping** | Implemented quadrature demodulation with proper phase difference calculation |
| **Memory pressure in tests** | Split test suites into smaller files to avoid CI memory issues |

---

## 4 · Digital Blocks Planning <!--────────────────────────────-->

Discussed the digital blocks implementation strategy:

* **Directory structure by use-case rather than block type**
* **Core implementation priorities:**
  * LFSR (Linear Feedback Shift Register)
  * CRC (Cyclic Redundancy Check)
  * Scrambler/Descrambler
  * Constellation representation

Josh and I agreed on a modular approach with distinct subdirectories:

```
digital/
├── core/          # Fundamental components (LFSR, CRC, etc.)
├── mapping/       # Symbol mapping/slicing
├── timing/        # Clock recovery, synchronization
├── equalizer/     # Channel equalization
├── ofdm/          # OFDM-specific components
└── compat/        # GR3 compatibility layers
```

This structure allows for focused development and testing.

---

## 5 · Porting Guide Updates <!--────────────────────────────-->

Added key sections to the porting guide:

* **State management patterns** - When to use member variables vs. `start()/stop()`
* **Error handling approaches** - How to handle edge cases gracefully
* **Performance considerations** - Guidelines for SIMD-friendly implementations
* **Reflection best practices** - When and how to expose parameters

---

## 6 · Action items <!--──────────────────────────────────-->

| Owner | Task |
|-------|------|
| **Krish** | • Create PR for completed Analog blocks<br>• Begin implementation of Digital core components<br>• Finalize initial version of porting guide |
| **Josh** | • Review Analog block implementations<br>• Provide feedback on Digital directory structure<br>• Connect with other maintainers about repository transition |

---

>**Next sync**: June 26th to review Analog PR and discuss Digital block implementation details.

---

## 7 · Repository Strategy <!--────────────────────────────-->

With the repository transition to `gnuradio/gnuradio4` in progress, we decided on the following approach:

1. Submit the Analog PR directly to the new repository
2. Eventually rebase the Math PR or submit a fresh PR to the new repository
3. Structure the Digital blocks PR for easier review given its larger scope

Josh mentioned:

> "The Digital blocks PR will be larger and more complex. We should structure it for incremental review, focusing first on the core components that other blocks will depend on."
