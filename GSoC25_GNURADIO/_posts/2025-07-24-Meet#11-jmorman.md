---
title: "Meet#11 jmorman"
date: 2025-07-24
description: "Weekly progress update for GNU Radio 4.0 Block Set Expansion project."
layout: post
---

# GSoC 2025 · GNU Radio 4.0 Block-Set Expansion  
### Week-11 check-in (mentor — **Josh Morman**)

| | |
|---|---|
| **Date** | 24 July 2025, 17:00 IST |
| **Duration** | 45 min |
| **Participants** | Josh Morman · Krish Gupta |

---

## 1 · Progress update <!--──────────────────────────-->

* **PR Review Status**  
  * Analog PR received positive feedback with only minor adjustments needed
  * Digital PR initial review highlighted documentation improvements
  * Math PR awaiting final resolution of CI Sonar issues

* **Timing Recovery**  
  * Completed implementation of all planned timing recovery blocks:
    * `MuellerMullerClockRecovery.hpp`
    * `SymbolSync.hpp`
    * `CostasLoop.hpp`
    * `PFBClockSync.hpp` (Polyphase filterbank clock sync)
  * All implementations thoroughly tested with comprehensive test cases

* **OFDM Components**  
  * Started implementation of key OFDM components:
    * `CyclicPrefixer.hpp` - Adds cyclic prefix to OFDM symbols
    * `Serializer.hpp` - Converts parallel subcarriers to serial
    * `CarrierAllocator.hpp` - Maps data to OFDM subcarriers

---

## 2 · Timing Recovery Innovations <!--──────────────────────────-->

Implemented several innovations in the timing recovery blocks:

### PFB Clock Sync Implementation

```cpp
template <typename T>
class PFBClockSync : public Block<PFBClockSync<T>> {
private:
    int nfilts_{32};          // Number of filterbank arms
    float rate_{1.0f};        // Resampling rate
    std::vector<std::vector<T>> filters_; // Filterbank taps
    
    // Minimal state with clear lifecycle
    float d_rate_f{0};
    float d_rate_i{0};
    float d_rate_error{0};
    
    // ... other implementation details

public:
    GR_MAKE_REFLECTABLE(PFBClockSync, nfilts_, rate_, filters_);
    
    void start() {
        d_rate_f = 0;
        d_rate_i = 0;
        d_rate_error = 0;
        // ... other initialization
    }
    
    template <typename InputIt, typename OutputIt>
    void processBulk(InputIt in, size_t n, OutputIt out, size_t& produced) {
        // Implementation of PFB clock synchronization
        // ...
    }
};
```

Key innovations:
* Memory-efficient implementation with minimal state
* Clear separation of configuration and runtime state
* Optimized filter application for better performance
* Robust error handling for edge cases

---

## 3 · OFDM Architecture <!--────────────────────────────-->

Designed a modular OFDM architecture with clear component separation:

```
ofdm/
├── CyclicPrefixer.hpp       # Add/remove cyclic prefix
├── Serializer.hpp           # Parallel<->Serial conversion
├── CarrierAllocator.hpp     # Data/pilot mapping
├── Equalizer/
│   ├── SimpleEqualizer.hpp  # One-tap equalization
│   └── LMSEqualizer.hpp     # Adaptive equalization
└── Sync/
    ├── ChannelEstimator.hpp # Pilot-based estimation
    └── FrameSync.hpp        # Frame synchronization
```

This structure:
* Separates concerns for better maintainability
* Enables focused testing of each component
* Provides clear interfaces between subsystems
* Follows the overall Digital blocks organization

---

## 4 · Testing Innovations <!--────────────────────────────-->

Developed several testing innovations to ensure robust implementations:

| Innovation | Benefit |
|------------|---------|
| **End-to-end test chains** | Verify correct operation in realistic scenarios |
| **Known-good reference data** | Compare against established implementations |
| **Parameter sweep tests** | Ensure stability across configuration space |
| **Edge case testing** | Verify behavior with extreme inputs |

Example of parameter sweep testing:

```cpp
"SymbolSync_ParameterSweep"_test = []{
    // Test across range of gain values
    for (float gain_mu : {0.01f, 0.05f, 0.1f, 0.2f, 0.5f}) {
        for (float gain_omega : {0.001f, 0.01f, 0.1f, 0.25f}) {
            SymbolSync<float> sync(1.0f, gain_mu, gain_omega);
            
            // Run stability test with these parameters
            // ...
            
            expect(that % stable == true);
        }
    }
};
```

---

## 5 · PR Review Feedback <!--────────────────────────────-->

Received and addressed key feedback on PRs:

| PR | Feedback | Resolution |
|----|----------|------------|
| **Analog** | Type casting in `QuadratureDemod` | Implemented explicit casts with range checking |
| **Analog** | Documentation examples needed | Added concrete usage examples |
| **Digital** | Directory structure questions | Clarified rationale in PR description |
| **Digital** | Test coverage gaps | Added additional tests for edge cases |

Josh mentioned:

> "The Analog PR is very close to being ready for merge. Just a few minor type casting issues to fix. The Digital PR will need more time due to its size, but the core components look good so far."

---

## 6 · Action items <!--──────────────────────────────────-->

| Owner | Task |
|-------|------|
| **Krish** | • Address final feedback on Analog PR<br>• Continue OFDM component implementation<br>• Add additional test coverage for Digital PR |
| **Josh** | • Coordinate final approval of Analog PR<br>• Continue reviewing Digital PR components<br>• Provide guidance on OFDM implementation |

---

>**Next sync**: July 31st to review OFDM progress and prepare for final project phase.

---

## 7 · Reflection on Modern C++ Usage <!--────────────────────────────-->

Josh and I discussed how our implementation approach leverages modern C++23 features:

| C++23 Feature | How We Used It | Benefit |
|---------------|----------------|---------|
| **std::span** | Non-owning buffer views | Zero-copy processing |
| **Concepts** | Type constraints | Clearer errors, better documentation |
| **Reflection** | Parameter configuration | Automatic GUI/Python binding |
| **std::simd** | Vectorized processing | Performance without intrinsics |

Our implementations demonstrate the power of modern C++ for DSP applications:
* Type-safe yet flexible interfaces
* Minimal runtime overhead
* Clear parameter configuration
* Maximum optimization opportunities
