dnl
dnl autoconf for Agents
dnl
dnl License: GNU General Public License (GPL)

dnl ===============================================
dnl Bootstrap 
dnl ===============================================
AC_PREREQ(2.63)

dnl Suggested structure:
dnl     information on the package
dnl     checks for programs
dnl     checks for libraries
dnl     checks for header files
dnl     checks for types
dnl     checks for structures
dnl     checks for compiler characteristics
dnl     checks for library functions
dnl     checks for system services

AC_INIT([resource-agents], 
	m4_esyscmd([make/git-version-gen .tarball-version]),
	[developers@clusterlabs.org])

AC_USE_SYSTEM_EXTENSIONS

CRM_DTD_VERSION="1.0"

AC_CONFIG_AUX_DIR(.)
AC_CONFIG_MACRO_DIR([m4])

AC_CANONICAL_HOST

dnl Where #defines go (e.g. `AC_CHECK_HEADERS' below)
dnl
dnl Internal header: include/config.h
dnl   - Contains ALL defines
dnl   - include/config.h.in is generated automatically by autoheader
dnl   - NOT to be included in any header files except lha_internal.h
dnl     (which is also not to be included in any other header files)
dnl
dnl External header: include/agent_config.h
dnl   - Contains a subset of defines checked here
dnl   - Manually edit include/agent_config.h.in to have configure include new defines
dnl   - Should not include HAVE_* defines
dnl   - Safe to include anywhere
AM_CONFIG_HEADER(include/config.h include/agent_config.h)
ALL_LINGUAS="en fr"

AC_ARG_WITH(version,
    [  --with-version=version   Override package version (if you're a packager needing to pretend) ],
    [ PACKAGE_VERSION="$withval" ])

AC_ARG_WITH(pkg-name,
    [  --with-pkg-name=name     Override package name (if you're a packager needing to pretend) ],
    [ PACKAGE_NAME="$withval" ])

dnl 
dnl AM_INIT_AUTOMAKE([1.11.1 foreign dist-bzip2 dist-xz])
dnl

AM_INIT_AUTOMAKE([1.10.1 foreign dist-bzip2])

AC_DEFINE_UNQUOTED(AGENTS_VERSION, "$PACKAGE_VERSION", Current agents version)

CC_IN_CONFIGURE=yes
export CC_IN_CONFIGURE

LDD=ldd

dnl ========================================================================
dnl Compiler characteristics
dnl ========================================================================

# check stolen from gnulib/m4/gnu-make.m4
if ! ${MAKE-make} --version /cannot/make/this >/dev/null 2>&1; then
	AC_MSG_ERROR([you don't seem to have GNU make; it is required])
fi

AC_PROG_CC dnl Can force other with environment variable "CC".
AM_PROG_CC_C_O
AC_PROG_CC_STDC
AC_PROG_CPP
AC_PROG_AWK
AC_PROG_LN_S
AC_PROG_INSTALL
AC_PROG_MAKE_SET

AC_C_STRINGIZE
AC_C_INLINE

AC_TYPE_SIZE_T
AC_TYPE_SSIZE_T
AC_TYPE_UID_T
AC_TYPE_UINT16_T
AC_TYPE_UINT8_T
AC_TYPE_UINT32_T

AC_CHECK_SIZEOF(char)
AC_CHECK_SIZEOF(short)
AC_CHECK_SIZEOF(int)
AC_CHECK_SIZEOF(long)
AC_CHECK_SIZEOF(long long)
AC_STRUCT_TIMEZONE

dnl ===============================================
dnl Helpers 
dnl ===============================================
cc_supports_flag() {
	local CPPFLAGS="$@"
	AC_MSG_CHECKING(whether $CC supports "$@")
	AC_PREPROC_IFELSE([AC_LANG_PROGRAM([])],
			  [RC=0; AC_MSG_RESULT([yes])],
			  [RC=1; AC_MSG_RESULT([no])])
	return $RC
}

extract_header_define() {
	  AC_MSG_CHECKING(for $2 in $1)
	  Cfile=$srcdir/extract_define.$2.${$}
	  printf "#include <stdio.h>\n" > ${Cfile}.c
	  printf "#include <%s>\n" $1 >> ${Cfile}.c
	  printf "int main(int argc, char **argv) { printf(\"%%s\", %s); return 0; }\n" $2 >> ${Cfile}.c
	  $CC $CFLAGS ${Cfile}.c -o ${Cfile}
	  value=`${Cfile}`
	  AC_MSG_RESULT($value)
	  printf $value
	  rm -f ${Cfile}.c ${Cfile}
	}

AC_MSG_NOTICE(Sanitizing prefix: ${prefix})
case $prefix in
  NONE)
	prefix=/usr
	dnl Fix default variables - "prefix" variable if not specified
	if test "$localstatedir" = "\${prefix}/var"; then
		localstatedir="/var"
	fi
	if test "$sysconfdir" = "\${prefix}/etc"; then
		sysconfdir="/etc"
	fi
	;;
esac

# ordering is important, PKG_PROG_PKG_CONFIG is to be invoked before any other PKG_* related stuff
PKG_PROG_PKG_CONFIG(0.18)

# PKG_CHECK_MODULES will fail if systemd is not found by default, so make sure
# we set the proper vars and deal with it
PKG_CHECK_MODULES([systemd], [systemd], [HAS_SYSTEMD=yes], [HAS_SYSTEMD=no])
if test "x$HAS_SYSTEMD" = "xyes"; then
	PKG_CHECK_VAR([SYSTEMD_UNIT_DIR], [systemd], [systemdsystemunitdir])
	if test "x$SYSTEMD_UNIT_DIR" = "x"; then
		AC_MSG_ERROR([Unable to detect systemd unit dir automatically])
	fi

	PKG_CHECK_VAR([SYSTEMD_TMPFILES_DIR], [systemd], [tmpfilesdir])
	if test "x$SYSTEMD_TMPFILES_DIR" = "x"; then
		AC_MSG_ERROR([Unable to detect systemd tmpfiles directory automatically])
	fi

	# sanitize systed vars when using non standard prefix
	if test "$prefix" != "/usr"; then
		SYSTEMD_UNIT_DIR="$prefix/$SYSTEMD_UNIT_DIR"
		AC_SUBST([SYSTEMD_UNIT_DIR])
		SYSTEMD_TMPFILES_DIR="$prefix/$SYSTEMD_TMPFILES_DIR"
		AC_SUBST([SYSTEMD_TMPFILES_DIR])
	fi

fi
AM_CONDITIONAL(HAVE_SYSTEMD, [test "x$HAS_SYSTEMD" = xyes ])


dnl ===============================================
dnl Configure Options
dnl ===============================================

dnl Some systems, like Solaris require a custom package name
AC_ARG_WITH(pkgname,
    [  --with-pkgname=name     name for pkg (typically for Solaris) ],
    [ PKGNAME="$withval" ],
    [ PKGNAME="LXHAhb" ],
  )
AC_SUBST(PKGNAME)

AC_ARG_ENABLE([ansi],
[  --enable-ansi force GCC to compile to ANSI/ANSI standard for older compilers.
     [default=no]])

AC_ARG_ENABLE([fatal-warnings],
[  --enable-fatal-warnings very pedantic and fatal warnings for gcc
     [default=yes]])

INITDIR=""
AC_ARG_WITH(initdir,
    [  --with-initdir=DIR      directory for init (rc) scripts [${INITDIR}]],
    [ INITDIR="$withval" ])

OCF_ROOT_DIR="${prefix}/lib/ocf"
AC_ARG_WITH(ocf-root,
    [  --with-ocf-root=DIR      directory for OCF scripts [${OCF_ROOT_DIR}]],
    [ OCF_ROOT_DIR="$withval" ])
HA_RSCTMPDIR=${localstatedir}/run/resource-agents
AC_ARG_WITH(rsctmpdir,
    [  --with-rsctmpdir=DIR      directory for resource agents state files [${HA_RSCTMPDIR}]],
    [ HA_RSCTMPDIR="$withval" ])

AC_ARG_ENABLE([libnet],
 [  --enable-libnet 	Use libnet for ARP based functionality, [default=try]], 
 [enable_libnet="$enableval"], [enable_libnet=try])

BUILD_RGMANAGER=0
BUILD_LINUX_HA=0

RASSET=linux-ha
AC_ARG_WITH(ras-set,
    [  --with-ras-set=SET     build/install only linux-ha, rgmanager or all resource-agents [default: linux-ha]],
    [ RASSET="$withval" ])

if test x$RASSET = xyes || test x$RASSET = xall ; then
	BUILD_RGMANAGER=1
	BUILD_LINUX_HA=1
fi

if test x$RASSET = xlinux-ha; then
	BUILD_LINUX_HA=1
fi

if test x$RASSET = xrgmanager; then
	BUILD_RGMANAGER=1
fi

if test $BUILD_LINUX_HA -eq 0 && test $BUILD_RGMANAGER -eq 0; then
	AC_MSG_ERROR([Are you really sure you want this package?])
	exit 1
fi

AM_CONDITIONAL(BUILD_LINUX_HA, test $BUILD_LINUX_HA -eq 1)
AM_CONDITIONAL(BUILD_RGMANAGER, test $BUILD_RGMANAGER -eq 1)

AC_ARG_WITH(compat-habindir,
    [  --with-compat-habindir      use HA_BIN directory with compatibility for the Heartbeat stack [${libexecdir}]],
    [],
    [with_compat_habindir=no])
AM_CONDITIONAL(WITH_COMPAT_HABINDIR, test "x$with_compat_habindir" != "xno")


dnl ===============================================
dnl General Processing
dnl ===============================================

echo Our Host OS: $host_os/$host

AC_MSG_NOTICE(Sanitizing exec_prefix: ${exec_prefix})
case $exec_prefix in
  dnl For consistency with Heartbeat, map NONE->$prefix
  NONE)	  exec_prefix=$prefix;;
  prefix) exec_prefix=$prefix;;
esac

AC_MSG_NOTICE(Sanitizing INITDIR: ${INITDIR})
case $INITDIR in
  prefix) INITDIR=$prefix;;
  "")
    AC_MSG_CHECKING(which init (rc) directory to use)
      for initdir in /etc/init.d /etc/rc.d/init.d /sbin/init.d	\
	   /usr/local/etc/rc.d /etc/rc.d
      do
        if
          test -d $initdir
        then
          INITDIR=$initdir
          break
        fi
      done
      if 
      	 test -z $INITDIR
      then
          INITDIR=${sysconfdir}/init.d
      fi
      AC_MSG_RESULT($INITDIR);;
esac
AC_SUBST(INITDIR)

if test "${prefix}" = "/usr"; then
 INITDIRPREFIX="$INITDIR" 
else
 INITDIRPREFIX="${prefix}/$INITDIR"
fi

AC_SUBST(INITDIRPREFIX)

AC_MSG_NOTICE(Sanitizing libdir: ${libdir})
case $libdir in
  dnl For consistency with Heartbeat, map NONE->$prefix
  *prefix*|NONE)
    AC_MSG_CHECKING(which lib directory to use)
    for aDir in lib64 lib
    do
      trydir="${exec_prefix}/${aDir}"
      if
        test -d ${trydir}
      then
        libdir=${trydir}
        break
      fi
    done
    AC_MSG_RESULT($libdir);
    ;;
esac

if test "x$with_compat_habindir" != "xno" ; then
  libexecdir=${libdir}
fi

dnl Expand autoconf variables so that we dont end up with '${prefix}' 
dnl in #defines and python scripts
dnl NOTE: Autoconf deliberately leaves them unexpanded to allow
dnl    make exec_prefix=/foo install
dnl No longer being able to do this seems like no great loss to me...

eval prefix="`eval echo ${prefix}`"
eval exec_prefix="`eval echo ${exec_prefix}`"
eval bindir="`eval echo ${bindir}`"
eval sbindir="`eval echo ${sbindir}`"
eval libexecdir="`eval echo ${libexecdir}`"
eval datadir="`eval echo ${datadir}`"
eval sysconfdir="`eval echo ${sysconfdir}`"
eval sharedstatedir="`eval echo ${sharedstatedir}`"
eval localstatedir="`eval echo ${localstatedir}`"
eval libdir="`eval echo ${libdir}`"
eval includedir="`eval echo ${includedir}`"
eval oldincludedir="`eval echo ${oldincludedir}`"
eval infodir="`eval echo ${infodir}`"
eval mandir="`eval echo ${mandir}`"

dnl docdir is a recent addition to autotools
eval docdir="`eval echo ${docdir}`"
if test "x$docdir" = "x"; then
   docdir="`eval echo ${datadir}/doc`"
fi
AC_SUBST(docdir)

dnl Home-grown variables
eval INITDIR="${INITDIR}"

for j in prefix exec_prefix bindir sbindir libexecdir datadir sysconfdir \
    sharedstatedir localstatedir libdir includedir oldincludedir infodir \
    mandir INITDIR docdir
do 
  dirname=`eval echo '${'${j}'}'`
  if
    test ! -d "$dirname"
  then
    AC_MSG_WARN([$j directory ($dirname) does not exist!])
  fi
done

dnl This OS-based decision-making is poor autotools practice;
dnl feature-based mechanisms are strongly preferred.
dnl
dnl So keep this section to a bare minimum; regard as a "necessary evil". 

REBOOT_OPTIONS="-f"
POWEROFF_OPTIONS="-f"

case "$host_os" in
*bsd*)		LIBS="-L/usr/local/lib"
		CPPFLAGS="$CPPFLAGS -I/usr/local/include"
		;;
*solaris*)
		REBOOT_OPTIONS="-n"
		POWEROFF_OPTIONS="-n"
		LDFLAGS+=" -lssp -lssp_nonshared"
		;;
*linux*)	
		AC_DEFINE_UNQUOTED(ON_LINUX, 1, Compiling for Linux platform)
		POWEROFF_OPTIONS="-nf"
		REBOOT_OPTIONS="-nf"
 		;;
darwin*)	
		AC_DEFINE_UNQUOTED(ON_DARWIN, 1, Compiling for Darwin platform)
  		LIBS="$LIBS -L${prefix}/lib"
  		CFLAGS="$CFLAGS -I${prefix}/include"
		;;
esac

AC_DEFINE_UNQUOTED(HA_LOG_FACILITY, LOG_DAEMON, Default logging facility)

AC_MSG_NOTICE(Host CPU: $host_cpu)

case "$host_cpu" in
  ppc64|powerpc64)
    case $CFLAGS in
     *powerpc64*)			;;
     *)	if test "$GCC" = yes; then
	  CFLAGS="$CFLAGS -m64"
	fi				;;
    esac
esac

AC_MSG_CHECKING(which format is needed to print uint64_t)
case "$host_cpu" in
  s390x)U64T="%lu";;
  *64*) U64T="%lu";;
  *)    U64T="%llu";;
esac
AC_MSG_RESULT($U64T)
AC_DEFINE_UNQUOTED(U64T, "$U64T", Correct printf format for logging uint64_t)


dnl Variables needed for substitution

AC_CHECK_HEADERS(heartbeat/glue_config.h)

if test "$ac_cv_header_heartbeat_glue_config_h" != "yes"; then
  enable_libnet=no
fi

AC_DEFINE_UNQUOTED(OCF_ROOT_DIR,"$OCF_ROOT_DIR", OCF root directory - specified by the OCF standard)
AC_SUBST(OCF_ROOT_DIR)

GLUE_STATE_DIR=${localstatedir}/run
AC_DEFINE_UNQUOTED(GLUE_STATE_DIR,"$GLUE_STATE_DIR", Where to keep state files and sockets)
AC_SUBST(GLUE_STATE_DIR)

AC_DEFINE_UNQUOTED(HA_VARRUNDIR,"$GLUE_STATE_DIR", Where Heartbeat keeps state files and sockets - old name)
HA_VARRUNDIR="$GLUE_STATE_DIR"
AC_SUBST(HA_VARRUNDIR)

# Expand $prefix
eval HA_RSCTMPDIR="`eval echo ${HA_RSCTMPDIR}`"
AC_DEFINE_UNQUOTED(HA_RSCTMPDIR,"$HA_RSCTMPDIR", Where Resource agents keep state files)
AC_SUBST(HA_RSCTMPDIR)

dnl Eventually move out of the heartbeat dir tree and create symlinks when needed
HA_VARLIBHBDIR=${localstatedir}/lib/heartbeat
AC_DEFINE_UNQUOTED(HA_VARLIBHBDIR,"$HA_VARLIBHBDIR", Whatever this used to mean)
AC_SUBST(HA_VARLIBHBDIR)

OCF_RA_DIR="${OCF_ROOT_DIR}/resource.d"
AC_DEFINE_UNQUOTED(OCF_RA_DIR,"$OCF_RA_DIR", Location for OCF RAs)
AC_SUBST(OCF_RA_DIR)

OCF_RA_DIR_PREFIX="$OCF_RA_DIR"
AC_SUBST(OCF_RA_DIR_PREFIX)

OCF_LIB_DIR="${OCF_ROOT_DIR}/lib"
AC_DEFINE_UNQUOTED(OCF_LIB_DIR,"$OCF_LIB_DIR", Location for shared code for OCF RAs)
AC_SUBST(OCF_LIB_DIR)

OCF_LIB_DIR_PREFIX="$OCF_LIB_DIR"
AC_SUBST(OCF_LIB_DIR_PREFIX)

dnl ===============================================
dnl rgmanager ras bits
dnl ===============================================

LOGDIR=${localstatedir}/log/cluster
CLUSTERDATA=${datadir}/cluster

AC_SUBST([LOGDIR])
AC_SUBST([CLUSTERDATA])

dnl ===============================================
dnl Program Paths
dnl ===============================================

PATH="$PATH:/sbin:/usr/sbin:/usr/local/sbin:/usr/local/bin"
export PATH

AC_CHECK_PROGS(MAKE, gmake make)
AC_CHECK_PROGS(SHELLCHECK, shellcheck)
AM_CONDITIONAL(CI_CHECKS, test "x$SHELLCHECK" != "x" )
AC_PATH_PROGS(BASH_SHELL, bash)
if test x"${BASH_SHELL}" = x""; then
   AC_MSG_ERROR(You need bash installed in order to build ${PACKAGE})
fi
AC_PATH_PROGS(XSLTPROC, xsltproc)
AM_CONDITIONAL(BUILD_DOC, test "x$XSLTPROC" != "x" )
if test "x$XSLTPROC" = "x"; then
   AC_MSG_WARN([xsltproc not installed, unable to (re-)build manual pages])
fi
AC_SUBST(XSLTPROC)
AC_PATH_PROGS(XMLCATALOG, xmlcatalog)
AC_PATH_PROGS(SSH, ssh, /usr/bin/ssh)
AC_PATH_PROGS(SCP, scp, /usr/bin/scp)
AC_PATH_PROGS(TAR, tar)
AC_PATH_PROGS(MD5, md5)
AC_PATH_PROGS(TEST, test)
AC_PATH_PROGS(PING, ping, /bin/ping)
AC_PATH_PROGS(IFCONFIG, ifconfig, /sbin/ifconfig)
AC_PATH_PROGS(MAILCMD, mailx mail, mail)
AC_PATH_PROGS(RM, rm)
AC_PROG_EGREP
AC_PROG_FGREP

AC_SUBST(BASH_SHELL)
AC_SUBST(MAILCMD)
AC_SUBST(SHELL)
AC_SUBST(PING)
AC_SUBST(RM)
AC_SUBST(TEST)

AM_PATH_PYTHON([3.6])
if test -z "$PYTHON"; then
    echo "*** Essential program python not found" 1>&2
    exit 1
fi

dnl Ensure PYTHON is an absolute path
AC_PATH_PROG([PYTHON], [$PYTHON])

AM_PATH_PYTHON
if test -z "$PYTHON"; then
    echo "*** Essential program python not found" 1>&2
fi

AC_PYTHON_MODULE(json)
AC_PYTHON_MODULE(pyroute2)
AC_PYTHON_MODULE(requests)
AC_PYTHON_MODULE(urllib3)
AC_PYTHON_MODULE(ibm_cloud_fail_over)

AS_VERSION_COMPARE([$PYTHON_VERSION], [3.6], [BUILD_OCF_PY=0], [BUILD_OCF_PY=1], [BUILD_OCF_PY=1])

BUILD_AZURE_EVENTS=1
if test -z "$PYTHON" || test $BUILD_OCF_PY -eq 0; then
    BUILD_AZURE_EVENTS=0
    AC_MSG_WARN("Not building azure-events")
fi
AM_CONDITIONAL(BUILD_AZURE_EVENTS, test $BUILD_AZURE_EVENTS -eq 1)

BUILD_AZURE_EVENTS_AZ=1
if test -z "$PYTHON" || test $BUILD_OCF_PY -eq 0; then
    BUILD_AZURE_EVENTS_AZ=0
    AC_MSG_WARN("Not building azure-events-az")
fi
AM_CONDITIONAL(BUILD_AZURE_EVENTS_AZ, test $BUILD_AZURE_EVENTS_AZ -eq 1)

BUILD_GCP_PD_MOVE=1
if test -z "$PYTHON" || test $BUILD_OCF_PY -eq 0; then
    BUILD_GCP_PD_MOVE=0
    AC_MSG_WARN("Not building gcp-pd-move")
fi
AM_CONDITIONAL(BUILD_GCP_PD_MOVE, test $BUILD_GCP_PD_MOVE -eq 1)

BUILD_GCP_VPC_MOVE_ROUTE=1
if test -z "$PYTHON" || test "x${HAVE_PYMOD_PYROUTE2}" != xyes || test $BUILD_OCF_PY -eq 0; then
    BUILD_GCP_VPC_MOVE_ROUTE=0
    AC_MSG_WARN("Not building gcp-vpc-move-route")
fi
AM_CONDITIONAL(BUILD_GCP_VPC_MOVE_ROUTE, test $BUILD_GCP_VPC_MOVE_ROUTE -eq 1)

BUILD_GCP_VPC_MOVE_VIP=1
if test -z "$PYTHON" || test $BUILD_OCF_PY -eq 0; then
    BUILD_GCP_VPC_MOVE_VIP=0
    AC_MSG_WARN("Not building gcp-vpc-move-vip")
fi
AM_CONDITIONAL(BUILD_GCP_VPC_MOVE_VIP, test $BUILD_GCP_VPC_MOVE_VIP -eq 1)

BUILD_POWERVS_SUBNET=1
if test -z "$PYTHON" || test $BUILD_OCF_PY -eq 0 || test "x${HAVE_PYMOD_REQUESTS}" != xyes || test "x${HAVE_PYMOD_URLLIB3}" != xyes; then
    BUILD_POWERVS_SUBNET=0
    AC_MSG_WARN("Not building powervs-subnet")
fi
AM_CONDITIONAL(BUILD_POWERVS_SUBNET, test $BUILD_POWERVS_SUBNET -eq 1)

BUILD_IBM_CLOUD_VPC_MOVE_ROUTE=1
if test -z "$PYTHON" || test $BUILD_OCF_PY -eq 0 || test "x${HAVE_PYMOD_IBM_CLOUD_FAIL_OVER}" != xyes; then
    BUILD_IBM_CLOUD_VPC_MOVE_ROUTE=0
    AC_MSG_WARN("Not building ibm-cloud-vpc-cr-vip")
fi
AM_CONDITIONAL(BUILD_IBM_CLOUD_VPC_MOVE_ROUTE, test $BUILD_IBM_CLOUD_VPC_MOVE_ROUTE -eq 1)

BUILD_IBM_CLOUD_VPC_MOVE_FIP=1
if test -z "$PYTHON" || test $BUILD_OCF_PY -eq 0 || test "x${HAVE_PYMOD_IBM_CLOUD_FAIL_OVER}" != xyes; then
    BUILD_IBM_CLOUD_VPC_MOVE_FIP=0
    AC_MSG_WARN("Not building ibm-cloud-vpc-move-fip")
fi
AM_CONDITIONAL(BUILD_IBM_CLOUD_VPC_MOVE_FIP, test $BUILD_IBM_CLOUD_VPC_MOVE_FIP -eq 1)

AC_PATH_PROGS(ROUTE, route)
AC_DEFINE_UNQUOTED(ROUTE, "$ROUTE", path to route command)

AC_MSG_CHECKING(ifconfig option to list interfaces)
for IFCONFIG_A_OPT in  "-A" "-a" ""
do
  $IFCONFIG $IFCONFIG_A_OPT > /dev/null 2>&1
  if
    test "$?" = 0
  then
    AC_DEFINE_UNQUOTED(IFCONFIG_A_OPT, "$IFCONFIG_A_OPT", option for ifconfig command)
    AC_MSG_RESULT($IFCONFIG_A_OPT)
    break
  fi
done

AC_SUBST(IFCONFIG_A_OPT)

if test x"${MAKE}" = x""; then
   AC_MSG_ERROR(You need (g)make installed in order to build ${PACKAGE})
fi

STYLESHEET_PREFIX=""
if test x"${XSLTPROC}" != x""; then
    AC_MSG_CHECKING(docbook to manpage transform)
    # first try to figure out correct template using xmlcatalog query,
    # resort to extensive (semi-deterministic) file search if that fails
    DOCBOOK_XSL_URI='http://docbook.sourceforge.net/release/xsl/current'
    DOCBOOK_XSL_PATH='manpages/docbook.xsl'
    STYLESHEET_PREFIX=$(${XMLCATALOG} "" ${DOCBOOK_XSL_URI} \
                   | sed -n 's|^file://||p;q')
    if test x"${STYLESHEET_PREFIX}" = x""; then
        DIRS=$(find "${datadir}" -name $(basename $(dirname ${DOCBOOK_XSL_PATH})) \
               -type d | LC_ALL=C sort)
        if test x"${DIRS}" = x""; then
            # when datadir is not standard OS path, we cannot find docbook.xsl
            # use standard OS path as backup
            DIRS=$(find "/usr/share" "/usr/local/share" -name $(basename $(dirname ${DOCBOOK_XSL_PATH})) \
                   -type d | LC_ALL=C sort)
        fi
        XSLT=$(basename ${DOCBOOK_XSL_PATH})
        for d in ${DIRS}; do
            if test -f "${d}/${XSLT}"; then
                 STYLESHEET_PREFIX=$(echo "${d}" | sed 's/\/manpages//')
                 break
            fi
        done
    fi
    if test x"${STYLESHEET_PREFIX}" = x""; then
        AC_MSG_ERROR(You need docbook-style-xsl installed in order to build ${PACKAGE})
    fi
fi
AC_MSG_RESULT($STYLESHEET_PREFIX)
AC_SUBST(STYLESHEET_PREFIX)

dnl ===============================================
dnl Libraries
dnl ===============================================
AC_CHECK_LIB(socket, socket)			
AC_CHECK_LIB(gnugetopt, getopt_long)		dnl if available

if test "x${enable_thread_safe}" = "xyes"; then
        GPKGNAME="gthread-2.0"
else
        GPKGNAME="glib-2.0"
fi

PKG_CHECK_MODULES([GLIB], [$GPKGNAME])
CPPFLAGS="$CPPFLAGS $GLIB_CFLAGS"
LIBS="$LIBS $GLIB_LIBS"
PKG_CHECK_MODULES([LIBQB], "libqb")

dnl ========================================================================
dnl Headers
dnl ========================================================================

AC_HEADER_STDC
AC_CHECK_HEADERS(sys/socket.h)
AC_CHECK_HEADERS(sys/sockio.h)
AC_CHECK_HEADERS([arpa/inet.h])
AC_CHECK_HEADERS([fcntl.h])
AC_CHECK_HEADERS([limits.h])
AC_CHECK_HEADERS([malloc.h])
AC_CHECK_HEADERS([netdb.h])
AC_CHECK_HEADERS([netinet/in.h])
AC_CHECK_HEADERS([sys/file.h])
AC_CHECK_HEADERS([sys/ioctl.h])
AC_CHECK_HEADERS([sys/param.h])
AC_CHECK_HEADERS([sys/time.h])
AC_CHECK_HEADERS([syslog.h])

dnl ========================================================================
dnl Functions
dnl ========================================================================

AC_FUNC_FORK
AC_FUNC_STRNLEN
AC_CHECK_FUNCS([alarm gettimeofday inet_ntoa memset mkdir socket uname])
AC_CHECK_FUNCS([strcasecmp strchr strdup strerror strrchr strspn strstr strtol strtoul])

AC_PATH_PROGS(REBOOT, reboot, /sbin/reboot)
AC_SUBST(REBOOT)
AC_SUBST(REBOOT_OPTIONS)
AC_DEFINE_UNQUOTED(REBOOT, "$REBOOT", path to the reboot command)
AC_DEFINE_UNQUOTED(REBOOT_OPTIONS, "$REBOOT_OPTIONS", reboot options)

AC_PATH_PROGS(POWEROFF_CMD, poweroff, /sbin/poweroff)
AC_SUBST(POWEROFF_CMD)
AC_SUBST(POWEROFF_OPTIONS)
AC_DEFINE_UNQUOTED(POWEROFF_CMD, "$POWEROFF_CMD", path to the poweroff command)
AC_DEFINE_UNQUOTED(POWEROFF_OPTIONS, "$POWEROFF_OPTIONS", poweroff options)

AC_PATH_PROGS(POD2MAN, pod2man)
AM_CONDITIONAL(BUILD_POD_DOC, test "x$POD2MAN" != "x" )
if test "x$POD2MAN" = "x"; then
   AC_MSG_WARN([pod2man not installed, unable to (re-)build ldirector manual page])
fi
AC_SUBST(POD2MAN)

dnl ========================================================================
dnl Functions
dnl ========================================================================

AC_CHECK_FUNCS(getopt, AC_DEFINE(HAVE_DECL_GETOPT,  1, [Have getopt function]))

dnl ========================================================================
dnl   sfex
dnl ========================================================================

build_sfex=no
case $host_os in
    *Linux*|*linux*) 
	if test "$ac_cv_header_heartbeat_glue_config_h" = "yes"; then
	    build_sfex=yes
	fi
	;;
esac

AM_CONDITIONAL(BUILD_SFEX, test "$build_sfex" = "yes" )


dnl ========================================================================
dnl   tickle (needs port to BSD platforms)
dnl ========================================================================

AC_CHECK_MEMBERS([struct iphdr.saddr],,,[[#include <netinet/ip.h>]])
AM_CONDITIONAL(BUILD_TICKLE, test "$ac_cv_member_struct_iphdr_saddr" = "yes" )

dnl ========================================================================
dnl   libnet
dnl ========================================================================

libnet=""
libnet_version="none"
LIBNETLIBS=""
LIBNETDEFINES=""

AC_MSG_CHECKING(if libnet is required)
libnet_fatal=$enable_libnet
case $enable_libnet in
     no) ;;
     yes|libnet10|libnet11|10|11) libnet_fatal=yes;;
     try)
	case $host_os in
	     *Linux*|*linux*) libnet_fatal=no;;
	     *) libnet_fatal=yes;; dnl legacy behavior
	esac
	;;
     *) libnet_fatal=yes; enable_libnet=try;;
esac
AC_MSG_RESULT($libnet_fatal)

if test "x$enable_libnet" != "xno"; then
   AC_PATH_PROGS(LIBNETCONFIG, libnet-config)

   AC_CHECK_LIB(nsl, t_open)			dnl -lnsl
   AC_CHECK_LIB(socket, socket)			dnl -lsocket
   AC_CHECK_LIB(net, libnet_get_hwaddr, LIBNETLIBS=" -lnet", [])
 fi
 
AC_MSG_CHECKING(for libnet)
if test "x$LIBNETLIBS" != "x" -o "x$enable_libnet" = "xlibnet11"; then
      LIBNETDEFINES=""
      if test "$ac_cv_lib_nsl_t_open" = yes; then
         LIBNETLIBS="-lnsl $LIBNETLIBS"
      fi
      if test "$ac_cv_lib_socket_socket" = yes; then
         LIBNETLIBS="-lsocket $LIBNETLIBS"
      fi

      libnet=net
      libnet_version="libnet1.1"
fi

if test "x$enable_libnet" = "xtry" -o "x$enable_libnet" = "xlibnet10"; then
   if test "x$LIBNETLIBS" = x -a "x${LIBNETCONFIG}" != "x" ; then
      LIBNETDEFINES="`$LIBNETCONFIG --defines` `$LIBNETCONFIG --cflags`";
      LIBNETLIBS="`$LIBNETCONFIG --libs`";
      libnet_version="libnet1.0 (old)"
      case $LIBNETLIBS in
        *-l*)	libnet=`echo $LIBNETLIBS | sed 's%.*-l%%'`;;
        *)	libnet_version=none;;
      esac

      CPPFLAGS="$CPPFLAGS $LIBNETDEFINES"

      AC_CHECK_HEADERS(libnet.h)
      if test "$ac_cv_header_libnet_h" = no; then
	libnet_version=none
      fi
   fi
fi
AC_MSG_RESULT(found $libnet_version)

if test "$libnet_version" = none; then
   LIBNETLIBS=""
   LIBNETDEFINES=""
   if test $libnet_fatal = yes; then
        AC_MSG_ERROR(libnet not found)
   fi

else
   AC_CHECK_LIB($libnet,libnet_init,
      [new_libnet=yes; AC_DEFINE(HAVE_LIBNET_1_1_API, 1, Libnet 1.1 API)],
      [new_libnet=no; AC_DEFINE(HAVE_LIBNET_1_0_API, 1, Libnet 1.0 API)],$LIBNETLIBS)
   AC_SUBST(LIBNETLIBS)
fi

if test "$new_libnet" = yes; then
   AC_MSG_CHECKING(for libnet API 1.1.4: )
   save_CFLAGS="$CFLAGS"
   CFLAGS="$CFLAGS -fgnu89-inline -Wall -Werror"
   AC_COMPILE_IFELSE([
	AC_LANG_SOURCE(#include <libnet.h>
			  int main(){libnet_t *l=NULL; libnet_pblock_record_ip_offset(l, l->total_size); return(0); })],
      [AC_MSG_RESULT(no)],
      [AC_DEFINE(HAVE_LIBNET_1_1_4_API, 1, Libnet 1.1.4 API) AC_MSG_RESULT(yes)])
   CFLAGS="$save_CFLAGS"
fi

sendarp_linux=0
case $host_os in
     *Linux*|*linux*) sendarp_linux=1;;
esac

redhat_based=0
AC_CHECK_FILE(/etc/redhat-release, [redhat_based=1])

AC_SUBST(LIBNETLIBS)
AC_SUBST(LIBNETDEFINES)

AM_CONDITIONAL(SENDARP_LINUX, test $sendarp_linux = 1 )
AM_CONDITIONAL(USE_LIBNET, test "x$libnet_version" != "xnone" )
AM_CONDITIONAL(NFSCONVERT, test $redhat_based = 1 )

dnl ************************************************************************
dnl * Check for netinet/icmp6.h to enable the IPv6addr resource agent
AC_CHECK_HEADERS(netinet/icmp6.h,[],[],[#include <sys/types.h>])
AM_CONDITIONAL(USE_IPV6ADDR_AGENT, test "$ac_cv_header_netinet_icmp6_h" = yes && test "$ac_cv_header_heartbeat_glue_config_h" = yes)
AM_CONDITIONAL(IPV6ADDR_COMPATIBLE, test "$ac_cv_header_netinet_icmp6_h" = yes)

dnl ========================================================================
dnl Compiler flags
dnl ========================================================================

dnl Make sure that CFLAGS is not exported. If the user did
dnl not have CFLAGS in their environment then this should have
dnl no effect. However if CFLAGS was exported from the user's
dnl environment, then the new CFLAGS will also be exported
dnl to sub processes.

CC_ERRORS=""
CC_EXTRAS=""

if export -p | fgrep " CFLAGS=" > /dev/null; then
	SAVED_CFLAGS="$CFLAGS"
	unset CFLAGS
	CFLAGS="$SAVED_CFLAGS"
	unset SAVED_CFLAGS
fi

if test "$GCC" != yes; then
        CFLAGS="$CFLAGS -g"
	enable_fatal_warnings=no
else
        CFLAGS="$CFLAGS -ggdb3"

	# We had to eliminate -Wnested-externs because of libtool changes
	# Also remove -Waggregate-return because we use one libnet
	# call which returns a struct
        EXTRA_FLAGS="-fgnu89-inline
		-fstack-protector-all
		-Wall
		-Wbad-function-cast 
		-Wcast-qual 
		-Wdeclaration-after-statement
		-Wendif-labels
		-Wfloat-equal
		-Wformat=2
		-Wformat-security
		-Wformat-nonliteral
		-Winline
		-Wmissing-prototypes 
		-Wmissing-declarations 
		-Wmissing-format-attribute
		-Wnested-externs
		-Wno-long-long
		-Wno-strict-aliasing
		-Wpointer-arith 
		-Wstrict-prototypes
		-Wunsigned-char
		-Wwrite-strings
		-Wno-maybe-uninitialized"

# Additional warnings it might be nice to enable one day
#		-Wshadow
#		-Wunreachable-code

	for j in $EXTRA_FLAGS
	do
	  if
	    cc_supports_flag $j
	  then
	    CC_EXTRAS="$CC_EXTRAS $j"
	  fi
	done

dnl In lib/ais/Makefile.am there's a gcc option available as of v4.x

	GCC_MAJOR=`gcc -v 2>&1 | awk 'END{print $3}' | sed 's/[.].*//'`
	AM_CONDITIONAL(GCC_4, test "${GCC_MAJOR}" = 4)

dnl System specific options

	case "$host_os" in
  	*linux*|*bsd*)
		if test "${enable_fatal_warnings}" = "unknown"; then
        		enable_fatal_warnings=yes
        	fi
          	;;
	esac

	if test "x${enable_fatal_warnings}" != xno && cc_supports_flag -Werror ; then
	   enable_fatal_warnings=yes
	else
	   enable_fatal_warnings=no
        fi

	if test "x${enable_ansi}" = xyes && cc_supports_flag -std=iso9899:199409; then
	  AC_MSG_NOTICE(Enabling ANSI Compatibility)
	  CC_EXTRAS="$CC_EXTRAS -ansi -D_GNU_SOURCE -DANSI_ONLY"
	fi

  	AC_MSG_NOTICE(Activated additional gcc flags: ${CC_EXTRAS})
fi

CFLAGS="$CFLAGS $CC_EXTRAS"

NON_FATAL_CFLAGS="$CFLAGS"
AC_SUBST(NON_FATAL_CFLAGS)

dnl
dnl We reset CFLAGS to include our warnings *after* all function
dnl checking goes on, so that our warning flags don't keep the
dnl AC_*FUNCS() calls above from working.  In particular, -Werror will
dnl *always* cause us troubles if we set it before here.
dnl
dnl
if test "x${enable_fatal_warnings}" = xyes ; then
   AC_MSG_NOTICE(Enabling Fatal Warnings)
   CFLAGS="$CFLAGS -Werror"
fi
AC_SUBST(CFLAGS)

dnl This is useful for use in Makefiles that need to remove one specific flag
CFLAGS_COPY="$CFLAGS"
AC_SUBST(CFLAGS_COPY)

AC_SUBST(LOCALE)

AC_SUBST(CC)
AC_SUBST(MAKE)

dnl The Makefiles and shell scripts we output
AC_CONFIG_FILES(Makefile				        \
resource-agents.pc						\
include/Makefile						\
heartbeat/Makefile						\
   heartbeat/ocf-binaries					\
   heartbeat/ocf-directories					\
   heartbeat/ocf-shellfuncs					\
   heartbeat/shellfuncs						\
systemd/Makefile						\
   systemd/resource-agents.conf					\
tools/Makefile							\
   tools/nfsconvert						\
   tools/ocf-tester						\
   tools/ocft/Makefile						\
   tools/ocft/ocft						\
   tools/ocft/caselib						\
   tools/ocft/README						\
   tools/ocft/README.zh_CN					\
ldirectord/Makefile						\
ldirectord/ldirectord   					\
	ldirectord/init.d/Makefile				\
	ldirectord/init.d/ldirectord   				\
	ldirectord/init.d/ldirectord.debian			\
	ldirectord/init.d/ldirectord.debian.default		\
	ldirectord/systemd/Makefile	\
	ldirectord/systemd/ldirectord.service	\
	ldirectord/logrotate.d/Makefile				\
	ldirectord/OCF/Makefile					\
	ldirectord/OCF/ldirectord				\
doc/Makefile							\
	doc/man/Makefile					\
rgmanager/Makefile						\
	rgmanager/src/Makefile					\
	rgmanager/src/resources/Makefile			\
		rgmanager/src/resources/ocf-shellfuncs		\
		rgmanager/src/resources/svclib_nfslock		\
		rgmanager/src/resources/lvm_by_lv.sh		\
		rgmanager/src/resources/lvm_by_vg.sh		\
	rgmanager/src/resources/utils/Makefile			\
		rgmanager/src/resources/utils/fs-lib.sh		\
		rgmanager/src/resources/utils/messages.sh	\
		rgmanager/src/resources/utils/config-utils.sh	\
		rgmanager/src/resources/utils/member_util.sh	\
		rgmanager/src/resources/utils/ra-skelet.sh	\
		)

dnl Files we output that need to be executable
AC_CONFIG_FILES([heartbeat/azure-events], [chmod +x heartbeat/azure-events])
AC_CONFIG_FILES([heartbeat/azure-events-az], [chmod +x heartbeat/azure-events-az])
AC_CONFIG_FILES([heartbeat/AoEtarget], [chmod +x heartbeat/AoEtarget])
AC_CONFIG_FILES([heartbeat/ManageRAID], [chmod +x heartbeat/ManageRAID])
AC_CONFIG_FILES([heartbeat/ManageVE], [chmod +x heartbeat/ManageVE])
AC_CONFIG_FILES([heartbeat/Squid], [chmod +x heartbeat/Squid])
AC_CONFIG_FILES([heartbeat/SysInfo], [chmod +x heartbeat/SysInfo])
AC_CONFIG_FILES([heartbeat/aws-vpc-route53], [chmod +x heartbeat/aws-vpc-route53])
AC_CONFIG_FILES([heartbeat/clvm], [chmod +x heartbeat/clvm])
AC_CONFIG_FILES([heartbeat/conntrackd], [chmod +x heartbeat/conntrackd])
AC_CONFIG_FILES([heartbeat/dnsupdate], [chmod +x heartbeat/dnsupdate])
AC_CONFIG_FILES([heartbeat/dummypy], [chmod +x heartbeat/dummypy])
AC_CONFIG_FILES([heartbeat/eDir88], [chmod +x heartbeat/eDir88])
AC_CONFIG_FILES([heartbeat/fio], [chmod +x heartbeat/fio])
AC_CONFIG_FILES([heartbeat/galera], [chmod +x heartbeat/galera])
AC_CONFIG_FILES([heartbeat/gcp-pd-move], [chmod +x heartbeat/gcp-pd-move])
AC_CONFIG_FILES([heartbeat/gcp-vpc-move-ip], [chmod +x heartbeat/gcp-vpc-move-ip])
AC_CONFIG_FILES([heartbeat/gcp-vpc-move-vip], [chmod +x heartbeat/gcp-vpc-move-vip])
AC_CONFIG_FILES([heartbeat/gcp-vpc-move-route], [chmod +x heartbeat/gcp-vpc-move-route])
AC_CONFIG_FILES([heartbeat/ibm-cloud-vpc-cr-vip], [chmod +x heartbeat/ibm-cloud-vpc-cr-vip])
AC_CONFIG_FILES([heartbeat/ibm-cloud-vpc-move-fip], [chmod +x heartbeat/ibm-cloud-vpc-move-fip])
AC_CONFIG_FILES([heartbeat/iSCSILogicalUnit], [chmod +x heartbeat/iSCSILogicalUnit])
AC_CONFIG_FILES([heartbeat/iSCSITarget], [chmod +x heartbeat/iSCSITarget])
AC_CONFIG_FILES([heartbeat/jira], [chmod +x heartbeat/jira])
AC_CONFIG_FILES([heartbeat/kamailio], [chmod +x heartbeat/kamailio])
AC_CONFIG_FILES([heartbeat/lxc], [chmod +x heartbeat/lxc])
AC_CONFIG_FILES([heartbeat/lxd-info], [chmod +x heartbeat/lxd-info])
AC_CONFIG_FILES([heartbeat/machine-info], [chmod +x heartbeat/machine-info])
AC_CONFIG_FILES([heartbeat/mariadb], [chmod +x heartbeat/mariadb])
AC_CONFIG_FILES([heartbeat/mpathpersist], [chmod +x heartbeat/mpathpersist])
AC_CONFIG_FILES([heartbeat/nfsnotify], [chmod +x heartbeat/nfsnotify])
AC_CONFIG_FILES([heartbeat/openstack-info], [chmod +x heartbeat/openstack-info])
AC_CONFIG_FILES([heartbeat/powervs-subnet], [chmod +x heartbeat/powervs-subnet])
AC_CONFIG_FILES([heartbeat/rabbitmq-cluster], [chmod +x heartbeat/rabbitmq-cluster])
AC_CONFIG_FILES([heartbeat/redis], [chmod +x heartbeat/redis])
AC_CONFIG_FILES([heartbeat/rsyslog], [chmod +x heartbeat/rsyslog])
AC_CONFIG_FILES([heartbeat/smb-share], [chmod +x heartbeat/smb-share])
AC_CONFIG_FILES([heartbeat/sg_persist], [chmod +x heartbeat/sg_persist])
AC_CONFIG_FILES([heartbeat/slapd], [chmod +x heartbeat/slapd])
AC_CONFIG_FILES([heartbeat/storage-mon], [chmod +x heartbeat/storage-mon])
AC_CONFIG_FILES([heartbeat/sybaseASE], [chmod +x heartbeat/sybaseASE])
AC_CONFIG_FILES([heartbeat/syslog-ng], [chmod +x heartbeat/syslog-ng])
AC_CONFIG_FILES([heartbeat/vsftpd], [chmod +x heartbeat/vsftpd])
AC_CONFIG_FILES([heartbeat/CTDB], [chmod +x heartbeat/CTDB])
AC_CONFIG_FILES([rgmanager/src/resources/ASEHAagent.sh], [chmod +x rgmanager/src/resources/ASEHAagent.sh])
AC_CONFIG_FILES([rgmanager/src/resources/apache.sh], [chmod +x rgmanager/src/resources/apache.sh])
AC_CONFIG_FILES([rgmanager/src/resources/bind-mount.sh], [chmod +x rgmanager/src/resources/bind-mount.sh])
AC_CONFIG_FILES([rgmanager/src/resources/clusterfs.sh], [chmod +x rgmanager/src/resources/clusterfs.sh])
AC_CONFIG_FILES([rgmanager/src/resources/db2.sh], [chmod +x rgmanager/src/resources/db2.sh])
AC_CONFIG_FILES([rgmanager/src/resources/drbd.sh], [chmod +x rgmanager/src/resources/drbd.sh])
AC_CONFIG_FILES([rgmanager/src/resources/fs.sh], [chmod +x rgmanager/src/resources/fs.sh])
AC_CONFIG_FILES([rgmanager/src/resources/ip.sh], [chmod +x rgmanager/src/resources/ip.sh])
AC_CONFIG_FILES([rgmanager/src/resources/lvm.sh], [chmod +x rgmanager/src/resources/lvm.sh])
AC_CONFIG_FILES([rgmanager/src/resources/mysql.sh], [chmod +x rgmanager/src/resources/mysql.sh])
AC_CONFIG_FILES([rgmanager/src/resources/named.sh], [chmod +x rgmanager/src/resources/named.sh])
AC_CONFIG_FILES([rgmanager/src/resources/netfs.sh], [chmod +x rgmanager/src/resources/netfs.sh])
AC_CONFIG_FILES([rgmanager/src/resources/nfsclient.sh], [chmod +x rgmanager/src/resources/nfsclient.sh])
AC_CONFIG_FILES([rgmanager/src/resources/nfsexport.sh], [chmod +x rgmanager/src/resources/nfsexport.sh])
AC_CONFIG_FILES([rgmanager/src/resources/nfsserver.sh], [chmod +x rgmanager/src/resources/nfsserver.sh])
AC_CONFIG_FILES([rgmanager/src/resources/openldap.sh], [chmod +x rgmanager/src/resources/openldap.sh])
AC_CONFIG_FILES([rgmanager/src/resources/oracledb.sh], [chmod +x rgmanager/src/resources/oracledb.sh])
AC_CONFIG_FILES([rgmanager/src/resources/oradg.sh], [chmod +x rgmanager/src/resources/oradg.sh])
AC_CONFIG_FILES([rgmanager/src/resources/orainstance.sh], [chmod +x rgmanager/src/resources/orainstance.sh])
AC_CONFIG_FILES([rgmanager/src/resources/oralistener.sh], [chmod +x rgmanager/src/resources/oralistener.sh])
AC_CONFIG_FILES([rgmanager/src/resources/postgres-8.sh], [chmod +x rgmanager/src/resources/postgres-8.sh])
AC_CONFIG_FILES([rgmanager/src/resources/samba.sh], [chmod +x rgmanager/src/resources/samba.sh])
AC_CONFIG_FILES([rgmanager/src/resources/script.sh], [chmod +x rgmanager/src/resources/script.sh])
AC_CONFIG_FILES([rgmanager/src/resources/service.sh], [chmod +x rgmanager/src/resources/service.sh])
AC_CONFIG_FILES([rgmanager/src/resources/smb.sh], [chmod +x rgmanager/src/resources/smb.sh])
AC_CONFIG_FILES([rgmanager/src/resources/tomcat-5.sh], [chmod +x rgmanager/src/resources/tomcat-5.sh])
AC_CONFIG_FILES([rgmanager/src/resources/tomcat-6.sh], [chmod +x rgmanager/src/resources/tomcat-6.sh])
AC_CONFIG_FILES([rgmanager/src/resources/vm.sh], [chmod +x rgmanager/src/resources/vm.sh])

dnl Now process the entire list of files added by previous 
dnl  calls to AC_CONFIG_FILES()
AC_OUTPUT()

dnl *****************
dnl Configure summary
dnl *****************

AC_MSG_RESULT([])
AC_MSG_RESULT([$PACKAGE configuration:])
AC_MSG_RESULT([  Version                  = ${VERSION}])
AC_MSG_RESULT([  Build Version            = $Format:%H$])
AC_MSG_RESULT([])
AC_MSG_RESULT([  Prefix                   = ${prefix}])
AC_MSG_RESULT([  Executables              = ${sbindir}])
AC_MSG_RESULT([  Man pages                = ${mandir}])
AC_MSG_RESULT([  Libraries                = ${libdir}])
AC_MSG_RESULT([  Header files             = ${includedir}])
AC_MSG_RESULT([  Arch-independent files   = ${datadir}])
AC_MSG_RESULT([  Documentation            = ${docdir}])
AC_MSG_RESULT([  State information        = ${localstatedir}])
AC_MSG_RESULT([  System configuration     = ${sysconfdir}])
AC_MSG_RESULT([  HA_BIN directory prefix  = ${libexecdir}])
AC_MSG_RESULT([  RA state files           = ${HA_RSCTMPDIR}])
AC_MSG_RESULT([  AIS Plugins              = ${LCRSODIR}])
AC_MSG_RESULT([])
AC_MSG_RESULT([  CPPFLAGS                 = ${CPPFLAGS}])
AC_MSG_RESULT([  CFLAGS                   = ${CFLAGS}])
AC_MSG_RESULT([  Libraries                = ${LIBS}])
AC_MSG_RESULT([  Stack Libraries          = ${CLUSTERLIBS}])

