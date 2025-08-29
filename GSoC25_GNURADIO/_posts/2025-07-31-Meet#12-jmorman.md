---
title: "Meet#12 jmorman"
date: 2025-07-31
description: "Weekly progress update for GNU Radio 4.0 Block Set Expansion project."
layout: post
---

# GSoC 2025 · GNU Radio 4.0 Block-Set Expansion  
### Week-12 check-in (mentor — **Josh Morman**)

| | |
|---|---|
| **Date** | 31 July 2025, 17:00 IST |
| **Duration** | 50 min |
| **Participants** | Josh Morman · Krish Gupta |

---

## 1 · Progress update <!--──────────────────────────-->

* **Analog PR Status**  
  * All feedback addressed and CI passing
  * Final approval expected soon
  * Documentation enhanced with concrete examples

* **OFDM Components**  
  * Completed implementation of core OFDM components:
    * `CyclicPrefixer.hpp` / `CyclicPrefixRemover.hpp`
    * `Serializer.hpp` / `Deserializer.hpp`
    * `CarrierAllocator.hpp`
  * Started implementation of OFDM synchronization:
    * `ChannelEstimator.hpp`
    * `FrameSynchronizer.hpp`

* **Packet Handling**  
  * Began implementation of packet processing components:
    * `HeaderFormat.hpp` - Base class for packet headers
    * `HeaderParser.hpp` - Extracts metadata from headers
    * `PacketSink.hpp` - Processes complete packets

---

## 2 · OFDM Implementation Highlights <!--──────────────────────────-->

Implemented a flexible OFDM system with clean interfaces:

### Carrier Allocator

```cpp
template <typename T>
class CarrierAllocator : public Block<CarrierAllocator<T>> {
private:
    size_t fft_len_{64};
    std::vector<int> occupied_carriers_;
    std::vector<int> pilot_carriers_;
    std::vector<std::complex<T>> pilot_symbols_;
    
public:
    GR_MAKE_REFLECTABLE(CarrierAllocator, 
                        fft_len_, occupied_carriers_, 
                        pilot_carriers_, pilot_symbols_);
    
    template <typename InputIt, typename OutputIt>
    void processBulk(InputIt in, size_t n, OutputIt out, size_t& produced) {
        // Maps input symbols to OFDM subcarriers
        // with proper handling of pilots and guard bands
        // ...
    }
    
    // Factory methods for common configurations
    static CarrierAllocator<T> make_standard(size_t fft_len);
    static CarrierAllocator<T> make_dvbt(size_t mode);
};
```

This implementation:
* Provides flexible carrier mapping
* Supports various standards through factory methods
* Handles pilot insertion automatically
* Maintains clean state management

---

## 3 · Packet Processing Architecture <!--────────────────────────────-->

Designed a modular packet processing system:

```
packet/
├── HeaderFormat.hpp         # Base class for header formats
├── HeaderFormats/
│   ├── DefaultFormat.hpp    # Simple length+flags header
│   └── CrcFormat.hpp        # Header with CRC protection
├── HeaderParser.hpp         # Extracts metadata from headers
├── HeaderGenerator.hpp      # Creates packet headers
├── PacketSink.hpp           # Processes complete packets
└── Demux.hpp                # Routes packets based on metadata
```

This architecture:
* Separates header format from processing logic
* Enables custom header formats through inheritance
* Provides robust error detection
* Maintains backward compatibility with GR3 patterns

---

## 4 · Final Phase Planning <!--────────────────────────────-->

Josh and I discussed plans for the final GSoC phase:

| Priority | Tasks |
|----------|-------|
| **1. PR Finalization** | • Address all remaining feedback<br>• Ensure CI is stable<br>• Complete documentation |
| **2. Documentation** | • Finalize porting guide<br>• Add usage examples<br>• Document design decisions |
| **3. Testing** | • Ensure comprehensive test coverage<br>• Verify performance in realistic scenarios<br>• Add benchmarks for key components |

Josh emphasized:

> "The priority now is to ensure that what we've done so far is solid, well-tested, and ready for merge. It's better to have fewer blocks that are production-ready than more blocks with incomplete testing or documentation."

---

## 5 · Template Diagnostics Improvements <!--────────────────────────────-->

Further refined our approach to template diagnostics:

```cpp
// Before: Cryptic error messages
template <typename T>
void processOne(const T& in, T& out) {
    out = std::sin(in); // Error if T isn't a floating-point type
}

// After: Clear constraints and messages
template <typename T>
requires std::floating_point<T> || 
         (gr::concepts::ComplexValue<T> && 
          std::floating_point<typename T::value_type>)
void processOne(const T& in, T& out) {
    out = std::sin(in);
}
```

This approach:
* Provides clear error messages
* Documents type requirements
* Prevents instantiation with invalid types
* Improves IDE auto-completion and documentation

---

## 6 · Action items <!--──────────────────────────────────-->

| Owner | Task |
|-------|------|
| **Krish** | • Complete OFDM synchronization components<br>• Continue packet handling implementation<br>• Prepare final documentation updates |
| **Josh** | • Finalize Analog PR approval<br>• Continue Digital PR review<br>• Coordinate with maintainers on repository transition |

---

>**Next sync**: August 7th to review final implementation status and prepare for project wrap-up.

---

## 7 · Reflections on Repository Transition <!--────────────────────────────-->

The ongoing transition from `fair-acc/gnuradio4` to `gnuradio/gnuradio4` has required careful coordination:

| Challenge | Approach |
|-----------|----------|
| **PR migration** | Submit new PRs to the new repository |
| **CI configuration** | Ensure consistent configuration across repositories |
| **Review continuity** | Document feedback from previous reviews |

Josh mentioned:

> "The repository transition adds complexity, but it's a good thing for the project's long-term home. The move to the official GNU Radio organization will increase visibility and adoption."

This transition represents a significant milestone for GNU Radio 4.0, signaling its move toward becoming the official next-generation implementation.
