COMMENT =	non-monolithic Wayland compositor

V =		0.4.1

DIST_TUPLE +=	codeberg river river v${V} .
DIST_TUPLE +=	codeberg ifreund zig-pixman v0.3.0 .
DIST_TUPLE +=	codeberg ifreund zig-wayland v0.5.0 .
DIST_TUPLE +=	codeberg ifreund zig-wlroots v0.19.4 .
DIST_TUPLE +=	codeberg ifreund zig-xkbcommon v0.4.0 .

CATEGORIES =	wayland

HOMEPAGE =	https://isaacfreund.com/software/river/

MAINTAINER =	Rob Lillack <rob@lillack.net>

# GPLv3
PERMIT_PACKAGE =	Yes

WRKDIST =	${WRKDIR}/river

MODULES =	wayland/wlroots

COMPILER =	base-clang
COMPILER_LANGS = c

BUILD_DEPENDS =	lang/zig \
		textproc/scdoc \
		wayland/wayland-protocols

LIB_DEPENDS =	wayland/libevdev-openbsd \
		wayland/libinput-openbsd \
		wayland/wayland \
		x11/xkbcommon

WANTLIB +=	c evdev input pixman-1 wayland-server xkbcommon

# Hashes of the Zig package dependencies in build.zig.zon; these become the
# directory names under the Zig global cache. Update whenever the deps in
# build.zig.zon are bumped.
PIXMAN_HASH =		pixman-0.3.0-LClMnz2VAAAs7QSCGwLimV5VUYx0JFnX5xWU6HwtMuDX
WAYLAND_HASH =		wayland-0.5.0-lQa1knz8AQCh08NA8BeQrwJB9U3CfqcVAdHZYGRKIGuu
WLROOTS_HASH =		wlroots-0.19.4-jmOlcqQMBABhKYH6NMSnoK1sohTbhc97_JP-hGg2UZaK
XKBCOMMON_HASH =	xkbcommon-0.4.0-VDqIe0i2AgDRsok2GpMFYJ8SVhQS10_PI2M_CnHXsJJZ

ZIG_CACHE =	${WRKBUILD}/zig-global-cache

MAKE_ENV +=	ZIG_GLOBAL_CACHE_DIR=${ZIG_CACHE} \
		ZIG_LOCAL_CACHE_DIR=${WRKBUILD}/zig-local-cache

ZIG_FLAGS =	--global-cache-dir ${ZIG_CACHE} \
		--cache-dir ${WRKBUILD}/zig-local-cache \
		-Doptimize=ReleaseSafe \
		-Dpie=true \
		-Dxwayland=true \
		-Dman-pages=true \
		-Dversion-string=${V}

# Place each Zig dependency tarball under the global cache where the Zig build
# system expects it, keyed by its content hash. This avoids any network access
# during the build.
post-extract:
	mkdir -p ${ZIG_CACHE}/p
	mv ${WRKDIR}/zig-pixman    ${ZIG_CACHE}/p/${PIXMAN_HASH}
	mv ${WRKDIR}/zig-wayland   ${ZIG_CACHE}/p/${WAYLAND_HASH}
	mv ${WRKDIR}/zig-wlroots   ${ZIG_CACHE}/p/${WLROOTS_HASH}
	mv ${WRKDIR}/zig-xkbcommon ${ZIG_CACHE}/p/${XKBCOMMON_HASH}

# Build into a staging prefix so the install step has somewhere writable.
# The fake-install phase re-invokes the install step with the real prefix.
do-build:
	cd ${WRKSRC} && ulimit -s 16384 && exec ${SETENV} ${MAKE_ENV} \
		zig build ${ZIG_FLAGS} --prefix ${WRKBUILD}/stage

do-install:
	cd ${WRKSRC} && ${SETENV} ${MAKE_ENV} \
		zig build install ${ZIG_FLAGS} --prefix ${PREFIX}
	# build.zig follows the Linux man path convention; OpenBSD uses
	# ${PREFIX}/man instead of ${PREFIX}/share/man.
	mkdir -p ${PREFIX}/man/man1
	mv ${PREFIX}/share/man/man1/river.1 ${PREFIX}/man/man1/river.1
	rmdir ${PREFIX}/share/man/man1 ${PREFIX}/share/man
	# The pkg-config file gets the staged WRKINST prefix baked in by
	# build.zig; rewrite it to the real install prefix.
	sed -i "s,^prefix=.*,prefix=${TRUEPREFIX}," \
		${PREFIX}/share/pkgconfig/river-protocols.pc

.include <bsd.port.mk>
