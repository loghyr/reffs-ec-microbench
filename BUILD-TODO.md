<!--
SPDX-FileCopyrightText: 2026 Tom Haynes <loghyr@gmail.com>
SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Build system TODO (v0.0.1 -> v0.1.0)

This file lists what still needs to be done to make
`reffs-ec-microbench` build standalone from a clean checkout.

## What's here

- All source files copied from reffs `0d885b9edff9`
- License documents (LICENSE, LICENSE-EXCEPTIONS.txt, README.md)
- Directory structure matching the intended v0.1.0 shape

## What's missing

### Autoconf / build system

The reffs `lib/ec/Makefile.am` carries substantial autoconf
integration that hasn't been carved out yet:

1. **Top-level `configure.ac`** -- needs to detect:
   - C compiler + pthreads
   - `-mavx2` support (sets `AVX2_CFLAGS`) for the Mojette forward
     transform SIMD fast path
   - `-mssse3` for Slice R.2's Reed-Solomon `pshufb` path
   - `HAVE_MD_X86_SIMD` conditional (Linux md RAID-6 x86 SIMD)
   - NASM presence for ISA-L's SIMD kernel (see
     `reference_fedora_reffs_bench_prep` memory: without NASM,
     ISA-L falls back to C-only at ~80 MB/s instead of ~30 GB/s)
   - `--enable-noscalar-vec` flag for the SIMD-vs-scalar honest-
     comparison experiments (sets `NOSCALAR_VEC_CFLAGS`)

2. **`Makefile.am` at each level**:
   - Top-level: `SUBDIRS = lib/ec vendor tools`
   - `lib/ec/Makefile.am` (adapt from reffs's -- most of it
     copies over verbatim; drop the reffs-specific `AM_CFLAGS
     -I$(top_srcdir)/lib/include` path if we relocate the header)
   - `vendor/isa-l-erasure/Makefile.am` (copies from reffs)
   - `vendor/linux-md-raid/Makefile.am` (copies from reffs)
   - `vendor/snapraid-raid/Makefile.am` (copies from reffs)
   - `tools/Makefile.am` (much simpler than reffs's -- only
     needs ec_bench + moj_bench)
   - `lib/ec/tests/Makefile.am` (adapt from reffs)

3. **`autogen.sh` or standard `autoreconf -fi` step** -- reffs
   uses the latter; either works.

### Registry glue (`lib/ec/ec.c`)

Reffs's encoding-registry API (`struct ec_encoding` in
`lib/include/reffs/ec.h`) has each encoder self-register via a
per-encoder `ec_XXX_create()` function.  Consumers call these
directly.  This means there is no central registry file to copy
from reffs -- the "wiring" is scattered.

For the carve-out this is fine: `ec_bench.c` already calls each
`ec_XXX_create()` explicitly.  No glue `ec.c` needed after all.

### CI / release

- GitHub Actions workflow (`.github/workflows/build.yml`) to
  smoke-build on `ubuntu-latest` (has NASM in apt) and
  `macos-latest` (no NASM -- ISA-L skipped on Mac).
- `AUTHORS` / `CONTRIBUTING.md`.
- Version stamp mechanism (probably `AC_INIT([reffs-ec-microbench],
  [0.1.0], [loghyr@gmail.com])`).

## Estimated effort

Per the Slice E.1 scoping doc (`slice-E1-ec-microbench-carveout.md`
in reffs-docs): **~3 days of author work to first release**.

- Day 1: configure.ac + Makefile.am at each level; verify
  Linux build works end-to-end
- Day 2: Mac build fix-ups (NASM absence, Darwin-specific
  shims); tests pass
- Day 3: CI workflow, AUTHORS, small documentation polish,
  tag v0.1.0

Total lines of code to write (all glue): ~200-400 lines of
autoconf + Makefile.am.

## What can be done RIGHT NOW without more work

The source is here.  Anyone who wants to reproduce a benchmark
number today can:

1. `cd ~/reffs && ./configure && make -C lib/ec && make -C tools ec_bench`
2. `./build/tools/ec_bench --sizes 4096,16384,65536,262144`

That's the reffs-current path, which is what the flexfiles-v2
draft family's numbers were generated on (Slices R.3 + R.5 +
Slice 7.8 all used this build).  The carve-out's value is having
this as a standalone artifact for those who don't want the full
reffs tree; until the build system is scaffolded here, that
audience is not yet served.

## Related

- Slice E.1 scoping: `~/Documents/reffs-docs/slice-E1-ec-microbench-carveout.md`
- reffs `lib/ec/Makefile.am` (source of the build patterns to adapt)
- reffs `configure.ac` (source of the SIMD-detection blocks to adapt)
