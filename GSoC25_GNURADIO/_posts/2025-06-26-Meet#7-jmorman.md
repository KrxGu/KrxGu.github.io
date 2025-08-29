---
title: "Meet#7 jmorman"
date: 2025-06-26
description: "Weekly progress update for GNU Radio 4.0 Block Set Expansion project."
layout: post
---

# GSoC 2025 · GNU Radio 4.0 Block-Set Expansion  
### Week-7 check-in (mentor — **Josh Morman**)

| | |
|---|---|
| **Date** | 26 June 2025, 17:00 IST |
| **Duration** | 45 min |
| **Participants** | Josh Morman · Krish Gupta |

---

## 1 · Progress update <!--──────────────────────────-->

* **Analog Blocks**  
  * Submitted PR [gnuradio/gnuradio4#5](https://github.com/gnuradio/gnuradio4/pull/5) with complete Analog block implementations
  * Addressed initial feedback from Ralph about parameter naming conventions
  * Improved documentation for each block with usage examples

* **Digital Core Components**  
  * Implemented key digital core components:
    * `Lfsr.hpp` - Linear Feedback Shift Register with configurable polynomial
    * `Crc.hpp` - Cyclic Redundancy Check with common polynomials
    * `Scrambler.hpp` / `Descrambler.hpp` - Bit scrambling with LFSR backing
    * First draft of `Constellation.hpp` template class

---

## 2 · Analog PR Highlights <!--──────────────────────────-->

The Analog PR includes comprehensive implementations:

* **Modulators:** `FrequencyMod`, `PhaseModulator`
* **Demodulators:** `QuadratureDemod`, `FmDet`, `AmDemod`
* **Gain Control:** `Agc`, `Agc2`

All blocks follow the agreed-upon patterns:
* Minimal state
* Clean CRTP implementation
* Extensive testing
* Proper reflection for runtime configuration

Initial feedback has been positive, with only minor style adjustments requested.

---

## 3 · Digital Core Implementation <!--──────────────────────────-->

### LFSR Implementation

Implemented a flexible LFSR with customizable polynomials:

```cpp
template <typename T = uint32_t>
class Lfsr {
private:
    T reg_{1};           // Shift register state
    T mask_{0x00000001}; // Polynomial mask
    T outXor_{0};        // Output XOR mask

public:
    GR_MAKE_REFLECTABLE(Lfsr, mask_, outXor_);
    
    void reset() { reg_ = 1; }
    
    T next() {
        T feedback = 0;
        if (reg_ & 0x1) {
            feedback = mask_;
            reg_ = ((reg_ >> 1) ^ feedback);
        } else {
            reg_ = (reg_ >> 1);
        }
        return reg_ ^ outXor_;
    }
    
    // Support for common polynomials
    static constexpr T POLY_CCITT() { return 0x00008810; }
    static constexpr T POLY_CDMA2K() { return 0x0000cc00; }
};

GR_REGISTER_BLOCK(Lfsr<uint32_t>);
```

This serves as the foundation for scramblers, random number generation, and more.

---

## 4 · Constellation Design <!--────────────────────────────-->

Josh and I spent considerable time discussing the ideal Constellation design:

| Approach | Pros | Cons |
|----------|------|------|
| **C++23 POD class** | Clean memory layout, SIMD-friendly | Requires C++23 for full benefits |
| **Legacy class hierarchy** | Compatible with existing code | Virtual overhead, complex inheritance |
| **Template-based approach** | Type-safe, optimizable | More complex for users |

We settled on a hybrid approach:
* Core `Constellation<N>` template for new code (C++23 POD)
* Thin legacy adapters for backward compatibility
* Stateless slicing/metrics for better optimization

---

## 5 · CI Improvements <!--────────────────────────────────-->

I implemented several CI improvements to address ongoing challenges:

* **Per-subtree CMake targets** to allow focused builds
* **Test splitting** to reduce memory pressure
* **Selective type matrices** to balance coverage and resource usage

These changes have significantly improved CI stability, especially for complex template code.

---

## 6 · Action items <!--──────────────────────────────────-->

| Owner | Task |
|-------|------|
| **Krish** | • Address feedback on Analog PR<br>• Continue Digital core implementations<br>• Begin work on `Constellation` implementation<br>• Update CI configuration for Digital blocks |
| **Josh** | • Review Digital core components<br>• Connect with CI team about forked PR Sonar issues<br>• Provide guidance on Constellation design |

---

>**Next sync**: July 3rd to review Digital core progress and discuss Constellation implementation details.

---

## 7 · Technical Notes <!--────────────────────────────────-->

### CMake Module Structure

We refined the CMake structure to better support modular development:

```cmake
# digital/CMakeLists.txt
add_library(gr4_digital INTERFACE)

# Core components (always built)
add_subdirectory(core)
target_link_libraries(gr4_digital INTERFACE gr4_digital_core)

# Optional components (can be built separately)
if(BUILD_DIGITAL_MAPPING)
    add_subdirectory(mapping)
    target_link_libraries(gr4_digital INTERFACE gr4_digital_mapping)
endif()

# ... other components similarly structured
```

This approach:
* Allows focused CI builds
* Reduces build times during development
* Maintains a clean interface for users
