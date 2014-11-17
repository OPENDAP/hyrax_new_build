#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ([2.69])
AC_INIT([Hyrax], [2.0.0], [support@opendap.org])
AC_CONFIG_SRCDIR([hyrax_version.h])

AC_CONFIG_AUX_DIR(conf)
AC_CONFIG_MACRO_DIR([conf])

# Set this to true here and use this in the submodule Makefile.ams
AM_CONDITIONAL([UNIFIED_HYRAX_BUILD], [true])

AC_CONFIG_HEADERS([libdap/config.h libdap/dods-datatypes-config.h libdap/xdr-datatypes-config.h])

AM_INIT_AUTOMAKE
AM_PROG_LEX

# Checks for programs.
AC_PROG_AWK
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

AC_CHECK_PROG(VALGRIND, valgrind, [valgrind --logfile=memcheck])
dnl Add valgrind options
DODS_GCOV_VALGRIND

gl_SOURCE_BASE([libdap/gl])
gl_M4_BASE([libdap/gl/m4])
gl_MODULES([regex btyeswap])

gl_INIT

# Checks for header files.
AC_HEADER_DIRENT
AC_HEADER_STDC
AC_HEADER_SYS_WAIT
AC_HEADER_TR1_FUNCTIONAL

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
# AC_FUNC_MKTIME
AC_FUNC_REALLOC
AC_FUNC_STRTOD

AC_CHECK_FUNCS([alarm atexit btowc floor getcwd getpagesize isascii \
localeconv localtime_r memchr memmove memset mkdir nl_langinfo putenv \
re_comp regcomp setenv setlocale strcasecmp strchr strerror strncasecmp \
strtol strtoul wcwidth timegm mktime])

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

# FIXME Make these calls into a function?
AC_ARG_WITH(curl,[  --with-curl=PFX   Prefix where curl/libcurl is installed (optional). This will override pkgconfig, etc.],
            with_curl_prefix="$withval", with_curl_prefix="")

dnl I wrote these checks because we need the *-config scripts to build, so 
dnl the AC_CHECK_LIB macro is not needed.

curlprivatereq=
curlprivatelibs=
curl_set=

if test -n "$with_curl_prefix" -a -x $with_curl_prefix/bin/curl-config
then
    AC_MSG_NOTICE([Using $with_curl_prefix as the curl prefix directory.])
    CURL_LIBS="`$with_curl_prefix/bin/curl-config --libs`"
    CURL_STATIC_LIBS=$CURL_LIBS
    curlprivatelibs="`$with_curl_prefix/bin/curl-config --libs`"
    CURL_CFLAGS="`$with_curl_prefix/bin/curl-config --cflags`"
    curl_set="yes"
elif test -n "$with_curl_prefix"
then
    AC_MSG_ERROR([You set the curl-prefix directory to $with_curl_prefix, but curl-config is not there.])
fi

if test -z "$curl_set"
then
    # curlprivatereq=
    # curlprivatelibs=
    libdap_pkgconfig_libcurl=yes
    libdap_libcurl_module='libcurl >= 7.19.0'
    PKG_CHECK_MODULES([CURL],[$libdap_libcurl_module],,
        [libdap_pkgconfig_libcurl=no])
    AC_MSG_CHECKING([for libcurl])
    
    if test $libdap_pkgconfig_libcurl = 'yes'
    then
	   curlprivatereq=$libdap_libcurl_module
	   CURL_STATIC_LIBS="`$PKG_CONFIG --static --libs libcurl`"
	   AC_MSG_RESULT([yes; used pkg-config])
    elif curl-config --version > /dev/null 2>&1
    then
	   version_libcurl=`curl-config --version | sed 's@libcurl \(.*\)@\1@'`

           AS_VERSION_COMPARE(["$version_libcurl"], ["7.19.0"], 
                [AC_MSG_ERROR([I could not find libcurl 7.19.0 or newer, found $version_libcurl])])
        
	   CURL_LIBS="`curl-config --libs`"
	   CURL_STATIC_LIBS=$CURL_LIBS
	   curlprivatelibs="`curl-config --libs`"
	   CURL_CFLAGS="`curl-config --cflags`"
	   AC_MSG_RESULT([yes; used curl-config and found version $version_libcurl])
    else
	   AC_MSG_ERROR([I could not find libcurl])
    fi
fi

AC_SUBST([curlprivatereq])
AC_SUBST([curlprivatelibs])
AC_SUBST([CURL_LIBS])
AC_SUBST([CURL_STATIC_LIBS])
AC_SUBST([CURL_CFLAGS])

AC_ARG_WITH(xml2,[  --with-xml2=PFX   Prefix where libxml2 is installed (optional). This will override pkgconfig, etc.],
            with_xml2_prefix="$withval", with_xml2_prefix="")

xmlprivatereq=
xmlprivatelibs=
xml_set=

if test -n "$with_xml2_prefix" -a -x $with_xml2_prefix/bin/xml2-config
then
    AC_MSG_NOTICE([Using $with_xml2_prefix as the libxml2 prefix directory.])
    XML2_LIBS="`$with_xml2_prefix/bin/xml2-config --libs`"
    dnl XML2_STATIC_LIBS=$XML2_LIBS
    xmlprivatelibs="`$with_xml2_prefix/bin/xml2-config --libs`"
    XML2_CFLAGS="`$with_xml2_prefix/bin/xml2-config --cflags`"
    xml_set="yes"
elif test -n "$with_xml2_prefix"
then
    AC_MSG_ERROR([You set the libxml2 prefix directory to $with_xml2_prefix, but xml2-config is not there.])
fi

if test -z "$xml_set"
then
libdap_pkgconfig_libxml2=yes
libdap_libxml2_module='libxml-2.0 >= 2.7.0'
PKG_CHECK_MODULES([XML2],[$libdap_libxml2_module], ,[libdap_pkgconfig_libxml2=no])
AC_MSG_CHECKING([for libxml2])
if test $libdap_pkgconfig_libxml2 = 'yes'
then
	xmlprivatereq=$libdap_libxml2_module
	dnl XML2_STATIC_LIBS="`$PKG_CONFIG --static --libs libxml-2.0`"
	XML2_LIBS="`$PKG_CONFIG --libs libxml-2.0`"
	AC_MSG_RESULT([yes; used pkg-config])
elif xml2-config --version > /dev/null 2>&1
then
	version_libxml2=`xml2-config --version`

        AS_VERSION_COMPARE(["$version_libxml2"], ["2.7.0"], 
                [AC_MSG_ERROR([I could not find libxml2 2.7.0 or newer])])
        
	XML2_LIBS="`xml2-config --libs`"
	dnl XML2_STATIC_LIBS=$XML2_LIBS
	XML2_CFLAGS="`xml2-config --cflags`"
	xmlprivatelibs="`xml2-config --libs`"
	AC_MSG_RESULT([yes; used xml2-config and found version $version_libxml2])
else
	AC_MSG_ERROR([I could not find ml2-config])
fi
fi

AC_SUBST([xmlprivatereq])
AC_SUBST([xmlprivatelibs])
AC_SUBST([XML2_LIBS])
dnl AC_SUBST([XML2_STATIC_LIBS])
AC_SUBST([XML2_CFLAGS])

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
AM_CONDITIONAL([BES_DEVELOPER], [test "x$build_developer" = "xtrue"])

## Things that are specific to libdap

# See util.cc:dods_root()
if test "$prefix" = "NONE"
then
AC_DEFINE([LIBDAP_ROOT], ["/usr/local"], [Set to the prefix directory])
else
AC_DEFINE_UNQUOTED([LIBDAP_ROOT], ["$prefix"], [Set to the prefix directory])
fi

AC_DEFINE(DAP_PROTOCOL_VERSION, ["3.5"], [Highest DAP version implemented?])
AC_SUBST(DAP_PROTOCOL_VERSION)

AC_DEFINE(CNAME, "libdap", [What sort of HTTP client is this?])
AC_DEFINE_UNQUOTED(CVER, "$PACKAGE_VERSION", [Client version number])
AC_DEFINE_UNQUOTED(DVR, "libdap/$PACKAGE_VERSION", [Client name and version combined])
AC_SUBST(DVR)

AC_DEFINE(EVAL, 1, [Should all the classes run ConstraintEvaluator::eval()?])
AC_SUBST(EVAL)

DAPLIB_CURRENT=17
DAPLIB_AGE=0
DAPLIB_REVISION=2
AC_SUBST(DAPLIB_CURRENT)
AC_SUBST(DAPLIB_AGE)
AC_SUBST(DAPLIB_REVISION)

LIBDAP_VERSION="$DAPLIB_CURRENT:$DAPLIB_REVISION:$DAPLIB_AGE"
AC_SUBST(LIBDAP_VERSION)

CLIENTLIB_CURRENT=6
CLIENTLIB_AGE=0
CLIENTLIB_REVISION=0
AC_SUBST(CLIENTLIB_CURRENT)
AC_SUBST(CLIENTLIB_AGE)
AC_SUBST(CLIENTLIB_REVISION)

CLIENTLIB_VERSION="$CLIENTLIB_CURRENT:$CLIENTLIB_REVISION:$CLIENTLIB_AGE"
AC_SUBST(CLIENTLIB_VERSION)

SERVERLIB_CURRENT=12
SERVERLIB_AGE=5
SERVERLIB_REVISION=1
AC_SUBST(SERVERLIB_CURRENT)
AC_SUBST(SERVERLIB_AGE)
AC_SUBST(SERVERLIB_REVISION)

SERVERLIB_VERSION="$SERVERLIB_CURRENT:$SERVERLIB_REVISION:$SERVERLIB_AGE"
AC_SUBST(SERVERLIB_VERSION)

## End of things for libdap

## BES specific configuration items

AM_CONDITIONAL([LIBDAP], [true]) # always true for the unified build

BES_OLD_LIBS=$LIBS
dnl z and bz2 library?
AC_CHECK_LIB( bz2, BZ2_bzReadOpen,
    [
	AM_CONDITIONAL([BZ2UNCOMPRESS], [true])
	BES_BZ2_LIBS=-lbz2
	AC_DEFINE([HAVE_LIBBZ2], [1], [libbz2])
    ],
    [ AM_CONDITIONAL([BZ2UNCOMPRESS], [false]) ] )
AC_CHECK_LIB( z, gzopen, [BES_ZLIB_LIBS=-lz])

dnl dl lib?
AC_CHECK_FUNC(dlclose, , [ AC_CHECK_LIB(dl, dlopen, [BES_DL_LIBS=-ldl]) ])
LIBS=$BES_OLD_LIBS

AC_SUBST(BES_DL_LIBS)
AC_SUBST(BES_ZLIB_LIBS)
AC_SUBST(BES_BZ2_LIBS)

## FIXME This should probably be moved to a 'find the 3rd party stuff' section
## of this file. jhrg 11/17/14
GDAL_FOUND=

AC_ARG_WITH(gdal, AS_HELP_STRING([--with-gdal], [Use the copy of GDAL at this location]),
            with_gdal_prefix="$withval", with_gdal_prefix="")

if test -n "$with_gdal_prefix" -a -x $with_gdal_prefix/bin/gdal-config
then
    AC_MSG_NOTICE([Using $with_gdal_prefix as the GDAL prefix directory.])
    GDAL_LDFLAGS="`$with_gdal_prefix/bin/gdal-config --libs`"
    GDAL_CFLAGS="`$with_gdal_prefix/bin/gdal-config --cflags`"
    GDAL_FOUND="yes"
elif test -n "$with_gdal_prefix"
then
    AC_MSG_ERROR([You set the gdal-prefix directory to $with_gdal_prefix, but gdal-config is not there.])
fi

if test -z "$GDAL_FOUND"
then
    AX_LIB_GDAL([1.10.0])
    if test ! -z "$GDAL_CFLAGS" -a ! -z "$GDAL_LDFLAGS"; then
        GDAL_FOUND="yes"
    fi
fi

AM_CONDITIONAL([GDAL_FOUND], [test x$GDAL_FOUND = xyes])

if test -z "$GDAL_FOUND"
then
    AC_MSG_WARN([I could not find GDAL. GDAL dependent code will not be built.])
else 
    AC_MSG_NOTICE([GDAL_CFLAGS:  $GDAL_CFLAGS])
    AC_MSG_NOTICE([GDAL_LDFLAGS: $GDAL_LDFLAGS])
fi

dnl library visioning: Update these when the interface changes. Generally,
dnl assume that the interface tracks the major and minor release numbers.
LIB_DIS_CURRENT=13
LIB_DIS_AGE=5
LIB_DIS_REVISION=0
AC_SUBST(LIB_DIS_CURRENT)
AC_SUBST(LIB_DIS_AGE)
AC_SUBST(LIB_DIS_REVISION)
LIBDISPATCH_VERSION="$LIB_DIS_CURRENT:$LIB_DIS_REVISION:$LIB_DIS_AGE"
AC_SUBST(LIBDISPATCH_VERSION)

LIB_PPT_CURRENT=4
LIB_PPT_AGE=0
LIB_PPT_REVISION=5
AC_SUBST(LIB_PPT_CURRENT)
AC_SUBST(LIB_PPT_AGE)
AC_SUBST(LIB_PPT_REVISION)
LIBPPT_VERSION="$LIB_PPT_CURRENT:$LIB_PPT_REVISION:$LIB_PPT_AGE"
AC_SUBST(LIBPPT_VERSION)

LIB_XML_CMD_CURRENT=2
LIB_XML_CMD_AGE=1
LIB_XML_CMD_REVISION=4
AC_SUBST(LIB_XML_CMD_CURRENT)
AC_SUBST(LIB_XML_CMD_AGE)
AC_SUBST(LIB_XML_CMD_REVISION)
LIBXMLCOMMAND_VERSION="$LIB_XML_CMD_CURRENT:$LIB_XML_CMD_REVISION:$LIB_XML_CMD_AGE"
AC_SUBST(LIBXMLCOMMAND_VERSION)

AC_DEFINE([DAPREADER_PACKAGE], ["dapreader_module"], [dapreader_module])
AC_DEFINE([DAPREADER_VERSION], ["0.0.1"], [0.0.1])

# check for readline
VL_LIB_READLINE
if test "$vl_cv_lib_readline" = "no"; then
   AC_MSG_ERROR([I could not find the readline library!])
fi

SIC_VAR_SYS_ERRLIST

BES_CHECK_OPENSSL

dnl extra argument: --with-libwrap
AC_MSG_CHECKING(whether to use libwrap)
AC_ARG_WITH(libwrap,
    [  --with-libwrap          Compile in libwrap (tcp_wrappers) support.],
    [ case "$withval" in
    yes) AC_MSG_RESULT(yes)
	AC_CHECK_LIB(wrap, request_init, 
	     [ LIBS="-lwrap $LIBS"
	       AC_DEFINE([HAVE_LIBWRAP], [1], [Description])],
	     [ AC_MSG_ERROR(Could not find libwrap library. You have to insall tcp-wrappers before using --with-libwrap option to configure.) ])
	;;
    *) AC_MSG_RESULT(no);;
    esac ],
    AC_MSG_RESULT(no)
)

## End of things for the BES

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

## libdap

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

## bes

AC_CONFIG_TESTDIR(bes/cmdln/testsuite, [.])
AC_CONFIG_TESTDIR(bes/dapreader/bes-testsuite, [.])
AC_CONFIG_TESTDIR(bes/functions/tests, [.])
dnl AC_CONFIG_TESTDIR(bes/dap/bes-testsuite, [.])

AC_CONFIG_FILES([bes/Makefile
		 bes/abi_checker.xml
		 bes/dispatch/Makefile
		 bes/dispatch/unit-tests/Makefile
		 bes/docs/Makefile
		 bes/xmlcommand/Makefile
		 bes/xmlcommand/unit-tests/Makefile
		 bes/ppt/Makefile
		 bes/ppt/unit-tests/Makefile
		 bes/cmdln/Makefile
		 bes/cmdln/unit-tests/Makefile
		 bes/cmdln/testsuite/Makefile
		 bes/cmdln/testsuite/atlocal
		 bes/standalone/Makefile
		 bes/server/Makefile
		 bes/server/test/Makefile
		 bes/bin/Makefile
		 bes/templates/Makefile
		 bes/bes_dispatch.pc
		 bes/bes_ppt.pc
		 bes/bes_xml_command.pc

		 bes/dapreader/Makefile
		 bes/dapreader/bes-testsuite/Makefile
		 bes/dapreader/bes-testsuite/atlocal

		 bes/dap/Makefile
		 bes/dap/unit-tests/Makefile
		 bes/dap/unit-tests/test_config.h

		 bes/functions/Makefile
		 bes/functions/tests/Makefile
		 bes/functions/tests/atlocal
 		 bes/functions/tests/package.m4
		 bes/functions/unit-tests/Makefile])

## FIXME 11/17/14
if test -n "$GDAL_FOUND"
then
    AC_CONFIG_FILES([bes/functions/swath2grid/Makefile])
fi

AC_CONFIG_FILES([bes/cmdln/testsuite/generate_data_baseline.sh], 
		[chmod +x bes/cmdln/testsuite/generate_data_baseline.sh])
AC_CONFIG_FILES([bes/cmdln/testsuite/generate_metadata_baseline.sh],
                [chmod +x bes/cmdln/testsuite/generate_metadata_baseline.sh])

AC_CONFIG_FILES([bes/functions/tests/generate_data_baseline.sh],
                [chmod +x bes/functions/tests/generate_data_baseline.sh])
AC_CONFIG_FILES([bes/functions/tests/generate_metadata_baseline.sh],
                [chmod +x bes/functions/tests/generate_metadata_baseline.sh])

AC_CONFIG_FILES([bes/bes-config],
                [chmod +x bes/bes-config]) 
AC_CONFIG_FILES([bes/besd],
                [chmod +x bes/besd]) 
AC_CONFIG_FILES([bes/server/besctl],
                [chmod +x bes/server/besctl]) 
AC_CONFIG_FILES([bes/server/hyraxctl],
                [chmod +x bes/server/hyraxctl]) 


AC_OUTPUT
