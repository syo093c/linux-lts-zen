# Maintainer: Your Name <youremail@example.com>

pkgbase=linux-lts-zen
pkgname=('linux-lts-zen' 'linux-lts-zen-headers')
pkgver=6.6.36
pkgrel=1
pkgdesc='LTS Linux with Zen patches'
url='https://www.kernel.org'
arch=('x86_64')
license=('GPL2')
makedepends=(
  'bc'
  'cpio'
  'gettext'
  'libelf'
  'pahole'
  'perl'
  'python'
  'tar'
  'xz'
  'graphviz'
  'imagemagick'
  'python-sphinx'
  'texlive-latexextra'
)
source=(
  "https://cdn.kernel.org/pub/linux/kernel/v${pkgver%%.*}.x/linux-${pkgver}.tar.xz"
  '0001-ZEN-Add-sysctl-and-CONFIG-to-disallow-unprivileged-C.patch'
  '0002-skip-simpledrm-if-nvidia-drm.modeset=1-is.patch'
  '0003-Default-to-maximum-amount-of-ASLR-bits.patch'
  'config'
)
sha256sums=('b9676828b737e8fb8eaa5198303d35d35e8df019550be153c8a42c99afe0cdd5'
            '21195509fded29d0256abfce947b5a8ce336d0d3e192f3f8ea90bde9dd95a889'
            '2f23be91455e529d16aa2bbf5f2c7fe3d10812749828fc752240c21b2b845849'
            '6400a06e6eb3a24b650bc3b1bba9626622f132697987f718e7ed6a5b8c0317bc'
            'c76831f33cba2a7737d6ec4e28793e649e86b63869f0f526d4cd2b546515ddf6')

prepare() {
  cd "$srcdir/linux-${pkgver}"
  
  echo "Setting version..."
  echo "-${pkgrel}" > localversion.10-pkgrel
  echo "${pkgbase#linux}" > localversion.20-pkgname

  for patch in "${source[@]}"; do
    patch="${patch##*/}"
    [[ $patch = *.patch ]] || continue
    echo "Applying patch $patch..."
    patch -Np1 < "../$patch"
  done

  echo "Setting config..."
  cp ../config .config
  make olddefconfig
}

build() {
  cd "$srcdir/linux-${pkgver}"
  make all
}

package_linux-lts-zen() {
  pkgdesc="The ${pkgdesc} kernel and modules"
  depends=('coreutils' 'initramfs' 'kmod')
  provides=("KSMBD-MODULE" "VIRTUALBOX-GUEST-MODULES" "WIREGUARD-MODULE")
  replaces=('wireguard-lts')
  license=('GPL2')

  cd "$srcdir/linux-${pkgver}"
  make INSTALL_MOD_PATH="$pkgdir/usr" modules_install

  install -Dt "$pkgdir/usr/lib/modules/$pkgver" -m644 \
    arch/x86/boot/bzImage \
    .config \
    System.map
}

package_linux-lts-zen-headers() {
  pkgdesc="Headers and scripts for building modules for the ${pkgdesc} kernel"
  depends=('pahole')
  provides=("linux-headers")

  cd "$srcdir/linux-${pkgver}"
  install -Dt "$pkgdir/usr/lib/modules/$pkgver/build" -m644 \
    .config \
    Makefile \
    Module.symvers \
    System.map \
    localversion.* \
    vmlinux
  install -Dt "$pkgdir/usr/lib/modules/$pkgver/build/kernel" -m644 \
    kernel/Makefile
  install -Dt "$pkgdir/usr/lib/modules/$pkgver/build/arch/x86" -m644 \
    arch/x86/Makefile
  cp -t "$pkgdir/usr/lib/modules/$pkgver/build" -a scripts
  cp -t "$pkgdir/usr/lib/modules/$pkgver/build" -a include
  cp -t "$pkgdir/usr/lib/modules/$pkgver/build/arch/x86" -a arch/x86/include

  # required when STACK_VALIDATION is enabled
  install -Dt "$pkgdir/usr/lib/modules/$pkgver/build/tools/objtool" tools/objtool/objtool
  # required when DEBUG_INFO_BTF_MODULES is enabled
  install -Dt "$pkgdir/usr/lib/modules/$pkgver/build/tools/bpf/resolve_btfids" tools/bpf/resolve_btfids/resolve_btfids
}

