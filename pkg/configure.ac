#!/usr/bin/env autoreconf
dnl Process this file with autoconf to produce a configure script.
define([pkgversion], esyscmd([sh -c "grep Version: DESCRIPTION | cut -d' ' -f2 | tr -d '\n'"]))dnl
AC_INIT(rgdal, [pkgversion], Roger.Bivand@nhh.no)
AC_CONFIG_SRCDIR(src/gdal-bindings.cpp)
AC_CONFIG_MACRO_DIRS([inst/m4])
AC_PROG_CXX()

# find R home and set correct compiler + flags
: ${R_HOME=`R RHOME`}
if test -z "${R_HOME}"; then
  AC_MSG_ERROR([cannot determine R_HOME. Make sure you use R CMD INSTALL!])
fi
AC_MSG_NOTICE([R_HOME: ${R_HOME}])
RBIN="${R_HOME}/bin/R"

# pick all flags for testing from R
: ${CC=`"${RBIN}" CMD config CC`}
: ${CXX=`"${RBIN}" CMD config CXX`}
: ${CPP=`"${RBIN}" CMD config CPP`}
: ${CFLAGS=`"${RBIN}" CMD config CFLAGS`}
: ${CPPFLAGS=`"${RBIN}" CMD config CPPFLAGS`}
: ${LDFLAGS=`"${RBIN}" CMD config LDFLAGS`}
# AC_SUBST([CC],["clang"])
# AC_SUBST([CXX],["clang++"])
AC_MSG_NOTICE([CC: ${CC}])
AC_MSG_NOTICE([CXX: ${CXX}])

#CXX_11

R_VERSION=`echo 'cat(unlist(R.Version()$minor))' | ${RBIN} --vanilla --slave`
R_MINOR=`echo $R_VERSION | cut -f1 -d"."`

#if test ${R_MINOR} -ge 4; then
#: ${CXX=`"${RBIN}" CMD config CXX11`}
#else
#: ${CXX=`"${RBIN}" CMD config CXX1X`}
#fi

if test ${R_MINOR} -ge 4; then
 CXX11=`"${RBIN}" CMD config CXX11`
 CXX11STD=`"${RBIN}" CMD config CXX11STD`
else
 CXX11=`"${RBIN}" CMD config CXX1X`
 CXX11STD=`"${RBIN}" CMD config CXX1XSTD`
fi

CXX="${CXX11} ${CXX11STD}"

if test -n "${CXX11}"; then
HAVE_CXX11=1
else
HAVE_CXX11=0
fi

#HAVE_CXX11=1

#m4_include([inst/m4/ax_cxx_compile_stdcxx.m4])
#AX_CXX_COMPILE_STDCXX([11], [ext], [optional])
if test [${HAVE_CXX11} = 1] ; then
  AC_MSG_NOTICE([C++11 support available])
else
  AC_MSG_NOTICE([C++11 support not available])
fi


AC_MSG_NOTICE([${PACKAGE_NAME}: ${PACKAGE_VERSION}])

#
# Use subversion version to report
#
AC_PATH_PROG([SVN_VERSION], [svnversion])
AS_IF([ test -n "${SVN_VERSION}" ], [
dnl extract version from checkout
  SVN_VERSION=`${SVN_VERSION} -n '.'`
  AS_IF([ test "${SVN_VERSION}" != "exported" &&
          test "${SVN_VERSION}" != "Unversioned directory" ], [
dnl write version to inst/SVN_VERSION
            echo "${SVN_VERSION}" > inst/SVN_VERSION
        ], [
dnl read version from inst/SVN_VERSION
            SVN_VERSION=`cat inst/SVN_VERSION | tr -d '\n'`
        ])
  ], [
dnl read version from inst/SVN_VERSION
    SVN_VERSION=`cat inst/SVN_VERSION | tr -d '\n'`
  ])
AC_MSG_NOTICE([svn revision: ${SVN_VERSION}])

#CXX_11
AX_CXX_COMPILE_STDCXX([11], [ext], [optional])
AS_IF([ test "x${HAVE_CXX11}" = "x1" ],
  [AC_MSG_NOTICE([C++11 support available])],
  [AC_MSG_NOTICE([C++11 support not available])]
)

#
# check packages exist with pkg-config
#
PKG_CHECK_EXISTS([gdal proj],
  [AC_MSG_NOTICE([gdal exists, continuing...])],
  [AC_MSG_ERROR([Failed to find gdal _and_ proj4 consider installing, setting PKG_CONFIG_PATH])]
  )

#
# GDAL - details with pkg-config and gdal-config
#
PKG_CHECK_MODULES([GDAL], [gdal >= 1.11.4], [
dnl copy CFLAGS to CPPFLAGS
  AC_SUBST([GDAL_CPPFLAGS], ["${GDAL_CFLAGS}"])
dnl get version of GDAL
  PKG_CHECK_VAR([GDAL_VERSION], [gdal], [CONFIG_VERSION],
    [AC_MSG_NOTICE([VERSION = "${GDAL_VERSION}"])],
    [AC_MSG_ERROR([gdal.pc missing version])])

dnl get data directory - may be incorrect https://github.com/OSGeo/gdal/issues/608
  PKG_CHECK_VAR([GDAL_DATADIR], [gdal], [datadir],
    [AC_MSG_NOTICE([DATADIR = "${GDAL_DATADIR}"])],
    [AC_MSG_ERROR([gdal.pc missing datadir])])

dnl get gdal prefix
  PKG_CHECK_VAR([GDAL_PREFIX], [gdal], [prefix],
    [AC_MSG_NOTICE([PREFIX  = "${GDAL_PREFIX}"])],
    [AC_MSG_ERROR([gdal.pc missing prefix])])

dnl check version and CXX11 availability
  AS_IF([ test `${PKG_CONFIG} gdal --atleast-version=2.3.0` ],[
    AC_MSG_CHECKING([C++11 support for GDAL >= 2.3.0])
    AS_IF([ test "x${HAVE_CXX11}" = "x1" ],
      [AC_MSG_RESULT([yes])],
      [
        AC_MSG_RESULT([no])
        AC_MSG_ERROR([provide at least CXX11 compiler support for GDAL >= 2.3.0])
      ]
    )
  ])
  ], [AC_MSG_ERROR([upgrade GDAL to 1.11.4 or later])])

#
# Need to use gdal-config - often more accurate
#
AC_ARG_WITH([gdal-config],
    AC_HELP_STRING([--with-gdal-config=GDAL_CONFIG],
           [the location of gdal-config]),
           [gdal_config=$withval])
AS_IF([ test  -n "$gdal_config" ], [
    AC_SUBST([GDAL_CONFIG],["${gdal_config}"])
    AC_MSG_NOTICE(gdal-config set to $GDAL_CONFIG)
], [
  AC_PATH_PROG([GDAL_CONFIG], [gdal-config],[
    AC_MSG_ERROR([gdal-config not found in PATH])
  ])
])

dnl catch user error specifying incorrect path in --with-gdal-config=
AC_MSG_CHECKING(gdal-config usability)
AS_IF([test `${GDAL_CONFIG} --version`], [
  AC_MSG_RESULT(yes)
  GDAL_DEP_LIBS=`${GDAL_CONFIG} --dep-libs`
  GDAL_DATADIR=`${GDAL_CONFIG} --datadir`
  ], [
  AC_MSG_RESULT(no)
  AC_MSG_ERROR([gdal-config not found
    The gdal-config script distributed with GDAL could not be found.
	  If you have not installed the GDAL libraries, you can
	  download the source from  http://www.gdal.org/
	  If you have installed the GDAL libraries, then make sure that
	  gdal-config is in your path. Try typing gdal-config at a
	  shell prompt and see if it runs. If not, use:
	    --configure-args='--with-gdal-config=/usr/local/bin/gdal-config'
	  with appropriate values for your installation.
  ])
])

INLIBS="${LIBS}"
INCPPFLAGS="${CPPFLAGS}"
INPKG_CPPFLAGS="${PKG_CPPFLAGS}"
INPKG_LIBS="${PKG_LIBS}"

AC_SUBST([PKG_CPPFLAGS], ["${INPKG_CPPFLAGS} ${GDAL_CPPFLAGS}"])
AC_SUBST([PKG_LIBS], ["${INPKG_LIBS} ${GDAL_LIBS}"])

# honor PKG_xx overrides
# for CPPFLAGS we will superfluously double R's flags
# since we'll set PKG_CPPFLAGS with this, but that shouldn't hurt
CPPFLAGS="${INCPPFLAGS} ${PKG_CPPFLAGS}"

AC_LANG_PUSH([C++])
AC_MSG_NOTICE(gdal: linking with --libs only)
AC_CHECK_LIB([gdal], [GDALAllRegister],
  [NEED_DEPS="no"],[NEED_DEPS="yes"], [${GDAL_LIBS}])

AS_IF([test "x${NEED_DEPS}" = "xyes"], [
  AC_MSG_NOTICE(gdal: linking with --libs and --dep-libs)
  dnl clear cache otherwise the previous result is returned from cache
  dnl and this check lib is never run
  AS_UNSET([ac_cv_lib_gdal_GDALAllRegister])
  AC_CHECK_LIB([gdal], [GDALAllRegister],
    [NEED_DEPS="yes"],[
      AC_MSG_NOTICE([Install failure: compilation and/or linkage problems.])
      AC_MSG_ERROR([GDALAllRegister not found in libgdal.])
    ], [${GDAL_LIBS} ${GDAL_DEP_LIBS}])
  ])
AC_LANG_POP([C++])

GDAL_DATA_TEST_FILE="${GDAL_DATADIR}/pcs.csv"
AC_MSG_CHECKING(GDAL: ${GDAL_DATADIR}/pcs.csv readable)
AS_IF([ test -r "${GDAL_DATA_TEST_FILE}" ], [
    AC_MSG_RESULT(yes)
  ], [
    AC_MSG_RESULT(no)
    AC_MSG_ERROR([pcs.csv not found in GDAL data directory.])
  ])

# PROJ.4

# honor PKG_xx overrides
# for CPPFLAGS we will superfluously double R's flags
# since we'll set PKG_CPPFLAGS with this, but that shouldn't hurt
CPPFLAGS="${INCPPFLAGS} ${PKG_CPPFLAGS}"

AC_ARG_WITH([proj-include],
    AC_HELP_STRING([--with-proj-include=DIR],
                   [location of proj header files]),
    [proj_include_path=$withval])

AC_ARG_WITH([proj-lib],
    AC_HELP_STRING([--with-proj-lib=LIB_PATH],
               [the location of proj libraries]),
               [proj_lib_path=$withval])

AS_IF(
  [ test -z "$proj_lib_path" && test -z "$proj_include_path" ],
  [ dnl test -z "$proj_lib_path" || test -z "$proj_include_path" success
    AC_MSG_NOTICE([no paths in --configure-args])
    PKG_CHECK_MODULES([PROJ4], [proj >= 4.8.0],
      [ dnl get version of proj4
      PROJ4_VERSION=`${PKG_CONFIG} proj --modversion`
      AC_MSG_NOTICE([VERSION = "${PROJ4_VERSION}"])
      AC_MSG_NOTICE([CFLAGS = ${PROJ4_CFLAGS}])
      AC_MSG_NOTICE([LIBS = ${PROJ4_LIBS}])
      ],
      [ dnl failed to find proj.pc
      AC_MSG_NOTICE([PROJ is required])
      AC_MSG_ERROR([upgrade PROJ to 4.8.0 or later])
      ])
  ],
  [ dnl test -z "$proj_lib_path" || test -z "$proj_include_path" fail
    AC_MSG_NOTICE([checking for proj4 in user specified dirs])
    AS_IF([ test -n "$proj_include_path" ],
          [ dnl user specified a path
            AC_MSG_NOTICE([--with-proj-include=${proj_include_path}])
            proj_include_path="-I${proj_include_path}"
            AC_SUBST([PKG_CPPFLAGS],["${proj_include_path} ${INPKG_CPPFLAGS}"])
          ])
    AS_IF([ test -n "$proj_lib_path" ],
          [ dnl user specified a path to library shared object
            dnl - does not add -lproj
            AC_MSG_NOTICE([--with-proj-lib=${proj_lib_path}])
            proj_lib_path="-L${proj_lib_path}"
            AC_SUBST([PKG_LIBS],["${proj_include_path} ${INPKG_LIBS}"])
          ])
  ]
)

CPPFLAGS="$CPPFLAGS $PROJ4_CFLAGS"
AC_CHECK_HEADER([proj_api.h], [], [
  AC_MSG_ERROR([proj_api.h not found in standard or given locations.])
  ])
AC_RUN_IFELSE([AC_LANG_PROGRAM([dnl
#include <stdio.h>
#include <proj_api.h>], [dnl
printf("%d\n", PJ_VERSION);
return 0;
])], [PROJ4_VERSION_H=`./conftest$EXEEXT`], [
  AC_MSG_ERROR([failed to get version from proj_api.h])
  ])

AC_MSG_NOTICE([proj.4 versions ${PROJ4_VERSION} vs ${PROJ4_VERSION_H}])
AC_MSG_CHECKING([PROJ version agreement])
AS_IF([ test `echo ${PROJ4_VERSION} | tr -d "."` = ${PROJ4_VERSION_H}],
  [ AC_MSG_RESULT([yes]) ], [
    AC_MSG_RESULT([no])
    AC_MSG_ERROR([proj_api.h and pkg-config give different versions])
  ])

AS_IF([ test "${PROJ4_VERSION}" = "4.8.0" ], [
dnl unsure if $srcdir would be enough here rather than pwd
  ADDED_PKG_INCL_480=`pwd`"/inst/include"
  AC_SUBST([PKG_CPPFLAGS],["${PKG_CPPFLAGS} -I${ADDED_PKG_INCL_480}"])
  proj_include_path="${PKG_CPPFLAGS} -I${ADDED_PKG_INCL_480}"
  CPPFLAGS="${INCPPFLAGS} ${PKG_CPPFLAGS}"
])

AC_CHECK_HEADER([projects.h], [], [
  AC_MSG_ERROR([projects.h not found in standard or given locations.
    PROJ 4.8.0 was bundled wrongly, and did not provide
    the file. A copy is provided in init/include in the source
    package for the users to place in the same directory
    as proj_api.h manually.
    ])
  ])

#AC_MSG_NOTICE([${CFLAGS} ${PKG_CPPFLAGS}])

AC_SUBST([PKG_CPPFLAGS],["-DP4CTX=1 ${PKG_CPPFLAGS}"])

#
# Check location of proj metadata files
#
AC_ARG_WITH([proj-share],
    AC_HELP_STRING([--with-proj-share=SHARE_PATH],
               [the location of proj metadata files]),
               [proj_share_path=$withval])
AS_IF([ test -n "$proj_share_path" ],
  [AC_MSG_NOTICE([PROJ_LIB: ${proj_share_path}])])

AS_IF([ test -n "$proj_share_path" ],[
  AC_MSG_CHECKING(PROJ.4: epsg found and readable)
  LIBS="${PKG_LIBS} ${PROJ4_LIBS}"
  PROJ_LIB="$proj_share_path"

  AC_RUN_IFELSE([AC_LANG_PROGRAM([dnl
#include <stdio.h>
#include <proj_api.h>
#if PJ_VERSION == 480
FILE *pj_open_lib(projCtx, const char *, const char *);
#endif], [dnl
#if PJ_VERSION == 480
    FILE *fp;
#else
    PAFile fp;
#endif
    projCtx ctx;
    ctx = pj_get_default_ctx();
    fp = pj_open_lib(ctx, "epsg", "rb");
    if (fp == NULL) return 1;
#if PJ_VERSION == 480
    fclose(fp);
#else
    pj_ctx_fclose(ctx, fp);
#endif
    return 0;
])], [AC_MSG_RESULT([yes])], [
  AC_MSG_RESULT([no])
  AC_MSG_ERROR([proj/epsg not found
    Either install missing proj support files, for example
    the proj-nad and proj-epsg RPMs on systems using RPMs,
    or if installed but not autodetected, set PROJ_LIB to the
    correct path, and if need be use the --with-proj-share=
    configure argument.
    ])])
dnl end of AC_RUN_IFELSE

  AC_MSG_CHECKING(PROJ.4: conus found and readable)
  AC_RUN_IFELSE([AC_LANG_PROGRAM([dnl
#include <stdio.h>
#include <proj_api.h>
#if PJ_VERSION == 480
FILE *pj_open_lib(projCtx, const char *, const char *);
#endif], [dnl
#if PJ_VERSION == 480
    FILE *fp;
#else
    PAFile fp;
#endif
    projCtx ctx;
    ctx = pj_get_default_ctx();
    fp = pj_open_lib(ctx, "conus", "rb");
    if (fp == NULL) return 1;
#if PJ_VERSION == 480
    fclose(fp);
#else
    pj_ctx_fclose(ctx, fp);
#endif
    return 0;
])], [AC_MSG_RESULT([yes])], [
  AC_MSG_RESULT([no])
  AC_MSG_WARN([Note: proj/conus not found
    No support available in PROJ4 for NAD grid datum transformations
    If required, consider re-installing from source with the contents
    of proj-datumgrid-1.<latest>.zip from http://download.osgeo.org/proj/ in nad/.
    ])])
dnl end of AC_RUN_IFELSE

])

# Optional local copy of GDAL datadir and PROJ_LIB

AC_ARG_WITH([proj-data],
  AC_HELP_STRING([--with-proj-data=DIR],
                 [location of PROJ.4 data directory]),
                 [proj_lib1=$withval])
AC_ARG_WITH([data-copy],
    AC_HELP_STRING([--with-data-copy=yes/no],
               [local copy of data directories in package, default no]),
               [data_copy=$withval])

AS_IF([ test "x${data_copy}" = "xyes" ], [ dnl try to copy
  AC_MSG_NOTICE([Copy data for:])
  proj_lib0="${PROJ_LIB}"
  dnl determine which directory
  AS_IF([ test -n "${proj_lib0}" ],
    [proj_lib="${proj_lib0}"],
    [proj_lib="${proj_lib1}"])

  dnl test directory exists
  AS_IF([ test -n "${proj_lib}" && test -d "${proj_lib}" ],
    [ dnl run copy command
      cp -r "${proj_lib}" "${R_PACKAGE_DIR}"
      AC_MSG_NOTICE([  PROJ.4: ${proj_lib}])
    ],
    [ dnl directory not found
      AC_MSG_ERROR([PROJ.4 data files not found; set environment variable PROJ_LIB=DIR or --with-proj-data=DIR.])
    ])
  dnl test gdal data dir exists - tested previously
  AS_IF([ test -d "${GDAL_DATADIR}" ],
    [ dnl run copy command
      cp -r "${GDAL_DATADIR}" "${R_PACKAGE_DIR}"
      AC_MSG_NOTICE([  GDAL: ${GDAL_DATADIR}])
    ],
    [ dnl directory disappeared
      AC_MSG_ERROR([GDAL data files not found.])
    ])
])

#AC_MSG_ERROR([Halt])

# concluding substitution

AC_SUBST([PKG_CPPFLAGS], ["${INPKG_CPPFLAGS} ${GDAL_CPPFLAGS}"])
if test [ -n "$proj_include_path" ] ; then
AC_SUBST([PKG_CPPFLAGS], ["${PKG_CPPFLAGS} ${proj_include_path}"])
fi

AC_SUBST([PKG_LIBS], ["${INPKG_LIBS} ${GDAL_LIBS}"])
if test "${NEED_DEPS}" = yes; then
   AC_SUBST([PKG_LIBS], ["${PKG_LIBS} ${GDAL_DEP_LIBS}"])
fi
if test [ -n "$proj_lib_path" ] ; then
AC_SUBST([PKG_LIBS], ["${PKG_LIBS} -L${proj_lib_path}"])
fi
AC_SUBST([PKG_LIBS], ["${PROJ4_LIBS} -lproj"])

AC_MSG_NOTICE([Package CPP flags: ${PKG_CPPFLAGS}])
AC_MSG_NOTICE([Package LIBS: ${PKG_LIBS}])

AC_CONFIG_FILES(src/Makevars)
AC_OUTPUT
