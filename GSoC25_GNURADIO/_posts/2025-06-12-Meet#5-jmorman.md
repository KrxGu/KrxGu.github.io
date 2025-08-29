---
title: Mentor Meeting #5 – jmorman
date: 2025-06-12
description: Week-5 sync – Analog modulators implementation, repository transition, and Math PR review.
layout: post
---

# GSoC 2025 · GNU Radio 4.0 Block-Set Expansion  
### Week-5 check-in (mentor — **Josh Morman**)

| | |
|---|---|
| **Date** | 12 June 2025, 17:00 IST |
| **Duration** | 40 min |
| **Participants** | Josh Morman · Krish Gupta |

---

## 1 · Repository Transition News <!--──────────────────────────-->

Josh shared important project updates:

* GNU Radio organization is transitioning from `fair-acc/gnuradio4` to `gnuradio/gnuradio4`
* Existing PRs will need to be rebased or re-submitted to the new repository
* This transition impacts our Math PR but creates a clean slate for Analog and Digital submissions

---

## 2 · Progress update <!--──────────────────────────-->

* **Math PR Review**  
  * Received feedback from maintainers on Math PR
  * Made requested changes to improve type safety and documentation
  * Added more comments explaining CRTP implementation

* **Analog Blocks**  
  * Completed initial implementation of:
    * `FrequencyMod.hpp` with configurable sensitivity
    * `PhaseModulator.hpp` with proper phase wrapping
  * Started on `QuadratureDemod.hpp` implementation
  * All implementations use minimal state and maximize `processOne()` efficiency

---

## 3 · Boost.UT Testing Strategy <!--──────────────────────────-->

Migrated from GoogleTest to Boost.UT for more concise tests:

```cpp
// Before (GoogleTest)
TEST(FrequencyModTest, BasicModulation) {
    FrequencyMod<float> mod(1.0f);
    EXPECT_NEAR(mod.processOne(1.0f), 0.15915f, 0.0001f);
}

// After (Boost.UT)
"BasicModulation"_test = []{
    FrequencyMod<float> mod(1.0f);
    expect(that % mod.processOne(1.0f) == approx(0.15915f, 0.0001f));
};
```

Benefits:
* More concise test code
* Faster compilation time
* Better integration with GR4 testing framework

---

## 4 · Technical Challenges <!--────────────────────────────-->

Discovered and resolved several technical challenges:

| Challenge | Solution |
|-----------|----------|
| **Phase wrapping** | Implemented custom wrapping function that avoids branches for better SIMD vectorization |
| **Complex type handling** | Created specialized templates for complex I/O types |
| **Gain adaptation rates** | Implemented smooth gain adjustment with configurable attack/decay rates |

---

## 5 · Documentation Progress <!--────────────────────────────-->

Major progress on the porting guide:

* Completed first draft of GR3 → GR4 porting patterns
* Added specific section on handling complex types
* Created diagrams showing the lifecycle of a block (initialization → processOne/Bulk → cleanup)
* Documented reflection macro usage with clear examples

---

## 6 · Action items <!--──────────────────────────────────-->

| Owner | Task |
|-------|------|
| **Krish** | • Complete `QuadratureDemod` and `AmDemod` implementations<br>• Begin work on AGC blocks<br>• Rebase Math PR against the new repository when ready |
| **Josh** | • Coordinate repository transition timeline<br>• Review Analog block implementations<br>• Provide feedback on porting guide draft |

---

>**Next sync**: June 19th with focus on completing the Analog demodulators and beginning AGC implementation.

---

## 7 · Implementation Insights <!--────────────────────────────-->

Key pattern emerging in our implementations:

```cpp
template <typename T>
class FrequencyMod : public Block<FrequencyMod<T>> {
private:
    T sensitivity_{1.0};
    // Minimal state, maximum efficiency

public:
    void processOne(const T& in, T& out) {
        // Direct, branch-free implementation
        out = std::sin(sensitivity_ * in);
    }
    
    // Configuration through reflection
    GR_MAKE_REFLECTABLE(FrequencyMod, sensitivity_);
};

GR_REGISTER_BLOCK(FrequencyMod<float>);
GR_REGISTER_BLOCK(FrequencyMod<double>);
```

This pattern ensures:
* Minimal memory footprint
* Maximum vectorization potential
* Type-safe instantiations
* Clean configuration via reflection
