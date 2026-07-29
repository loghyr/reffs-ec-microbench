<!--
SPDX-FileCopyrightText: 2026 Tom Haynes <loghyr@gmail.com>
SPDX-License-Identifier: AGPL-3.0-or-later
-->

# Contributing to reffs-ec-microbench

## Scope

This project is a carve-out of the erasure-coding subsystem from
[reffs](https://github.com/loghyr/reffs), packaged as a standalone
microbenchmark tree for anyone who wants to cross-verify FFv2 pNFS
erasure-coding numbers without checking out the full reffs source.

**Upstream is reffs.** Changes to encoder implementations, unit
tests, or the ec_bench / moj_bench tools SHOULD land in reffs first,
then get copied here for the next release.  Changes that make sense
in the standalone context only (build system, CI, README) can go
here directly.

## Build + test

    autoreconf -fi
    mkdir build && cd build
    ../configure --disable-shared
    make -j
    make -C lib/ec check       # unit tests
    ./tools/ec_bench --iters 20 --sizes 65536

On macOS: NASM is not required.  Without it, ISA-L falls back to
portable-C (~80 MB/s at k=4 m=2 65 KiB); with NASM it runs at
multi-GB/s via SIMD.  Install via Homebrew: `brew install nasm`.

On Linux: `apt install nasm libcheck-dev` (or the equivalent on
your distribution) picks up both the ISA-L SIMD path and the unit-
test framework.

## Coding standards

- **SPDX headers on every file.**  License is AGPL-3.0-or-later
  unless the file carries an inherited license from an upstream
  vendored subtree (see `vendor/*/NOTICE.md` and
  `LICENSE-EXCEPTIONS.txt`).
- Match the surrounding file's indentation and formatting.  Reffs
  uses `clang-format` with the config carried at the reffs top
  level; formatting drift is caught by `make style` there.
- Any new encoder registered in `lib/ec/` MUST come with a
  corresponding `tests/*_test.c` and MUST be added to
  `tools/ec_bench.c`'s encoder registry.

## Vendored subtrees

The `vendor/` tree contains verbatim (or nearly-verbatim) copies
of SnapRAID's raid/, Linux md/raid6/, and Intel ISA-L
erasure_code/.  Refresh procedure:

1. Copy the upstream sources into the corresponding
   `vendor/<name>/` directory.
2. Preserve the vendored file's original SPDX header untouched.
3. Do NOT edit the vendored sources to fix warnings or style; the
   per-directory `Makefile.am` handles suppression via `-w` or
   sanitizer ignorelists.

## Bug reports

Open an issue at
<https://github.com/loghyr/reffs-ec-microbench/issues>.  Include:

- Host OS + version, CPU model, compiler + version
- Output of `./configure` summary
- Full `make -j 2>&1` output
- Output of `./tools/ec_bench --iters 20 --sizes 65536`
- Reproducer command line if applicable

Behavioural bugs in the encoders themselves should ideally be
filed upstream at <https://github.com/loghyr/reffs/issues>.
