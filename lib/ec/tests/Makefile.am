# SPDX-FileCopyrightText: 2026 Tom Haynes <loghyr@gmail.com>
# SPDX-License-Identifier: AGPL-3.0-or-later

# The unit tests use libcheck.  When check(1) is not installed
# (HAVE_CHECK is false), skip the tests directory entirely.
if HAVE_CHECK

AM_CFLAGS = -I$(top_srcdir)/lib/include \
	-I$(top_srcdir)/lib/ec \
	$(CHECK_CFLAGS)

LDADD = $(CHECK_LIBS) \
	$(top_builddir)/lib/ec/libreffs_ec.la

check_PROGRAMS = $(TESTS)

TESTS = \
	gf_test \
	matrix_test \
	mirror_test \
	mojette_test \
	mojette_encoding_test \
	rs_test \
	snapraid_test \
	xor_test \
	linux_md_test \
	isa_l_test

endif
