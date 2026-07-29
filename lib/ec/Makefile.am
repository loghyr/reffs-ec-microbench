# SPDX-FileCopyrightText: 2026 Tom Haynes <loghyr@gmail.com>
# SPDX-License-Identifier: AGPL-3.0-or-later

SUBDIRS = tests

AM_CFLAGS = -I$(top_srcdir)/lib/include

noinst_LTLIBRARIES = libreffs_ec.la
libreffs_ec_la_SOURCES = gf.c isa_l.c linux_md.c matrix.c mirror.c \
	mojette.c mojette_encoding.c rs.c snapraid.c stripe.c xor.c

# Vendor sub-libraries live under top-level vendor/ in this carve-out
# (in reffs they live under lib/ec/).  Adjust LIBADD paths accordingly.
libreffs_ec_la_LIBADD = \
	$(top_builddir)/vendor/snapraid-raid/libreffs_snapraid_raid.la \
	$(top_builddir)/vendor/linux-md-raid/libreffs_linux_md_raid.la \
	$(top_builddir)/vendor/isa-l-erasure/libreffs_isa_l_erasure.la

# isa_l.c needs to see the vendored ISA-L public header.
isa_l.lo: AM_CFLAGS += -I$(top_srcdir)/vendor/isa-l-erasure/include

# linux_md.c and snapraid.c use paths relative to the vendor
# subdirs (in reffs those live under lib/ec/; here they live
# under vendor/).  Add -I$(top_srcdir)/vendor so the includes
# resolve without touching the source.
linux_md.lo:  AM_CFLAGS += -I$(top_srcdir)/vendor
snapraid.lo:  AM_CFLAGS += -I$(top_srcdir)/vendor

# Enable AVX2 for the Mojette forward transform SIMD fast path.
mojette.lo: AM_CFLAGS += @AVX2_CFLAGS@

# --enable-noscalar-vec applies -fno-tree-vectorize -fno-slp-vectorize
# to the encoding source files so the "scalar" path used by the
# SIMD-vs-scalar honest comparison cannot be auto-vectorized by the
# compiler.  When the flag is absent NOSCALAR_VEC_CFLAGS is empty so
# this is a no-op for normal builds.
gf.lo:            AM_CFLAGS += @NOSCALAR_VEC_CFLAGS@
matrix.lo:        AM_CFLAGS += @NOSCALAR_VEC_CFLAGS@
mojette.lo:       AM_CFLAGS += @NOSCALAR_VEC_CFLAGS@
mojette_encoding.lo: AM_CFLAGS += @NOSCALAR_VEC_CFLAGS@
stripe.lo:        AM_CFLAGS += @NOSCALAR_VEC_CFLAGS@

# rs.lo needs @NOSCALAR_VEC_CFLAGS@ (like the peers above) AND on
# x86_64 also -mssse3 so Slice R.2's pshufb intrinsic path
# activates (__SSSE3__ is only defined when the flag is on the
# compile line).
if HAVE_MD_X86_SIMD
rs.lo:            AM_CFLAGS += @NOSCALAR_VEC_CFLAGS@ -mssse3
else
rs.lo:            AM_CFLAGS += @NOSCALAR_VEC_CFLAGS@
endif
