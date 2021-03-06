#                                               -*- Autoconf -*-
# Process this file with autoconf to produce a configure script.

AC_PREREQ(2.61)
AC_INIT(libimobiledevice, 1.1.5, nospam@nowhere.com)
AM_INIT_AUTOMAKE([dist-bzip2 no-dist-gzip])
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES])
AC_CONFIG_SRCDIR([src/])
AC_CONFIG_HEADERS([config.h])
AC_CONFIG_MACRO_DIR([m4])

dnl libtool versioning
# +1 : 0 : +1  == adds new functions to the interface
# +1 : 0 : 0   == changes or removes functions (changes include both
#                 changes to the signature and the semantic)
#  ? :+1 : ?   == just internal changes
# CURRENT : REVISION : AGE
LIBIMOBILEDEVICE_SO_VERSION=3:1:0

AC_SUBST(LIBIMOBILEDEVICE_SO_VERSION)

# Checks for programs.
AC_PROG_CC
AC_PROG_CXX
AM_PROG_CC_C_O
AC_PROG_LIBTOOL

# Checks for libraries.
PKG_CHECK_MODULES(libusbmuxd, libusbmuxd >= 1.0.8)
PKG_CHECK_MODULES(libplist, libplist >= 1.8)
PKG_CHECK_MODULES(libplistmm, libplist++ >= 1.8)
AC_CHECK_LIB(pthread, [pthread_create, pthread_mutex_lock], [AC_SUBST(libpthread_LIBS,[-lpthread])], [AC_MSG_ERROR([libpthread is required to build libimobiledevice])])

# Checks for header files.
AC_HEADER_STDC
AC_CHECK_HEADERS([stdint.h stdlib.h string.h gcrypt.h])

# Checks for typedefs, structures, and compiler characteristics.
AC_C_CONST
AC_TYPE_SIZE_T
AC_TYPE_SSIZE_T
AC_TYPE_UINT16_T
AC_TYPE_UINT32_T
AC_TYPE_UINT8_T

# Checks for library functions.
AC_FUNC_MALLOC
AC_FUNC_REALLOC
AC_CHECK_FUNCS([strcasecmp strdup strerror strndup])

AC_CHECK_FUNC(asprintf, [have_asprintf="yes"], [have_asprintf="no"])
if test "x$have_asprintf" = "xyes"; then
  AC_DEFINE(HAVE_ASPRINTF,1,[define if asprintf is available])
fi
AC_CHECK_FUNC(vasprintf, [have_vasprintf="yes"], [have_vasprintf="no"])
if test "x$have_vasprintf" = "xyes"; then
  AC_DEFINE(HAVE_VASPRINTF,1,[define if vasprintf is available])
fi

AC_CHECK_HEADER(endian.h, [ac_cv_have_endian_h="yes"], [ac_cv_have_endian_h="no"])
if test "x$ac_cv_have_endian" = "xno"; then
  AC_DEFINE(__LITTLE_ENDIAN,1234,[little endian])
  AC_DEFINE(__BIG_ENDIAN,4321,[big endian])
  AC_C_BIGENDIAN([ac_cv_c_bigendian="yes"], [ac_cv_c_bigendian="no"], [], [])
  if test "x$ac_cv_c_bigendian" = "xyes"; then
    AC_DEFINE(__BYTE_ORDER,4321,[big endian byte order])
  else
    AC_DEFINE(__BYTE_ORDER,1234,[little endian byte order])
  fi
fi

# Cython Python Bindings
AC_ARG_WITH([cython],
            [AS_HELP_STRING([--without-cython],
            [build Python bindings using Cython (default is yes)])],
            [build_cython=false],
            [build_cython=true])
if test "$build_cython" = "true"; then
            AM_PATH_PYTHON(2.3)
            AC_PROG_CYTHON(0.13.0)
            CYTHON_PYTHON
else
            CYTHON=false
fi
if [test "x$CYTHON" != "xfalse"]; then
            AC_MSG_CHECKING([for libplist Cython bindings])
            CYTHON_PLIST_INCLUDE_DIR=$($PKG_CONFIG --variable=includedir libplist)/plist/cython
            if [test ! -d "$CYTHON_PLIST_INCLUDE_DIR"]; then
                CYTHON=false
                CYTHON_SUB=
                cython_python_bindings=no
                AC_MSG_RESULT([no])
                AC_MSG_WARN([cannot find libplist Cython bindings. You should install your distribution specific libplist Cython bindings package.])
            else
                AC_SUBST([CYTHON_PLIST_INCLUDE_DIR])
                AC_MSG_RESULT([$CYTHON_PLIST_INCLUDE_DIR])
                CYTHON_SUB=cython
                cython_python_bindings=yes
            fi
else
            CYTHON_SUB=
            cython_python_bindings=no
fi
AM_CONDITIONAL([HAVE_CYTHON],[test "x$CYTHON_SUB" = "xcython"])
AC_SUBST([CYTHON_SUB])

AC_ARG_ENABLE([openssl],
            [AS_HELP_STRING([--disable-openssl],
            [Do not look for OpenSSL])],
            [use_openssl=no],
            [use_openssl=yes])

pkg_req_openssl="openssl >= 0.9.8"
PKG_CHECK_MODULES(openssl, $pkg_req_openssl, have_openssl=yes, have_openssl=no)
if test "x$have_openssl" = "xyes"; then
  if test "x$use_openssl" != "xyes"; then
    enable_openssl=no
    echo "*** Note: OpenSSL support explicitly disabled ***"
  else
    enable_openssl=yes
  fi
else
  if test "x$use_openssl" == "xyes" -a "x$have_openssl" != "xyes"; then
    AC_MSG_ERROR([OpenSSL support explicitly requested but OpenSSL could not be found])
  fi
fi

if test "x$enable_openssl" = "xyes"; then
  AC_DEFINE(HAVE_OPENSSL, 1, [Define if you have OpenSSL support])
  AC_SUBST(openssl_CFLAGS)
  AC_SUBST(openssl_LIBS)
  ssl_provider="OpenSSL";
  ssl_requires="$pkg_req_openssl"
  AC_SUBST(ssl_requires)
else
  pkg_req_gnutls="gnutls >= 2.2.0"
  pkg_req_libtasn1="libtasn1 >= 1.1"
  PKG_CHECK_MODULES(libgnutls, $pkg_req_gnutls)
  AC_CHECK_LIB(gcrypt, gcry_control, [AC_SUBST(libgcrypt_LIBS,[-lgcrypt])], [AC_MSG_ERROR([libgcrypt is required to build libimobiledevice with GnuTLS])])
  PKG_CHECK_MODULES(libtasn1, $pkg_req_libtasn1)
  ssl_provider="GnuTLS"
  ssl_requires="$pkg_req_gnutls $pkg_req_libtasn1"
  AC_SUBST(ssl_requires)
fi

AC_ARG_ENABLE([dev-tools],
            [AS_HELP_STRING([--enable-dev-tools],
            [build development helper tools (default is no)])],
            [build_dev_tools=true],
            [build_dev_tools=false])
if test "$build_dev_tools" = true; then
            DEV_SUB=dev
            AC_CHECK_HEADERS([readline/readline.h],
			     [],
			     [AC_MSG_ERROR([Please install readline development headers])]
	    )
	    building_dev_tools=yes
else
            DEV_SUB=
	    building_dev_tools=no
fi

AM_CONDITIONAL([ENABLE_DEVTOOLS],[test "x$DEV_SUB" = "xdev"])
AC_SUBST([DEV_SUB])

AC_ARG_ENABLE([debug-code],
            [AS_HELP_STRING([--enable-debug-code],
            [enable debug message reporting in library (default is no)])],
            [no_debug_code=false],
            [no_debug_code=true])
if test "$no_debug_code" = true; then
	building_debug_code=no
	AC_DEFINE(STRIP_DEBUG_CODE,1,[Strip debug reporting code])
else
	building_debug_code=yes
fi

AS_COMPILER_FLAGS(GLOBAL_CFLAGS, "-Wall -Wextra -Wmissing-declarations -Wredundant-decls -Wshadow -Wpointer-arith  -Wwrite-strings -Wswitch-default -Wno-unused-parameter")
AC_SUBST(GLOBAL_CFLAGS)

# check for large file support
AC_SYS_LARGEFILE
LFS_CFLAGS=''
if test "$enable_largefile" != no; then
    if test "$ac_cv_sys_file_offset_bits" != 'no'; then
	LFS_CFLAGS="$LFS_CFLAGS -D_FILE_OFFSET_BITS=$ac_cv_sys_file_offset_bits"
    else
	AC_MSG_CHECKING(for native large file support)
	AC_RUN_IFELSE([AC_LANG_SOURCE([#include <unistd.h>
	  int main (int argc, char **argv)
	  {
	      exit(!(sizeof(off_t) == 8));
	  }])],
	[ac_cv_sys_file_offset_bits=64; AC_DEFINE(_FILE_OFFSET_BITS,64)
	 AC_MSG_RESULT(yes)],
	[AC_MSG_RESULT(no)])
    fi
    if test "$ac_cv_sys_large_files" != 'no'; then
	LFS_CFLAGS="$LFS_CFLAGS -D_LARGE_FILES=1"
    fi
    AC_FUNC_FSEEKO
    if test "$ac_cv_sys_largefile_source" != 'no'; then
	LFS_CFLAGS="$LFS_CFLAGS -D_LARGEFILE_SOURCE=1"
    fi
fi
AC_SUBST(LFS_CFLAGS)

m4_ifdef([AM_SILENT_RULES],[AM_SILENT_RULES([yes])])

AC_OUTPUT([
Makefile
src/Makefile
include/Makefile
dev/Makefile
tools/Makefile
cython/Makefile
docs/Makefile
libimobiledevice-1.0.pc
doxygen.cfg
])

echo "
Configuration for $PACKAGE $VERSION:
-------------------------------------------

  Install prefix: .........: $prefix
  Debug code ..............: $building_debug_code
  Dev tools ...............: $building_dev_tools
  Python bindings .........: $cython_python_bindings
  SSL support .............: $ssl_provider

  Now type 'make' to build $PACKAGE $VERSION,
  and then 'make install' for installation.
"
