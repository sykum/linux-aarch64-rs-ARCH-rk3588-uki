# / AArch64 multi-platform
# Maintainer: Kevin Mihelich <kevin@archlinuxarm.org>
# Maintainer: graysky <therealgraysky AT proton DOT me>

buildarch=8

pkgbase=linux-aarch64-rs
pkgname=(
  "${pkgbase}"
  "${pkgbase}-headers"
)
_srcname=linux-6.18.6
_kernelname=${pkgbase#linux}
_desc="AArch64 multi-platform"
pkgver=6.18.6
pkgrel=1
arch=('aarch64')
url="http://www.kernel.org/"
license=('GPL-2.0-only')
makedepends=('xmlto' 'docbook-xsl' 'kmod' 'inetutils' 'bc' 'git' 'uboot-tools' 'vboot-utils' 'dtc' 'python')
options=('!strip')
source=("https://cdn.kernel.org/pub/linux/kernel/v6.x/${_srcname}.tar.xz"
        #"https://www.kernel.org/pub/linux/kernel/v6.x/patch-${pkgver}.xz"
        'config'
        'linux.preset'
        '001-Fix-media-uAPI-cross-references.patch'
        '002-media-rkvdec-Add-HEVC-backend.patch'
        '003-media-rkvdec-Add-support-for-VDPU381-and-VDPU383.patch'
        '004-arm64-dts-rockchip-Add-vdpu-381-and-383-nodes.patch'
        '005-Add-HDMI-CEC-support-to-Rockchip-RK3588-RK3576-SoCs.patch'
        '006-fix-hym8563-pinctrl.patch'
        '007-PCI-dwc-Revert-Link-Up-IRQ-support.patch')
sha256sums=('472497197b2f68d4dbf1bc32cc6dc669ca220ff4c0eb0dc39a9cff9a88f9a31b'
            '37cc42595f230a7ab7996729efc22f2573a50e8e9da368fd25dfb19022506c53'
            'ec409dd9079403969e26c55d942082ec8a4889d5652cbf873b3a52f5d3d23b2c'
            'd8481de4a4586d0d9149e641a318df08b0d2c7687be0b32abed6b43374a6a877'
            '7a23e87702537b65bb651f4965e0eb0dca0cbebdf13f5c019139adc7faa833a9'
            '9ca614cb767965f4de9e66d48914a97f9a581311f55515bd578c9f85f802ac7d'
            '0adf05ad233b2c590a2f537e0d002292a325eab02d6796efacd02942db244c38'
            '22054c22bad89de7703daa52f6b90f4db3bed8c28dd2da256cd99b3291ca1920'
            'a8e95f206f25ec03381d88a71bcdd09b47b2a5380891b07ca0c49feab3ea6761'
            'a904cd3e219ca0c7e98b96a2f23ff4fdb6c21d81703fde7ca2ae3541208ae200')

prepare() {
  cd $_srcname

  echo "Setting version..."
  echo "-$pkgrel" > localversion.10-pkgrel
  echo "${pkgbase#linux}" > localversion.20-pkgname

  # add upstream patch
  if [[ -f ../patch-${pkgver} ]]; then
    git apply --whitespace=nowarn ../patch-${pkgver}
  fi

  # RK patches
  git apply ../001-Fix-media-uAPI-cross-references.patch
  git apply ../002-media-rkvdec-Add-HEVC-backend.patch
  git apply ../003-media-rkvdec-Add-support-for-VDPU381-and-VDPU383.patch
  git apply ../004-arm64-dts-rockchip-Add-vdpu-381-and-383-nodes.patch
  git apply ../005-Add-HDMI-CEC-support-to-Rockchip-RK3588-RK3576-SoCs.patch
  git apply ../006-fix-hym8563-pinctrl.patch
  git apply ../007-PCI-dwc-Revert-Link-Up-IRQ-support.patch
  
  cat "${srcdir}/config" > ./.config
}

build() {
  cd ${_srcname}

  # get kernel version
  make prepare
  make -s kernelrelease > version

  # build!
  unset LDFLAGS
  make ${MAKEFLAGS} Image Image.gz modules
  # Generate device tree blobs with symbols to support applying device tree overlays in U-Boot
  make ${MAKEFLAGS} DTC_FLAGS="-@" dtbs
}

package_linux-aarch64-rs() {
  pkgdesc="The Linux Kernel and modules - ${_desc}"
  depends=('coreutils' 'kmod' 'mkinitcpio>=0.7')
  optdepends=('linux-firmware: firmware images needed for some devices'
              'wireless-regdb: to set the correct wireless channels of your country')
  provides=("linux=${pkgver}" "KSMBD-MODULE" "WIREGUARD-MODULE")
  conflicts=('linux')
  install=${pkgname}.install

  cd $_srcname
  local kernver="$(<version)"
  local modulesdir="$pkgdir/usr/lib/modules/$kernver"

  echo "Installing boot image and dtbs..."
  install -Dm644 arch/arm64/boot/Image{,.gz} -t "${pkgdir}/boot"
  make INSTALL_DTBS_PATH="${pkgdir}/boot/dtbs" dtbs_install

  echo "Installing modules..."
  make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 DEPMOD=/doesnt/exist modules_install

  # remove build link
  rm "$modulesdir"/build

  # sed expression for following substitutions
  local _subst="
    s|%PKGBASE%|${pkgbase}|g
    s|%KERNVER%|${kernver}|g
  "

  # install mkinitcpio preset file
  sed "${_subst}" ../linux.preset |
    install -Dm644 /dev/stdin "${pkgdir}/etc/mkinitcpio.d/${pkgbase}.preset"

  # rather than use another hook (90-linux.hook) rely on mkinitcpio's 90-mkinitcpio-install.hook
  # which avoids a double run of mkinitcpio that can occur
  install -d "${pkgdir}/usr/lib/initcpio/"
  echo "dummy file to trigger mkinitcpio to run" > "${pkgdir}/usr/lib/initcpio/$(<version)"
}

package_linux-aarch64-rs-headers() {
  pkgdesc="Header files and scripts for building modules for linux kernel - ${_desc}"
  provides=("linux-headers=${pkgver}")
  conflicts=('linux-headers')

  cd $_srcname
  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  echo "Installing build files..."
  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
    localversion.* version vmlinux
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/arm64" -m644 arch/arm64/Makefile
  cp -t "$builddir" -a scripts

  # add xfs and shmem for aufs building
  mkdir -p "$builddir"/{fs/xfs,mm}

  echo "Installing headers..."
  cp -t "$builddir" -a include
  cp -t "$builddir/arch/arm64" -a arch/arm64/include
  install -Dt "$builddir/arch/arm64/kernel" -m644 arch/arm64/kernel/asm-offsets.s
  mkdir -p "$builddir/arch/arm"
  cp -t "$builddir/arch/arm" -a arch/arm/include

  install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
  install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h

  # https://bugs.archlinux.org/task/13146
  install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h

  # https://bugs.archlinux.org/task/20402
  install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h

  # https://bugs.archlinux.org/task/71392
  install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

  echo "Installing KConfig files..."
  find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

  echo "Removing unneeded architectures..."
  local arch
  for arch in "$builddir"/arch/*/; do
    [[ $arch = */arm64/ || $arch == */arm/ ]] && continue
    echo "Removing $(basename "$arch")"
    rm -r "$arch"
  done

  echo "Removing documentation..."
  rm -r "$builddir/Documentation"

  echo "Removing broken symlinks..."
  find -L "$builddir" -type l -printf 'Removing %P\n' -delete

  echo "Removing loose objects..."
  find "$builddir" -type f -name '*.o' -printf 'Removing %P\n' -delete

  echo "Stripping build tools..."
  local file
  while read -rd '' file; do
    case "$(file -bi "$file")" in
      application/x-sharedlib\;*)      # Libraries (.so)
        strip -v $STRIP_SHARED "$file" ;;
      application/x-archive\;*)        # Libraries (.a)
        strip -v $STRIP_STATIC "$file" ;;
      application/x-executable\;*)     # Binaries
        strip -v $STRIP_BINARIES "$file" ;;
      application/x-pie-executable\;*) # Relocatable binaries
        strip -v $STRIP_SHARED "$file" ;;
    esac
  done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

  echo "Adding symlink..."
  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}
