# Created by: Patrick Tracanelli <eksffa@freebsdbrasil.com.br>
# $FreeBSD: head/security/suricata/Makefile 377282 2015-01-18 07:12:37Z koobs $

PORTNAME=	suricata
PORTVERSION=	2.1.20141213
CATEGORIES=	security
MASTER_SITES=	http://www.openinfosecfoundation.org/download/ \
		http://mirrors.rit.edu/zi/
PKGNAMESUFFIX=	-git

MAINTAINER=	koobs@FreeBSD.org
COMMENT=	High Performance Network IDS, IPS and Security Monitoring engine

LICENSE=	GPLv2
LICENSE_FILE=	${WRKSRC}/LICENSE

LIB_DEPENDS=	libpcre.so:${PORTSDIR}/devel/pcre \
		libnet.so:${PORTSDIR}/net/libnet \
		libyaml.so:${PORTSDIR}/textproc/libyaml

USE_GITHUB=	yes
GH_ACCOUNT=	inliniac
GH_PROJECT=	suricata
GH_TAGNAME=	${GH_COMMIT}
GH_COMMIT=	1010406

USES=		autoreconf gmake pkgconfig libtool pathfix
USE_LDCONFIG=	yes
USE_RC_SUBR=	${PORTNAME}
GNU_CONFIGURE=	yes
CONFLICTS?=	suricata-2.* suricata-devel-2.*

INSTALL_TARGET=		install-strip
PATHFIX_MAKEFILEIN=	Makefile.am

OPTIONS_DEFINE=		GEOIP HTP_PORT IPFW JSON NETMAP NSS PORTS_PCAP PRELUDE SC TESTS
OPTIONS_DEFAULT=	GEOIP HTP_PORT IPFW JSON LUAJIT NETMAP NSS PRELUDE SC
OPTIONS_RADIO=		SCRIPTS
OPTIONS_RADIO_SCRIPTS=	LUA LUAJIT
OPTIONS_SUB=		yes

SCRIPTS_DESC=		Scripting

GEOIP_DESC=		GeoIP support
HTP_PORT_DESC=		Use libhtp from ports
IPFW_DESC=		IPFW and IP Divert support for inline IDP
JSON_DESC=		JSON output support
LUA_DESC=		LUA scripting support
LUAJIT_DESC=		LuaJIT scripting support
NSS_DESC=		File checksums and SSL/TLS fingerprinting
PORTS_PCAP_DESC=	Use libpcap from ports
PRELUDE_DESC=		Prelude support for NIDS alerts
SC_DESC=		Suricata socket client (suricatasc)
TESTS_DESC=		Unit tests in suricata binary

GEOIP_LIB_DEPENDS=		libGeoIP.so:${PORTSDIR}/net/GeoIP
GEOIP_CONFIGURE_ON=		--enable-geoip

HTP_PORT_LIB_DEPENDS=		libhtp.so:${PORTSDIR}/devel/libhtp
HTP_PORT_CONFIGURE_ON=		--enable-non-bundled-htp
HTP_PORT_CONFIGURE_OFF=		--enable-bundled-htp
HTP_PORT_CONFLICTS_INSTALL_OFF=	libhtp-[0-9]* libhtp-suricata
HTP_PORT_USES_OFF=		iconv

IPFW_CONFIGURE_ON=		--enable-ipfw

PORTS_PCAP_LIB_DEPENDS=		libpcap.so:${PORTSDIR}/net/libpcap
PORTS_PCAP_CONFIGURE_ON=	--with-libpcap-includes=${LOCALBASE}/include \
				--with-libpcap-libraries=${LOCALBASE}/lib
PORTS_PCAP_CONFIGURE_OFF=	--with-libpcap-includes=/usr/include \
				--with-libpcap-libraries=/usr/lib

PRELUDE_LIB_DEPENDS=		libprelude.so:${PORTSDIR}/security/libprelude
PRELUDE_CONFIGURE_ENABLE=	prelude
PRELUDE_CONFIGURE_ON=		--with-libprelude-prefix=${LOCALBASE}

JSON_LIB_DEPENDS=		libjansson.so:${PORTSDIR}/devel/jansson
JSON_CONFIGURE_ON=		--with-libjansson-includes=${LOCALBASE}/include \
				--with-libjansson-libraries=${LOCALBASE}/lib

LUA_USES=			lua
LUA_CONFIGURE_ON=		--enable-lua \
				--with-liblua-includes=${LUA_INCDIR} \
				--with-liblua-libraries=${LUA_LIBDIR}

LUAJIT_LIB_DEPENDS=		libluajit-5.1.so:${PORTSDIR}/lang/luajit
LUAJIT_CONFIGURE_ON=		--enable-luajit

NSS_LIB_DEPENDS=		libnss3.so:${PORTSDIR}/security/nss \
				libnspr4.so:${PORTSDIR}/devel/nspr

NETMAP_CONFIGURE_ON=		--enable-netmap

NSS_CONFIGURE_ON=		--with-libnss-includes=${LOCALBASE}/include/nss/nss \
				--with-libnss-libraries=${LOCALBASE}/lib \
				--with-libnspr-libraries=${LOCALBASE}/lib \
				--with-libnspr-includes=${LOCALBASE}/include/nspr

SC_USES=			python
SC_CONFIGURE_ENV=		ac_cv_path_HAVE_PYTHON_CONFIG=yes
SC_CONFIGURE_ENV_OFF=		ac_cv_path_HAVE_PYTHON_CONFIG=no

TESTS_CONFIGURE_ENABLE=		unittests

SUB_FILES=	pkg-message

CONFIGURE_ARGS+=--enable-gccprotect \
		--with-libpcre-includes=${LOCALBASE}/include \
		--with-libpcre-libraries=${LOCALBASE}/lib \
		--with-libyaml-includes=${LOCALBASE}/include \
		--with-libyaml-libraries=${LOCALBASE}/lib \
		--with-libnet-includes=${LOCALBASE}/include/libnet11 \
		--with-libnet-libraries=${LOCALBASE}/lib/libnet11 \
		--with-libhtp-includes=${LOCALBASE}/include/ \
		--with-libhtp-libraries=${LOCALBASE}/lib \
		--localstatedir=/var/

LIBNET_CONFIG?=	${LOCALBASE}/bin/libnet11-config
CONFIG_DIR?=	${ETCDIR}
CONFIG_FILES=	suricata.yaml classification.config reference.config
RULES_DIR=	${CONFIG_DIR}/rules
RULES_FILES=	decoder-events.rules dns-events.rules files.rules http-events.rules smtp-events.rules stream-events.rules tls-events.rules
LOGS_DIR?=	/var/log/${PORTNAME}

.include <bsd.port.pre.mk>

.if ${ARCH} == "ia64" || ${ARCH} == "powerpc" || ${ARCH} == "sparc64"
BROKEN=		Does not compile on ia64, powerpc, or sparc64
.endif

pre-patch:
	${CP} ${FILESDIR}/ax_check_compile_flag.m4 ${WRKSRC}/m4

post-install:
	${MKDIR} ${STAGEDIR}${CONFIG_DIR}
	${MKDIR} ${STAGEDIR}${RULES_DIR}
	${MKDIR} ${STAGEDIR}${LOGS_DIR}

.for f in ${CONFIG_FILES}
	${INSTALL_DATA} ${WRKSRC}/${f} ${STAGEDIR}${CONFIG_DIR}/${f}.sample
.endfor

.for f in ${RULES_FILES}
	${INSTALL_DATA} ${WRKSRC}/rules/${f} ${STAGEDIR}${RULES_DIR}/${f}
.endfor

.if ${PORT_OPTIONS:MSC}
	(cd ${STAGEDIR}${PREFIX} \
	&& ${PYTHON_CMD} ${PYTHON_LIBDIR}/compileall.py \
	-d ${PYTHONPREFIX_SITELIBDIR} -f ${PYTHONPREFIX_SITELIBDIR:S;${PREFIX}/;;})
.endif

TMPDIR?=	/tmp
TESTDIR=	${TMPDIR}/${PORTNAME}

regression-test: build
	${RM} -rf ${TESTDIR}
	${MKDIR} ${TESTDIR}
	cd ${WRKSRC}/src && ./suricata -u -l ${TESTDIR}
	${RM} -rf ${TESTDIR}

.include <bsd.port.post.mk>
