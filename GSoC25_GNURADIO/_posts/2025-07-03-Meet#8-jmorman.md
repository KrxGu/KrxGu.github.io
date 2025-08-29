---
title: "Meet#8 jmorman"
date: 2025-07-03
description: "Weekly progress update for GNU Radio 4.0 Block Set Expansion project."
layout: post
---

# GSoC 2025 · GNU Radio 4.0 Block-Set Expansion  
### Week-8 check-in (mentor — **Josh Morman**)

| | |
|---|---|
| **Date** | 3 July 2025, 17:00 IST |
| **Duration** | 50 min |
| **Participants** | Josh Morman · Krish Gupta |

---

## 1 · Progress update <!--──────────────────────────-->

* **Analog PR Review**  
  * Addressed all feedback from Ralph and Josh on the Analog PR
  * Fixed minor type casting issues highlighted by CI
  * Improved documentation with more detailed examples

* **Digital Components**  
  * Completed `Constellation<N>` implementation with:
    * Stateless slicing functions
    * Distance metrics (Euclidean, Manhattan)
    * Decision boundaries
  * Implemented mapping utilities:
    * `ChunksToSymbols` for bit→symbol mapping
    * `SymbolsToChunks` for symbol→bit mapping
    * `DifferentialEncoder`/`DifferentialDecoder`

---

## 2 · Constellation Implementation <!--──────────────────────────-->

The `Constellation<N>` template represents a significant modernization:

```cpp
template <size_t N>
class Constellation {
private:
    std::array<std::complex<float>, N> points_;
    std::array<uint32_t, N> symbols_;
    float scaling_{1.0f};

public:
    GR_MAKE_REFLECTABLE(Constellation, points_, symbols_, scaling_);
    
    // Stateless slicing - perfect for SIMD
    template <typename T>
    uint32_t slice(const std::complex<T>& sample) const {
        // Find nearest constellation point
        size_t min_index = 0;
        T min_distance = std::numeric_limits<T>::max();
        
        for (size_t i = 0; i < N; ++i) {
            T dist = distance(sample, points_[i]);
            if (dist < min_distance) {
                min_distance = dist;
                min_index = i;
            }
        }
        
        return symbols_[min_index];
    }
    
    // Common constellation factories
    static Constellation<4> make_qpsk();
    static Constellation<8> make_8psk();
    static Constellation<16> make_qam16();
    // ...
};
```

This approach:
* Uses fixed-size arrays for better optimization
* Provides factory methods for common constellations
* Supports custom symbol mappings
* Enables direct SIMD optimization

---

## 3 · CI Challenges for Digital PR <!--──────────────────────────-->

Given the size of the Digital blocks PR (projected 4k+ LOC), we discussed CI optimization strategies:

| Challenge | Solution |
|-----------|----------|
| **Build time** | Per-subdirectory CMake targets |
| **Memory usage** | Focused test matrices with selective instantiation |
| **PR size** | Structured commit history with logical grouping |

Josh shared his experience:

> "Large PRs need careful staging. We should structure the Digital PR with a clear progression: core components → mapping → timing → specialized blocks. This makes review much more manageable."

---

## 4 · Porting Guide Completion <!--────────────────────────────-->

Finalized the initial version of the porting guide with:

* Complete GR3 → GR4 migration patterns
* Code examples for each block family
* Common pitfalls and solutions
* Best practices for state management
* Guidelines for testing

The guide is available in both HTML and Markdown formats for maximum accessibility.

---

## 5 · Template Diagnostics Improvements <!--────────────────────────────-->

Addressed template diagnostic challenges across different compilers:

| Issue | Solution |
|-------|----------|
| **Cryptic GCC errors** | Added explicit typedefs and concepts |
| **Clang/GCC differences** | Used cleaner concept constraints |
| **Type conversion warnings** | Implemented systematic conversion handling |

These improvements make the codebase more maintainable and easier to debug.

---

## 6 · Action items <!--──────────────────────────────────-->

| Owner | Task |
|-------|------|
| **Krish** | • Finalize Digital core and mapping components<br>• Prepare Digital blocks PR with structured commit history<br>• Update CI configuration for focused testing |
| **Josh** | • Continue reviewing Analog PR details<br>• Provide feedback on Constellation implementation<br>• Coordinate with CI team on large PR strategy |

---

>**Next sync**: July 10th to review Digital PR preparation and Constellation implementation details.

---

## 7 · Learning from Analog PR Review <!--────────────────────────────-->

The Analog PR review process highlighted several important patterns:

1. **Type safety** is critical - explicit conversions are better than implicit
2. **Documentation** should include concrete examples
3. **Parameter naming** should be consistent across block families
4. **Edge cases** need explicit handling (NaN, Inf, extreme values)
5. **CI configuration** needs careful tuning for template-heavy code

These learnings are being applied to the Digital blocks implementation to ensure a smoother review process.
