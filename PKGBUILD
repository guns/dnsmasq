# Maintainer: Sung Pae <self@sungpae.com>
pkgname=dnsmasq-nerv
pkgver=0
pkgrel=1
pkgdesc="Custom dnsmasq build"
arch=('x86_64')
url="https://github.com/guns/dnsmasq"
license=('GPL')
groups=('nerv')
backup=('etc/dnsmasq.conf'
        'etc/dnsmasq.resolv.conf')
depends=('glibc' 'libidn' 'libidn2' 'libnetfilter_conntrack' 'nettle' 'gmp')
install=dnsmasq.install
makedepends=('git')
provides=('dnsmasq')
conflicts=('dnsmasq')
replaces=('dnsmasq-guns')

envmake() {
    DESTDIR="$pkgdir" PREFIX=/usr BINDIR=/usr/bin RCDIR= \
    make -e -j "$(nproc)" "$@"
}

pkgver() {
    printf %s "$(git describe --long | tr - .)"
}

build() {
    cd "$startdir"
    envmake all-i18n
}

package() {
    cd "$startdir"
    envmake install
}
