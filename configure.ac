dnl SPDX-FileCopyrightText: 2026 Tom Haynes <loghyr@gmail.com>
dnl SPDX-License-Identifier: AGPL-3.0-or-later

AC_INIT([reffs-ec-microbench], [0.1.0], [loghyr@gmail.com])
AC_CONFIG_AUX_DIR([.])
AC_CONFIG_MACRO_DIRS([m4])
AM_INIT_AUTOMAKE([foreign subdir-objects -Wall -Wno-portability])
AM_SILENT_RULES([yes])

AC_CONFIG_SRCDIR([lib/ec/mojette.c])
AC_CONFIG_HEADERS([config.h])

AC_PROG_CC
AC_PROG_AWK
AC_USE_SYSTEM_EXTENSIONS
m4_ifdef([AM_PROG_AR], [AM_PROG_AR])
LT_INIT

dnl --- host + Darwin/BSD path adjustments (from reffs) -----------------
AC_CANONICAL_HOST
host_is_freebsd=no
host_is_darwin=no
AS_CASE([$host_os],
    [freebsd*|dragonfly*], [
        CPPFLAGS="-I/usr/local/include $CPPFLAGS"
        LDFLAGS="-L/usr/local/lib $LDFLAGS"
        host_is_freebsd=yes
    ],
    [darwin*], [
        AS_IF([test -d /opt/homebrew/include],
              [CPPFLAGS="-I/opt/homebrew/include $CPPFLAGS"])
        AS_IF([test -d /opt/homebrew/lib],
              [LDFLAGS="-L/opt/homebrew/lib $LDFLAGS"])
        AS_IF([test -d /usr/local/include],
              [CPPFLAGS="-I/usr/local/include $CPPFLAGS"])
        AS_IF([test -d /usr/local/lib],
              [LDFLAGS="-L/usr/local/lib $LDFLAGS"])
        host_is_darwin=yes
    ])
AM_CONDITIONAL([HOST_FREEBSD], [test "x$host_is_freebsd" = "xyes"])
AM_CONDITIONAL([HOST_DARWIN],  [test "x$host_is_darwin"  = "xyes"])

dnl --- per-arch SIMD gates for vendored Linux md/raid6 (from reffs) ---
have_md_x86_simd=no
have_md_neon_simd=no
AS_CASE([$host_cpu],
    [x86_64|amd64],  [have_md_x86_simd=yes],
    [aarch64|arm64], [have_md_neon_simd=yes])
AM_CONDITIONAL([HAVE_MD_X86_SIMD],  [test "x$have_md_x86_simd" = "xyes"])
AM_CONDITIONAL([HAVE_MD_NEON_SIMD], [test "x$have_md_neon_simd" = "xyes"])

dnl --- ISA-L x86 SIMD gate (needs x86_64 AND NASM) (from reffs) -------
AC_CHECK_PROG([NASM], [nasm], [nasm])
have_isa_l_x86_simd=no
AS_IF([test "x$have_md_x86_simd" = "xyes" && test "x$NASM" != "x"],
    [have_isa_l_x86_simd=yes])
AM_CONDITIONAL([HAVE_ISA_L_X86_SIMD], [test "x$have_isa_l_x86_simd" = "xyes"])

dnl --- HAVE_ASSEMBLY for vendored SnapRAID raid/ (from reffs) ---------
AC_DEFINE([HAVE_ASSEMBLY], [1],
    [Define to 1 to enable inline assembly in the vendored SnapRAID raid/ library.])

dnl --- -fsanitize-ignorelist= support (clang only; gcc rejects) -------
dnl The snapraid-raid/Makefile.am references this flag; if the compiler
dnl does not accept it (gcc), leave the substitution empty so the
dnl vendor Makefile does not pass a flag that will fatal-error the
dnl build.  Under UBSan+clang this preserves the alignment-check
dnl carve-out described in vendor/snapraid-raid/NOTICE.md.
UBSAN_IGNORELIST_CFLAGS=""
AC_MSG_CHECKING([whether $CC supports -fsanitize-ignorelist=])
echo "" > conftest.ignorelist
saved_CFLAGS="$CFLAGS"
CFLAGS="$CFLAGS -fsanitize-ignorelist=conftest.ignorelist"
AC_COMPILE_IFELSE(
    [AC_LANG_PROGRAM([], [])],
    [AC_MSG_RESULT([yes])
     UBSAN_IGNORELIST_SUPPORTED=yes],
    [AC_MSG_RESULT([no])
     UBSAN_IGNORELIST_SUPPORTED=no])
CFLAGS="$saved_CFLAGS"
rm -f conftest.ignorelist
AC_SUBST([UBSAN_IGNORELIST_SUPPORTED])
AM_CONDITIONAL([HAVE_UBSAN_IGNORELIST],
    [test "x$UBSAN_IGNORELIST_SUPPORTED" = "xyes"])

dnl --- AVX2 support for Mojette SIMD fast path (from reffs) -----------
AVX2_CFLAGS=""
AC_MSG_CHECKING([whether $CC supports -mavx2])
saved_CFLAGS="$CFLAGS"
CFLAGS="$CFLAGS -mavx2"
AC_COMPILE_IFELSE(
  [AC_LANG_PROGRAM(
    [[#include <immintrin.h>]],
    [[__m256i a = _mm256_setzero_si256(); (void)a;]])],
  [AC_MSG_RESULT([yes])
   AVX2_CFLAGS="-mavx2"],
  [AC_MSG_RESULT([no])])
CFLAGS="$saved_CFLAGS"
AC_SUBST([AVX2_CFLAGS])

dnl --- --enable-noscalar-vec toggle (from reffs) ----------------------
NOSCALAR_VEC_CFLAGS=""
AC_ARG_ENABLE([noscalar-vec],
    [AS_HELP_STRING([--enable-noscalar-vec],
        [disable compiler auto-vectorization of the scalar encoding paths (for SIMD-vs-scalar honest benchmarks)])],
    [], [enable_noscalar_vec=no])
AS_IF([test "x$enable_noscalar_vec" = "xyes"],
    [NOSCALAR_VEC_CFLAGS="-fno-tree-vectorize -fno-slp-vectorize"])
AC_SUBST([NOSCALAR_VEC_CFLAGS])

dnl --- Check unit-test framework for lib/ec/tests --------------------
PKG_PROG_PKG_CONFIG
have_check=no
PKG_CHECK_MODULES([CHECK], [check >= 0.15.2],
    [have_check=yes], [have_check=no])
AM_CONDITIONAL([HAVE_CHECK], [test "x$have_check" = "xyes"])

dnl --- Stricter -Wall unless user overrode CFLAGS (from reffs pattern)
AC_MSG_CHECKING([for C11 support])
saved_CFLAGS="$CFLAGS"
CFLAGS="-std=c11 $CFLAGS"
AC_COMPILE_IFELSE([AC_LANG_PROGRAM([],[])],
    [AC_MSG_RESULT([yes])],
    [AC_MSG_RESULT([no]); CFLAGS="$saved_CFLAGS"])

AC_CONFIG_FILES([
    Makefile
    lib/Makefile
    lib/ec/Makefile
    lib/ec/tests/Makefile
    vendor/Makefile
    vendor/snapraid-raid/Makefile
    vendor/linux-md-raid/Makefile
    vendor/isa-l-erasure/Makefile
    tools/Makefile
])

AC_OUTPUT

echo ""
echo "  reffs-ec-microbench $PACKAGE_VERSION configured"
echo ""
echo "  Host:              $host"
echo "  Compiler:          $CC"
echo "  AVX2 support:      $AVX2_CFLAGS"
echo "  md x86 SIMD:       $have_md_x86_simd"
echo "  md NEON SIMD:      $have_md_neon_simd"
echo "  ISA-L x86 SIMD:    $have_isa_l_x86_simd (NASM=$NASM)"
echo "  Check unit tests:  $have_check"
echo ""
