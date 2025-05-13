---
title: "GSoC 2025 Kick‑off: Expanding the GNU Radio 4.0 Block Set"
date: 2025-05-13 00:00:00 +0530
tags: [GSoC, GNU Radio, SDR, Progress]
---

👋 Hello GNU Radio community—and everyone following my GSoC journey!

I’m Krish Gupta, a 2nd‑year CSE student at Manipal University Jaipur. This summer I’m contributing to Google Summer of Code 2025 under GNU Radio with a project titled “Expanding the GNU Radio 4.0 Block Set.”
(https://summerofcode.withgoogle.com/programs/2025/projects/ES4VxTjr)

**🚀 Project in one sentence**

Port and modernise the most‑used analog & digital signal‑processing blocks from GNU Radio 3.x to the brand‑new GNU Radio 4.0 architecture so that users can adopt GR4 without losing any favourite functionality.

**🔍 Why does this matter?**

GR4 performance: lock‑free buffers, compile‑time flow‑graph optimisation, built‑in SIMD/SYCL hooks.

Adoption blocker: most real‑world flow‑graphs still rely on GR3 blocks (WBFM Rx, QPSK mod/demod, etc.).

My goal: deliver a ready‑to‑use library of GR4‑native blocks + documentation + tests so researchers, hobbyists and industry engineers can migrate painlessly.

**📝 What I proposed (and accepted!)**

Analog blocks — I’ll port waveform and noise sources, AM / FM modulators & demodulators, plus operational helpers such as AGC, squelch and throttle. Together they enable a complete FM broadcast‑receiver chain that turns raw IQ samples into clear audio.

Digital blocks — Starting with BPSK and QPSK mod/demod pairs and extending (time‑permitting) to 16‑QAM and GMSK, I’ll add constellation helpers, Mueller‑&‑Müller timing recovery and a Costas loop. These pieces combine to form a QPSK loop‑back link that converts bits to RF (or a file) and faithfully recovers them.

I/O layer — Real‑time audio I/O comes via ALSA/PulseAudio Audio Sink and Audio Source, with File/TCP/UDP blocks as stretch goals. The payoff: plug headphones or a microphone straight into a GR4 flow‑graph.

Infrastructure — Every block ships with a gtest‑powered unit‑test harness, continuous‑integration workflow, example flow‑graphs and a detailed porting guide so future developers can extend the library with confidence.

**🛠️ Work done so far (pre‑GSoC)**

Deep‑dive into GR4 internals (reflection macros, templated blocks, new scheduler).

Prototype block my_adder—first successful GR4 OOT build & test proving tool‑chain.

Active on Matrix #architecture—feedback from maintainers Josh Morman & Daniel Estévez.

**🔗 Follow the journey**

Source repo: https://github.com/KrxGu/gnuradio4

All blog posts RSS: https://krxgu.github.io/

Ping me on Matrix: @krish_gupta:gnuradio.org

Next up: Meet on 16th May with mentors mostly will be setting up the project!Stay tuned, and feel free to drop suggestions or questions on my gmail: krishgupta2832@gmail.com