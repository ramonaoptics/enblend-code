AC_PREREQ([2.69])
AC_INIT([enblend-enfuse],
        [m4_esyscmd([tr -d '\n' < VERSION])],
        [https://bugs.launchpad.net/enblend],
        [],
        [m4_esyscmd([sed -e 's,^\([^-]*\)-.*$,http://sourceforge.net/projects/enblend/files/enblend-enfuse/enblend-enfuse-\1/,' < VERSION | tr -d '\n'])])
AC_CONFIG_SRCDIR([src/enblend.cc])
AC_CONFIG_AUX_DIR([.])
AC_CONFIG_MACRO_DIR(m4)
AC_CONFIG_HEADER([config.h])

AM_INIT_AUTOMAKE([foreign no-installinfo])
AM_SILENT_RULES()

AC_PROG_CXX
AX_CXX_COMPILE_STDCXX_17(ext)
AC_PROG_CC
AC_PROG_AR
AM_PROG_AR
AC_PROG_EGREP
AC_PROG_GREP
AC_PROG_RANLIB
AC_PROG_SED
AC_LANG(C++)

AC_C_BIGENDIAN

enable_on_demand_dynamic_linking=check
AC_MSG_CHECKING([if on-demand dynamic linking is desired])
AC_ARG_ENABLE([on-demand-dynlink],
              [AS_HELP_STRING([--enable-on-demand-dynlink],
                              [use on-demand dynamic linking @<:@default=check@:>@])],
              [AC_MSG_RESULT($enableval);
               enable_on_demand_dynamic_linking=$enableval],
              [AC_MSG_RESULT(check);
               enable_on_demand_dynamic_linking=check])

have_on_demand_dynamic_linking=no
AS_IF([test "$enable_on_demand_dynamic_linking" = no],
      [have_on_demand_dynamic_linking=no],
      [if test -n "$enable_on_demand_dynamic_linking" || \
          test "$enable_on_demand_dynamic_linking" = yes || \
          test "$enable_on_demand_dynamic_linking" = check;
       then
          AC_MSG_CHECKING([for on-demand dynamic linking])
          save_ldflags="$LDFLAGS"
          LDFLAGS="-Wl,--as-needed -Wl,--no-copy-dt-needed-entries ${LDFLAGS}"
          AC_LINK_IFELSE([AC_LANG_PROGRAM([[#include <cstring>]],
                                          [[std::strlen("foo")]])],
                         [AC_MSG_RESULT(yes);
                          have_on_demand_dynamic_linking=yes],
                         AC_MSG_RESULT(no))
          LDFLAGS="$save_ldflags"
       fi])
if test "$have_on_demand_dynamic_linking" = yes; then
  LDFLAGS="-Wl,--as-needed -Wl,--no-copy-dt-needed-entries ${LDFLAGS}"
fi


# Checks for libraries.

STDCXX_FILESYSTEM_LIBRARY='-lstdc++fs'
AC_MSG_CHECKING([whether we must link with an extra library to get C++17 filesystem support])
AC_LINK_IFELSE([AC_LANG_PROGRAM([[#include <filesystem>]],
                                [[std::filesystem::path p;
                                  p.parent_path()]])],
               [AC_MSG_RESULT(no)],
               [AC_MSG_RESULT(yes)
                LIBS="$LIBS $STDCXX_FILESYSTEM_LIBRARY"])

# We check for TCMalloc so early to make it the last user library on
# the linker command line.  It will then gobble all undefined
# references to malloc(3) and free(3) and thus relieve the O/S even
# more.
use_tcmalloc=no
AC_ARG_WITH([tcmalloc],
            [AS_HELP_STRING([--with-tcmalloc], [use Google's TCMalloc library @<:@default=check@:>@])],
            [],
            [with_tcmalloc=check])
AS_IF([test "$with_tcmalloc" = check -o "$with_tcmalloc" = yes],
      [AC_CHECK_HEADERS([gperftools/tcmalloc.h tcmalloc.h], [break])
       if test "$ac_cv_header_tcmalloc_h" = yes -o "$with_tcmalloc" = yes; then
           AC_SEARCH_LIBS(tc_malloc,
                          [tcmalloc, tcmalloc_minimal],
                          [use_tcmalloc=yes
                           AC_DEFINE(HAVE_TCMALLOC, 1, [Define if you have Google's TCMalloc library])])
       fi])

AC_CHECK_LIB([m], [sqrt])

AC_CHECK_LIB(z, gzopen, [], AC_MSG_NOTICE([Compiling without libz.]), [])
AC_CHECK_LIB(jpeg, jpeg_finish_compress,
             [LIBS="-ljpeg ${LIBS}"; AC_DEFINE(HasJPEG, 1, [Define if you have the jpeg library])],
             AC_MSG_NOTICE([Compiling without support for jpeg files.]), [])
AC_CHECK_LIB(png, png_init_io,
             [LIBS="-lpng ${LIBS}"; AC_DEFINE(HasPNG, 1, [Define if you have the png library])],
             AC_MSG_NOTICE([Compiling without support for png files.]), [])
AC_CHECK_LIB(tiff, TIFFOpen,
             [LIBS="-ltiff ${LIBS}"; AC_DEFINE(HasTIFF, 1, [Define if you have the tiff library])],
             AC_MSG_ERROR([libtiff is required to compile Enblend.]), [])

AC_MSG_CHECKING([if OpenEXR is wanted])
AC_ARG_WITH([openexr],
            [AS_HELP_STRING([--with-openexr],
                            [use OpenEXR @<:@default=check@:>@])],
            [],
            [with_openexr=check])
AS_IF([test "$with_openexr" = no],
      [AC_MSG_NOTICE([disabling OpenEXR])
       have_exr=no],
      [if test "$with_openexr" = yes || test "$with_openexr" = check; then
          PKG_CHECK_MODULES(OPENEXR, OpenEXR >= 1.0,
                            [AC_DEFINE(HasEXR, 1, [Define if you have EXR library])
                             have_exr=yes],
                            [AC_MSG_WARN("OpenEXR support disabled: " $OPENEXR_PKG_ERRORS)
                             have_exr=no])
          LIBS="${OPENEXR_LIBS} $LIBS"
          CFLAGS="${OPENEXR_CFLAGS} $CFLAGS"
          CXXFLAGS="${OPENEXR_CFLAGS} $CXXFLAGS"
       else
          have_exr=no
       fi])

AC_ARG_VAR([VIGRAIMPEX_LIB], [name of Vigra import/export-library])
if test -z "$VIGRAIMPEX_LIB"; then
   VIGRAIMPEX_LIB='-lvigraimpex'
fi

AC_MSG_CHECKING([for Vigra import/export-library])
LIBS="$VIGRAIMPEX_LIB $LIBS"
AC_LINK_IFELSE([AC_LANG_PROGRAM([[#include <vigra/imageinfo.hxx>]],
                                [[vigra::impexListFormats()]])],
               AC_MSG_RESULT(yes),
               [AC_MSG_RESULT(no)
                AC_MSG_ERROR([libvigraimpex is required to compile Enblend.])])
AC_MSG_CHECKING([whether Vigra import/export-library is sufficiently recent])
AC_LINK_IFELSE([AC_LANG_PROGRAM([[#include <vigra/imageinfo.hxx>
                                  #include <vigra/impexalpha.hxx>]],
                                [[vigra::BRGBImage image;
                                  vigra::BImage alpha;
                                  vigra::ImageImportInfo info("image.tif");
                                  info.setImageIndex(99);
                                  vigra::importImageAlpha(info, destImage(image), destImage(alpha));]])],
               AC_MSG_RESULT(yes),
               [AC_MSG_RESULT(no)
                AC_MSG_ERROR([Vigra was found, but it was not recent enough.])])

use_exiv2=no
AC_ARG_WITH([exiv2],
            [AS_HELP_STRING([--with-exiv2], [use Exiv2 library @<:@default=check@:>@])],
            [],
            [with_exiv2=check])
AS_IF([test "$with_exiv2" = check -o "$with_exiv2" = yes],
      [AC_MSG_CHECKING([for Exiv2 library])
       SAVED_LIBS=$LIBS
       LIBS="$LIBS -lexiv2"
       AC_LINK_IFELSE([AC_LANG_PROGRAM([[#include <exiv2/image.hpp>]],
                                       [[Exiv2::Image* image {nullptr};
                                         image->iptcData();]])],
                      [AC_MSG_RESULT(yes)
                       use_exiv2=yes
                       AC_DEFINE(HAVE_EXIV2, 1, [Define if you have the Exiv2 library])],
                      [AC_MSG_RESULT(no)
                       LIBS="$SAVED_LIBS"])])
if test "$use_exiv2" = yes; then
  AC_MSG_CHECKING([whether Exiv2 library is sufficiently recent])
  AC_LINK_IFELSE([AC_LANG_PROGRAM([[#include <exiv2/image.hpp>]],
                                  [[Exiv2::Image::UniquePtr p {nullptr}]])],
                 AC_MSG_RESULT(yes),
                 [AC_MSG_RESULT(no)
                  use_exiv2=no
                  LIBS="$SAVED_LIBS"
                  if test "$with_exiv2" = check; then
                    AC_MSG_WARN([Exiv2 was found, but it was not recent enough.])
                  else
                    AC_MSG_ERROR([Exiv2 was found, but it was not recent enough.])
                  fi])
fi

AC_MSG_CHECKING([for support of partially static linking])
original_LIBS="${LIBS}"
LIBS="-Wl,-Bstatic -lm -Wl,-Bdynamic ${LIBS}"
AC_LINK_IFELSE([AC_LANG_PROGRAM([[#include <math.h>]],
                                [[sin(0.5)]])],
               [AC_MSG_RESULT(yes);
                partially_static_linking_support=yes],
               [AC_MSG_RESULT(no);
                partially_static_linking_support=no])
LIBS="${original_LIBS}"

STATIC_LIBS=""
partially_static_linking=no
AC_ARG_ENABLE([partially-static-linking],
              AC_HELP_STRING([--enable-partially-static-linking],
                             [static link of performance critical libraries @<:@default=no@:>@]),
              [partially_static_linking=$enableval],
              [partially_static_linking=no])

if test "$partially_static_linking_support" = yes && test "$partially_static_linking" = yes; then
  dnl https://www.gnu.org/software/gsl/manual/html_node/Linking-with-an-alternative-BLAS-library.html
  AC_CHECK_LIB(atlas, ATL_cputime, [], [])
  AC_CHECK_LIB(cblas, cblas_dgemm, [have_cblas=true], [have_cblas=false], [-latlas])
  if $have_cblas; then
    STATIC_LIBS="-lcblas -latlas ${STATIC_LIBS}"
    AC_CHECK_LIB(gsl, gsl_blas_dgemm, [STATIC_LIBS="-lgsl ${STATIC_LIBS}"], [], [-lcblas -latlas])
  else
    AC_CHECK_LIB(gslcblas, cblas_dgemm, [STATIC_LIBS="-lgslcblas ${STATIC_LIBS}"])
    AC_CHECK_LIB(gsl, gsl_blas_dgemm, [STATIC_LIBS="-lgsl ${STATIC_LIBS}"], [], [-lgslcblas])
  fi

  AC_CHECK_LIB(lcms2, cmsCreateTransform, [STATIC_LIBS="-llcms2 ${STATIC_LIBS}"])
  AC_SUBST(STATIC_LIBS, ["-Wl,-Bstatic ${STATIC_LIBS} -Wl,-Bdynamic"])
else
  AC_CHECK_LIB(gslcblas, cblas_dgemm)
  AC_CHECK_LIB(gsl, gsl_blas_dgemm)
  AC_CHECK_LIB(lcms2, cmsCreateTransform)
  AC_SUBST(STATIC_LIBS)
fi


# Memory allocation debug support
AC_MSG_CHECKING([if malloc debugging is wanted])
AC_ARG_WITH(dmalloc,
            [  --with-dmalloc          use dmalloc, as in
                          http://www.dmalloc.com/dmalloc.tar.gz],
            [if test "$withval" = yes; then
                AC_MSG_RESULT(yes)
                AC_DEFINE(WITH_DMALLOC, 1,
                    [Define if using the dmalloc debugging malloc package])
                if test $acx_pthread_ok = yes; then
                    LIBS="$LIBS -ldmallocthcxx"
                    enable_dmalloc="yes (thread aware)"
                else
                    LIBS="$LIBS -ldmalloccxx"
                    enable_dmalloc=yes
                fi
                LDFLAGS="$LDFLAGS -g"
                AC_DEFINE(DMALLOC, 1,
                          [Define to enable malloc debugger library])
                AC_DEFINE(DMALLOC_FUNC_CHECK, 1,
                          [Define to enable malloc debugger function checking])
            else
                AC_MSG_RESULT(no)
                enable_dmalloc=no
            fi],
            [AC_MSG_RESULT(no)
             enable_dmalloc=no])

DL_IMPLEMENTATION=''
dynload_implementation=''
enable_dynload=no
AC_ARG_WITH([dl],
            [AS_HELP_STRING([--with-dl], [add support for dynamic loading @<:@default=check@:>@])],
            [],
            [with_dl=check])
AS_IF([test "$with_dl" != no],
      [if test "$with_dl" = check -o "$with_dl" = yes -o "$with_dl" = dl; then
           AC_CHECK_HEADERS(dlfcn.h)
           if test "$ac_cv_header_dlfcn_h" = yes; then
               AC_SEARCH_LIBS(dlopen,
                              [dl],
                              [AC_DEFINE(HAVE_DL, 1, [Define if you have support for dynamic loading])
                               AC_DEFINE(POSIX_DL, 1, [Use POSIX dl implementation])
                               DL_IMPLEMENTATION='libdynamic_loader_a-posix_implementation.o'
                               dynload_implementation='(via dl)'
                               enable_dynload=yes])
           fi
       fi
       if test "$enable_dynload" != yes -a \( "$with_dl" = check -o "$with_dl" = gmodule \); then
           AC_CHECK_HEADERS(gmodule.h)
           if test "$ac_cv_header_gmodule_h" = yes; then
               AC_SEARCH_LIBS(g_module_open,
                              [gmodule-2.0],
                              [AC_DEFINE(HAVE_DL, 1, [Define if you have support for dynamic loading])
                               AC_DEFINE(GMODULE_DL, 1, [Use gmodule-2.0 dl implementation])
                               DL_IMPLEMENTATION='libdynamic_loader_a-gmodule_implementation.o'
                               dynload_implementation='(via gmodule-2.0)'
                               enable_dynload=yes])
           fi
       fi])
AC_SUBST(DL_IMPLEMENTATION)
AM_CONDITIONAL([DYNAMIC_EXPOSURE_WEIGHTS], [test "$enable_dynload" = yes])

# Checks for header files.
AC_HEADER_DIRENT
AC_HEADER_STDC

AC_CHECK_HEADERS([fenv.h limits.h stdlib.h string.h unistd.h])

AC_CHECK_HEADER(sys/times.h,
                [AC_DEFINE([HAVE_SYS_TIMES_H], [1], [Define if <sys/times.h> exists.])])
AC_CHECK_HEADER(tiffio.h, [],
                AC_MSG_ERROR([libtiff-devel header files are required to compile Enblend.]))
AC_CHECK_HEADER(jpeglib.h, [],
                AC_MSG_ERROR([libjpeg-devel header files are required to compile Enblend.]))
AC_CHECK_HEADER(png.h, [],
                AC_MSG_ERROR([libpng-devel header files are required to compile Enblend.]))

AC_LANG_PUSH([C++])

AC_CHECK_HEADER(vigra/basicimage.hxx, [],
                AC_MSG_ERROR([Vigra "basicimage.hxx" header file is required to compile Enblend.]))

AC_CHECK_HEADER(gsl/gsl_errno.h, [],
                AC_MSG_ERROR([GNU Scientific Library (GSL) header file "gsl_errno" is required to compile Enblend.]))
AC_CHECK_HEADER(gsl/gsl_min.h, [],
                AC_MSG_ERROR([GNU Scientific Library (GSL) header file "gsl_min" is required to compile Enblend.]))
AC_CHECK_HEADER(gsl/gsl_multimin.h, [],
                AC_MSG_ERROR([GNU Scientific Library (GSL) header file "gsl_multimin" is required to compile Enblend.]))
AC_CHECK_HEADER(gsl/gsl_rng.h, [],
                AC_MSG_ERROR([GNU Scientific Library (GSL) header file "gsl_rng" is required to compile Enblend.]))
AC_CHECK_HEADER(gsl/gsl_vector.h, [],
                AC_MSG_ERROR([GNU Scientific Library (GSL) header file "gsl_vector" is required to compile Enblend.]))

AC_LANG_POP([])
AC_SUBST(EXTRA_LIBS)

AC_CHECK_HEADER(lcms2.h, [],
                AC_MSG_ERROR([lcms2 header files are required to compile Enblend.]))

# Checks for typedefs, structures, and compiler characteristics.
AC_CHECK_HEADER_STDBOOL
AC_C_CONST
AC_C_INLINE
if test "$ac_cv_c_inline" != no; then
   AC_DEFINE(HAVE_INLINE, 1, [Define if functions can be declared `inline'.])
   AC_SUBST(HAVE_INLINE)
fi
AC_C_RESTRICT
AC_DEFINE([RESTRICT], [restrict], [Map RESTRICT onto C99 restrict keyword])

AC_TYPE_OFF_T

AC_TYPE_SIG_ATOMIC_T
AC_TYPE_SIZE_T

AC_CHECK_TYPES([ptrdiff_t])

# Checks for library functions.
AC_FUNC_CLOSEDIR_VOID
AC_FUNC_ERROR_AT_LINE
AC_FUNC_FSEEKO
AC_FUNC_SETVBUF_REVERSED
AC_FUNC_STRERROR_R
AC_FUNC_STRTOD

AC_CHECK_FUNCS([atexit clock_gettime \
                fesetround floor \
                memset mkstemp select \
                pow sqrt \
                sqrt strchr strcspn strdup strerror strpbrk strrchr strtok_r strtol strtoul])

# lrint and lrintf
AC_C99_FUNC_LRINT
AC_C99_FUNC_LRINTF

AC_PATH_PROG(PERL, perl, false)
if test "$PERL" = false; then
    AC_MSG_ERROR(cannot find perl)
fi
AX_PROG_PERL_MODULES(Sys::Hostname, [],
                     AC_MSG_ERROR(missing Perl module Sys::Hostname))
AX_PROG_PERL_MODULES(Time::Zone, [],
                     AC_MSG_WARN(missing Perl module Time::Zone))

# Documentation
if test "$cross_compiling" = no; then
    AM_MISSING_PROG(HELP2MAN, help2man)
else
    HELP2MAN=:
fi


can_build_doc=yes

AC_PATH_PROGS(LATEX, [latex elatex lambda])

if test -n "$LATEX"
then
  AC_LATEX_CLASS(report,report,[],
                 [AC_MSG_WARN(missing document class report.cls)
                  can_build_doc=no
                  missing_for_doc="$missing_for_doc report.cls"])
  AC_LATEX_CLASS(refrep,refrep,[],
                 [AC_MSG_WARN([missing document class refrep.cls -- no Reference Manual Style available])
                  missing_for_doc="$missing_for_doc refrep.cls"])

dnl  FIXME: We ought to add package `hypcap' to the list, but
dnl  `AC_LATEX_PACKAGE' does not allow for prerequisite pqckages to be
dnl  mentioned while testing and `hypcap' needs `hyperref' to be
dnl  included first.
  m4_foreach([package_name],
             [amsmath, bold-extra, color, enumitem,
              float, footnote, graphicx, hyperref, hyphenat,
              ifpdf, index, latexsym, listings,
              microtype, ragged2e, shorttoc, suffix,
              url, xstring],
             [AC_LATEX_PACKAGE([package_name],
                               [report],
                               [m4_translit(package_name,[-],[_])],
                               [],
                               [AC_MSG_WARN(missing LaTeX package package_name)
                                can_build_doc=no
                                missing_for_doc="$missing_for_doc package_name.sty"])])
else
  AC_MSG_WARN(missing LaTeX translator)
  can_build_doc=no
  missing_for_doc="$missing_for_doc latex"
fi

if test "$PERL" != false; then
  AX_PROG_PERL_MODULES(File::Basename, [],
                       [AC_MSG_WARN(missing Perl module File::Basename)
                        can_build_doc=no
                        missing_for_doc="$missing_for_doc File::Basename"])
  AX_PROG_PERL_MODULES(FindBin, [],
                       [AC_MSG_WARN(missing Perl module FindBin)
                        can_build_doc=no
                        missing_for_doc="$missing_for_doc FindBin"])
  AX_PROG_PERL_MODULES(Getopt::Long, [],
                       [AC_MSG_WARN(missing Perl module Getopt::Long)
                        can_build_doc=no
                        missing_for_doc="$missing_for_doc Getopt::Long"])
  AX_PROG_PERL_MODULES(IO::File, [],
                       [AC_MSG_WARN(missing Perl module IO::File)
                        can_build_doc=no
                        missing_for_doc="$missing_for_doc IO::File"])
  AX_PROG_PERL_MODULES(IO::Handle, [],
                       [AC_MSG_WARN(missing Perl module IO::Handle)
                        can_build_doc=no
                        missing_for_doc="$missing_for_doc IO::Handle"])
  AX_PROG_PERL_MODULES(Readonly, [],
                       [AC_MSG_WARN(missing Perl module Readonly)
                        can_build_doc=no
                        missing_for_doc="$missing_for_doc Readonly"])
fi


AC_PATH_PROG(PDFLATEX, pdflatex)
if test -z "$PDFLATEX"; then
     AC_MSG_WARN([missing PDFLaTeX translator -- no direct translation of LaTeX to PDF available])
     missing_for_doc="$missing_for_doc pdflatex"
fi


AC_PATH_PROG(TEXLOGANALYSER, texloganalyser, cat)
if test "$TEXLOGANALYSER" = cat; then
    AC_MSG_WARN([cannot find texloganalyser; will substitute cat(1)])
    missing_for_doc="$missing_for_doc texloganalyser"
fi


AC_PATH_PROG(MAKEINDEX, makeindex, false)
if test "$MAKEINDEX" = false; then
    AC_MSG_WARN(cannot find makeindex; will not be able to build documentation with index)
    can_build_doc=no
    missing_for_doc="$missing_for_doc makeindex"
fi


AC_PATH_PROG(DVIPS, dvips, false)
if test "$DVIPS" = false; then
    AC_MSG_WARN(cannot find dvips; will not be able to build PostScript documentation)
    can_build_doc=no
    missing_for_doc="$missing_for_doc dvips"
fi

AC_PATH_PROG(CONVERT, convert, false)
if test "$CONVERT" = false; then
    AC_MSG_WARN(cannot find convert; will not be able to build documentation)
    can_build_doc=no
    missing_for_doc="$missing_for_doc convert"
fi

AX_WITH_PROG(LATEX2HTML, hevea, false, [])
if test "$LATEX2HTML" = false; then
    AC_MSG_WARN(cannot find LaTeX to HTML translator; will not be able to build HTML documentation)
    can_build_doc=no
    missing_for_doc="$missing_for_doc hevea"
else
    AC_MSG_CHECKING([$LATEX2HTML library directory])
    AC_ARG_VAR(LATEX2HTML_LIBDIR,
               [Directory where LaTeX to HTML translator's additional files reside])
    LATEX2HTML_LIBDIR=$($LATEX2HTML -version | sed -n 's/^library directory: \(.*\)/\1/p')
    if test -z "$LATEX2HTML_LIBDIR"; then
        AC_MSG_RESULT(none)
    elif test -d "$LATEX2HTML_LIBDIR"; then
        AC_MSG_RESULT($LATEX2HTML_LIBDIR)
    else
        AC_MSG_RESULT(missing)
    fi
fi

AC_PATH_PROG(SPLITHTML, hacha, false)
if test "$SPLITHTML" = false; then
    AC_MSG_WARN(cannot find hacha; will not be able to split HTML documentation on request)
fi

AC_PATH_PROG(GNUPLOT, gnuplot, false)
if test "$GNUPLOT" = false; then
    AC_MSG_WARN(cannot find gnuplot; will not be able to build documentation)
    can_build_doc=no
    missing_for_doc="$missing_for_doc gnuplot"
fi

AC_PATH_PROG(M4, m4, false)
if test "$M4" = false; then
    AC_MSG_WARN(cannot find m4; will not be able to build documentation)
    can_build_doc=no
    missing_for_doc="$missing_for_doc m4"
fi

AC_PATH_PROG(DOT, dot, false)
if test "$DOT" = false; then
    AC_MSG_WARN(cannot find dot; will not be able to build documentation)
    can_build_doc=no
    missing_for_doc="$missing_for_doc dot"
fi

AC_PATH_PROG(SVGCONVERT, rsvg-convert, false)
if test "$SVGCONVERT" = false; then
    AC_MSG_WARN(cannot find rsvg-convert; will not be able to build documentation)
    can_build_doc=no
    missing_for_doc="$missing_for_doc rsvg-convert"
fi

AC_PATH_PROGS(PS2PDF, [ps2pdf ps2pdf14], false)
if test "$PDFLATEX" = false -a "$PS2PDF" != false; then
  AC_MSG_NOTICE([Alternative PDF generation via ps2pdf available.])
fi

AC_PATH_PROGS(DVIPDF, [dvipdfmx dvipdfm dvipdf], false)
if test "$PDFLATEX" = false -a "$DVIPDF" != false; then
  AC_MSG_NOTICE([Alternative PDF generation via dvipdf available.])
fi


AM_CONDITIONAL([BUILD_DOC], [test "$can_build_doc" = yes])
if test "$can_build_doc" = no; then
    no_doc_reason=", because of missing$missing_for_doc"
fi


AC_ARG_VAR(DYNAMIC_TEX_PREAMBLE, [LaTeX commands to add to top of preamble])
DYNAMIC_TEX_PREAMBLE=\\def\\finishdynamicpreamble{\\overfullrule=2pt\\relax}


AC_MSG_CHECKING(whether to enable debugging)
debug_default="no"
AC_ARG_ENABLE(debug,
              AS_HELP_STRING([--enable-debug],
                             [turn on debugging @<:@default=no@:>@]),
              [enable_debug=$enableval],
              [enable_debug=$debug_default])
if test "$enable_debug" = yes; then
    CXXFLAGS="$CXXFLAGS -g -DDEBUG"
    AC_MSG_RESULT(yes)
    enable_debug=yes
else
    CXXFLAGS="$CXXFLAGS -O2 -DNDEBUG"
    AC_MSG_RESULT(no)
    enable_debug=no
fi

AC_MSG_CHECKING(whether to compile with OpenMP)
openmp_default="no"
AC_ARG_ENABLE(openmp,
              AS_HELP_STRING([--enable-openmp],
                             [compile with OpenMP @<:@default=no@:>@]),
              [enable_openmp=$enableval],
              [enable_openmp=$openmp_default])
if test "$enable_openmp" = yes; then
    AC_MSG_RESULT(yes)
    AX_OPENMP([enable_openmp=yes])
    CFLAGS="$CFLAGS $OPENMP_CFLAGS"
    CXXFLAGS="$CXXFLAGS $OPENMP_CXXFLAGS"
fi
if test -z "$OPENMP_CXXFLAGS"; then
    AC_MSG_RESULT(no)
    enable_openmp=no
else
    enable_openmp=yes
fi

built_in_opencl_path=/usr/local/share/enblend/kernels:/usr/share/enblend/kernels
AC_ARG_WITH([opencl-path],
            AS_HELP_STRING([--with-opencl-path=<PATH>],
                           [set to-be-built-in OpenCL search path]),
            [opencl_path="$withval"],
            [opencl_path="$built_in_opencl_path"])
AC_DEFINE_UNQUOTED(DEFAULT_OPENCL_PATH, ["$opencl_path"], [Default search path for OpenCL files])
AC_DEFINE(PREFER_SEPARATE_OPENCL_SOURCE, 1,
          [Define if you want to access OpenCL files, not compile-in their string equivalents])


AX_OPENCL([C++])
CFLAGS="$CFLAGS $CL_CFLAGS"
CXXFLAGS="$CXXFLAGS $CL_CFLAGS"
LIBS="$LIBS $CL_LIBS"

AC_CONFIG_FILES([doc/Makefile
                 doc/examples/Makefile
                 doc/examples/enfuse/Makefile
                 doc/examples/enfuse/Makefile.userweight])

AC_CONFIG_FILES([Makefile
                 src/Makefile
                 src/dynamic_loader/Makefile
                 src/layer_selection/Makefile
                 src/win32helpers/Makefile])
AC_OUTPUT

AC_MSG_RESULT([
 enblend-enfuse now configured for ${host}
   Source directory:               ${srcdir}
   Installation directory:         ${prefix}
   C++ compiler:                   ${CXX}
   CFLAGS:                         ${CFLAGS:-<none required>}
   CXXFLAGS:                       ${CXXFLAGS:-<none required>}
   LDFLAGS:                        ${LDFLAGS:-<none required>}
   LIBS:                           ${LIBS:-<none required>}
   STATIC_LIBS:                    ${STATIC_LIBS:-<none defined>}
   EXTRA_LIBS (optional):          ${EXTRA_LIBS:-<none selected>}

 can build all documentation:      ${can_build_doc}${no_doc_reason}

 feature selection:
   enable debugging support:       ${enable_debug}
   enable malloc debugging:        ${enable_dmalloc}
   enable dynamic loading          ${enable_dynload} ${dynload_implementation}
   OpenEXR image format            ${have_exr}
   use OpenMP:                     ${enable_openmp}
   use OpenCL:                     ${enable_opencl} (search path: $opencl_path)
   use Exiv2:                      ${use_exiv2}
   use TCMalloc:                   ${use_tcmalloc}
${warnings}])
