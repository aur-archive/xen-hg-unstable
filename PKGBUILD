# Maintainer: Xaseron <xaseron@googlemail.com>

pkgname=xen-hg-unstable
pkgver=4.3
pkgrel=5
pkgdesc="Xen Hypervisor & Tools"
arch=(i686 x86_64)
url=http://xen.org
license=(GPL)
depends=(bin86 bluez bridge-utils glib2 gnutls libaio libjpeg-turbo libpng lzo2 sdl wget vde2 yajl pixman nss)
[[ "$CARCH" == "x86_64" ]] && depends+=(lib32-glibc)
makedepends=(dev86 git iasl markdown ocaml-findlib mercurial)
optdepends=('xen-docs: Official Xen Documentation')
conflicts=(xen-4.2{,-testing-hg} xen-{gdbsx,hg-unstable,rc})
backup=(etc/xen/xend-{config,pci-{permissive,quirks}}.sxp etc/modules-load.d/xen.conf)
options=(!buildflags !emptydirs !strip)
install=$pkgname.install
provides=(xen)
source=(09_xen
    texi2html.patch
    librt.patch
    proc-xen.mount
    var-lib-xenstored.mount
    xenconsoled.service
    xendomains.service
    xendomU@.service
    xenstored.service
    tmpfiles.d-xen.conf
    xen.conf)
sha256sums=('89f0076485d3f253b55aa6bb067003e74b4a6d6d559a87258fcaa8f93123693e'
            '08e5bf65c833a608470ad118ce369e32f9c267e1787f2900c06708af321225e6'
            'ffaac47599703949a42ecb9952aa476bb3bf869503f18c4faeffe48b8a355096'
            '139eed988bfaf8edc8ccdfd0b668382bd63db48ce17be91776182a7e28e9d88c'
            'c19146931c6ab8e53092bd9b2ebbfda5c76fd22ad3b1d42dcda3dd1b61f123ff'
            'ba8f1c10b3f3df1f9fda0782a691fed67661e36f49be74471c86850639fee3ba'
            '0bd45d9de6456c4f9adf32e726f2db3a3cd0423c1d161b442e8a1666d2e68e3f'
            '1862a14607582d14247b74435dfb16411fd68904aa19e2a93c5e6ac301169d3c'
            'a0ad5a7d9262c2d22a8875a47cff2c821885ddb65c0c9eb7518befb0f42fcce7'
            '6bddcea43922f72a1c8ab556c3f20067d7f817220bcd9c1c61d18f3a58dfaa9d'
            '50a9b7fd19e8beb1dea09755f07318f36be0b7ec53d3c9e74f3266a63e682c0c')

_hgroot=http://xenbits.xen.org/hg/
_hgrepo=xen-unstable.hg

build() {
     cd "$srcdir"
    msg "Connecting to Mercurial server...."

    if [[ -d "$_hgrepo" ]]; then
        cd "$_hgrepo"
        hg pull -u
        msg "The local files are updated."
    else
        hg clone "$_hgroot" "$_hgrepo"
    fi

    msg "Mercurial checkout done or server
    timeout"
    msg "Starting build..."

    rm -rf "$srcdir/$_hgrepo-build"
    cp -r "$srcdir/$_hgrepo" "$srcdir/$_hgrepo-build"
    cd "$srcdir/$_hgrepo-build"

    patch -Np1 -i ../texi2html.patch
    patch -Np1 -i ../librt.patch

    ./autogen.sh
    ./configure PYTHON=/usr/bin/python2
}

package() {
    cd "$srcdir"/"$_hgrepo"-build/

    make PYTHON=python2 DESTDIR="$pkgdir" install-xen install-tools
    # stubdom won't build with multiple makethreads
    make -j1 PYTHON=python2 DESTDIR="$pkgdir" install-stubdom

    cd ../
    for f in ${source[@]}; do
        [[ $f =~ .mount || $f =~ .service ]] && install -Dm644 $f "$pkgdir"/usr/lib/systemd/system/$f
    done
    install -Dm644 tmpfiles.d-xen.conf "$pkgdir"/usr/lib/tmpfiles.d/xen.conf
    install -Dm644 xen.conf "$pkgdir"/etc/modules-load.d/xen.conf
    install -Dm755 09_xen "$pkgdir"/etc/grub.d/09_xen

    cd "$pkgdir"
    sed -i ':XENDOM_CONFIG=/etc/:s:sysconfig/xendomains:conf.d/xendomains:' etc/init.d/xendomains
    sed -i 's:touch /var/lock/subsys/xend:mkdir -p /var/lock/subsys\n       &:' etc/init.d/xend

    if [[ -d usr/lib64 ]]; then
        cd usr/
        cp -r lib64/* lib/
        rm -rf lib64
    fi

    mv etc/{init,rc}.d

    mv usr/local/etc/qemu/ etc/
    rm -rf usr/local/share/
    mv etc/rc.d/xendomains etc/xen/scripts/xendomains

    ##### Kill unwanted stuff #####
    # stubdom: newlib
    rm -rf usr/*-xen-elf

    # hypervisor symlinks
    rm -f boot/xen{,-4,-4.2}.gz

    # silly doc dir fun
    rm -rf usr/share/doc/xen
    rm -rf usr/share/doc/qemu

    # Pointless helper
    rm -f usr/sbin/xen-python-path

    # qemu stuff (unused or available from upstream)
    rm -rf usr/share/xen/man
    rm -rf usr/bin/qemu-*-xen
    for file in bios.bin openbios-sparc32 openbios-sparc64 ppc_rom.bin \
        pxe-e1000.bin pxe-ne2k_pci.bin pxe-pcnet.bin pxe-rtl8139.bin \
        vgabios.bin vgabios-cirrus.bin video.x openbios-ppc bamboo.dtb; do
        rm -f usr/share/xen/qemu/$file
    done

    # adhere to Static Library Packaging Guidelines
    rm -rf usr/lib/*.a

    # Fix errors from deprecated xend
    rm etc/udev/rules.d/xend.rules
}
