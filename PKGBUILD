# Maintainer: Stephan Springer <buzo+arch@Lini.de>
# Contributor: Hyacinthe Cartiaux <hyacinthe.cartiaux@free.fr>
# Contributor: korjjj <korjjj+aur[at]gmail[dot]com>

pkgname=gns3-gui
pkgver=2.2.10
pkgrel=1
pkgdesc='GNS3 network simulator. Graphical user interface package.'
arch=('any')
url='https://github.com/GNS3/gns3-gui'
license=('GPL3')
groups=('gns3')
makedepends=('python-setuptools')
depends=(
    'desktop-file-utils'
    'gns3-converter'
    'python-distro'
    'python-jsonschema'
    'python-psutil'
    'python-pyqt5'
    'python-sentry_sdk'
    'python-sip'
    'qt5-svg'
    'qt5-websockets'
)
optdepends=(
    'gns3-server: GNS3 backend. Manages emulators such as Dynamips, VirtualBox or Qemu/KVM.'
    'xterm: Default terminal emulator for CLI management of virtual instances.'
    'wireshark-qt: Live packet capture.')
source=("$pkgname-$pkgver.tar.gz::https://github.com/GNS3/$pkgname/archive/v$pkgver.tar.gz"
        'gns3.desktop')
sha256sums=('0f04fe1b3b328bab5ee80755ea90e18e0d25828fbf4a644f3fbec29189f19bc8'
            '51e6db5b47e6af3d008d85e8c597755369fafb75ddb2af9e79a441f943f4c166')

prepare() {
    cd "$pkgname-$pkgver"
    # Arch usually has the latest versions. Patch requirements to allow them.
    sed -i -e 's|^psutil==5\.6\.6$|psutil>=5.6.6|' requirements.txt
}

build() {
    cd "$pkgname-$pkgver"
    python setup.py build
}

package() {
  cd "$pkgname-$pkgver"
  python setup.py install --root="$pkgdir" --optimize=1
  install -Dm644 "$srcdir"/gns3.desktop "$pkgdir"/usr/share/applications/gns3.desktop
  install -Dm644 resources/images/gns3_icon_256x256.png "$pkgdir"/usr/share/pixmaps/gns3.png
  install -Dm644 LICENSE "$pkgdir/usr/share/licenses/$pkgname/LICENSE"
}
