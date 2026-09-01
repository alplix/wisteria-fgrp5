# Wisteria — FGRP5 Fortran Port (Einstein@Home Anonymous Platform App)

A modern Fortran reimplementation of the Einstein@Home FGRP5 gamma-ray
pulsar search, packaged as a BOINC anonymous-platform application
(version 99). Numerically cross-validated against the upstream
application: identical candidate toplists, FFT spectra agree to float32
precision (~1e-7), and checkpoint/resume survives `kill -9`.

**This repository distributes binary releases only** — no source code
(source availability is restricted per the upstream authors' request).

## Install (Linux x86-64)

1. Download the latest release zip and unpack it into
   `<BOINC data dir>/projects/einstein.phys.uwm.edu/`
   (typically `/var/lib/boinc/projects/einstein.phys.uwm.edu/`).
2. Edit `app_info.xml`: set `<plan_class>` to the plan class your host
   receives for FGRP5 (check `client_state.xml` after running the stock
   application once, e.g. `hsgamma_FGRP5_cpu`). Remove the XML comment.
3. Keep `app_config.xml` if you want one task at a time with exact
   progress reporting (optional).
4. In BOINC Manager: Options → Read config files, then request FGRP5 work.

The binary is largely statically linked (FFTW, BOINC API, C++ runtime);
only standard system libraries (glibc, libm, libgomp) are required.
OpenMP-parallel — set `Max CPUs` / `app_config` to control thread use.

## Validation summary

| Check | Result |
|---|---|
| FFT spectrum vs upstream C (4.19M bins) | max rel. diff 1.1e-7 |
| Toplist vs upstream C | 50/50 candidates identical, powers 1.3e-7 rel |
| Toplist vs Julia port | identical digit-for-digit |
| Injected pulsar recovery | f0 found at grid resolution, correct sky position |
| kill -9 + resume | identical result from checkpoint |

## Credits

Science algorithms from the Einstein@Home FGRP5 application
(H. J. Pletsch et al., Albert-Einstein-Institut). Port by Alperen Yavuz.
Run in accordance with Einstein@Home's custom-application policy;
please notify sourcecode@einsteinathome.org about custom app versions.
