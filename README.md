# Wisteria - FGRP5 at warp speed

A from-scratch modern reimplementation of the Einstein@Home FGRP5
gamma-ray pulsar search, packaged as ready-to-run BOINC anonymous-platform
apps for **Linux x86-64, Linux x86-64-v3 (AVX2/FMA), Linux aarch64 and
Windows x64**.

**Coded by Alperen Yavuz.** Binary releases only - source availability is
restricted per the upstream authors' request. GPLv2+ like upstream.

## Downloads

- **[Latest release](https://github.com/alplix/wisteria-fgrp5/releases/latest)** - v0.3.1
  - Linux x86-64 tarball: static build, baseline ISA (runs on any 64-bit x86 CPU)
  - Linux x86-64 v3 tarball: AVX2/FMA optimized static build (2013+ CPUs, ~15-30% faster)
  - Linux aarch64 tarball: Jetson, Raspberry Pi 5, Ampere
  - Windows x64 zip: self-contained executable (no runtime installs needed)

### Why are there two Linux x64 versions?

Both target 64-bit Intel/AMD Linux, but for different generations of CPUs - the same
split the stock Einstein@Home app uses (SSE2 vs AVX builds):

- **x86-64 (baseline)** - the safest choice. Runs on every 64-bit x86 CPU since 2003,
  including very old or low-end machines. Pick this if you don't know your CPU.
- **x86-64-v3 (AVX2/FMA)** - compiled for the x86-64-v3 instruction set (Intel Haswell+,
  AMD Excavator+/2013+). Clearly faster on supported CPUs: roughly 15-30% shorter
  runtimes thanks to the 256-bit SIMD FFT path and fused multiply-add.

The v3 build will **not** run on CPUs without AVX2/FMA - it crashes with SIGILL
(illegal instruction), which is why the baseline is kept. BOINC picks the correct
plan class per machine automatically; for manual installs, match your CPU.

The other two packages are single-variant: **aarch64** for ARM64 (Jetson, Raspberry Pi 5,
Ampere) and **Windows x64**.

Every archive includes `app_info.xml` (with the confirmed
`<plan_class>FGRPSSE</plan_class>`) and `app_config.xml` - unpack into your
BOINC project folder, restart the client, done.

The executable is named `wisteria` (Linux) / `wisteria.exe` (Windows). On
start it prints a stderr banner with the GPL v2 notice, app name/version,
author (Alperen Yavuz) and project URL (github.com/alplix/wisteria-fgrp5).

## What changed in v0.3.1

Code-review pass over the full Fortran pipeline, 6 fixes applied and all
packages rebuilt/verified:

- Coherent follow-up: added the missing `- mjd_ref_frac` subtraction
  (consistent with the semicoherent stage; prevented a shift with
  fractional-day `--reftime`)
- Switched the monotonic clock to `system_clock` (no negative durations
  across midnight)
- Fixed `bfraction_done` normalization so progress reaches 1.0 on the last
  sky point
- Checkpoint `read`: clamped to the toplist size (no overflow on corrupt
  files); `write`: fixed Windows `rename()` failure by deleting the old
  checkpoint first (no stale `.tmp` residue)
- Removed a dead-statistic expression

## Verified

- Top candidate `f0=12.3457260` **identical on all four platforms**
- DE430 ephemeris, Windows 11 & Linux: `S=29.73155 P=47.64821`, top candidate
  bit-exact; lower toplist ranks match to 1e-5 (FMA rounding noise)
- `SHA256SUMS` checked with `sha256sum -c`

## This build is for you if...

- you want faster FGRP5 CPU tasks on any machine, old or new
- you run ARM boards (Jetson / RPi 5) that the stock app barely supports
- you like watching a task finish before your coffee does :)

## Skip it if...

- you only chase credit - validate your expectations against stock first.

Feedback very welcome - especially from ARM board owners and anyone running
long uninterrupted sessions.
