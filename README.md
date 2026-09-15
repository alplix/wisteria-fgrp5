# Wisteria - FGRP5 at warp speed

A from-scratch modern reimplementation of the Einstein@Home FGRP5
gamma-ray pulsar search, packaged as ready-to-run BOINC anonymous-platform
apps for **Linux x86-64, Linux x86-64-v3 (AVX2/FMA), Linux aarch64 and
Windows x64**.

**Coded by Alperen Yavuz.** Binary releases only - source availability is
restricted per the upstream authors' request. GPLv2+ like upstream.

## Downloads

- **[Latest release](https://github.com/alplix/wisteria-fgrp5/releases/latest)** - v0.3.7
  - Linux x86-64 tarball: static build, baseline ISA (runs on any 64-bit x86 CPU)
  - Linux x86-64 v3 tarball: AVX2/FMA optimized static build (2013+ CPUs, ~15-30% faster)
  - Linux aarch64 tarball: Jetson, Raspberry Pi 5, Ampere
  - Windows x64 baseline zip: self-contained executable, runs on any x64 CPU
  - Windows x64 v3 zip: AVX2/FMA build (2013+ CPUs, ~15-30% faster)

### v0.3.7 (current) - hotfix: diagnostics now go to stderr

v0.3.6's diagnostics were written to **stdout**; the BOINC client only
captures **stderr**, so in BOINC the log showed just the banner, the
correct output names and a bare `STOP 1` - the new "bad LAL ephemeris
header" message and the file dump never appeared (confirmed by @baracutio's
v0.3.6 stderr on task `LATeah2223F_872.0_258069_0.0_0`).

v0.3.7 sends **every** progress and error line to stderr, so the ephemeris
diagnostics are now visible in the task log. With a non-FITS ephemeris the
next failing stderr should show:

```
% loading ephemeris: JPLEPH.405
% not JPL FITS; loading as LAL text: JPLEPH.405
ERROR: bad LAL ephemeris header, first token of:
> <the actual first non-# line>
file size: ...
first bytes (hex): ...
first bytes (text): ...
```

If you saw the bare `STOP 1` with v0.3.6, install v0.3.7 and paste the full
stderr of one failing task - that message identifies the real ephemeris
file format. `app_info.xml` bumped to **133**.

### v0.3.6 - ephemeris diagnostics ("Bad real number in item 1")

v0.3.5 fixed the output naming (confirmed by @toggleton and @baracutio: the
`_0` and `_1` outputs now appear under exactly the names BOINC expects) but
a second deterministic crash surfaced in the **ephemeris** loader:

```
At line 252 of file fgrp5_support.f90
Fortran runtime error: Bad real number in item 1 of list input
```

That is the generic **LAL-text** ephemeris parser being handed a file that
is not a JPL FITS file (a FITS file starts with `SIMPLE  =`). In the BOINC
slot the file referenced as `--ephemdir JPLEPH.405` turns out not to be the
standard FITS ephemeris, so detection falls through and the text parser
trips over the first non-numeric line, exiting with code 2.

**v0.3.6 turns that obscure crash into a real diagnostic:** the app now
prints the offending header line, the file size and the first 128 bytes of
the file (hex + as text) and stops with a clear
`ERROR: bad LAL ephemeris header` message - so the next stderr shows
exactly what that ephemeris file really is. Genuine LAL text tables and
JPL FITS files keep working unchanged. `app_info.xml` version bumped to
**132**.

### v0.3.5 - the real BOINC fix

Modern FGRP5 workunits from Einstein@Home no longer send `-o/--outputfile`
on the command line (newer workunit generator; the client_state.xml command
lines now end in `--debug 0 --debugCommandLineMangling` with no `-o`).
Every earlier build required `-o`, so at startup they printed an
`argc/argv` dump and exited before producing anything - and BOINC reported
**"Output file ... absent"** for every single task. That argv dump is
exactly what showed up in @baracutio's and @toggleton's stderr.

**Root cause found and fixed.** Wisteria now reads the result name from
`init_data.xml` (written by BOINC into the slot directory) and writes
exactly the files BOINC expects:

- `<result_name>_0` - the toplist,
- `<result_name>_1` - the coherent follow-up.

Verified with a BOINC-style command line (no `-o`, bare `--inputfile` and
`--ephemdir JPLEPH.405`, `--debugCommandLineMangling` flag): both output
files are created under the exact names the validator looks for, RC=0.
Standalone runs with `-o` keep working unchanged. `app_info.xml` version
bumped to **131** so BOINC picks up the new binaries.

### v0.3.4 - the semicoherent crash fix

The big one. Every real multi-sky-point BOINC task (a typical FGRP5 search
uses **12 sky points**) was dying with **"Output file ... absent"** right after
the first sky point. Root cause: the module-level pair arrays inside
`setup_pairs()` were allocated again on the 2nd sky point without being freed
first, raising a Fortran runtime **"already allocated"** error that terminated
the app (exit code 2). The arrays are now released before re-allocation.

Verified with the official BOINC command line and explicit multi-sky-point
runs: **12/12 sky points complete, RC=0, output file produced**. Single-sky
and `--ephemdir` tests still pass unchanged.

### v0.3.3 - BOINC app_info schema fix + --ephemdir directory support

Two follow-up fixes from the Einstein@Home forum (reported by @baracutio
and @toggleton):

- **app_info.xml `<file_name>` → `<name>`**: the `<file_info>` block must
  use `<name>wisteria</name>` (or `wisteria.exe`); the previous
  `<file_name>` tag is only valid inside `<file_ref>` and made BOINC report
  "missing application file". This was identified by @baracutio's own
  modified app_info.xml which worked correctly.
- **`--ephemdir` now accepts a directory**: BOINC passes
  `--ephemdir .../einstein.phys.uwm.edu/JPLEPH` which is a directory, not a
  file. Wisteria now probes the plain path first, then searches well-known
  ephemeris file names (`JPLEPH.405`, `lnxp1600p1658.405`, `DE430.dat`, ...)
  inside the directory before giving up.

All five packages rebuilt; same top candidate `f0=12.3457260` verified.

### v0.3.2 - BOINC CLI parser + app_config fixes

Reported on the Einstein@Home forum right after v0.3.1: the app received a
`STOP 1`/`process exited with code 1` from BOINC before doing any work, and
BOINC sometimes reported "Not requesting tasks: don't need (no
applications)". Three root causes found and fixed:

- **`app_config.xml` was invalid**: `<fraction_done_xml>1</fraction_done_xml>`
  is not a valid tag and an unknown `<options>` block made BOINC reject the
  file. It now uses the correct `<fraction_done_exact/>` and nothing else.
- **`app_info.xml` was missing the `<platform>` tag**. Without it BOINC did
  not properly match the app to workunits, so tasks either never arrived or
  the app was launched with no usable arguments (immediate `STOP 1`). All
  packages now ship the correct platform tag for their architecture.
- **The CLI parser was rebuilt from scratch**: it now handles the full
  official FGRP5 command line exactly as BOINC delivers it in
  anonymous-platform mode - split args, long flags, negative-number values
  (e.g. `--f1dot -1e-13`), and even the entire command line arriving as a
  single argv element. If required arguments are still missing at startup,
  the app prints the exact `argc/argv` it received so any remaining issue
  can be reported precisely.

Direct command line still works unchanged on every platform.

### Why are there two versions (Linux and Windows)?

Both the Linux and Windows releases come in a baseline and an x86-64-v3
(AVX2/FMA) variant - the same split the stock Einstein@Home app uses
(SSE2 vs AVX builds):

- **x86-64 (baseline)** - the safest choice. Runs on every 64-bit x86 CPU since 2003,
  including very old or low-end machines. Pick this if you don't know your CPU.
- **x86-64-v3 (AVX2/FMA)** - compiled for the x86-64-v3 instruction set (Intel Haswell+,
  AMD Excavator+/2013+). Clearly faster on supported CPUs: roughly 15-30% shorter
  runtimes thanks to the 256-bit SIMD FFT path and fused multiply-add.

The v3 build will **not** run on CPUs without AVX2/FMA - it crashes with SIGILL
(illegal instruction), which is why the baseline is kept. BOINC picks the correct
plan class per machine automatically; for manual installs, match your CPU.

The remaining package is single-variant: **aarch64** for ARM64 (Jetson,
Raspberry Pi 5, Ampere).

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
