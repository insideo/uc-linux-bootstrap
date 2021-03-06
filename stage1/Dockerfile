FROM scratch
MAINTAINER ccondit@randomcoder.com
ADD stage1.tar.xz /
ADD stage1-tools.tar.xz /tools/
ADD stage1-packages.tar /packages/

ENV PATH /usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/tools/bin

# tools setup
RUN [ "/tools/bin/ln", "-sfv", "/tools/bin/bash", "/bin/sh" ]

RUN \
	set +xe && \
	umask 022 && \
	ln -sfv /tools/bin/bash /usr/bin && \
	ln -sfv /tools/bin/cat  /usr/bin && \
	ln -sfv /tools/bin/echo /usr/bin && \
	ln -sfv /tools/bin/pwd /usr/bin && \
	ln -sfv /tools/bin/stty /usr/bin && \
	ln -sfv /tools/bin/perl /usr/bin && \
	ln -sfv /tools/lib/libgcc_s.so /usr/lib && \
	ln -sfv /tools/lib/libgcc_s.so.1 /usr/lib && \
	ln -sfv /tools/lib/libstdc++.so /usr/lib && \
	ln -sfv /tools/lib/libstdc++.so.6 /usr/lib && \
	sed 's/tools/usr/' /tools/lib/libstdc++.la > /usr/lib/libstdc++.la && \
	ln -sfv bash /bin/sh && \
	ln -sfv /tools/lib/ld-linux-x86-64.so.2 /lib

# build env setup
RUN \
	set -xe && \
	umask 022 && \
	mkdir -p /build /download /control

# linux-devel
ADD control/linux-devel /control/linux-devel
RUN \
	set -xe && \
	umask 022 && \
	LC_ALL=POSIX && \
	wget -O /download/linux-4.8.6.tar.xz -c \
		https://www.kernel.org/pub/linux/kernel/v4.x/linux-4.8.6.tar.xz && \
	cd /build && \
	pre-root destroot && \
	mkdir -p destroot/usr/include && \
	tar xf /download/linux-4.8.6.tar.xz && \
	cd linux-4.8.6 && \
	make mrproper && \
	make INSTALL_HDR_PATH=dest headers_install && \
	cp -rv dest/include/* /build/destroot/usr/include && \
	post-root /build/destroot && \
	make-deb \
		/build/linux-devel-deb \
		/build/destroot \
		/control/linux-devel \
		/packages/linux-devel_4.8.6-1_amd64.deb && \
	cd / && \
	rm -rf /build/* /download/* /control/* && \
	dpkg -i /packages/linux-devel_*.deb

# glibc-bootstrap
ADD control/glibc-bootstrap-all /control
RUN \
	set -x && \
	umask 022 && \
	LC_ALL=POSIX && \
	wget -O /download/glibc-2.24.tar.xz \
		http://ftp.gnu.org/gnu/glibc/glibc-2.24.tar.xz && \
	wget -O /download/glibc-2.24-fhs-1.patch \
		http://www.linuxfromscratch.org/patches/lfs/7.10/glibc-2.24-fhs-1.patch && \
	cd /build && \
	tar xf /download/glibc-2.24.tar.xz && \
	cd glibc-2.24 && \
	patch -Np1 -i /download/glibc-2.24-fhs-1.patch && \
	mkdir -p ../glibc-build && \
	cd ../glibc-build && \
	../glibc-2.24/configure \
		--prefix=/usr --disable-profile --enable-kernel=2.6.32\
		--enable-obsolete-rpc && \
	MAKE="make -j4" make && \
	pre-root /build/glibc-root && \
	mkdir -p /build/glibc-root/etc && \
	touch /build/glibc-root/etc/ld.so.conf && \
	make install_root=/build/glibc-root install && \
	rm -f /build/glibc-root/etc/ld.so.conf && \
	mkdir -p /build/glibc-root/usr/lib/locale && \
	I18NPATH=/build/glibc-2.24/localedata \
	locale/localedef --alias-file=../glibc-2.24/intl/locale.alias \
		-i ../glibc-2.24/localedata/locales/en_US -c \
		-f ../glibc-2.24/localedata/charmaps/ISO-8859-1 \
		--prefix=/build/glibc-root en_US && \
	I18NPATH=/build/glibc-2.24/localedata \
	locale/localedef --alias-file=../glibc-2.24/intl/locale.alias \
		-i ../glibc-2.24/localedata/locales/en_US -c \
		-f ../glibc-2.24/localedata/charmaps/UTF-8 \
		--prefix=/build/glibc-root en_US && \
	cd /build/glibc-root && \
	(strip --strip-debug usr/lib/* usr/lib/*/* || true) && \
	(strip --strip-unneeded sbin/* usr/bin/* usr/sbin/* || true) && \
	rm -f etc/ld.so.cache && \
	rm -rf usr/share/locale && \
	rm -rf usr/lib/gconv && \
	rm -rf usr/libexec/getconf && \
	rm -rf usr/libexec && \
	rm -rf usr/lib/locale && \
	rm -rf usr/share/i18n && \
	rm -f etc/rpc usr/sbin/sln && \
	rm -rf var && \
	sed -i 's@/usr/bin/bash@/bin/sh@' usr/bin/ldd && \
	post-root /build/glibc-root && \

	cp -a /build/glibc-root /build/libc-bootstrap-root && \
	cd /build/libc-bootstrap-root && \
	cd usr/bin && \
	find -type f -not -name ldd -delete && \
	cd ../.. && \
	rm -rf etc && \
	cd usr/sbin && \
	find -type f -not -name ldconfig -delete && \
	cd ../.. && \
	rm -rf usr/share && \
	rm -rf usr/include && \
	find . -name "*.a" -delete && \
	find . -name "*.o" -delete && \
	make-deb \
		/build/libc-bootstrap-deb \
		/build/libc-bootstrap-root \
		/control/libc-bootstrap \
		/packages/libc-bootstrap_2.24-1_amd64.deb && \

	cp -a /build/glibc-root /build/glibc-bin-bootstrap-root && \
	cd /build/glibc-bin-bootstrap-root && \
	rm -f usr/bin/ldd && \
	rm -f usr/sbin/ldconfig && \
	rm -rf etc && \
	rm -rf usr/lib && \
	rm -rf usr/share && \
	rm -rf usr/include && \
	find . -name "*.a" -delete && \
	find . -name "*.o" -delete && \
	make-deb \
		/build/glibc-bin-bootstrap-deb \
		/build/glibc-bin-bootstrap-root \
		/control/glibc-bin-bootstrap \
		/packages/glibc-bin-bootstrap_2.24-1_amd64.deb && \

	cp -a /build/glibc-root /build/libc-devel-bootstrap-root && \
	cd /build/libc-devel-bootstrap-root && \
	rm -rf etc usr/bin usr/share usr/sbin && \
	find usr/lib -name "*.so*" -delete && \
	make-deb \
		/build/libc-devel-bootstrap-deb \
		/build/libc-devel-bootstrap-root \
		/control/libc-devel-bootstrap \
		/packages/libc-devel-bootstrap_2.24-1_amd64.deb && \
	cd / && \
	rm -rf /build/* /download/* /control/*

# install new packages and fixup toolchain
RUN \
	set -x && \
	set +h && \
	umask 022 && \
	LC_ALL=POSIX && \
	dpkg --force-depends -i /packages/linux-devel_*.deb && \
	dpkg --force-depends -i /packages/libc-bootstrap_*.deb && \
	/sbin/ldconfig && \
	dpkg --force-depends -i /packages/glibc-bin-bootstrap_*.deb && \
	dpkg --force-depends -i /packages/libc-devel-bootstrap_*.deb && \
	mv -v /tools/bin/{ld,ld-old} && \
	mv -v /tools/$(gcc -dumpmachine)/bin/{ld,ld-old} && \
	mv -v /tools/bin/{ld-new,ld} && \
	ln -sv /tools/bin/ld /tools/$(gcc -dumpmachine)/bin/ld && \
	gcc -dumpspecs | \
		sed 's@/tools@@g' | \
		sed '/\*startfile_prefix_spec:/{n;s@.*@/usr/lib/ @}' | \
		sed '/\*cpp:/{n;s@$@ -isystem /usr/include@}' > \
		`dirname $(gcc --print-libgcc-file-name)`/specs && \
	cd /build && \
	echo 'main(){}' > dummy.c && \
	cc dummy.c -v -Wl,--verbose &> dummy.log && \
	readelf -l a.out | grep ': /lib' && \
	grep -o '/usr/lib.*/crt[1in].*succeeded' dummy.log && \
	grep -B1 '^ /usr/include' dummy.log && \
	grep 'SEARCH.*/usr/lib' dummy.log |sed 's|; |\n|g' && \
	grep "/lib.*/libc.so.6 " dummy.log && \
	grep found dummy.log && \
	rm -v dummy.c a.out dummy.log

# libz-bootstrap
ADD control/libz-bootstrap-all /control
RUN \
	set -x && \
	set +h && \
	umask 022 && \
	LC_ALL=POSIX && \
	wget -O /download/zlib-1.2.8.tar.xz \	
		http://www.zlib.net/zlib-1.2.8.tar.xz && \
	cd /build && \
	tar xf /download/zlib-1.2.8.tar.xz && \
	cd /build/zlib-1.2.8 && \
	CFLAGS='-mstackrealign -fPIC -O3' ./configure --prefix=/usr && \
	MAKE="make -j4" make && \
	pre-root /build/zlib-root && \
	make DESTDIR=/build/zlib-root install && \
	cd /build/zlib-root && \
	rm -rf usr/share && \
	rm -f lib/libz.so && \
	post-root /build/zlib-root && \

	cp -a /build/zlib-root /build/libz-bootstrap-root && \
	cd /build/libz-bootstrap-root && \
	rm -rf usr/include usr/lib/pkgconfig && \
	rm -f usr/lib/*.a && \
	(strip --strip-debug usr/lib/*.so* || true) && \
	make-deb \
		/build/libz-bootstrap-deb \
		/build/libz-bootstrap-root \
		/control/libz-bootstrap \
		/packages/libz-bootstrap_1.2.8-2_amd64.deb && \

	cp -a /build/zlib-root /build/libz-devel-bootstrap-root && \
	cd /build/libz-devel-bootstrap-root && \
	rm -f usr/lib/*.so* && \
	make-deb \
		/build/libz-devel-bootstrap-deb \
		/build/libz-devel-bootstrap-root \
		/control/libz-devel-bootstrap \
		/packages/libz-devel-bootstrap_1.2.8-2_amd64.deb && \
	cd / && \
	rm -rf /build/* /download/* /control/* && \
	dpkg --force-depends -i /packages/libz-bootstrap_*.deb && \
	dpkg --force-depends -i /packages/libz-devel-bootstrap_*.deb

# file-bootstrap
ADD control/file-bootstrap-all /control
RUN \
	set -x && \
	set +h && \
	umask 022 && \
	LC_ALL=POSIX && \
	wget -O /download/file-5.29.tar.gz \
		ftp://ftp.astron.com/pub/file/file-5.29.tar.gz && \
	cd /build && \
	tar xf /download/file-5.29.tar.gz && \
	cd file-5.29 && \
	./configure --prefix=/usr && \
	MAKE="make -j4" make && \
	pre-root /build/file-root && \
	make DESTDIR=/build/file-root install && \
	cd /build/file-root && \
	rm -rf usr/share/man && \
	(strip --strip-unneeded usr/bin/* || true) && \
	(strip --strip-debug usr/lib/*.so* || true) && \
	post-root /build/file-root && \

	cp -a /build/file-root /build/file-bootstrap-root && \
	cd /build/file-bootstrap-root && \
	rm -rf usr/lib usr/include && \
	make-deb \
		/build/file-bootstrap-deb \
		/build/file-bootstrap-root \
		/control/file-bootstrap \
		/packages/file-bootstrap_5.29-1_amd64.deb && \

	cp -a /build/file-root /build/libmagic-bootstrap-root && \
	cd /build/libmagic-bootstrap-root && \
	rm -rf usr/include usr/share usr/bin && \
	rm -f usr/lib/*.la && \
	make-deb \
		/build/libmagic-bootstrap-deb \
		/build/libmagic-bootstrap-root \
		/control/libmagic-bootstrap \
		/packages/libmagic-bootstrap_5.29-1_amd64.deb && \

	cp -a /build/file-root /build/libmagic-devel-bootstrap-root && \
	cd /build/libmagic-devel-bootstrap-root && \
	rm -rf usr/share usr/bin usr/lib/*.so* && \
	make-deb \
		/build/libmagic-devel-bootstrap-deb \
		/build/libmagic-devel-bootstrap-root \
		/control/libmagic-devel-bootstrap \
		/packages/libmagic-devel-bootstrap_5.29-1_amd64.deb && \

	cd / && \
	rm -rf /build/* /download/* /control/* && \
	dpkg --force-depends -i /packages/file-bootstrap_*.deb

# binutils-bootstrap
ADD control/binutils-bootstrap-all /control
RUN \
	set -x && \
	set +h && \
	umask 022 && \
	LC_ALL=POSIX && \
	TARGET_PLATFORM=$(uname -m)-uc-linux-gnu && \
	wget -O /download/binutils-2.27.tar.bz2 \
		http://ftp.gnu.org/gnu/binutils/binutils-2.27.tar.bz2 && \
	cd /build && \
	tar xf /download/binutils-2.27.tar.bz2 && \
	mkdir binutils-build && \
	cd binutils-build && \
	CC=gcc CXX=g++ AR=ar RANLIB=ranlib LD=ld STRIP=strip \
	../binutils-2.27/configure \
		--prefix=/usr --enable-shared --disable-werror \
		--target="${TARGET_PLATFORM}" --host="${TARGET_PLATFORM}" \
		--disable-multilib --with-build-sysroot=/ && \
	MAKE="make -j4" make tooldir=/usr && \
	pre-root /build/binutils-root && \
	make DESTDIR=/build/binutils-root tooldir=/usr install && \
	cd /build/binutils-root && \
	rm -rf usr/share && \
	(strip --strip-debug usr/lib/* || true) && \
	(strip --strip-unneeded usr/bin/* || true) && \
	post-root /build/binutils-root && \

	cp -a /build/binutils-root /build/binutils-bootstrap-root && \
	cd /build/binutils-bootstrap-root && \
	rm -rf usr/include usr/lib/*.a usr/lib/*.la && \
	make-deb \
		/build/binutils-bootstrap-deb \
		/build/binutils-bootstrap-root \
		/control/binutils-bootstrap \
		/packages/binutils-bootstrap_2.27.0-1_amd64.deb && \

	cp -a /build/binutils-root /build/binutils-devel-bootstrap-root && \
	cd /build/binutils-devel-bootstrap-root && \
	rm -rf usr/bin usr/lib/ldscripts usr/lib/*.so* && \
	make-deb \
		/build/binutils-devel-bootstrap-deb \
		/build/binutils-devel-bootstrap-root \
		/control/binutils-devel-bootstrap \
		/packages/binutils-devel-bootstrap_2.27.0-1_amd64.deb && \

	cd / && \
	rm -rf /build/* /download/* /control/* && \
	dpkg --force-depends -i /packages/binutils-bootstrap_*.deb && \
	dpkg --force-depends -i /packages/libmagic-bootstrap_*.deb

# m4-bootstrap
ADD control/m4-bootstrap /control/m4-bootstrap
RUN \
	set -x && \
	set +h && \
	umask 022 && \
	LC_ALL=POSIX && \
	wget -O /download/m4-1.4.17.tar.xz \
		http://ftp.gnu.org/gnu/m4/m4-1.4.17.tar.xz && \
	cd /build && \
	tar xf /download/m4-1.4.17.tar.xz && \
	cd m4-1.4.17 && \
	./configure --prefix=/usr && \
	MAKE="make -j4" make && \
	pre-root /build/m4-bootstrap-root && \
	make DESTDIR=/build/m4-bootstrap-root install && \
	cd /build/m4-bootstrap-root && \
	rm -rf usr/share && \
	(strip --strip-unneeded usr/bin/* || true) && \
	post-root /build/m4-bootstrap-root && \
	make-deb \
		/build/m4-bootstrap-deb \
		/build/m4-bootstrap-root \
		/control/m4-bootstrap \
		/packages/m4-bootstrap_1.4.17-1_amd64.deb && \
	cd / && \
	rm -rf /build/* /download/* /control/* && \
	dpkg --force-depends -i /packages/m4-bootstrap_*.deb

# libgmp-bootstrap
ADD control/libgmp-bootstrap-all /control
RUN \
	set -x && \
	set +h && \
	umask 022 && \
	LC_ALL=POSIX && \
	wget -O /download/gmp-6.1.1.tar.xz \
		http://ftp.gnu.org/gnu/gmp/gmp-6.1.1.tar.xz && \
	cd /build && \
	tar xf /download/gmp-6.1.1.tar.xz && \
	cd gmp-6.1.1 && \
	./configure --prefix=/usr --enable-cxx --disable-static && \
	MAKE="make -j4" make && \
	pre-root /build/gmp-root && \
	make DESTDIR=/build/gmp-root install && \
	cd /build/gmp-root && \
	rm -rf usr/share && \
	(strip --strip-debug usr/lib/* || true) && \
	post-root /build/gmp-root && \

	cp -a /build/gmp-root /build/libgmp-bootstrap-root && \
	cd /build/libgmp-bootstrap-root && \
	rm -rf usr/include && \
	rm -rf usr/lib/*.la && \
	make-deb \
		/build/libgmp-bootstrap-deb \
		/build/libgmp-bootstrap-root \
		/control/libgmp-bootstrap \
		/packages/libgmp-bootstrap_6.1.1-1_amd64.deb && \

	cp -a /build/gmp-root /build/libgmp-devel-bootstrap-root && \
	cd /build/libgmp-devel-bootstrap-root && \
	rm -f usr/lib/*.so* && \
	make-deb \
		/build/libgmp-devel-bootstrap-deb \
		/build/libgmp-devel-bootstrap-root \
		/control/libgmp-devel-bootstrap \
		/packages/libgmp-devel-bootstrap_6.1.1-1_amd64.deb && \

	cd / && \
	rm -rf /build/* /download/* /control/* && \
	dpkg --force-depends -i /packages/libgmp-bootstrap_*.deb && \
	dpkg --force-depends -i /packages/libgmp-devel-bootstrap_*.deb

# patch-bootstrap
ADD control/patch-bootstrap /control/patch-bootstrap
RUN \
	set -x && \
	set +h && \
	umask 022 && \
	LC_ALL=POSIX && \
	wget -O /download/patch-2.7.5.tar.xz \
		http://ftp.gnu.org/gnu/patch/patch-2.7.5.tar.xz && \
	cd /build && \
	tar xf /download/patch-2.7.5.tar.xz && \
	cd patch-2.7.5 && \
	./configure --prefix=/usr && \
	MAKE="make -j4" make && \
	pre-root /build/patch-bootstrap-root && \
	make DESTDIR=/build/patch-bootstrap-root install && \
	cd /build/patch-bootstrap-root && \
	rm -rf usr/share && \
	(strip --strip-unneeded usr/bin/* || true) && \
	post-root /build/patch-bootstrap-root && \
	make-deb \
		/build/patch-bootstrap-deb \
		/build/patch-bootstrap-root \
		/control/patch-bootstrap \
		/packages/patch-bootstrap_2.7.5-1_amd64.deb && \
	cd / && \
        rm -rf /build/* /download/* /control/* && \
	dpkg --force-depends -i /packages/patch-bootstrap_*.deb

# libmpfr-bootstrap
ADD control/libmpfr-bootstrap-all /control
RUN \
	set -x && \
	set +h && \
	umask 022 && \
	LC_ALL=POSIX && \
	wget -O /download/mpfr-3.1.4.tar.xz \
		http://www.mpfr.org/mpfr-3.1.4/mpfr-3.1.4.tar.xz && \
	cd /build && \
	tar xf /download/mpfr-3.1.4.tar.xz && \
	cd mpfr-3.1.4 && \
	./configure --prefix=/usr --disable-static --enable-thread-safe && \
	MAKE="make -j4" make && \
	pre-root /build/mpfr-root && \
	make DESTDIR=/build/mpfr-root install && \
	cd /build/mpfr-root && \
	rm -rf usr/share && \
	(strip --strip-debug usr/lib/* || true) && \
	post-root /build/mpfr-root && \

	cp -a /build/mpfr-root /build/libmpfr-bootstrap-root && \
	cd /build/libmpfr-bootstrap-root && \
	rm -rf usr/include && \
	rm -f usr/lib/*.la && \
	make-deb \
		/build/libmpfr-bootstrap-deb \
		/build/libmpfr-bootstrap-root \
		/control/libmpfr-bootstrap \
		/packages/libmpfr-bootstrap_3.1.4-1_amd64.deb && \

	cp -a /build/mpfr-root /build/libmpfr-devel-bootstrap-root && \
	cd /build/libmpfr-devel-bootstrap-root && \
	rm -f usr/lib/*.so* && \
	make-deb \
		/build/libmpfr-devel-bootstrap-deb \
		/build/libmpfr-devel-bootstrap-root \
		/control/libmpfr-devel-bootstrap \
		/packages/libmpfr-devel-bootstrap_3.1.4-1_amd64.deb && \

	cd / && \
	rm -rf /build/* /download/* /control/* && \
	dpkg --force-depends -i /packages/libmpfr-bootstrap_*.deb && \
	dpkg --force-depends -i /packages/libmpfr-devel-bootstrap_*.deb

# libmpc-bootstrap
ADD control/libmpc-bootstrap-all /control
RUN \
	set -x && \
	set +h && \
	umask 022 && \
	LC_ALL=POSIX && \
	wget -O /download/mpc-1.0.3.tar.gz \
		http://www.multiprecision.org/mpc/download/mpc-1.0.3.tar.gz && \
	cd /build && \
	tar xf /download/mpc-1.0.3.tar.gz && \
	cd mpc-1.0.3 && \
	./configure --prefix=/usr --disable-static && \
	MAKE="make -j4" make && \
	pre-root /build/mpc-root && \
	make DESTDIR=/build/mpc-root install && \
        cd /build/mpc-root && \
	rm -rf usr/share && \
	(strip --strip-debug usr/lib/* || true) && \
	post-root /build/mpc-root && \

	cp -a /build/mpc-root /build/libmpc-bootstrap-root && \
	cd /build/libmpc-bootstrap-root && \
	rm -rf usr/include && \
	rm -f usr/lib/*.la && \
	make-deb \
		/build/libmpc-bootstrap-deb \
		/build/libmpc-bootstrap-root \
		/control/libmpc-bootstrap \
		/packages/libmpc-bootstrap_1.0.3-2_amd64.deb && \

	cp -a /build/mpc-root /build/libmpc-devel-bootstrap-root && \
	cd /build/libmpc-devel-bootstrap-root && \
	rm -f usr/lib/*.so* && \
	make-deb \
		/build/libmpc-devel-bootstrap-deb \
		/build/libmpc-devel-bootstrap-root \
		/control/libmpc-devel-bootstrap \
		/packages/libmpc-devel-bootstrap_1.0.3-2_amd64.deb && \

	cd / && \
        rm -rf /build/* /download/* /control/* && \
	dpkg --force-depends -i /packages/libmpc-bootstrap_*.deb && \
	dpkg --force-depends -i /packages/libmpc-devel-bootstrap_*.deb

# gcc-bootstrap
ADD control/gcc-bootstrap-all /control
RUN \
	set -x && \
	set +h && \
	umask 022 && \
	LC_ALL=POSIX && \
	TARGET_PLATFORM=$(uname -m)-uc-linux-gnu && \
	wget -O /download/gcc-6.2.0.tar.bz2 \
		http://ftp.gnu.org/gnu/gcc/gcc-6.2.0/gcc-6.2.0.tar.bz2 && \
	cd /build && \
	tar xf /download/gcc-6.2.0.tar.bz2 && \
	mkdir gcc-build && \
	cd gcc-build && \
	SED=sed CC=gcc CXX=g++ AR=ar RANLIB=ranlib LD=ld STRIP=strip \
	../gcc-6.2.0/configure \
		--prefix=/usr --enable-languages=c,c++ --disable-multilib \
		--disable-bootstrap --with-system-zlib \
		--host="${TARGET_PLATFORM}" --target="${TARGET_PLATFORM}" \
		--with-build-sysroot=/ && \
	MAKE="make -j4" make && \
	pre-root /build/gcc-bootstrap-root && \
	make DESTDIR=/build/gcc-bootstrap-root install-strip && \
	post-root /build/gcc-bootstrap-root && \
	cd /build/gcc-bootstrap-root && \
	rm -f usr/bin/${TARGET_PLATFORM}-* && \
	for tool in c++ g++ gcc gcc-ar gcc-nm gcc-ranlib ; do ln -sf $tool usr/bin/${TARGET_PLATFORM}-$tool ; done && \
	ln -sf gcc usr/bin/${TARGET_PLATFORM}-gcc-6.2.0 && \
	ln -sv ../bin/cpp usr/lib/cpp && \
	ln -sv gcc usr/bin/cc && \
	mkdir -p usr/lib/bfd-plugins && \
	ln -sfv ../../libexec/gcc/${TARGET_PLATFORM}/6.2.0/liblto_plugin.so usr/lib/bfd-plugins/ && \
	mkdir -pv usr/share/gdb/auto-load/usr/lib && \
	mv -v usr/lib/*gdb.py usr/share/gdb/auto-load/usr/lib && \
	rm -f usr/lib/libsupc++* && \
	rm -rf usr/share/info usr/share/locale usr/share/man && \
	for pkg in libasan libatomic libcilkrts libgcc libgomp libitm \
		liblsan libquadmath libssp libtsan libubsan ; do ( \
		mkdir -p /build/${pkg}-bootstrap-root/usr/lib && \
		cp -a usr/lib/${pkg}*.so* /build/${pkg}-bootstrap-root/usr/lib/ && \
		rm -f usr/lib/${pkg}*.so* && \
		make-deb \
			/build/${pkg}-bootstrap-deb \
			/build/${pkg}-bootstrap-root \
			/control/${pkg}-bootstrap \
			/packages/${pkg}-bootstrap_6.2.0-1_amd64.deb) ; done && \

	mkdir -p /build/libstdcxx-bootstrap-root/usr/lib && \
	cp -a usr/lib/libstdc++*.so* /build/libstdcxx-bootstrap-root/usr/lib/ && \
	rm -f usr/lib/libstdc++*.so* && \
	make-deb \
		/build/libstdcxx-bootstrap-deb \
		/build/libstdcxx-bootstrap-root \
		/control/libstdcxx-bootstrap \
		/packages/libstdcxx-bootstrap_6.2.0-1_amd64.deb && \

	make-deb \
		/build/gcc-bootstrap-deb \
		/build/gcc-bootstrap-root \
		/control/gcc-bootstrap \
		/packages/gcc-bootstrap_6.2.0-1_amd64.deb && \

	cd / && \
	rm -rf /build/* /download/* /control/* && \
	for pkg in libasan libatomic libcilkrts libgcc libgomp libitm \
		liblsan libquadmath libssp libstdcxx libtsan libubsan gcc ; do \
		dpkg --force-depends -i /packages/${pkg}-bootstrap_*.deb ; done && \
	echo 'int main(){}' > dummy.c && \
	cc dummy.c -v -Wl,--verbose &> dummy.log && \
	readelf -l a.out | grep ': /lib' && \
	grep -o '/usr/lib.*/crt[1in].*succeeded' dummy.log && \
	grep -B4 '^ /usr/include' dummy.log && \
	grep 'SEARCH.*/usr/lib' dummy.log |sed 's|; |\n|g' && \
	grep "/lib.*/libc.so.6 " dummy.log && \
	grep found dummy.log && \
	rm -v dummy.c a.out dummy.log

# busybox-bootstrap
ADD control/busybox-bootstrap control/busybox-bootstrap
RUN \
	set -x && \
	set +h && \
	umask 022 && \
	LC_ALL=POSIX && \
        wget -O /download/busybox-1.25.1.tar.bz2 \
		http://www.busybox.net/downloads/busybox-1.25.1.tar.bz2 && \
	cd /build && \
	tar xf /download/busybox-1.25.1.tar.bz2 && \
	cd busybox-1.25.1 && \
	confs=' \
		CONFIG_INSTALL_NO_USR \
		CONFIG_AR \
		CONFIG_FEATURE_AR_LONG_FILENAMES \
		CONFIG_FEATURE_AR_CREATE \
		CONFIG_DPKG \
		CONFIG_DPKG_DEB' && \
	make defconfig && \
	for conf in $confs; do \
		sed -i "s!^# $conf is not set\$!$conf=y!" .config; \
		grep -q "^$conf=y" .config || echo "$conf=y" >> .config; \
	done && \
	make oldconfig && \
	for conf in $confs; do grep -q "^$conf=y" .config; done && \
	MAKE="make -j4" make && \
	make install && \
	rm -f _install/linuxrc && \
	mkdir -p /build/busybox-bootstrap-root/usr && \
	cp -a _install/* /build/busybox-bootstrap-root/usr/ && \
	cd /build/busybox-bootstrap-root && \
	chmod 4755 /build/busybox-bootstrap-root/usr/bin/busybox && \
	make-deb \
		/build/busybox-bootstrap-deb \
		/build/busybox-bootstrap-root \
		/control/busybox-bootstrap \
		/packages/busybox-bootstrap_1.25.1-1_amd64.deb && \

	cd / && \
	rm -rf /build/* /download/* /control/* && \
	dpkg --force-depends -i /packages/busybox-bootstrap_*.deb && \
	dpkg --force-depends -i /packages/patch-bootstrap_*.deb && \
	dpkg --force-depends -i /packages/binutils-bootstrap_*.deb && \
	ln -sf bash /bin/sh

# openssl-bootstrap
ADD control/openssl-bootstrap-all /control
RUN \
	set -x && \
	set +h && \
	umask 022 && \
	LC_ALL=POSIX && \
	wget -O /download/openssl-1.0.2j.tar.gz \
		https://www.openssl.org/source/openssl-1.0.2j.tar.gz && \
	cd /build && \
	tar xf /download/openssl-1.0.2j.tar.gz && \
	cd openssl-1.0.2j && \
	./config --prefix=/usr --openssldir=/etc/ssl --libdir=lib shared zlib-dynamic && \
	sed -i 's# libcrypto.a##;s# libssl.a##' Makefile && \
	make && \
	pre-root /build/openssl-root && \
	make INSTALL_PREFIX=/build/openssl-root install && \
	cd /build/openssl-root && \
	rm -rf etc/ssl/man && \
	(strip --strip-debug usr/lib/* usr/lib/*/* || true) && \
	(strip --strip-unneeded usr/bin/* || true) && \
	post-root /build/openssl-root && \

	cp -a /build/openssl-root /build/libssl-bootstrap-root && \
	cd /build/libssl-bootstrap-root && \
	rm -rf etc usr/bin usr/include usr/lib/pkgconfig && \
	make-deb \
		/build/libssl-bootstrap-deb \
		/build/libssl-bootstrap-root \
		/control/libssl-bootstrap \
		/packages/libssl-bootstrap_1.0.2j-1_amd64.deb && \

	cp -a /build/openssl-root /build/libssl-devel-bootstrap-root && \
	cd /build/libssl-devel-bootstrap-root && \
	rm -rf etc usr/bin usr/lib/engines && \
	rm -f usr/lib/lib*  && \
	make-deb \
		/build/libssl-devel-bootstrap-deb \
		/build/libssl-devel-bootstrap-root \
		/control/libssl-devel-bootstrap \
		/packages/libssl-devel-bootstrap_1.0.2j-1_amd64.deb && \

	cp -a /build/openssl-root /build/openssl-bootstrap-root && \
	cd /build/openssl-bootstrap-root && \
	rm -rf usr/include usr/lib && \
	make-deb \
		/build/openssl-bootstrap-deb \
		/build/openssl-bootstrap-root \
		/control/openssl-bootstrap \
		/packages/openssl-bootstrap_1.0.2j-1_amd64.deb && \

	cd / && \
	rm -rf /build/* /download/* /control/* && \
	dpkg --force-depends -i /packages/libssl-bootstrap_*.deb && \
	dpkg --force-depends -i /packages/libssl-devel-bootstrap_*.deb && \
	dpkg --force-depends -i /packages/openssl-bootstrap_*.deb

# final prep for stage2
RUN \
	set -x && \
	set +h && \
	umask 022 && \
	LC_ALL=POSIX && \
	mkdir -p /stage2 && \
	cd /stage2 && \
	dpkg-deb -x /packages/base-files_*.deb . && \
	dpkg-deb -x /packages/libc-bootstrap_*.deb . && \
	dpkg-deb -x /packages/libssl-bootstrap_*.deb . && \
	dpkg-deb -x /packages/openssl-bootstrap_*.deb . && \
	dpkg-deb -x /packages/busybox-bootstrap_*.deb . && \
	mkdir packages && \
	cp \
		/packages/base-files_*.deb \
		/packages/libc-bootstrap_*.deb \
		/packages/libssl-bootstrap_*.deb \
		/packages/openssl-bootstrap_*.deb \
		/packages/busybox-bootstrap_*.deb \
		packages && \
	chroot /stage2 /usr/bin/dpkg --force-depends -i /packages/base-files_*.deb && \
	chroot /stage2 /usr/bin/dpkg --force-depends -i /packages/libc-bootstrap_*.deb && \
	chroot /stage2 /sbin/ldconfig && \
	chroot /stage2 /usr/bin/dpkg --force-depends -i /packages/libssl-bootstrap_*.deb && \
	chroot /stage2 /usr/bin/dpkg --force-depends -i /packages/openssl-bootstrap_*.deb && \
	chroot /stage2 /usr/bin/dpkg --force-depends -i /packages/busybox-bootstrap_*.deb && \
	cd / && \
	rm -rf /stage2/packages /packages/base-files_*.deb
	
CMD ["/bin/sh"]
