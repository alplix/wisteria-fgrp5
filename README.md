# ✧ Wisteria — FGRP5 at warp speed ✧

A from-scratch modern reimplementation of the Einstein@Home FGRP5
gamma-ray pulsar search, packaged as ready-to-run BOINC anonymous
platform apps for **Linux x86-64, Linux ARM64 and Windows x64**.

**Coded by Alperen Yavuz.** Binary releases only — source availability
is restricted per the upstream authors' request. GPLv2+ like upstream.

## Downloads

⬤ **[Latest release](https://github.com/alplix/wisteria-fgrp5/releases/latest)** — v0.2.1
  - Linux x86-64 tarball: baseline + AVX2 + AVX-512 builds (static FFTW/BOINC)
  - Linux ARM64 tarball: Jetson, Raspberry Pi 5, Ampere
  - Windows x64 zip: self-contained executable (no runtime installs needed)

Every archive includes `app_info.xml` (with the confirmed
`<plan_class>FGRPSSE</plan_class>`) and `app_config.xml` — unpack into
your project folder, restart BOINC, done.

## Why another build?

The stock CPU application still targets a ~2008 baseline: SSE2, no FMA,
an old compiler, no multi-core. Wisteria rebuilds the entire pipeline
for modern hardware:

- **Per-microarchitecture builds** — AVX2 + FMA, AVX-512, ARM NEON
- **Measured 2.1× speedup** of the semicoherent stage (AVX2 build vs baseline)
- **Windows build JIT-compiles to your exact CPU** on first run
- **OpenMP parallel** scan engine with per-sky-point checkpointing
  (survives kill -9, validated)

## Numerical fidelity

Not just fast — verified against the upstream application on identical
input: FFT spectra agree to float32 precision (1.1e-7 over 4.19M bins),
candidate toplists identical (50/50, powers within 1.3e-7 relative).
The same injected-pulsar test passes on x86-64, ARM64 (qemu) and Windows.

## This build is for you if...

- you want faster FGRP5 CPU tasks on any machine, old or new
- you run ARM boards (Jetson / RPi 5) that the stock app barely supports
- you like watching a task finish before your coffee does :)

## Skip it if...

- you only chase credit — validate your expectations against stock first.

Feedback very welcome — especially from ARM board owners and anyone
running long uninterrupted sessions.
