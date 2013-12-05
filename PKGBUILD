# Maintainer: Sung Pae <self@sungpae.com>
pkgname=dnsmasq-guns
pkgver=v2.68rc5.63.gf2588a1
pkgrel=1
pkgdesc="Sung Pae's dnsmasq build"
arch=('x86_64')
url="https://github.com/guns/dnsmasq"
license=('GPL')
groups=('guns')
backup=('etc/dnsmasq/Rakefile'
        'etc/dnsmasq/dnsmasq.conf'
        'etc/dnsmasq/hosts'
        'etc/dnsmasq/resolv.conf')
depends=('glibc' 'libidn')
install=dnsmasq.install
makedepends=('git')
provides=('dnsmasq')
conflicts=('dnsmasq')

envmake() {
    DESTDIR="$pkgdir/" PREFIX=/usr BINDIR=/usr/bin RCDIR= \
    make -e -j $(grep -c ^processor /proc/cpuinfo) "$@"
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
