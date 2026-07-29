# SPDX-FileCopyrightText: 2026 Tom Haynes <loghyr@gmail.com>
# SPDX-License-Identifier: AGPL-3.0-or-later

ACLOCAL_AMFLAGS = -I m4

# vendor/ MUST come before lib/ec/ so the sub-libraries libtool
# LIBADDs are resolved from lib/ec/Makefile.am are already built.
SUBDIRS = vendor lib tools

EXTRA_DIST = README.md LICENSE LICENSE-EXCEPTIONS.txt BUILD-TODO.md
