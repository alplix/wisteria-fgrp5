# Wisteria - FGRP5 at warp speed

A from-scratch modern reimplementation of the Einstein@Home FGRP5
gamma-ray pulsar search, packaged as ready-to-run BOINC anonymous-platform
apps for **Linux x86-64, Linux x86-64-v3 (AVX2/FMA), Linux aarch64 and
Windows x64**.

**Coded by Alperen Yavuz.** Binary releases only - source availability is
restricted per the upstream authors' request. GPLv2+ like upstream.

## Downloads

- **[Latest release](https://github.com/alplix/wisteria-fgrp5/releases/latest)** - v0.3.1
  - Linux x86-64 tarball: static build, baseline ISA
  - Linux x86-64 v3 tarball: AVX2/FMA optimized static build
  - Linux aarch64 tarball: Jetson, Raspberry Pi 5, Ampere
  - Windows x64 zip: self-contained executable (no runtime installs needed)

Every archive includes `app_info.xml` (with the confirmed
`<plan_class>FGRPSSE</plan_class>`) and `app_config.xml` - unpack into your
BOINC project folder, restart the client, done.

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