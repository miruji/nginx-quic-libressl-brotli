_pkgbase=nginx
_commit=e24f7ccc161f1a2a759eb27263ec9af4fc7c8e96
_libressl_ver=4.0.0
pkgbase=nginx-quic-libressl-brotli
pkgname=($pkgbase $pkgbase-src)
pkgver=1.27.2
pkgrel=1
pkgdesc='Lightweight HTTP server and IMAP/POP3 proxy server'
arch=('i686' 'x86_64')
url='https://nginx.org'
license=('BSD-2-Clause-Views AND OpenSSL')
depends=('geoip' 'libxcrypt' 'pcre2' 'zlib' 'glibc' 'mailcap')
makedepends=('cmake' 'git' 'go' 'mercurial')
backup=('etc/nginx/fastcgi.conf'
        'etc/nginx/fastcgi_params'
        'etc/nginx/koi-win'
        'etc/nginx/koi-utf'
        'etc/nginx/nginx.conf'
        'etc/nginx/scgi_params'
        'etc/nginx/uwsgi_params'
        'etc/nginx/win-utf'
        'etc/logrotate.d/nginx')
install=nginx.install
provides=('nginx' 'nginx-mainline')
conflicts=('nginx')
source=("git+https://github.com/nginx/nginx.git#commit=${_commit}"
#       "hg+https://hg.nginx.org/nginx#revision=$_commit"
        "https://cdn.openbsd.org/pub/OpenBSD/LibreSSL/libressl-${_libressl_ver}.tar.gz"
        "service"
        "logrotate")
sha512sums=('SKIP' 'SKIP' 'SKIP' 'SKIP')

_common_flags=(
  --with-compat
  --with-debug
  --with-file-aio
  --with-http_addition_module
  --with-http_auth_request_module
  --with-http_dav_module
  --with-http_degradation_module
  --with-http_flv_module
  --with-http_geoip_module
  --with-http_gunzip_module
  --with-http_gzip_static_module
  --with-http_mp4_module
  --with-http_realip_module
  --with-http_secure_link_module
  --with-http_slice_module
  --with-http_ssl_module
  --with-http_stub_status_module
  --with-http_sub_module
  --with-http_v2_module
  --with-mail
  --with-mail_ssl_module
  --with-pcre-jit
  --with-stream
  --with-stream_geoip_module
  --with-stream_realip_module
  --with-stream_ssl_module
  --with-stream_ssl_preread_module
  --with-threads
)

_mainline_flags=(
)

_quic_flags=(
  --with-http_v3_module
#  --with-stream_quic_module
)

prepare() {
  # Backup pristine version of nginx source for -src package
  test -d ${srcdir}/${pkgname}-src && rm -r ${srcdir}/${pkgname}-src
  cp -r ${srcdir}/nginx ${srcdir}/nginx-src
}

pkgver() {
  cd nginx
  #git describe --long --tags | sed -e 's/release-\([^-]*\)-.*/\1/'
  echo "1.27.2"
}

build() {
  # Clear -D_FORTIFY_SOURCE from C++ build flags, it causes Boringssl tests to fail to compile
  export CPPFLAGS=${CPPFLAGS/-D_FORTIFY_SOURCE=[1-9]/-D_FORTIFY_SOURCE=0}
  export CXXFLAGS=${CXXFLAGS/-D_FORTIFY_SOURCE=[1-9]/-D_FORTIFY_SOURCE=0}

  export CXXFLAGS="$CXXFLAGS -fPIC"
  export CFLAGS="$CFLAGS -fPIC"
  export LDFLAGS="$LDFLAGS"

  if [[ $CC == "clang" ]];then
    _cc_opt="-flto $CPPFLAGS $CFLAGS"
    _ld_opt="-flto -fuse-ld=lld $LDFLAGS"
  else
    # Disable some warnings that make Boringssl fail to compile due to a forced -Werror in CMakeLists.txt
    # -Wno-array-bounds: 2022-05-21 for compatiblity with GCC 12.1 (https://bugs.chromium.org/p/boringssl/issues/detail?id=492&sort=-modified)
    export CFLAGS="$CFLAGS -Wno-stringop-overflow -Wno-array-parameter -Wno-array-bounds"
    _cc_opt="$CPPFLAGS $CFLAGS"
    _ld_opt="$LDFLAGS"
  fi

#  cd ${srcdir}/boringssl
#  mkdir build && cd build && cmake -DCMAKE_BUILD_TYPE=Release ../ && make crypto ssl
#  cd ${srcdir}/boringssl
#  mkdir -p .openssl/lib && cd .openssl && ln -s ../include . && cd ../
#  cp ${srcdir}/boringssl/build/crypto/libcrypto.a ${srcdir}/boringssl/build/ssl/libssl.a .openssl/lib && cd ..

  cd ${srcdir}/nginx
  ./auto/configure \
    --prefix=/etc/nginx \
    --conf-path=/etc/nginx/nginx.conf \
    --sbin-path=/usr/bin/nginx \
    --pid-path=/run/nginx.pid \
    --lock-path=/run/lock/nginx.lock \
    --user=http \
    --group=http \
    --http-log-path=/var/log/nginx/access.log \
    --error-log-path=stderr \
    --http-client-body-temp-path=/var/lib/nginx/client-body \
    --http-proxy-temp-path=/var/lib/nginx/proxy \
    --http-fastcgi-temp-path=/var/lib/nginx/fastcgi \
    --http-scgi-temp-path=/var/lib/nginx/scgi \
    --http-uwsgi-temp-path=/var/lib/nginx/uwsgi \
    --with-openssl=${srcdir}/libressl-${_libressl_ver} \
    --with-cc-opt="${_cc_opt}" \
    --with-ld-opt="${_ld_opt}" \
    ${_common_flags[@]} \
    ${_mainline_flags[@]} \
    ${_quic_flags[@]}

  #touch ${srcdir}/boringssl/.openssl/include/openssl/ssl.h
  make
}

package_nginx-quic-libressl-brotli() {
  cd nginx
  make DESTDIR="$pkgdir" install

  sed -e 's|\<user\s\+\w\+;|user http;|g' \
    -e '44s|html|/usr/share/nginx/html|' \
    -e '54s|html|/usr/share/nginx/html|' \
    -i "$pkgdir"/etc/nginx/nginx.conf

  rm "$pkgdir"/etc/nginx/*.default
  rm "$pkgdir"/etc/nginx/mime.types  # in mailcap

  install -d "$pkgdir"/var/lib/nginx
  install -dm700 "$pkgdir"/var/lib/nginx/proxy

  chmod 755 "$pkgdir"/var/log/nginx
  chown root:root "$pkgdir"/var/log/nginx

  install -d "$pkgdir"/usr/share/nginx
  mv "$pkgdir"/etc/nginx/html/ "$pkgdir"/usr/share/nginx

  install -Dm644 ../logrotate "$pkgdir"/etc/logrotate.d/nginx
  install -Dm644 ../service "$pkgdir"/usr/lib/systemd/system/nginx.service
  install -Dm644 LICENSE "$pkgdir"/usr/share/licenses/$provides/LICENSE
  install -d "$pkgdir"/usr/share/licenses/$pkgname
  ln -s /usr/share/licenses/$provides/LICENSE "$pkgdir"/usr/share/licenses/$pkgname/LICENSE

  rmdir "$pkgdir"/run

  install -d "$pkgdir"/usr/share/man/man8/
  gzip -9c docs/man/nginx.8 > "$pkgdir"/usr/share/man/man8/nginx.8.gz

  for i in ftdetect indent syntax; do
    install -Dm644 contrib/vim/${i}/nginx.vim \
      "${pkgdir}/usr/share/vim/vimfiles/${i}/nginx.vim"
  done
  install -Dm644 "$srcdir"/libressl-${_libressl_ver}/COPYING "$pkgdir"/usr/share/licenses/$pkgname/LICENSE-LIBRESSL

}

package_nginx-quic-libressl-brotli-src() {
  pkgdesc="Source code of nginx-quic $pkgver, useful for building modules"
  arch=('any')
  provides=('nginx-src' 'nginx-mainline-src')
  conflicts=($_pkgbase-src)
  depends=("sh" "bash" "perl")
  backup=()
  install -d "$pkgdir/usr/src"
  test -d "$pkgdir/usr/src/nginx" && rm -r "$pkgdir/usr/src/nginx"
  cp -r ${srcdir}/nginx-src "$pkgdir/usr/src/nginx"
  # Delete the .hg directory, it is huge and not needed
  #rm -r ${pkgdir}/usr/src/nginx/{.hg,.hgtags}
  #rm -r ${pkgdir}/usr/src/nginx/.hg
  # Link the 'configure' script to its location in release tarballs,
  # as this is where modules expect it
  ln -s /usr/src/nginx/auto/configure "$pkgdir/usr/src/nginx"
  cd nginx-src
  install -Dm644 LICENSE "$pkgdir"/usr/share/licenses/$provides/LICENSE
  install -d "$pkgdir"/usr/share/licenses/$pkgname
  ln -s /usr/share/licenses/$provides/LICENSE "$pkgdir"/usr/share/licenses/$pkgname/LICENSE
  install -Dm644 "$srcdir"/libressl-${_libressl_ver}/COPYING "$pkgdir"/usr/share/licenses/$pkgname/LICENSE-LIBRESSL
}

# vim:set ts=2 sw=2 et:
