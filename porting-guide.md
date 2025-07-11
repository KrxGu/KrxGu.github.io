---
layout: default
title: "GNU Radio 3.x to 4.0 Porting Guide"
permalink: /porting-guide/
---

<style>
/* GNU Radio Documentation Style */
body {
    font-family: sans-serif;
    line-height: 1.4;
    color: #000;
    font-size: 15px;
}

.banner-wrapper {
    text-align: center;
    padding: 2rem 1rem;
    margin-bottom: 2rem;
    background: #fff;
    border-bottom: 1px solid #ddd;
}

.banner-logos {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 2rem;
    margin-bottom: 1rem;
}

.banner-logos img {
    height: 80px;
    width: auto;
}

.banner-title {
    font-size: 26px;
    font-weight: normal;
    color: #000;
    margin: 0;
    font-family: sans-serif;
}

.banner-title .highlight-orange {
    color: #ff6a00;
}

.banner-title .highlight-blue {
    color: #4a90e2;
}

/* Continue with rest of styles... */
</style>

<!-- Banner Section -->
<div class="banner-wrapper">
    <div class="banner-logos">
        <img src="https://ettus.com/wp-content/uploads/2018/11/GNU.png" alt="GNU Radio logo" />
        <span style="font-size: 48px; color: #ff6a00; font-weight: bold; margin: 0 1rem;">×</span>
        <img src="https://www.pymc.io/_images/GSoC_banner.png" alt="Google Summer of Code logo" style="height: 100px;" />
    </div>
    <h1 class="banner-title">
        <span class="highlight-orange">From 3.x</span> <span class="highlight-blue">to 4.0</span><br>
        GNU Radio Porting Guide<br>
        <span style="font-size: 20px; color: #666;">(GSoC 2025)</span>
    </h1>
    <div style="text-align: center; margin-top: 1rem; font-size: 16px; color: #333;">
        <strong>By Krish Gupta</strong><br>
        <em>Mentor: Josh Morman</em>
    </div>
</div>

## 📋 Contents

1. [Overview & Motivation](#overview--motivation)
2. [Prerequisites & Setup](#prerequisites--setup)
3. [Architecture Changes – 3.x vs 4.0](#architecture-changes--3x-vs-40)
4. [Step‑by‑Step Porting Workflow](#step-by-step-porting-workflow)
5. [Block API Migration](#block-api-migration)
6. [SIMD Optimisation](#simd-optimisation)
7. [Reflection & Registration System](#reflection--registration-system)
8. [Testing & Validation](#testing--validation)
9. [Complete Porting Examples](#complete-porting-examples)
10. [Best Practices & Conventions](#best-practices--conventions)
11. [Troubleshooting Common Issues](#troubleshooting-common-issues)
12. [Resources & References](#resources--references)

---

## 1 Overview & Motivation

### 🎯 Why Port to GNU Radio 4.0?

GNU Radio 4.0 ("GR4") is a **ground‑up rewrite** of the 3.x series. It trades Boost and decade‑old macros for modern C++23 features, an all‑new lock‑free scheduler, and auto‑vectorised math. Porting frequently‑used GR3 blocks means **everyone**—students, hobbyists, scientists—can enjoy:

* **2–3× speed‑ups** on the same hardware (lock‑free buffers + `std::simd`).
* **Type‑safe ports & parameters** that surface cleanly in Python and soon‑to‑arrive GUI tools.
* **Zero‑boilerplate GUIs**: one macro registers the block, generates YAML + Python stubs.

### 🔍 What This Guide Covers

* **Audience** Absolute beginners to GNU Radio internals.
* **Scope** End‑to‑end porting **inside the *main* `fair‑acc/gnuradio4` repo**—no OOT module glue required. You'll learn how to:

  1. Read an old GR3 block.
  2. Map its concepts to the new API.
  3. Open a Pull Request (PR) with DCO sign‑off.
* **Non‑Goals** FPGA (RFNoC) paths, GUI designer, or containerising flowgraphs.

### 📊 Is Your Block a Good Candidate?

Ask yourself:

| Question                            | Heuristic                                  |
| ----------------------------------- | ------------------------------------------ |
| *Used by many flowgraphs?*          | Check GitHub search or your own notebooks. |
| *Algorithm self‑contained?*         | Fewer external deps = faster port.         |
| *Contains hand‑written intrinsics?* | Great! We'll swap them for `std::simd`.    |
| *Does it allocate big buffers?*     | Might need redesign with `std::span`.      |

If you answer **yes** to the first two, dive in.

---

## 2 Prerequisites & Setup

> **Quick‑start: one command, zero surprises**
>
> ```bash
> docker pull --platform linux/amd64 ghcr.io/fair-acc/gr4-build-container:latest
> ```

### 🛠 Toolchain Basics (Bare‑Metal)

| Tool       | Min Version      | Why                              |
| ---------- | ---------------- | -------------------------------- |
| **GCC**    | ≥13.3 (≥14.2 for full `<experimental/simd>`) | minimum to compile GR4 |
| **Clang**  | 18 (recommended) | faster builds, great diagnostics |
| **CMake**  | 3.25             | presets & fetched content        |
| **Ninja**  | any              | parallel build engine            |
| **Python** | 3.10             | unit‑test harness & bindings     |

Install on Ubuntu 24.04:

```bash
sudo add-apt-repository ppa:ubuntu-toolchain-r/test -y
sudo apt update
sudo apt install gcc-14 g++-14 clang-18 cmake ninja-build git python3-pip -y
export CC=gcc-14 CXX=g++-14  # or clang-18/clang++-18
```

### 🐳 Docker Route (Recommended for Beginners)

1. **Pull** the container (see quick‑start).
2. **Run** with the current repo mounted:

   ```bash
   docker run --rm -it \
     --volume="${PWD}:/work/src" \
     --workdir="/work/src" \
     ghcr.io/fair-acc/gr4-build-container:latest bash
   ```
3. Inside the shell, configure + build:

   ```bash
   rm -rf build && mkdir build && cd build
   cmake -DCMAKE_BUILD_TYPE=RelWithAssert -DGR_ENABLE_BLOCK_REGISTRY=ON ..
   cmake --build . -j$(nproc)
   ctest --output-on-failure -j$(nproc)
   ```

### 💡 ZRAM Route (If Your Laptop Runs Out of RAM)

Building GR4 can transiently spike to **6 GB+** of RAM with GCC's heavy templates. If your system swaps to disk, compile times crawl. **zram** swaps to *compressed RAM* instead of SSD.

```bash
# Enable 8 GiB zram swap (needs sudo)
cd gnuradio4
sudo ./enableZRAM.sh  # provided in repo root

# Build as usual
mkdir -p build && cd build
cmake -DCMAKE_BUILD_TYPE=RelWithAssert ..
cmake --build . -j$(nproc)

# Afterwards, free the device
sudo swapoff /dev/zram0
echo 1 | sudo tee /sys/block/zram0/reset
```

♻️ *Rule of thumb:* if `free -h` shows >80 % memory used during compile, enable zram.

### 🗂 Working Inside the Main Repo

```bash
# 1. Fork the repo on GitHub
# 2. Clone your fork

git clone https://github.com/<you>/gnuradio4.git
cd gnuradio4
git remote add upstream https://github.com/fair-acc/gnuradio4.git

git checkout -b port/math/MultiplyConst
# ...hack...

# 3. Commit w/ DCO & push

git add .
git commit -s -m "feat(math): port MultiplyConst block (float & complex)"
git push origin port/math/MultiplyConst
# 4. Open PR → choose 'Create pull request'
```

> **Tip** Use `git push --force-with-lease` (not `--force`) when you squash‑rebase.

---

## 3 Architecture Changes – 3.x vs 4.0

### 3.1 What You're Really Looking At

| Version | What the snippet actually is                                                                                                   | Where it lives                                                                      |
| ------- | ------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------- |
| **3.x** | `gr_math.h` – utility header with inline helpers (fast complex‑multiply, lookup tables, etc.). **Not** a block.                | Part of *gr‑core*; many math blocks call these helpers.                             |
| **4.0** | `math.hpp` – **collection of fully‑fledged blocks** (`Add`, `Multiply`, `AddConst`, …) written with the new `gr::Block<>` API. | Lives in *gr::blocks::math*; each instantiation registered via `GR_REGISTER_BLOCK`. |

> **So:** GR3's header *supports* math **inside** blocks, whereas GR4's header *is* the math blocks.

### 3.2 Key API Differences

| Category                  | GNU Radio 3.x                               | GNU Radio 4.0                                    | Why it matters                                    |
| ------------------------- | ------------------------------------------- | ------------------------------------------------ | ------------------------------------------------- |
| **Language era**          | C++03/11, Boost, macros                     | Modern C++23: concepts, `std::span`, `std::simd` | Safer code, fewer macros, faster binaries         |
| **Block base**            | Inherit `gr::sync_block`, override `work()` | `gr::Block<Derived>` (CRTP) → `processBulk()`    | Compile‑time errors instead of run‑time segfaults |
| **Port model**            | Hard‑coded indices                          | `PortIn<T>`/`PortOut<T>` templates               | Type‑safe, introspectable                         |
| **Registration**          | Custom `make()` + YAML                      | One‑liner `GR_REGISTER_BLOCK`                    | Slashes boilerplate                               |
| **Vectorisation**         | Manual intrinsics                           | `std::simd` auto‑vector                          | Free speed on AVX/NEON                            |
| **Scheduler**             | Thread‑per‑block                            | Task‑based (TBB)                                 | Better CPU utilisation                            |
| **Live parameter change** | Custom setters & mutex                      | Any reflected member mutable at run‑time         | Zero extra C++ for AGC, etc.                      |

### 3.3 Concrete Example – *Multiply‑by‑Constant*

| Aspect           | Old 3.x                          | New 4.0                           | Benefit                      |
| ---------------- | -------------------------------- | --------------------------------- | ---------------------------- |
| Source files     | 10+ variants (`*_ff`, `*_cc`, …) | **1** template                    | 10× less code                |
| Port declaration | Raw pointer arrays               | `PortIn<T> in; PortOut<T> out;`   | Compilation catches mismatch |
| Parameter        | Private `k_`, setter             | Public `value` reflected          | No boilerplate               |
| SIMD             | Optional helper call             | Automatic if compiler supports it | Speed                        |
| GUI              | Separate YAML + Python glue      | Code‑gen from reflection          | Less maintenance             |

---

## 4 Step‑by‑Step Porting Workflow

> **Roadmap** Follow each sub‑step and run tests after *every* compile.

### 4.1 Pre‑Port Checklist

* [x] Build **current** GR4 master so you know environment is sane.
* [x] Pick a small block first (e.g., `AddConst`).
* [x] Search for duplicated code (`grep -R "fast_.*_cc"`).
* [ ] Sketch a tiny flowgraph (.py) that exercises the block.

### 4.2 Porting at a Glance

```mermaid
graph LR
 A[Understand GR3 block] --> B[Copy tests to /tests]
 B --> C[Write GR4 skeleton header]
 C --> D[Port algorithm -> processBulk]
 D --> E[Add GR_MAKE_REFLECTABLE]
 E --> F[Register block]
 F --> G[Compile & run tests]
 G --> H[Open PR]
```

### 4.3 Incremental Commits

Commit **every compiling state**:

```bash
# good habit
cmake --build build -j$(nproc) && ctest --output-on-failure
```

When tests fail, `git stash` small experiments—keep main branch green.

---

## 5 Block API Migration

### 5.1 From `work()` to `processBulk()` (Beginner Friendly)

**Old way** (pseudo‑code):

```cpp
int work(int noutput_items,
         gr_vector_const_void_star& input_items,
         gr_vector_void_star&       output_items) {
    const float*  in  = (const float*)input_items[0];
    float*        out = (float*)output_items[0];
    for (int i=0;i<noutput_items;i++)
        out[i] = in[i] * k_;
    return noutput_items;
}
```

**Problems:** raw casts, no bounds check.

**New way:**

```cpp
work_return_t processBulk(std::span<const float> in,
                          std::span<float>      out) {
    for (std::size_t i = 0; i < in.size(); ++i)
        out[i] = in[i] * value;   // 'value' is reflected param
    return {out.size(), Status::OK};
}
```

*Key points for beginners:*

1. `std::span` is like a safe pointer + length.
2. The `work_return_t` tells the scheduler how many items you produced.
3. No virtual calls—`processBulk()` is a normal function.

### 5.2 Declaring Ports

```cpp
GR_DECLARE_PORT(in,  PortIn<float>);
GR_DECLARE_PORT(out, PortOut<float>);
```

That's it—no magic numbers like `input_items[0]`.

### 5.3 Reflection One‑Liner

```cpp
GR_MAKE_REFLECTABLE(MultiplyConst,
    (float, value, "Multiplier", 1.0f, "Constant gain factor"));
```

This auto‑generates:

* YAML entry for future GUI.
* Python binding (`gr.blocks.math.multiply_const`).
* Run‑time property that can be tweaked live.

---

## 6 SIMD Optimisation

`std::simd` landed in C++23 and gives you **auto‑vectorisation without writing AVX/NEON intrinsics**. GR4's meta‑helpers make it trivial to support both scalar *and* vector paths.

### 6.1 Detecting Whether SIMD Is Available

```cpp
#include <experimental/simd>
#if defined(__cpp_lib_simd)   // GCC ≥13 / Clang ≥16
  // you can safely #include <simd>
#endif
```

GR4 wraps this behind the convenience concept `gr::meta::any_simd<V,T>` used in the generated `processOne()` template.

### 6.2 A Minimal Example

Below is the **SIMD‑aware branch** extracted from *Analog.hpp* (MultiplyConst):

```cpp
template<gr::meta::t_or_simd<T> V>
[[nodiscard]] constexpr V processOne(const V& a) const noexcept {
    if constexpr (gr::meta::any_simd<V, T>) {
        return a * value;      // element‑wise vector multiply (AVX/NEON)
    } else {
        return a * value;      // plain scalar – same line, different type
    }
}
```

Note the single line of math appears **twice**—one inside the SIMD branch, one outside. Most real blocks need *no extra code*: the compiler emits the vector loop for you.

### 6.3 Fallback Strategy

If your compiler is older than GCC 13/Clang 16, `__cpp_lib_simd` is undefined and the scalar branch is chosen. Performance degrades gracefully, functionality remains identical.

### 6.4 Benchmarking Tips

| Tip                              | Command                                          |                      |
| -------------------------------- | ------------------------------------------------ | -------------------- |
| **Build with ‑O3 ‑march=native** | `cmake -DCMAKE_CXX_FLAGS="-O3 -march=native" ..` |                      |
| **Time one block**               | `gr_benchmark -b math::MultiplyConst -n 10M`     |                      |
| **Check assembly**               | `objdump -dS libgnuradio‑math.so | grep ymm` (for AVX) |              |

> **Rule of thumb:** if you *see* `vfmadd` or `vmulps` in the disassembly, SIMD is working.

---

## 7 Reflection & Registration System

### 7.1 Why Reflection?

GR4 can **introspect** a block at run‑time—ports, parameters, doc‑strings—thanks to a tiny header‑only reflection system. This powers future GUI builders and Python auto‑bindings.

### 7.2 Dissecting an Example

```cpp
struct MultiplyConst : gr::Block<MultiplyConst> {
    PortIn<float>  in;
    PortOut<float> out;
    float          value = 1.0f;

    GR_MAKE_REFLECTABLE(MultiplyConst, in, out, value);
};
```

`GR_MAKE_REFLECTABLE`:

1. Registers each **public** member (`in`, `out`, `value`).
2. Emits metadata (type, default, doc string).
3. Auto‑generates setters/getters used by Python and YAML.

### 7.3 Registration One‑Liner

```cpp
GR_REGISTER_BLOCK("gr::blocks::math::MultiplyConst",
                  MultiplyConst,
                  ([T], std::multiplies<[T]>),   // template params & functor
                  [ float, double, std::complex<float>, std::complex<double> ])
```

Parameters:

1. **Fully‑qualified name** – becomes YAML path *and* Python import path.
2. **C++ symbol** – the class or alias to instantiate.
3. **Template pack** – how to splice the functor/type into the template.
4. **Type list** – every concrete data type you want exposed.

### 7.4 Hot‑Reloading Parameters

Because `value` is reflected, you can change it in a flowgraph at run‑time:

```python
blk = gr.blocks.math.multiply_const_f()  # Python auto‑binding
blk.value = 0.5   # halves the gain while the graph runs
```

No extra C++ needed!

---

## 8 Testing & Validation

A good port is **bit‑exact** and has **unit tests** covering corner cases. GR4 ships a thin Boost.UT harness in `gnuradio‑4.0/testing`.

### 8.1 Minimal Test Skeleton

```cpp
#include <boost/ut.hpp>
#include <gnuradio-4.0/testing/TagMonitors.hpp>

using namespace boost::ut;
using gr::blocks::math::MultiplyConst;

"MultiplyConst scalar"_test = [] {
    constexpr float k = 2.0f;
    MultiplyConst<float> blk({{"value", k}});
    expect(eq(blk.processOne(3.0f), 6.0f));
};
```

### 8.2 Graph‑Level Tests (qa\_analog.cpp)

`qa_analog.cpp` wires **sources → DUT → sink** inside an in‑memory graph and validates the full scheduler path.

```cpp
Graph g;
auto& src = g.emplaceBlock<TagSource<float>>(property_map{{"values", {1,2,3}}});
auto& mul = g.emplaceBlock<MultiplyConst<float>>(property_map{{"value",2.0f}});
auto& snk = g.emplaceBlock<TagSink<float, ProcessFunction::USE_PROCESS_ONE>>();
g.connect(src,"out",mul,"in");
g.connect<"out">(mul).to<"in">(snk);
expect(eq(scheduler::Simple{std::move(g)}.runAndWait().has_value(), true));
```

### 8.3 CI Tips

Add a quick workflow to `.github/workflows/ci.yml`:

```yaml
- name: Build & test (Clang 18)
  run: |
    cmake -B build -S . -DCMAKE_CXX_COMPILER=clang++-18 -GNinja
    ninja -C build -j$(nproc)
    ctest --test-dir build -j$(nproc) --output-on-failure
```

Enable multiple jobs for GCC/Clang, and add ASAN if possible.

---

## 9 Complete Porting Examples *(Coming soon)*

* **Math / MultiplyConst** – finished; see `gr/blocks/math/Analog.hpp`.
* **Integrate** – running sum with decimation.
* **Argmax** – index of maximum element in a vector.

> These will be expanded with full walk‑throughs and side‑by‑side GR3 vs GR4 diffs.

---

## 10 Best Practices & Conventions *(stub)*

## 11 Troubleshooting Common Issues *(stub)*

## 12 Resources & References *(stub)*

---

> **© 2025 Krish Gupta** · Mentor: *Josh Morman* · Created for **Google Summer of Code 2025**

*Last updated: 2025‑07‑11* 