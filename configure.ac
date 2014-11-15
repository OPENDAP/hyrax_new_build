#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.69])
AC_INIT([Hyrax], [2.0.0], [support@opendap.org])
AC_CONFIG_SRCDIR([hyrax_version.h])

AC_CONFIG_AUX_DIR(conf)
AC_CONFIG_MACRO_DIR([conf])

AC_CONFIG_HEADERS([libdap/config.h libdap/dods-datatypes-config.h libdap/xdr-datatypes-config.h])

AM_INIT_AUTOMAKE
AM_PROG_LEX

# Checks for programs.
AC_PROG_AWK
AC_PROG_YACC
AC_PROG_CXX
AC_PROG_CC

# Call this gnulib macro right after a working C Compiler is found
gl_EARLY

# echo "CC = $CC"
if test "$CC" = "gcc"
then
    AM_CONDITIONAL([COMPILER_IS_GCC],[true])
else
    AM_CONDITIONAL([COMPILER_IS_GCC],[false])
fi

# AC_PROG_LEX
AC_PROG_INSTALL
AC_PROG_LN_S
AC_PROG_MAKE_SET
AC_PROG_LIBTOOL

dnl We really need bison and not yacc. If you use AC_PROG_YACC, the resulting 
dnl Makefile will call bison -y which does not know how to make the parsers 
dnl libdap requires. jhrg 6/15/05
AC_CHECK_PROG(YACC,[bison],[bison])

gl_SOURCE_BASE([libdap/gl])
gl_M4_BASE([libdap/gl/m4])
gl_MODULES([regex btyeswap])

gl_INIT

# Checks for header files.
AC_HEADER_DIRENT
AC_HEADER_STDC
AC_HEADER_SYS_WAIT

AC_FUNC_ALLOCA
AC_CHECK_HEADERS([OS.h fcntl.h inttypes.h langinfo.h libintl.h limits.h locale.h malloc.h netdb.h \
netinet/in.h stddef.h stdint.h stdlib.h string.h sys/file.h sys/param.h sys/time.h unistd.h wchar.h wctype.h])

# Checks for typedefs, structures, and compiler characteristics.
AC_CHECK_HEADER_STDBOOL
AC_TYPE_UID_T
AC_C_INLINE
AC_TYPE_INT16_T
AC_TYPE_INT32_T
AC_TYPE_INT64_T
AC_TYPE_INT8_T
AC_TYPE_MODE_T
AC_TYPE_OFF_T
AC_C_RESTRICT
AC_TYPE_SIZE_T
AC_TYPE_SSIZE_T
AC_CHECK_MEMBERS([struct stat.st_blksize])
AC_TYPE_UINT16_T
AC_TYPE_UINT32_T
AC_TYPE_UINT64_T
AC_TYPE_UINT8_T
AC_CHECK_TYPES([ptrdiff_t])

# Checks for library functions.
# AM_GNU_GETTEXT jhrg 11/14/14
AC_FUNC_ERROR_AT_LINE
AC_FUNC_MALLOC
AC_FUNC_MBRTOWC
AC_FUNC_MKTIME
AC_FUNC_REALLOC
AC_FUNC_STRTOD
AC_CHECK_FUNCS([alarm atexit btowc floor getcwd getpagesize isascii localeconv localtime_r memchr memmove \
 memset mkdir nl_langinfo putenv re_comp regcomp setenv setlocale strcasecmp strchr strerror strncasecmp \
 strtol strtoul wcwidth])

dnl Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_HEADER_TIME
AC_STRUCT_TM
AC_C_VOLATILE
AC_C_BIGENDIAN
        
DODS_CHECK_SIZES

AC_ARG_ENABLE([runtime-endian-check],
    AC_HELP_STRING([--enable-runtime-endian-check], [Enable runtime tests for big- or little-endian byte order (default is NO)])
)

AS_IF([test "x$enable_runtime_endian_check" = "xyes"], [
    dnl Do the stuff needed for enabling the feature
    AC_DEFINE([COMPUTE_ENDIAN_AT_RUNTIME], 1, [Should a function (run-time) be used to determine the byte order?])
])

# FIXME Add the --with-libcurl and --with-libxml2 code in here

AC_CHECK_LIB([pthread], [pthread_kill], 
    [PTHREAD_LIBS="-lpthread"],
    [AC_MSG_ERROR([I could not find pthreads])])
AC_SUBST([PTHREAD_LIBS])

AC_CHECK_LIB([uuid], [uuid_generate], 
    [UUID_LIBS="-luuid"],
    [UUID_LIBS=""])
AC_SUBST([UUID_LIBS])

AM_PATH_CPPUNIT(1.12.0,
    [AM_CONDITIONAL([CPPUNIT], [true])],
    [AM_CONDITIONAL([CPPUNIT], [false])])

AC_ARG_ENABLE([dap4], 
    [AC_HELP_STRING([--enable-dap4], [Enable DAP4 types and responses (default is NO)])],
    [case "${enableval}" in
       yes) enable_dap4=yes ;;
       no)  enable_dap4=no ;;
       *) AC_MSG_ERROR([bad value ${enableval} for --enable-dap4]) ;;
     esac],
[enable_dap4=no])

AS_IF([test "x$enable_dap4" = "xyes"], [
    dnl Do the stuff needed for enabling the feature
    dnl echo "Defining DAP4! "
    AC_DEFINE([DAP4], 1, [Should DAP4 support be enabled?])
    AM_CONDITIONAL([DAP4_DEFINED], [true]) ],
    [ dnl else enable_dap4 is false
    AM_CONDITIONAL([DAP4_DEFINED], [false])
])

AC_ARG_ENABLE([developer],
[AS_HELP_STRING([--enable-developer],
[Build a debug (-g3 -O0) version of this code and include assert() calls in the code (default is no)])],
[case "${enableval}" in
       yes) build_developer=true ;;
       no)  build_developer=false ;;
       *) AC_MSG_ERROR([bad value ${enableval} for --enable-developer]) ;;
     esac],
[build_developer=false])

if test "x$build_developer" = "xtrue"; then
   AC_MSG_NOTICE(["Building developer version"])
else
   AC_MSG_NOTICE(["Not building developer version"])
   AC_DEFINE([NDEBUG], [1], [Remove this to activate assert() calls.])
fi
AM_CONDITIONAL([BUILD_DEVELOPER], [test "x$build_developer" = "xtrue"])

AH_TOP([#ifndef _config_h
#define _config_h])

AH_BOTTOM([
#if defined(__GNUG__) || defined(__GNUC__)
#define not_used __attribute__ ((unused))
#else
#define not_used 
#endif /* __GNUG__ || __GNUC__ */

#endif /* _config_h */])

AC_CONFIG_FILES([Makefile])

AC_CONFIG_TESTDIR(libdap/tests, [.])

AC_CONFIG_FILES([libdap/Makefile
                 libdap/libdap.pc
                 libdap/libdapclient.pc
                 libdap/libdapserver.pc
                 libdap/main_page.doxygen
                 libdap/doxy.conf
                 libdap/abi_checker.xml

                 libdap/gl/Makefile
                 
                 libdap/tests/Makefile
                 libdap/tests/atlocal
                 
                 libdap/unit-tests/Makefile
                 libdap/unit-tests/cache-testsuite/Makefile])
                 
AC_CONFIG_FILES([libdap/dap-config], [chmod +x libdap/dap-config])

AC_OUTPUT
