<!--
SPDX-FileCopyrightText: 2026 Tom Haynes <loghyr@gmail.com>
SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Build system status (v0.1.0 = shipped)

This file was v0.0.1's TODO list.  As of v0.1.0 the build scaffold
is complete and this file is retained as a historical marker.

## What v0.1.0 delivered

- `configure.ac` with SIMD detection: `HAVE_MD_X86_SIMD`,
  `HAVE_MD_NEON_SIMD`, `HAVE_ISA_L_X86_SIMD` (guarded on
  NASM presence), `AVX2_CFLAGS` probe, `HOST_DARWIN`/`HOST_FREEBSD`
  conditionals, `--enable-noscalar-vec` flag.
- `Makefile.am` at each level: top-level, `lib/`, `lib/ec/`,
  `lib/ec/tests/`, `vendor/`, `vendor/{snapraid,linux-md,isa-l}-raid/`,
  `tools/`.  Sub-libraries link the way reffs's do, adjusted for the
  top-level `vendor/` layout.
- `libcheck`-driven unit tests (10/10 pass on mana ARM 2026-07-29).
- Two binaries: `ec_bench` (encoder sweep + correctness check) and
  `moj_bench` (Mojette inverse peel-vs-GD microbench).
- CI: `.github/workflows/build.yml` -- smoke-build + tests + bench
  on `ubuntu-latest` and `macos-latest`.
- `AUTHORS`, `CONTRIBUTING.md`.
- `.gitignore` covers autotools-generated files.

## Build from a clean checkout

    autoreconf -fi
    mkdir build && cd build
    ../configure --disable-shared
    make -j
    make -C lib/ec check
    ./tools/ec_bench --iters 20 --sizes 65536

## What comes next (post-v0.1.0)

- Refresh cadence policy: how often to pull upstream reffs source
  changes.  Currently ad-hoc.
- Homebrew formula and Debian packaging (both nice-to-have; the
  autotools tarball is the primary artifact for now).
- More extensive CI matrix (FreeBSD, aarch64 Linux, x86_64 Linux
  with + without NASM).
- **Vendored-SIMD test triple on x86_64 CI**: `snapraid_test`,
  `isa_l_test`, `linux_md_test` all crash with SIGSEGV on GitHub
  Actions `ubuntu-latest` runners (they pass on real x86 hardware
  via reffs's nightly on shadow + garbo).  The CI workflow
  currently narrows to the seven portable-C tests
  (`gf/matrix/mirror/mojette/mojette_encoding/rs/xor`) on Linux,
  which prove the build and core math.  Root-cause options:
  runner-specific CPU quirk, alignment assumption in the vendored
  dispatch code, or a build-time flag mismatch vs the reffs
  parent's proven-good build.  Not blocking v0.1.0 -- users run
  the bench on their own hardware -- but worth chasing before the
  next release.

## Related

- Slice E.1 scoping: `~/Documents/reffs-docs/slice-E1-ec-microbench-carveout.md`
- Upstream reffs erasure-coding sources:
  <https://github.com/loghyr/reffs>, `lib/ec/`
