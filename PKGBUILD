# Maintainer: loathingkernel <loathingkernel _a_ gmail _d_ com>
# Contributor: Felix Yan <felixonmars@archlinux.org>
# Contributor: Sven-Hendrik Haase <sh@lutzhaase.com>
# Contributor: Jan "heftig" Steffens <jan.steffens@gmail.com>
# Contributor: Eduardo Romero <eduardo@archlinux.org>
# Contributor: Giovanni Scafora <giovanni@archlinux.org>

pkgbase=wine-ge-custom
pkgname=(wine-ge-custom wine-ge-custom-standalone)
_srctag=GE-Proton8-10
_commit=ff9f66c9cdf8562f4d3338cc9900c1f15de5e01f
pkgver=${_srctag//-/.}
pkgrel=5
epoch=1

_pkgbasever=${pkgver/rc/-rc}
_winever=$_pkgbasever
#_winever=${_pkgbasever%.*}

source=(wine-ge-custom::git+https://github.com/GloriousEggroll/wine-ge-custom.git#commit=${_commit}
        proton-wine-ge::git+https://github.com/GloriousEggroll/proton-wine.git
        wine-wmclass.patch
        wine-isolate_home.patch
        30-win32-aliases.conf
        wine-binfmt.conf)
sha512sums=('SKIP'
            'SKIP'
            '30437d8ee92c5741fa50a7fe346ccfc48ba809dad0d740903a05a67781d23ea38a5094038a070a253e3fdd8046783b46a5420df6361bdd30cb229d3d88107569'
            '3dcdbd523fcbe79b9e9e9b026b9d0a5edf296514c7b48bd465d2dc05a8ca08e23ba8817e2de08edfe52286a2a2f81db42b65f71254cabe496752b9d45131d282'
            '6e54ece7ec7022b3c9d94ad64bdf1017338da16c618966e8baf398e6f18f80f7b0576edf1d1da47ed77b96d577e4cbb2bb0156b0b11c183a0accf22654b0a2bb'
            'bdde7ae015d8a98ba55e84b86dc05aca1d4f8de85be7e4bd6187054bfe4ac83b5a20538945b63fb073caab78022141e9545685e4e3698c97ff173cf30859e285')
validpgpkeys=(5AC1A08B03BD7A313E0A955AF5E6E9EEB9461DD7
              DA23579A74D4AD9AF9D3F945CEFAC8EAAF17519D)

pkgdesc="A compatibility layer for running Windows programs - GloriousEggroll branch"
url="https://github.com/GloriousEggroll/wine-ge-custom"
arch=(x86_64 x86_64_v3)
options=(!staticlibs !lto)
license=(LGPL)

depends=(
  attr             lib32-attr
  fontconfig       lib32-fontconfig
  libxcursor       lib32-libxcursor
  libxrandr        lib32-libxrandr
  libxi            lib32-libxi
  gettext          lib32-gettext
  freetype2        lib32-freetype2
  gcc-libs         lib32-gcc-libs
  libpcap          lib32-libpcap
  desktop-file-utils
)

makedepends=(autoconf bison perl flex mingw-w64-gcc
  git
  python
  giflib                lib32-giflib
  gnutls                lib32-gnutls
  libxinerama           lib32-libxinerama
  libxcomposite         lib32-libxcomposite
  libxxf86vm            lib32-libxxf86vm
  v4l-utils             lib32-v4l-utils
  alsa-lib              lib32-alsa-lib
  libxcomposite         lib32-libxcomposite
  mesa                  lib32-mesa
  mesa-libgl            lib32-mesa-libgl
  opencl-icd-loader     lib32-opencl-icd-loader
  libpulse              lib32-libpulse
  libva                 lib32-libva
  gtk3                  lib32-gtk3
  gst-plugins-base-libs lib32-gst-plugins-base-libs
  gst-plugins-good      lib32-gst-plugins-good
  vulkan-icd-loader     lib32-vulkan-icd-loader
  sdl2                  lib32-sdl2
  libgphoto2
  ffmpeg
  opencl-headers
)

optdepends=(
  giflib                lib32-giflib
  gnutls                lib32-gnutls
  v4l-utils             lib32-v4l-utils
  libpulse              lib32-libpulse
  alsa-plugins          lib32-alsa-plugins
  alsa-lib              lib32-alsa-lib
  libxcomposite         lib32-libxcomposite
  libxinerama           lib32-libxinerama
  opencl-icd-loader     lib32-opencl-icd-loader
  libva                 lib32-libva
  gtk3                  lib32-gtk3
  gst-plugins-base-libs lib32-gst-plugins-base-libs
  gst-plugins-good      lib32-gst-plugins-good
  vulkan-icd-loader     lib32-vulkan-icd-loader
  sdl2                  lib32-sdl2
  libgphoto2
  ffmpeg
  dosbox
)

optdepends+=(
  "wine-ge-custom-standalone: Makes wine-ge-custom behave as your system's default wine"
)

install=wine.install

prepare() {
  # Get rid of old build dirs
  rm -rf $pkgbase-{32,64}-build
  mkdir $pkgbase-{32,64}-build

  pushd $pkgbase
    git submodule init proton-wine
    git submodule set-url proton-wine "$srcdir"/proton-wine-ge
    git -c protocol.file.allow=always submodule update proton-wine
    pushd proton-wine
      patch -p1 -i "$srcdir"/wine-wmclass.patch
      patch -p1 -i "$srcdir"/wine-isolate_home.patch
      git config user.email "makepkg@aur.not"
      git config user.name "makepkg aur"
      git tag wine-ge-8.0 --annotate -m "$pkgver"
      dlls/winevulkan/make_vulkan
      tools/make_requests
      autoreconf -f
    popd
  popd

  # Doesn't compile without remove these flags as of 4.10
  export CFLAGS="${CFLAGS/-fno-plt/}"
  export LDFLAGS="${LDFLAGS/,-z,now/}"
}

build() {
  cd "$srcdir"

  # Doesn't compile without remove these flags as of 4.10
  export CFLAGS="${CFLAGS/-fno-plt/}"
  export CXXFLAGS="${CXXFLAGS/-fno-plt/}"
  export LDFLAGS="${LDFLAGS/,-z,now/}"
  # MingW Wine builds fail with relro
  export LDFLAGS="${LDFLAGS/,-z,relro/}"

  export LDFLAGS="-Wl,-O1,--sort-common,--as-needed"
  export CROSSLDFLAGS="$LDFLAGS -Wl,--file-alignment,4096"

  # From Proton
  OPTIMIZE_FLAGS="-O2 -march=nocona -mtune=core-avx2 -mfpmath=sse -pipe"
  SANITY_FLAGS="-fwrapv -fno-strict-aliasing"
  COMMON_FLAGS="$OPTIMIZE_FLAGS $SANITY_FLAGS -mno-avx2"

  msg2 "Building Wine-64..."

  export CFLAGS="$COMMON_FLAGS -mcmodel=small"
  export CXXFLAGS="$COMMON_FLAGS -mcmodel=small -std=c++17"
  export CROSSCFLAGS="$CFLAGS"
  export CROSSCXXFLAGS="$CXXFLAGS"
  export PKG_CONFIG_PATH="/usr/lib/pkgconfig:/usr/share/pkgconfig"
  cd "$srcdir/$pkgbase-64-build"
  ../$pkgbase/proton-wine/configure \
    --prefix=/opt/"$pkgbase" \
    --libdir=/opt/"$pkgbase"/lib \
    --with-x \
    --with-gstreamer \
    --with-mingw \
    --with-alsa \
    --without-oss \
    --disable-winemenubuilder \
    --disable-tests \
    --enable-win64 \
    --with-xattr

  make

  msg2 "Building Wine-32..."

  # Disable AVX instead of using 02, for 32bit
  export CFLAGS="$COMMON_FLAGS -mstackrealign -mno-avx"
  export CXXFLAGS="$COMMON_FLAGS -mstackrealign -mno-avx -std=c++17"
  export CROSSCFLAGS="$CFLAGS"
  export CROSSCXXFLAGS="$CXXFLAGS"
  export PKG_CONFIG_PATH="/usr/lib32/pkgconfig:/usr/share/pkgconfig"
  cd "$srcdir/$pkgbase-32-build"
  ../$pkgbase/proton-wine/configure \
    --prefix=/opt/"$pkgbase" \
    --with-x \
    --with-gstreamer \
    --with-mingw \
    --with-alsa \
    --without-oss \
    --disable-winemenubuilder \
    --disable-tests \
    --with-xattr \
    --libdir=/opt/"$pkgbase"/lib32 \
    --with-wine64="$srcdir/$pkgbase-64-build"

  make
}

package_wine-ge-custom() {
  msg2 "Packaging Wine-32..."
  cd "$srcdir/$pkgbase-32-build"

  make prefix="$pkgdir/opt/$pkgbase" \
    libdir="$pkgdir/opt/$pkgbase/lib32" \
    dlldir="$pkgdir/opt/$pkgbase/lib32/wine" install

  msg2 "Packaging Wine-64..."
  cd "$srcdir/$pkgbase-64-build"
  make prefix="$pkgdir/opt/$pkgbase" \
    libdir="$pkgdir/opt/$pkgbase/lib" \
    dlldir="$pkgdir/opt/$pkgbase/lib/wine" install

  i686-w64-mingw32-strip --strip-unneeded "$pkgdir"/opt/"$pkgbase"/lib32/wine/i386-windows/*.{dll,exe}
  x86_64-w64-mingw32-strip --strip-unneeded "$pkgdir"/opt/"$pkgbase"/lib/wine/x86_64-windows/*.{dll,exe}

  find "$pkgdir"/opt/"$pkgbase"/lib{,32}/wine -iname "*.a" -delete
  find "$pkgdir"/opt/"$pkgbase"/lib{,32}/wine -iname "*.def" -delete
}

package_wine-ge-custom-standalone() {
  pkgdesc+=". Makes wine-ge-custom behave as your system's default wine"
  depends=(wine-ge-custom)
  provides=("wine=8.0" "wine-wow64=8.0")
  conflicts=('wine' 'wine-wow64')

  _executables=(
    function_grep.pl msiexec regedit widl wine64 wineboot winecfg winecpp winedump
    wineg++ winemaker winepath wineserver wrc msidb notepad regsvr32 wine wine64-preloader
    winebuild wineconsole winedbg winefile winegcc winemine wine-preloader wmc
  )
  _winepath="/opt/$pkgbase"
  for _exe in "${_executables[@]}"; do
    install -Dm755 /dev/stdin "$pkgdir"/usr/bin/"$_exe" <<EOF
#!/usr/bin/sh
export PATH="$_winepath/bin/:\$PATH"
export LD_LIBRARY_PATH="$_winepath/lib:$_winepath/lib32:\$LD_LIBRARY_PATH"
export WINEDLLPATH="$_winepath/lib/wine:$_winepath/lib32/wine:\$WINEDLLPATH"
$_winepath/bin/$_exe
EOF
  done

  # Font aliasing settings for Win32 applications
  install -d "$pkgdir"/usr/share/fontconfig/conf.{avail,default}
  install -m644 "$srcdir/30-win32-aliases.conf" "$pkgdir/usr/share/fontconfig/conf.avail"
  ln -s ../conf.avail/30-win32-aliases.conf "$pkgdir/usr/share/fontconfig/conf.default/30-win32-aliases.conf"
  install -Dm 644 "$srcdir/wine-binfmt.conf" "$pkgdir/usr/lib/binfmt.d/wine.conf"

}

# vim:set ts=8 sts=2 sw=2 et:
