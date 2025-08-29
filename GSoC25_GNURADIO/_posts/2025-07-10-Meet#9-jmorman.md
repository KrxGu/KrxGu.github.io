---
title: "Meet#9 jmorman"
date: 2025-07-10
description: "Weekly progress update for GNU Radio 4.0 Block Set Expansion project."
layout: post
---

# GSoC 2025 · GNU Radio 4.0 Block-Set Expansion  
### Week-9 check-in (mentor — **Josh Morman**)

| | |
|---|---|
| **Date** | 10 July 2025, 17:00 IST |
| **Duration** | 45 min |
| **Participants** | Josh Morman · Krish Gupta |

---

## 1 · Progress update <!--──────────────────────────-->

* **Analog PR Status**  
  * All CI tests now passing after fixing type casting issues
  * Waiting for final approval from maintainers
  * Minor documentation improvements added

* **Digital Blocks**  
  * Finalized directory structure for all digital components
  * Completed core components implementation:
    * `Lfsr.hpp`, `Crc.hpp`, `Scrambler.hpp`, `Bits.hpp`
    * `Constellation<N>` with all common mappings
    * `ChunksToSymbols` and `SymbolsToChunks`
  * Set up CI configuration for digital module with focused test matrices

---

## 2 · Digital Directory Structure <!--──────────────────────────-->

Finalized the directory structure for the digital module:

```
digital/
├── core/          # Core components (LFSR, CRC, Scrambler, etc.)
│   ├── Lfsr.hpp
│   ├── Crc.hpp
│   ├── Scrambler.hpp
│   ├── Constellation.hpp
│   └── detail/
│       └── Bits.hpp  # Bit manipulation utilities
├── mapping/       # Symbol mapping and slicing
│   ├── ChunksToSymbols.hpp
│   ├── SymbolsToChunks.hpp
│   ├── DifferentialEncoder.hpp
│   └── DifferentialDecoder.hpp
├── timing/        # Clock recovery, synchronization
├── measure/       # SNR estimation, EVM, etc.
├── equalizer/     # Channel equalization
├── ofdm/          # OFDM-specific components
├── packet/        # Packet handling
├── misc/          # Miscellaneous utilities
└── compat/        # Compatibility with GR3
```

This organization:
* Groups related functionality together
* Provides clear boundaries between subsystems
* Supports focused testing and CI
* Enables incremental development

---

## 3 · CI Configuration for Digital PR <!--──────────────────────────-->

Implemented a specialized CI configuration for the digital module:

```yaml
# .github/workflows/digital.yml
jobs:
  core:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Build and test core
        run: |
          cmake -B build -DBUILD_TESTING=ON -DGR4_DIGITAL_CORE_ONLY=ON
          cmake --build build --target gr4_digital_core_test
          cd build && ctest -R "digital_core" -V

  mapping:
    needs: core
    runs-on: ubuntu-latest
    steps:
      # Similar structure, focused on mapping tests
      
  # Additional jobs for other components
```

This approach:
* Focuses testing on specific components
* Reduces memory pressure
* Provides faster feedback on PRs
* Makes it easier to identify failures

---

## 4 · Implementation Highlights <!--────────────────────────────-->

### Bits Utility Class

Created a utilities class for efficient bit manipulation:

```cpp
namespace gr::digital::detail {

class Bits {
public:
    // Pack bits into bytes (MSB or LSB first)
    template <typename InputIt, typename OutputIt>
    static void pack(InputIt first, InputIt last, 
                     OutputIt d_first, bool msb_first = true);
    
    // Unpack bytes into bits
    template <typename InputIt, typename OutputIt>
    static void unpack(InputIt first, InputIt last,
                      OutputIt d_first, bool msb_first = true);
    
    // Count set bits in a value
    template <typename T>
    static constexpr int popcount(T value);
    
    // Reverse bit order
    template <typename T>
    static constexpr T reverse(T value);
};

} // namespace gr::digital::detail
```

This utility class provides building blocks for many digital components.

---

## 5 · Addressing Sonar on Forked PRs <!--────────────────────────────-->

Discussed strategies for handling the Sonar CI issue with forked PRs:

| Option | Pros | Cons |
|--------|------|------|
| **Skip Sonar checks** | Unblocks merges | Misses quality checks |
| **Maintainer adjusts CI** | Preserves all checks | Requires organization changes |
| **Clone PR manually** | Works with current system | Extra maintainer effort |

Josh mentioned:

> "For the Digital PR, I'm working with the CI team to temporarily adjust the workflow configuration to allow Sonar checks on external PRs. This should unblock the Math PR as well."

---

## 6 · Action items <!--──────────────────────────────────-->

| Owner | Task |
|-------|------|
| **Krish** | • Prepare and submit Digital blocks PR<br>• Continue implementing remaining digital components<br>• Update test coverage for edge cases |
| **Josh** | • Coordinate with CI team on Sonar issues<br>• Review Digital core components<br>• Facilitate final approval of Analog PR |

---

>**Next sync**: July 17th to review Digital PR and discuss remaining implementation priorities.

---

## 7 · Key Decisions <!--────────────────────────────────-->

Several important decisions were made during this week's development:

1. **Focus on quality over quantity** - Prioritize robust implementations of core components
2. **Modular directory structure** - Organize by functionality for better maintainability
3. **Focused CI configuration** - Optimize CI for faster feedback and better resource usage
4. **Documentation-first approach** - Ensure all components are well-documented
5. **Type-safe interfaces** - Use modern C++ features for safer APIs

These decisions align with the project's goal of creating a maintainable, high-performance block set for GNU Radio 4.0.
