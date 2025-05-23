---
title: Mentor Meeting #2 – jmorman
date: 2025-05-22
description: Naming conventions, repository layout, and next-port priorities after the second week’s progress call with Josh Morman (plus #architecture sync).
layout: post
---

# GSoC 2025 · GNU Radio 4.0 Block-Set Expansion  
### Week-2 check-in (mentor – Josh Morman)

| | |
|---|---|
| **Date** | 22 May 2025 |
| **Meetings** | (1) one-to-one progress sync · (2) open **#architecture** channel call |
| **Participants** | Josh Morman, Krish Gupta · community contributors (Ralph, John Sally, Alex) |
| **Topics** | File-naming rules, OOT vs. sub-module layout, current porting status, upcoming priorities |

---

## 1 · Progress since last call <!--————————————————————-->

* **Structure research**  Drafted several repo-layout variants (monolithic-plus-plugins, layered, micro-repos, hybrid monorepo, feature branches).  
* **Porting milestone**  ✔ `iir_filter` block compiles under GR-4.0; extended unit tests now hit edge cases and SIMD paths.

---

## 2 · Repository & workflow decisions <!--——————————————————-->

| Decision | Rationale |
|----------|-----------|
| **Work stays in an OOT module** (`gr-incubator`) | Keeps GR-4.0 mainline clean until merge week. |
| **Initial folder skeleton** will be scaffolded by **Josh** | Gives everyone a stable starting point. |
| **Ralph’s “multi-header-in-one-file” style accepted** | Easier to locate related templates; reduces header churn. |

---

## 3 · Naming-convention consensus <!--———————————————————-->

| Rule | Agreed behaviour |
|------|------------------|
| **Block class names** | **PascalCase**, e.g. `IIRFilter`, `MultiplyConst`. |
| **File names** | Descriptive, no underscores; e.g. `IIRFilter.(hpp|cpp)`. |
| **Parentheses in filenames** | Use only where GR-4.0 already does (`Foo(simd).hpp`)—signals SIMD specialisations without underscores. |
| **Avoid one-word abbreviations** | Helps users grep by intent rather than internal shorthand. |

*John Sally favoured 1:1 file–class mapping; Ralph preferred “logical grouping”. Both agreed underscore-free names make discovery simpler.*

---

## 4 · Tooling idea

> Draft a lightweight **naming-linter** that scans `gr-incubator` and flags files or classes breaking the new rules (regex-based, hooked into CI).

---

## 5 · Port-order roadmap

1. **Math** (block family already underway)  
2. **ZMQ** (critical for network demos)  
3. **Filters** (FIR/IIR, complex variants)  
4. **Analog → Digital → the rest** (as originally proposed)

---

## 6 · Immediate action items

* **Krish**  
  * Finish SIMD test coverage for `iir_filter` which was already covered by ralph but was oversighted by me due to different naming.  
  * Start `math/AddConst` refactor with new naming pattern.  
  * Draft spec for the naming-linter (CLI + GitHub-Action prototype).
* **Josh**  
  * Push initial `gr-incubator` skeleton with MIT licence.  
  * Document preferred file-grouping examples in the repo README.

---

> _“Consistency beats cleverness—pick a name you’d expect to type in the search bar.”_  
> — **Ralph**, #architecture call, 22 May 2025
