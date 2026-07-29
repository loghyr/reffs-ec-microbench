# SPDX-FileCopyrightText: 2026 Tom Haynes <loghyr@gmail.com>
# SPDX-License-Identifier: AGPL-3.0-or-later

AM_CFLAGS = \
	-I$(top_srcdir)/lib/include \
	-I$(top_srcdir)/lib/ec

# Both tools link only against libreffs_ec.la (no NFS, no RPC, no TLS).
# ec_bench: sweep of every registered encoder + memcmp correctness check.
# moj_bench: Mojette inverse algorithm microbench (peel vs geometry-driven).

ec_bench_SOURCES = ec_bench.c
ec_bench_LDADD = $(top_builddir)/lib/ec/libreffs_ec.la

moj_bench_SOURCES = moj_bench.c
moj_bench_LDADD = $(top_builddir)/lib/ec/libreffs_ec.la

bin_PROGRAMS = ec_bench moj_bench
