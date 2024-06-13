# Maintainer: loathingkernel <loathingkernel _a_ gmail _d_ com>

pkgname=proton-experimental
_srctag=9.0-20240522
_commit=
pkgver=${_srctag//-/.}
_geckover=2.47.4
_monover=9.1.0
_xaliaver=0.4.2
pkgrel=2
epoch=1
pkgdesc="Compatibility tool for Steam Play based on Wine and additional components, experimental branch"
url="https://github.com/ValveSoftware/Proton"
arch=(x86_64 x86_64_v3)
options=(!staticlibs !lto !debug emptydirs)
license=('custom')

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
  lzo              lib32-lzo
  libxkbcommon     lib32-libxkbcommon
  libvpx           lib32-libvpx
  sdl2             lib32-sdl2
  libsoup          lib32-libsoup
  libgudev         lib32-libgudev
#  blas             lib32-blas
#  lapack           lib32-lapack
  desktop-file-utils
  python
  steam-native-runtime
)
depends+=(
  wayland          lib32-wayland
)

makedepends=(autoconf bison perl flex mingw-w64-gcc
  git wget rsync unzip mingw-w64-tools lld nasm
  meson cmake fontforge afdko python-pefile glib2-devel
  glslang vulkan-headers
  clang
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
  gtk3                  lib32-gtk3
  gst-plugins-base-libs lib32-gst-plugins-base-libs
  vulkan-icd-loader     lib32-vulkan-icd-loader
  sdl2                  lib32-sdl2
  rust                  lib32-rust-libs
  libgphoto2
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
  gtk3                  lib32-gtk3
  gst-plugins-base-libs lib32-gst-plugins-base-libs
  vulkan-icd-loader     lib32-vulkan-icd-loader
  libgphoto2
)

provides=('proton')
install=${pkgname}.install
source=(
    proton::git+https://github.com/ValveSoftware/Proton.git#tag=experimental-${_srctag}
    https://dl.winehq.org/wine/wine-gecko/${_geckover}/wine-gecko-${_geckover}-x86{,_64}.tar.xz
    https://github.com/madewokherd/wine-mono/releases/download/wine-mono-${_monover}/wine-mono-${_monover}-x86.tar.xz
    https://github.com/madewokherd/xalia/releases/download/xalia-${_xaliaver}/xalia-${_xaliaver}-net48-mono.zip
    0001-AUR-Pkgbuild-changes.patch
    0002-AUR-Do-not-update-cargo-crates.patch
    0003-AUR-Remove-kaldi-openfst-vosk-api-modules-because-of.patch
    0004-AUR-Copy-DLL-dependencies-of-32bit-libvkd3d-dlls-int.patch
    0005-AUR-Strip-binaries-early.patch
    0006-AUR-Fix-hwnd-redefinition.patch
)
noextract=(
    wine-gecko-${_geckover}-{x86,x86_64}.tar.xz
    wine-mono-${_monover}-x86.tar.xz
    xalia-${_xaliaver}-net48-mono.zip
)

_make_wrappers () {
    #     _arch     prefix   gcc    ld             as     strip
    local _i686=(  "i686"   "-m32" "-melf_i386"   "--32" "elf32-i386")
    local _x86_64=("x86_64" "-m64" "-melf_x86_64" "--64" "elf64-x86-64")
    local _opts=(_i686 _x86_64)
    declare -n _opt
    for _opt in "${_opts[@]}"; do
        for l in ar ranlib nm; do
            ln -s /usr/bin/gcc-$l wrappers/${_opt[0]}-pc-linux-gnu-$l
        done
        for t in gcc g++; do
            install -Dm755 /dev/stdin wrappers/${_opt[0]}-pc-linux-gnu-$t <<EOF
#!/usr/bin/bash
$(which ccache 2> /dev/null) /usr/bin/$t ${_opt[1]} "\$@"
EOF
        done
        install -Dm755 /dev/stdin wrappers/${_opt[0]}-pc-linux-gnu-ld <<EOF
#!/usr/bin/bash
/usr/bin/ld ${_opt[2]} "\$@"
EOF
        install -Dm755 /dev/stdin wrappers/${_opt[0]}-pc-linux-gnu-as <<EOF
#!/usr/bin/bash
/usr/bin/as ${_opt[3]} "\$@"
EOF
        install -Dm755 /dev/stdin wrappers/${_opt[0]}-pc-linux-gnu-strip <<EOF
#!/usr/bin/bash
/usr/bin/strip -F ${_opt[4]} "\$@"
EOF
    done
}

prepare() {

    # Provide wrappers to compiler tools
    rm -rf wrappers && mkdir wrappers
    _make_wrappers

    [ ! -d build ] && mkdir build

    cd proton

    [ ! -d contrib ] && mkdir -p contrib
    mv "$srcdir"/wine-gecko-${_geckover}-x86{,_64}.tar.xz contrib/
    mv "$srcdir"/wine-mono-${_monover}-x86.tar.xz contrib/
    mv "$srcdir"/xalia-${_xaliaver}-net48-mono.zip contrib/

    # Explicitly set origin URL for submodules using relative paths
    git remote set-url origin https://github.com/ValveSoftware/Proton.git
    git submodule update --init --filter=tree:0 --recursive

    for rustlib in gst-plugins-rs; do
    pushd $rustlib
        export RUSTUP_TOOLCHAIN=stable
        export CARGO_HOME="${SRCDEST}"/proton-cargo
        cargo update
        cargo fetch --locked --target "i686-unknown-linux-gnu"
        cargo fetch --locked --target "x86_64-unknown-linux-gnu"
    popd
    done

    patch -p1 -i "$srcdir"/0001-AUR-Pkgbuild-changes.patch
    #patch -p1 -i "$srcdir"/0002-AUR-Do-not-update-cargo-crates.patch
    patch -p1 -i "$srcdir"/0003-AUR-Remove-kaldi-openfst-vosk-api-modules-because-of.patch
    patch -p1 -i "$srcdir"/0004-AUR-Copy-DLL-dependencies-of-32bit-libvkd3d-dlls-int.patch
    patch -p1 -i "$srcdir"/0005-AUR-Strip-binaries-early.patch
    patch -p1 -i "$srcdir"/0006-AUR-Fix-hwnd-redefinition.patch
}

build() {
    export PATH="$(pwd)/wrappers:$PATH"

    cd build
    ROOTLESS_CONTAINER="" \
    ../proton/configure.sh \
        --container-engine="none" \
        --proton-sdk-image="" \
        --build-name="${pkgname}"

    local -a split=($CFLAGS)
    local -A flags
    for opt in "${split[@]}"; do flags["${opt%%=*}"]="${opt##*=}"; done
    local march="${flags["-march"]:-nocona}"
    local mtune="generic" #"${flags["-mtune"]:-core-avx2}"

    CFLAGS="-O3 -march=$march -mtune=$mtune -pipe -fno-semantic-interposition"
    CXXFLAGS="-O3 -march=$march -mtune=$mtune -pipe -fno-semantic-interposition"
    RUSTFLAGS="-C opt-level=3 -C target-cpu=$march"
    LDFLAGS="-Wl,-O1,--sort-common,--as-needed"

    # AVX is "hard" disabled for 32bit in any case.
    # AVX/AVX2 for 64bit is disabled below.
    # Seems unnecessery for 64bit if -mtune=generic is used
    #CFLAGS+=" -mno-avx2 -mno-avx"
    #CXXFLAGS+=" -mno-avx2 -mno-avx"

    export CFLAGS CXXFLAGS RUSTFLAGS LDFLAGS

    export RUSTUP_TOOLCHAIN=stable
    export CARGO_HOME="${SRCDEST}"/proton-cargo
    export WINEESYNC=0
    export WINEFSYNC=0
    unset DISPLAY

    SUBJOBS=$([[ "$MAKEFLAGS" =~ -j\ *([1-9][0-9]*) ]] && echo "${BASH_REMATCH[1]}" || echo "$(nproc)") \
        make -j1 dist
}

package() {
    cd build

    # Delete the intermediate build directories to free space (mostly for my github actions)
    rm -rf dst-* obj-* src-* pfx-*

    local _compatdir="$pkgdir/usr/share/steam/compatibilitytools.d"
    mkdir -p "$_compatdir/${pkgname}"
    rsync --delete -arx dist/* "$_compatdir/${pkgname}"

    # For some unknown to me reason, 32bit vkd3d (not vkd3d-proton) always links
    # to libgcc_s_dw2-1.dll no matter what linker options I tried.
    # Copy the required dlls into the package, they will be copied later into the prefix
    # by the patched proton script. Bundle them to not depend on mingw-w64-gcc being installed.
    cp /usr/i686-w64-mingw32/bin/{libgcc_s_dw2-1.dll,libwinpthread-1.dll} \
        "$_compatdir/${pkgname}"/files/lib/vkd3d/
    cp /usr/x86_64-w64-mingw32/bin/{libgcc_s_seh-1.dll,libwinpthread-1.dll} \
        "$_compatdir/${pkgname}"/files/lib64/vkd3d/

    mkdir -p "$pkgdir/usr/share/licenses/${pkgname}"
    mv "$_compatdir/${pkgname}"/LICENSE{,.OFL} \
        "$pkgdir/usr/share/licenses/${pkgname}"

    cd "$_compatdir/${pkgname}/files"
    i686-w64-mingw32-strip --strip-unneeded \
        $(find lib/wine \( -iname fakedlls -or -iname i386-windows \) -prune -false -or -iname "*.dll" -or -iname "*.exe")
    x86_64-w64-mingw32-strip --strip-unneeded \
        $(find lib64/wine \( -iname fakedlls -or -iname x86_64-windows \) -prune -false -or -iname "*.dll" -or -iname "*.exe")

    local _geckodir="share/wine/gecko/wine-gecko-${_geckover}"
    i686-w64-mingw32-strip --strip-unneeded \
        $(find "$_geckodir"-x86 -iname "*.dll" -or -iname "*.exe")
    x86_64-w64-mingw32-strip --strip-unneeded \
        $(find "$_geckodir"-x86_64 -iname "*.dll" -or -iname "*.exe")

    local _monodir="share/wine/mono/wine-mono-${_monover}"
    i686-w64-mingw32-strip --strip-unneeded \
        $(find "$_monodir"/lib/mono -iname "*.dll" -or -iname "*.exe")
    i686-w64-mingw32-strip --strip-unneeded \
        "$_monodir"/lib/x86/*.dll \
        $(find "$_monodir" -iname "*x86.dll" -or -iname "*x86.exe")
    x86_64-w64-mingw32-strip --strip-unneeded \
        "$_monodir"/lib/x86_64/*.dll \
        $(find "$_monodir" -iname "*x86_64.dll" -or -iname "*x86_64.exe")
}

sha256sums=('0b6f758340e9cfa8d5595203dbfb69d0775cb6ac8294730d8aa60fe007957772'
            '2cfc8d5c948602e21eff8a78613e1826f2d033df9672cace87fed56e8310afb6'
            'fd88fc7e537d058d7a8abf0c1ebc90c574892a466de86706a26d254710a82814'
            '601169d0203b291fbfd946b356a9538855e01de22abd470ded73baf312c88767'
            '50ce2cc85162343e62340b0ca7994ceba94592ab395fb99711e94e108e991f0c'
            'c3cad6dec42b5123b7ab67901fa146bb9461f7bc79fbaf7ace093c2db7ce7069'
            'd2f6fbd72ff6a542334cf8dc89c8c36cc9aea06a0a7396c0a1c24c9f32d7ad00'
            '07005979bb430f983ea32aec3603e214a65feef4410285c728495f29d18852ac'
            '96c1c93a7397ecb682b02296914b3a7fc16013809b313ac670960d20cc81df70'
            'eea7d4e53e28e99efd6bf6bd77350511a78ce848f39e92730e377333b4c2f872'
            '60013412e8f792f0bc79df32294ab533aa7a085fce041951d3128cdb0b21c60d')
