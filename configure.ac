dnl -*- Mode: Autoconf; tab-width: 4; indent-tabs-mode: nil; fill-column: 102 -*-
# Process this file with autoconf to create configure.

AC_PREREQ([2.65])

# ====================
# Version informations
# ====================
m4_define([libqxp_version_major],[0])
m4_define([libqxp_version_minor],[0])
m4_define([libqxp_version_micro],[2])
m4_define([libqxp_version],[libqxp_version_major.libqxp_version_minor.libqxp_version_micro])

# =============
# Automake init
# =============
AC_INIT([libqxp],[libqxp_version])
AC_CONFIG_MACRO_DIR([m4])
AC_CONFIG_HEADER([config.h])
AM_INIT_AUTOMAKE([1.11 foreign dist-xz dist-bzip2])
AM_SILENT_RULES([yes])
AC_LANG([C++])

# ===========================
# Find required base packages
# ===========================
m4_ifdef([AM_PROG_AR], [AM_PROG_AR])
AC_PROG_CC
AC_PROG_CPP
AC_PROG_CXX
AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_MAKE_SET
LT_INIT([win32-dll disable-static pic-only])
AC_CANONICAL_HOST

AX_CXX_COMPILE_STDCXX_11
AX_GCC_FUNC_ATTRIBUTE([format])
AX_GCC_FUNC_ATTRIBUTE([visibility])
DLP_FALLTHROUGH

PKG_PROG_PKG_CONFIG([0.20])

# ====================
# Find additional apps
# ====================
PKG_CHECK_MODULES([REVENGE],[librevenge-0.0])

# ==================
# Find boost headers
# ==================
saved_CPPFLAGS="${CPPFLAGS}"
CPPFLAGS="${BOOST_CFLAGS} ${CPPFLAGS}"
AC_CHECK_HEADERS(
        boost/cstdint.hpp \
        boost/math/constants/constants.hpp \
        boost/optional.hpp \
        boost/range/adaptor/reversed.hpp \
        boost/variant.hpp \
        ,
    [],
    [AC_MSG_ERROR([Required boost headers not found.])],
    []
)
CPPFLAGS="${saved_CPPFLAGS}"
AC_SUBST([BOOST_CFLAGS])

# ========
# Find icu
# ========
PKG_CHECK_MODULES([ICU], [icu-uc])
AC_SUBST(ICU_CFLAGS)
AC_SUBST(ICU_LIBS)

# =================================
# Libtool/Version Makefile settings
# =================================
AC_SUBST(QXP_MAJOR_VERSION, [libqxp_version_major])
AC_SUBST(QXP_MINOR_VERSION, [libqxp_version_minor])
AC_SUBST(QXP_MICRO_VERSION, [libqxp_version_micro])
AC_SUBST(QXP_VERSION, [libqxp_version])
# AC_SUBST(LT_RELEASE, [libqxp_version_major.libqxp_version_minor])
LT_CURRENT=`expr 100 '*' libqxp_version_major + libqxp_version_minor`
# For 1.0.0 comment the first line and uncomment the second
LT_AGE=0
# LT_AGE=libqxp_version_minor
AC_SUBST(LT_CURRENT)
AC_SUBST(LT_REVISION, [libqxp_version_micro])
AC_SUBST(LT_AGE)

# ==========================
# Platform check for windows
# ==========================
AC_MSG_CHECKING([for native Win32])
AS_CASE([$host],
    [*-*-mingw*], [
        native_win32=yes
        LIBQXP_WIN32_RESOURCE=libqxp-win32res.lo
        QXP2RAW_WIN32_RESOURCE=qxp2raw-win32res.lo QXP2SVG_WIN32_RESOURCE=qxp2svg-win32res.lo 
    ], [
        native_win32=no
        LIBQXP_WIN32_RESOURCE=
        QXP2RAW_WIN32_RESOURCE= QXP2SVG_WIN32_RESOURCE= 
    ]
)
AC_MSG_RESULT([$native_win32])
AM_CONDITIONAL(OS_WIN32, [test "x$native_win32" = "xyes"])
AC_SUBST([LIBQXP_WIN32_RESOURCE])
AC_SUBST([QXP2RAW_WIN32_RESOURCE]) AC_SUBST([QXP2SVG_WIN32_RESOURCE]) 

AC_MSG_CHECKING([for Win32 platform in general])
AS_CASE([$host],
    [*-*-mingw*|*-*-cygwin*], [platform_win32=yes],
    [platform_win32=no]
)
AC_MSG_RESULT([$platform_win32])
AM_CONDITIONAL([PLATFORM_WIN32], [test "x$platform_win32" = "xyes"])

AS_IF([test "x$platform_win32" = "xyes"],
    [],
    [
        AC_MSG_CHECKING([for -fvisibility=hidden compiler flag])
        saved_CXXFLAGS="$CXXFLAGS"
        CXXFLAGS="$CXXFLAGS -fvisibility=hidden"
        AC_TRY_COMPILE([], [], [have_visibility=yes], [have_visibility=no])
        AC_MSG_RESULT([$have_visibility])
        CXXFLAGS="$saved_CXXFLAGS"
    ]
)
AM_CONDITIONAL([HAVE_VISIBILITY], [
    test x"$have_visibility" = xyes && test x"$ax_cv_have_func_attribute_visibility" = xyes])

# ================
# Check for cflags
# ================
AC_ARG_ENABLE([werror],
    [AS_HELP_STRING([--enable-werror], [Treat all warnings as errors, usefull for development])],
    [enable_werror="$enableval"],
    [enable_werror=no]
)
AS_IF([test x"$enable_werror" != "xno"], [
    CFLAGS="$CFLAGS -Werror"
    CXXFLAGS="$CXXFLAGS -Werror"
])
AC_ARG_ENABLE([weffc],
    [AS_HELP_STRING([--disable-weffc], [ Disable -Weffc++ warnings, usefull when using an old version of gcc or of boost])],
    [enable_weffc="$enableval"],
    [enable_weffc=yes]
)
AC_ARG_ENABLE([wparanoic],
    [AS_HELP_STRING([--enable-wparanoic], [Enable a lot of warnings...])],
    [enable_wparanoic="$enableval"],
    [enable_wparanoic=no]
)
# Courtesy of Glib: Ensure MSVC-compatible struct packing convention
# is used when compiling for Win32 with gcc.
AS_IF([test "x$native_win32" = "xyes"], [
    AC_CHECK_TOOL(WINDRES, windres)
    AS_IF([test x"$GCC" = xyes], [
        AC_MSG_CHECKING([how to get MSVC-compatible struct packing])
        AS_IF([test -z "$ac_cv_prog_CC"], [
            our_gcc="$CC"
        ], [
            our_gcc="$ac_cv_prog_CC"
        ])
        AS_IF([$our_gcc -v --help 2>/dev/null | grep ms-bitfields >/dev/null], [
            msnative_struct="-mms-bitfields"
            CFLAGS="$CFLAGS $msnative_struct"
            CXXFLAGS="$CXXFLAGS $msnative_struct"
            AC_MSG_RESULT([${msnative_struct}])
        ], [
            AC_MSG_RESULT([no way])
            AC_MSG_WARN([produced libraries might be incompatible with MSVC-compiled code])
        ])
    ])
    CFLAGS="$CFLAGS -Wall -Wextra -pedantic"
    CXXFLAGS="$CXXFLAGS -Wall -Wextra -Wshadow -pedantic"
], [
    AS_IF([test x"$GCC" = xyes], [
        # Be tough with warnings and produce less careless code
        CFLAGS="$CFLAGS -Wall -Wextra -pedantic"
        CXXFLAGS="$CXXFLAGS -Wall -Wextra -pedantic -Wshadow -Wunused-variable"
        AS_IF([test x"$enable_weffc" != "xno"], [
            CXXFLAGS="$CXXFLAGS -Weffc++"
        ])
        AS_IF([test x"$enable_wparanoic" != "xno"], [
            CXXFLAGS="$CXXFLAGS -Wcast-align -Wcast-qual -Wchar-subscripts"
            CXXFLAGS="$CXXFLAGS -Wcomment -Wconversion -Wdisabled-optimization"
            CXXFLAGS="$CXXFLAGS -Wfloat-equal -Wformat -Wformat=2"
            CXXFLAGS="$CXXFLAGS -Wformat-nonliteral -Wformat-security"
            CXXFLAGS="$CXXFLAGS -Wformat-y2k -Wimport -Winit-self -Winvalid-pch"
            CXXFLAGS="$CXXFLAGS -Wmissing-braces -Wmissing-field-initializers"
            CXXFLAGS="$CXXFLAGS -Wmissing-format-attribute -Wmissing-include-dirs"
            CXXFLAGS="$CXXFLAGS -Wmissing-noreturn -Wpacked -Wparentheses"
            CXXFLAGS="$CXXFLAGS -Wpointer-arith -Wredundant-decls -Wreturn-type"
            CXXFLAGS="$CXXFLAGS -Wsequence-point -Wsign-compare -Wstrict-aliasing"
            CXXFLAGS="$CXXFLAGS -Wstrict-aliasing=2 -Wswitch -Wswitch-default"
            CXXFLAGS="$CXXFLAGS -Wswitch-enum -Wtrigraphs -Wunknown-pragmas -Wunused"
            CXXFLAGS="$CXXFLAGS -Wunused-function -Wunused-label -Wunused-parameter"
            CXXFLAGS="$CXXFLAGS -Wunused-value -Wvariadic-macros"
            CXXFLAGS="$CXXFLAGS -Wvolatile-register-var -Wwrite-strings"
        ])
    ])
])
AC_SUBST(DEBUG_CXXFLAGS)

# ============
# Debug switch
# ============
AC_ARG_ENABLE([debug],
    [AS_HELP_STRING([--enable-debug], [Turn on debugging])],
    [enable_debug="$enableval"],
    [enable_debug=no]
)
AS_IF([test "x$enable_debug" = "xyes"], [
    DEBUG_CXXFLAGS="-DDEBUG -g"
    CXXFLAGS="$CXXFLAGS -O0"
    CFLAGS="$CFLAGS -O0"
], [
    DEBUG_CXXFLAGS="-DNDEBUG"
])
AC_SUBST(DEBUG_CXXFLAGS)

# ==========
# Unit tests
# ==========
AC_ARG_ENABLE([tests],
    [AS_HELP_STRING([--enable-tests], [Build and run unit tests])],
    [enable_tests="$enableval"],
    [enable_tests=yes]
)
AS_IF([test "x$enable_tests" = "xyes"], [
    PKG_CHECK_MODULES([CPPUNIT], [cppunit])
], [])
AC_SUBST([CPPUNIT_CFLAGS])
AC_SUBST([CPPUNIT_LIBS])
AM_CONDITIONAL(WITH_TESTS, [test "x$enable_tests" = "xyes"])
AS_IF([test "x$enable_tests" = "xyes"], [need_stream=yes])

# =============
# Documentation
# =============
AC_ARG_WITH(docs,
    [AS_HELP_STRING([--without-docs], [Do not build documentation])],
    [with_docs="$withval"],
    [AS_IF([test "x$native_win32" = "xyes"], [with_docs=no], [with_docs=yes])]
)
AS_IF([test "x$with_docs" != "xno"], [
    AC_PATH_PROG(DOXYGEN, [doxygen])
    AS_IF([test -z "$DOXYGEN"], [
        AC_MSG_WARN([*** Could not find doxygen in your PATH.])
        AC_MSG_WARN([*** The documentation will not be built.])
        build_docs=no
    ], [build_docs=yes])
], [build_docs=no])
AM_CONDITIONAL([WITH_DOCS], [test "x$build_docs" != "xno"])

# ================
# Conversion tools
# ================
AC_ARG_ENABLE(tools,
    [AS_HELP_STRING([--disable-tools], [Do not build conversion tools])],
    [enable_tools="$enableval"],
    [AS_IF([test "x$native_win32" = "xyes"], [enable_tools=no], [enable_tools=yes])]
)
AM_CONDITIONAL([BUILD_TOOLS], [test "x$enable_tools" = "xyes"])
AS_IF([test "x$enable_tools" = "xyes"], [need_stream=yes; need_generators=yes])

# =======
# Fuzzers
# =======
AC_ARG_ENABLE([fuzzers],
    [AS_HELP_STRING([--enable-fuzzers], [Build fuzzer(s)])],
    [enable_fuzzers="$enableval"],
    [enable_fuzzers=no]
)
AM_CONDITIONAL([BUILD_FUZZERS], [test "x$enable_fuzzers" = "xyes"])
AS_IF([test "x$enable_fuzzers" = "xyes"], [need_stream=yes; need_generators=yes])

AS_IF([test "x$need_generators" = "xyes"], [
    PKG_CHECK_MODULES([REVENGE_GENERATORS],[librevenge-generators-0.0])
])
AC_SUBST([REVENGE_GENERATORS_CFLAGS])
AC_SUBST([REVENGE_GENERATORS_LIBS])

AS_IF([test "x$need_stream" = "xyes"], [
    PKG_CHECK_MODULES([REVENGE_STREAM],[librevenge-stream-0.0])
])
AC_SUBST([REVENGE_STREAM_CFLAGS])
AC_SUBST([REVENGE_STREAM_LIBS])

# =====================
# Prepare all .in files
# =====================
AC_CONFIG_FILES([
Makefile
inc/Makefile
inc/libqxp/Makefile
src/Makefile
src/conv/Makefile
src/fuzz/Makefile
src/lib/Makefile
src/lib/libqxp.rc
src/conv/raw/Makefile
src/conv/raw/qxp2raw.rc
src/conv/svg/Makefile
src/conv/svg/qxp2svg.rc
src/conv/text/Makefile
src/conv/text/qxp2text.rc
src/test/Makefile
build/Makefile
build/win32/Makefile
docs/Makefile
docs/doxygen/Makefile
libqxp-$QXP_MAJOR_VERSION.$QXP_MINOR_VERSION.pc:libqxp.pc.in
])
AC_OUTPUT

# ==============================================
# Display final informations about configuration
# ==============================================
AC_MSG_NOTICE([
==============================================================================
Build configuration:
    debug:           ${enable_debug}
    docs:            ${build_docs}
    fuzzers:         ${enable_fuzzers}
    tests:           ${enable_tests}
    tools:           ${enable_tools}
    werror:          ${enable_werror}
==============================================================================
])

dnl vim:set shiftwidth=4 softtabstop=4 expandtab:
