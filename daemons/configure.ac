AC_PREREQ(2.59)
m4_include(../version.m4)
AC_INIT([ipa-server],
        IPA_VERSION,
        [https://hosted.fedoraproject.org/projects/freeipa/newticket])

AC_CONFIG_HEADERS([config.h])
AC_CONFIG_SUBDIRS([../asn1])

AM_INIT_AUTOMAKE([foreign])
m4_ifdef([AM_SILENT_RULES], [AM_SILENT_RULES])

AM_MAINTAINER_MODE
AC_PROG_CC_C99
AC_DISABLE_STATIC
LT_INIT

AC_HEADER_STDC

AM_CONDITIONAL([HAVE_GCC], [test "$ac_cv_prog_gcc" = yes])

AC_SUBST(VERSION)
AC_SUBST([INSTALL_DATA], ['$(INSTALL) -m 644 -p'])

dnl ---------------------------------------------------------------------------
dnl - Check for NSPR/NSS
dnl ---------------------------------------------------------------------------
PKG_CHECK_MODULES([NSPR], [nspr], [], [AC_MSG_ERROR([libnspr not found])])
PKG_CHECK_MODULES([NSS], [nss], [], [AC_MSG_ERROR([libnss not found])])

dnl ---------------------------------------------------------------------------
dnl - Check for DS slapi plugin
dnl ---------------------------------------------------------------------------

# Need to hack CPPFLAGS to be able to correctly detetct slapi-plugin.h
SAVE_CPPFLAGS=$CPPFLAGS
CPPFLAGS=$NSPR_CFLAGS
AC_CHECK_HEADER(dirsrv/slapi-plugin.h)
if test "x$ac_cv_header_dirsrv_slapi-plugin_h" = "xno" ; then
	AC_MSG_ERROR([Required 389-ds header not available (389-ds-base-devel)])
fi
AC_CHECK_HEADER(dirsrv/repl-session-plugin.h)
if test "x$ac_cv_header_dirsrv_repl_session_plugin_h" = "xno" ; then
	AC_MSG_ERROR([Required 389-ds header not available (389-ds-base-devel)])
fi
CPPFLAGS=$SAVE_CPPFLAGS

if test "x$ac_cv_header_dirsrv_slapi_plugin_h" = "xno" ; then
	AC_MSG_ERROR([Required DS slapi plugin header not available (fedora-ds-base-devel)])
fi

dnl ---------------------------------------------------------------------------
dnl - Check for KRB5
dnl ---------------------------------------------------------------------------

PKG_CHECK_MODULES([KRB5], [krb5], [], [AC_MSG_ERROR([libkrb5 not found])])
AC_CHECK_HEADER(krad.h, [], [AC_MSG_ERROR([krad.h not found])])
AC_CHECK_LIB(krad, main, [], [AC_MSG_ERROR([libkrad not found])])
KRAD_LIBS="-lkrad"
krb5rundir="${localstatedir}/run/krb5kdc"
AC_SUBST(KRAD_LIBS)
AC_SUBST(krb5rundir)

dnl ---------------------------------------------------------------------------
dnl - Check for OpenLDAP SDK
dnl ---------------------------------------------------------------------------

SAVE_CPPFLAGS=$CPPFLAGS
CPPFLAGS="$NSPR_CFLAGS $NSS_CFLAGS"
AC_CHECK_LIB([ldap_r], [ldap_search], [], AC_MSG_ERROR([libldap_r not found]))
AC_CHECK_LIB([lber], [ber_peek_tag], [], AC_MSG_ERROR([liblber not found]))
LDAP_LIBS="-lldap_r -llber"
LDAP_CFLAGS=""
AC_SUBST(LDAP_LIBS)
AC_SUBST(LDAP_CFLAGS)

dnl ---------------------------------------------------------------------------
dnl - Check for OpenSSL Crypto library
dnl ---------------------------------------------------------------------------
PKG_CHECK_MODULES([CRYPTO], [libcrypto], [], [AC_MSG_ERROR([libcrypto not found])])

dnl ---------------------------------------------------------------------------
dnl - Check for UUID library
dnl ---------------------------------------------------------------------------
PKG_CHECK_MODULES([UUID], [uuid], [], [AC_MSG_ERROR([libuuid not found])])

dnl ---------------------------------------------------------------------------
dnl - Check for Python
dnl ---------------------------------------------------------------------------

AC_MSG_NOTICE([Checking for Python])
have_python=no
AM_PATH_PYTHON(2.7)

if test "x$PYTHON" = "x" ; then
  AC_MSG_ERROR([Python not found])
fi

dnl ---------------------------------------------------------------------------
dnl Check for ndr_krb5pac and other samba libraries
dnl ---------------------------------------------------------------------------

PKG_PROG_PKG_CONFIG()
PKG_CHECK_MODULES([TALLOC], [talloc])
PKG_CHECK_MODULES([TEVENT], [tevent])
PKG_CHECK_MODULES([NDRPAC], [ndr_krb5pac])
PKG_CHECK_MODULES([NDRNBT], [ndr_nbt])
PKG_CHECK_MODULES([NDR], [ndr])
PKG_CHECK_MODULES([SAMBAUTIL], [samba-util])
SAMBA40EXTRA_LIBPATH="-L`$PKG_CONFIG --variable=libdir samba-util`/samba -Wl,-rpath=`$PKG_CONFIG --variable=libdir samba-util`/samba"
AC_SUBST(SAMBA40EXTRA_LIBPATH)

bck_cflags="$CFLAGS"
CFLAGS="$NDRPAC_CFLAGS"
AC_CHECK_MEMBER(
    [struct PAC_DOMAIN_GROUP_MEMBERSHIP.domain_sid],
    [AC_DEFINE([HAVE_STRUCT_PAC_DOMAIN_GROUP_MEMBERSHIP], [1],
               [struct PAC_DOMAIN_GROUP_MEMBERSHIP is available.])],
    [AC_MSG_NOTICE([struct PAC_DOMAIN_GROUP_MEMBERSHIP is not available])],
                 [[#include <ndr.h>
                   #include <gen_ndr/krb5pac.h>]])

CFLAGS="$bck_cflags"

LIBPDB_NAME=""
AC_CHECK_LIB([samba-passdb],
             [make_pdb_method],
             [LIBPDB_NAME="samba-passdb"; HAVE_LIBPDB=1],
             [LIBPDB_NAME="pdb"],
             [$SAMBA40EXTRA_LIBPATH])

if test "x$LIB_PDB_NAME" = "xpdb" ; then
  AC_CHECK_LIB([$LIBPDB_NAME],
               [make_pdb_method],
               [HAVE_LIBPDB=1],
               [AC_MSG_ERROR([Neither libpdb nor libsamba-passdb does have make_pdb_method])],
               [$SAMBA40EXTRA_LIBPATH])
fi

AC_SUBST(LIBPDB_NAME)

AC_CHECK_LIB([$LIBPDB_NAME],[pdb_enum_upn_suffixes],
             [AC_DEFINE([HAVE_PDB_ENUM_UPN_SUFFIXES], [1], [Ability to enumerate UPN suffixes])],
             [AC_MSG_WARN([libpdb does not have pdb_enum_upn_suffixes, no support for realm domains in ipasam])],
             [$SAMBA40EXTRA_LIBPATH])

dnl ---------------------------------------------------------------------------
dnl Check for libunistring
dnl ---------------------------------------------------------------------------
AC_CHECK_HEADERS([unicase.h],,AC_MSG_ERROR([Could not find unicase.h]))
AC_CHECK_LIB([unistring],
             [ulc_casecmp],
             [UNISTRING_LIBS="-lunistring"],
             [AC_MSG_ERROR([libunistring does not have ulc_casecmp])])
AC_SUBST(UNISTRING_LIBS)

dnl ---------------------------------------------------------------------------
dnl Check for libverto
dnl ---------------------------------------------------------------------------
PKG_CHECK_MODULES([LIBVERTO], [libverto])

dnl ---------------------------------------------------------------------------
dnl - Check for cmocka unit test framework http://cmocka.cryptomilk.org/
dnl ---------------------------------------------------------------------------
PKG_CHECK_EXISTS(cmocka,
    [AC_CHECK_HEADERS([stdarg.h stddef.h setjmp.h],
        [], dnl We are only intrested in action-if-not-found
        [AC_MSG_WARN([Header files stdarg.h stddef.h setjmp.h are required by cmocka])
         cmocka_required_headers="no"
        ]
    )
    AS_IF([test x"$cmocka_required_headers" != x"no"],
          [PKG_CHECK_MODULES([CMOCKA], [cmocka], [have_cmocka="yes"])]
    )],
    dnl PKG_CHECK_EXISTS ACTION-IF-NOT-FOUND
    [AC_MSG_WARN([No libcmocka library found, cmocka tests will not be built])]
)
AM_CONDITIONAL([HAVE_CMOCKA], [test x$have_cmocka = xyes])

dnl A macro to check presence of a cwrap (http://cwrap.org) wrapper on the system
dnl Usage:
dnl     AM_CHECK_WRAPPER(name, conditional)
dnl If the cwrap library is found, sets the HAVE_$name conditional
AC_DEFUN([AM_CHECK_WRAPPER],
[
    FOUND_WRAPPER=0

    AC_MSG_CHECKING([for $1])
    PKG_CHECK_EXISTS([$1],
                     [
                        AC_MSG_RESULT([yes])
                        FOUND_WRAPPER=1
                     ],
                     [
                        AC_MSG_RESULT([no])
                        AC_MSG_WARN([cwrap library $1 not found, some tests will not run])
                     ])

    AM_CONDITIONAL($2, [ test x$FOUND_WRAPPER = x1])
])

AM_CHECK_WRAPPER(nss_wrapper, HAVE_NSS_WRAPPER)

dnl -- dirsrv is needed for the extdom unit tests --
PKG_CHECK_MODULES([DIRSRV], [dirsrv  >= 1.3.0])
dnl -- sss_idmap is needed by the extdom exop --
PKG_CHECK_MODULES([SSSIDMAP], [sss_idmap])
PKG_CHECK_MODULES([SSSNSSIDMAP], [sss_nss_idmap >= 1.13.90])

dnl ---------------------------------------------------------------------------
dnl - Check for systemd unit directory
dnl ---------------------------------------------------------------------------
PKG_CHECK_EXISTS([systemd], [], [AC_MSG_ERROR([systemd not found])])
AC_ARG_WITH([systemdsystemunitdir],
            AS_HELP_STRING([--with-systemdsystemunitdir=DIR], [Directory for systemd service files]),
            [], [with_systemdsystemunitdir=$($PKG_CONFIG --variable=systemdsystemunitdir systemd)])
AC_SUBST([systemdsystemunitdir], [$with_systemdsystemunitdir])

dnl ---------------------------------------------------------------------------
dnl - Check for program paths
dnl ---------------------------------------------------------------------------
AC_PATH_PROG(UNLINK, unlink, [AC_MSG_ERROR([unlink not found])])

dnl ---------------------------------------------------------------------------
dnl - Set the data install directory since we don't use pkgdatadir
dnl ---------------------------------------------------------------------------

IPA_DATA_DIR="$datadir/ipa"
AC_SUBST(IPA_DATA_DIR)

dnl ---------------------------------------------------------------------------
dnl Finish
dnl ---------------------------------------------------------------------------

# Turn on the additional warnings last, so -Werror doesn't affect other tests.

AC_ARG_ENABLE(more-warnings,
	[AC_HELP_STRING([--enable-more-warnings],
	[Maximum compiler warnings])],
	set_more_warnings="$enableval",[
        	if test -d $srcdir/../.hg; then
        		set_more_warnings=yes
              	else
                  	set_more_warnings=no
              	fi
        ])
AC_MSG_CHECKING(for more warnings)
if test "$GCC" = "yes" -a "$set_more_warnings" != "no"; then
        AC_MSG_RESULT(yes)
        CFLAGS="\
        -Wall \
        -Wchar-subscripts -Wmissing-declarations -Wmissing-prototypes \
        -Wnested-externs -Wpointer-arith \
        -Wcast-align -Wsign-compare \
        $CFLAGS"

        for option in -Wno-strict-aliasing -Wno-sign-compare; do
                SAVE_CFLAGS="$CFLAGS"
                CFLAGS="$CFLAGS $option"
                AC_MSG_CHECKING([whether gcc understands $option])
                AC_TRY_COMPILE([], [],
                        has_option=yes,
                        has_option=no,)
                if test $has_option = no; then
                        CFLAGS="$SAVE_CFLAGS"
                fi
                AC_MSG_RESULT($has_option)
                unset has_option
                unset SAVE_CFLAGS
        done
        unset option
else
        AC_MSG_RESULT(no)
fi

# Flags

AC_SUBST(CFLAGS)
AC_SUBST(CPPFLAGS)
AC_SUBST(LDFLAGS)

# Files

AC_CONFIG_FILES([
    Makefile
    ../asn1/Makefile
    ipa-kdb/Makefile
    ipa-sam/Makefile
    ipa-otpd/Makefile
    ipa-slapi-plugins/Makefile
    ipa-slapi-plugins/libotp/Makefile
    ipa-slapi-plugins/ipa-cldap/Makefile
    ipa-slapi-plugins/ipa-dns/Makefile
    ipa-slapi-plugins/ipa-enrollment/Makefile
    ipa-slapi-plugins/ipa-lockout/Makefile
    ipa-slapi-plugins/ipa-otp-counter/Makefile
    ipa-slapi-plugins/ipa-otp-lasttoken/Makefile
    ipa-slapi-plugins/ipa-pwd-extop/Makefile
    ipa-slapi-plugins/ipa-extdom-extop/Makefile
    ipa-slapi-plugins/ipa-winsync/Makefile
    ipa-slapi-plugins/ipa-version/Makefile
    ipa-slapi-plugins/ipa-uuid/Makefile
    ipa-slapi-plugins/ipa-modrdn/Makefile
    ipa-slapi-plugins/ipa-sidgen/Makefile
    ipa-slapi-plugins/ipa-range-check/Makefile
    ipa-slapi-plugins/topology/Makefile
])

AC_OUTPUT

echo "
                    IPA Server $VERSION
                    ========================

        prefix:                   ${prefix}
        exec_prefix:              ${exec_prefix}
        libdir:                   ${libdir}
        bindir:                   ${bindir}
        sbindir:                  ${sbindir}
        sysconfdir:               ${sysconfdir}
        localstatedir:            ${localstatedir}
        datadir:                  ${datadir}
        krb5rundir:               ${krb5rundir}
        systemdsystemunitdir:     ${systemdsystemunitdir}
        source code location:     ${srcdir}
        compiler:                 ${CC}
        cflags:                   ${CFLAGS}
        LDAP libs:                ${LDAP_LIBS}
        KRB5 libs:                ${KRB5_LIBS}
        KRAD libs:                ${KRAD_LIBS}
        OpenSSL crypto libs:      ${CRYPTO_LIBS}
        Maintainer mode:          ${USE_MAINTAINER_MODE}
"
