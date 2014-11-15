# Build libdap, libtest-types.a, getdap

AUTOMAKE_OPTIONS = foreign check-news
ACLOCAL_AMFLAGS = -I conf -I libdap/gl/m4

AM_CPPFLAGS = 
AM_CXXFLAGS = 

if COMPILER_IS_GCC
AM_CXXFLAGS += -Wall -W -Wcast-align
endif

# These are not used by automake but are often useful for certain types of
# debugging. The best way to use these is to run configure as:
#     export CXXFLAGS='...'; ./configure --disable-shared
# or ./configure --enable-developer --disable-shared
# the --disable-shared is not required, but it seems to help with debuggers.
CXXFLAGS_DEBUG = -g3 -O0  -Wall -W -Wcast-align 
# -Werror There are some warnings in this code that are here to stay...
TEST_COV_FLAGS = -ftest-coverage -fprofile-arcs

if BUILD_DEVELOPER
AM_CXXFLAGS += $(CXXFLAGS_DEBUG)
endif

SUBDIRS = libdap
DIST_SUBDIRS = 

aclocaldir=$(datadir)/aclocal
dist_aclocal_DATA = conf/libdap.m4

