<!--
SPDX-FileCopyrightText: 2026 Tom Haynes <loghyr@gmail.com>
SPDX-License-Identifier: AGPL-3.0-or-later
-->

# reffs-ec-microbench

Standalone microbenchmark harness for the erasure-encoding
implementations that back the reference implementation
[reffs](https://github.com/loghyr/reffs), carved out so that
implementers and reviewers of
[draft-haynes-nfsv4-flexfiles-v2](https://datatracker.ietf.org/doc/draft-haynes-nfsv4-flexfiles-v2)
can reproduce the benchmark numbers cited in that draft family
without cloning or building the full reffs tree.

## What's inside

**Encoders** (`lib/ec/`) implementing the seven MTI-candidate
encodings from the flexfiles-v2 encoding registry:

- `xor.c` -- FFV2_ENCODING_XOR_PARITY (m=1 XOR parity)
- `mirror.c` -- FFV2_ENCODING_MIRRORED (n-way mirror)
- `rs.c` -- FFV2_ENCODING_RS_VANDERMONDE (Reed-Solomon Vandermonde)
- `mojette.c` + `mojette.h` + `mojette_encoding.c` --
  FFV2_ENCODING_MOJETTE_SYSTEMATIC + FFV2_ENCODING_MOJETTE_NON_SYSTEMATIC
- `isa_l.c` -- FFV2_ENCODING_ISA_L_RS (Intel ISA-L Cauchy wrapper)
- `linux_md.c` -- FFV2_ENCODING_LINUX_MD_RAID (Linux md P+Q wrapper)
- `snapraid.c` -- FFV2_ENCODING_SNAPRAID_CAUCHY (SnapRAID Cauchy wrapper)

Supporting infrastructure: `gf.c/h` (GF(2^8) primitives), `matrix.c/h`
(Vandermonde / Cauchy matrix operations), `stripe.c` (stripe utility).

**Public API** (`lib/include/reffs/ec.h`) -- the encoding-registry
interface.

**Benchmarks** (`tools/`) -- `ec_bench` (algorithmic microbenchmark
across all encoders + geometries + shard sizes) and `moj_bench`
(Mojette-focused microbench).

**Tests** (`lib/ec/tests/`) -- per-encoder correctness tests.

**Vendored** (`vendor/`) -- upstream code kept unchanged:

- `isa-l-erasure/` (Intel ISA-L, BSD-3)
- `linux-md-raid/` (Linux md RAID-6, GPL-2)
- `snapraid-raid/` (SnapRAID, GPL-2)

## License

**Primary**: AGPL-3.0-or-later.  See `LICENSE`.

The Mojette-derived files (`lib/ec/mojette.c`, `lib/ec/mojette.h`,
`lib/ec/mojette_encoding.c`, `tools/moj_bench.c`) are covered by
Pierre Evenou's 2026-05-04 grant permitting relicense to
AGPL-3.0-or-later.  See `LICENSE-EXCEPTIONS.txt`.

The vendored subdirectories each carry their own upstream
licenses (see per-file SPDX headers): ISA-L is BSD-3, Linux md
RAID-6 is GPL-2, SnapRAID is GPL-2.  These do not further
restrict the AGPL-3.0-or-later posture of this repository as a
whole for use, only for redistribution.

## Building

**Not yet.**  See `BUILD-TODO.md` for the outstanding scaffolding
work.  This is a v0.0.1 scaffold state; the source is here but the
build system has not been carved out from the reffs autotools
tree yet.  See `BUILD-TODO.md` for the shopping list.

## Provenance

Every source file in `lib/ec/`, `lib/include/reffs/`, `tools/`,
and `lib/ec/tests/` was copied verbatim from
[reffs](https://github.com/loghyr/reffs) commit `0d885b9edff9` on
2026-07-28, preserving SPDX headers.  Vendored subdirectories were
copied under `vendor/` (in reffs they live under `lib/ec/` proper;
the move to `vendor/` makes the license posture more visible to
first-time readers).

## Reproducing the flexfiles-v2 draft's benchmark numbers

Once the build system is in place (see `BUILD-TODO.md`), the
intended reproduction command is:

    ./tools/ec_bench --sizes 4096,16384,65536,262144

which prints a matrix of (encoder, geometry, size) -> MB/s cells
matching the numbers cited in the flexfiles-v2 draft family's
encoding-cost material.

## Related

- Draft:
  [draft-haynes-nfsv4-flexfiles-v2](https://datatracker.ietf.org/doc/draft-haynes-nfsv4-flexfiles-v2)
- Reference implementation:
  [reffs](https://github.com/loghyr/reffs)
