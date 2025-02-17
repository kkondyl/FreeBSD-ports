# Created by: pst

PORTNAME=	panda-imap
PORTVERSION=	20130621
PORTREVISION=	1
CATEGORIES=	mail

MAINTAINER=	thierry@FreeBSD.org
COMMENT=	IMAP4rev1/POP2/POP3 mail servers forked from imap-uw

LICENSE=	APACHE20
LICENSE_FILE=	${WRKSRC}/LICENSE

LIB_DEPENDS=	libc-client4.so.10:mail/panda-cclient

USE_GITHUB=	yes
GH_ACCOUNT=	jonabbey
GH_TAGNAME=	7905901

CONFLICTS_INSTALL=	imap-uw # bin/dmail

DOCSDIR=	${PREFIX}/share/doc/${PORTNAME}${PKGNAMESUFFIX}
SUB_FILES=	pkg-message

MAKE_JOBS_UNSAFE=	yes

ALL_TARGET=	bsf

# This port must have the same SSL settings as mail/cclient, which it depends on
# To make MBX format the default mailbox format, change the settings of cclient

OPTIONS_DEFINE=	SSL SSL_AND_PLAINTEXT NETSCAPE_BRAIN_DAMAGE DOCS
OPTIONS_DEFAULT=	SSL
OPTIONS_SUB=	yes

SSL_AND_PLAINTEXT_DESC=	Allow plain text passwords and SSL
NETSCAPE_BRAIN_DAMAGE_DESC=	See Makefile for documentation

.include <bsd.port.options.mk>

.if ! ${PORT_OPTIONS:MSSL}
MAKE_ARGS+=	SSLTYPE=none SSLDIR=${OPENSSLBASE}
.else
USES+=		ssl
.if ${PORT_OPTIONS:MSSL_AND_PLAINTEXT}
MAKE_ARGS+=	SSLTYPE=unix
.else
MAKE_ARGS+=	SSLTYPE=unix.nopwd
.endif
.endif

# Define this to get somewhat better interoperability with Netscape.
.if ${PORT_OPTIONS:MNETSCAPE_BRAIN_DAMAGE}
MAKE_ARGS+=	WITH_NETSCAPE_BRAIN_DAMAGE=yes
.endif
# See src/imapd/Makefile for more information about these three options.

.include <bsd.port.pre.mk>
post-patch:
	@${REINPLACE_CMD} -e "s:/etc/c-client.cf:${PREFIX}/etc/c-client.cf:" \
		${WRKSRC}/docs/imaprc.txt
	@${RM} ${WRKSRC}/docs/imaprc.txt.bak

pre-build:
	@${SH} ${FILESDIR}/pkg-req.rev ${LOCALBASE}/include/c-client/portrevision.h ${PORTVERSION}
.if ${PORT_OPTIONS:MSSL}
	@${SH} ${FILESDIR}/pkg-req.ssl ${LOCALBASE}/include/c-client/portrevision.h yes
.else
	@${SH} ${FILESDIR}/pkg-req.ssl ${LOCALBASE}/include/c-client/portrevision.h no
.endif

do-install:
	${INSTALL_PROGRAM} ${WRKSRC}/imapd/imapd ${STAGEDIR}${PREFIX}/libexec/
	${INSTALL_PROGRAM} ${WRKSRC}/ipopd/ipop2d ${STAGEDIR}${PREFIX}/libexec/
	${INSTALL_PROGRAM} ${WRKSRC}/ipopd/ipop3d ${STAGEDIR}${PREFIX}/libexec/
	${INSTALL} ${COPY} ${STRIP}	\
		${WRKSRC}/mlock/mlock ${STAGEDIR}${PREFIX}/libexec
	${INSTALL_PROGRAM} ${WRKSRC}/mtest/mtest ${STAGEDIR}${PREFIX}/bin/mboxtest
	${INSTALL_MAN} \
		${WRKSRC}/src/imapd/imapd.8 ${STAGEDIR}${PREFIX}/man/man8/imapd.8
	${INSTALL_MAN} \
		${WRKSRC}/src/ipopd/ipopd.8 ${STAGEDIR}${PREFIX}/man/man8/ipopd.8
	${INSTALL_PROGRAM} ${WRKSRC}/dmail/dmail ${STAGEDIR}${PREFIX}/bin/
	${INSTALL_PROGRAM} ${WRKSRC}/mailutil/mailutil ${STAGEDIR}${PREFIX}/bin/
	${INSTALL_PROGRAM} ${WRKSRC}/tmail/tmail ${STAGEDIR}${PREFIX}/bin/
	${INSTALL_MAN} ${WRKSRC}/src/dmail/dmail.1 ${STAGEDIR}${PREFIX}/man/man1/
	${INSTALL_MAN} ${WRKSRC}/src/mailutil/mailutil.1 ${STAGEDIR}${PREFIX}/man/man1/
	${INSTALL_MAN} ${WRKSRC}/src/tmail/tmail.1 ${STAGEDIR}${PREFIX}/man/man1/

post-install-DOCS-on:
	${MKDIR} -m 0755 -p ${STAGEDIR}${DOCSDIR}
	${TAR} -C ${WRKSRC}/docs -cf - . | \
		(umask 022; ${TAR} -C ${STAGEDIR}${DOCSDIR} -xf -)

cert:
	${INSTALL} -d -o root -g wheel -m 0755 ${PREFIX}/certs
	openssl req -new -x509 -days 365 -nodes -config ${FILESDIR}/imap-uw.cnf -out ${PREFIX}/certs/imapd.pem -keyout ${PREFIX}/certs/imapd.pem
	openssl x509 -subject -dates -fingerprint -noout -in ${PREFIX}/certs/imapd.pem
	${CHMOD} 700 ${PREFIX}/certs/imapd.pem
	${LN} -s ${PREFIX}/certs/imapd.pem ${PREFIX}/certs/ipop3d.pem

.include <bsd.port.post.mk>
