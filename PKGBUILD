# Maintainer: Jack Chen <redchenjs@live.com>

pkgbase=linux-sunxi
pkgname=('linux-sunxi' 'linux-sunxi-headers')
pkgver=5.9.11
_armbver=21.02.0
_armbrel=1
_kernver="$pkgver-sunxi"
pkgrel=1
arch=('armv7h')
_desc="ARMv7 multi-platform Sunxi"
url="https://github.com/armbian/build"
license=('GPL2')
makedepends=('curl')
options=('!strip')
source=(
  "linux.preset"
  "60-linux.hook"
  "90-linux.hook"
  "https://beta.armbian.com/pool/main/l/linux-$_kernver/linux-dtb-current-sunxi_$_armbver-trunk.${_armbrel}_armhf.deb"
  "https://beta.armbian.com/pool/main/l/linux-$_kernver/linux-image-current-sunxi_$_armbver-trunk.${_armbrel}_armhf.deb"
  "https://beta.armbian.com/pool/main/l/linux-$_kernver/linux-headers-current-sunxi_$_armbver-trunk.${_armbrel}_armhf.deb"
)
sha512sums=(
  'a492aae17ee4a316ce03faf9f1b284b2529c485f4b092cc4a1f865a6c68d482fd356fd30efa296c116975a3bdf3922f5bf03912a8d0e76f4ab24aa6ab9f8c276'
  '7ad5be75ee422dda3b80edd2eb614d8a9181e2c8228cd68b3881e2fb95953bf2dea6cbe7900ce1013c9de89b2802574b7b24869fc5d7a95d3cc3112c4d27063a'
  'a8fc668de860cf7b44269e35f879d130ccbc7db84d159ffcac6a92c02324f09c08fb57cccad18332f423bf2153c928633fa681ee96482c778dc76bffe80dd4f5'
  'SKIP'
  'SKIP'
  'SKIP'
)
noextract=("${source[@]##*/}")

prepare() {
  cd "$srcdir"

  rm -rf $(find -mindepth 1 -maxdepth 1 -type d)
}

package_linux-sunxi() {
  pkgdesc="The Linux Kernel and modules - $_desc"
  depends=('coreutils' 'linux-firmware' 'kmod' 'mkinitcpio>=0.7')
  optdepends=('crda: to set the correct wireless channels of your country')
  backup=("etc/mkinitcpio.d/$pkgbase.preset")
  provides=("linux=$pkgver" "WIREGUARD-MODULE")
  conflicts=('linux')
  install="$pkgname.install"

  cd "$srcdir"

  ar x "linux-dtb-current-sunxi_$_armbver-trunk.${_armbrel}_armhf.deb"
  tar -xf data.tar.xz
  ar x "linux-image-current-sunxi_$_armbver-trunk.${_armbrel}_armhf.deb"
  tar -xf data.tar.xz

  install -Dm644 "boot/vmlinuz-$_kernver" "$pkgdir/boot/zImage"
  cp -r "boot/dtb-$_kernver" "$pkgdir/boot/dtbs"

  install -dm755 "$pkgdir/usr"
  cp -r lib "$pkgdir/usr/lib"

  # make room for external modules
  local _extramodules="extramodules-$(echo $_kernver | sed -r 's|^([0-9]+.[0-9]+).*|\1|')-armv7"
  ln -s "../$_extramodules" "$pkgdir/usr/lib/modules/$_kernver/extramodules"

  # add real version for building modules and running depmod from hook
  echo "$_kernver" |
    install -Dm644 /dev/stdin "$pkgdir/usr/lib/modules/$_extramodules/version"

  # sed expression for following substitutions
  local _subst="
    s|%PKGBASE%|$pkgbase|g
    s|%KERNVER%|$_kernver|g
    s|%EXTRAMODULES%|$_extramodules|g
  "

  # install mkinitcpio preset file
  sed "$_subst" ../linux.preset |
    install -Dm644 /dev/stdin "$pkgdir/etc/mkinitcpio.d/$pkgbase.preset"

  # install pacman hooks
  sed "$_subst" ../60-linux.hook |
    install -Dm644 /dev/stdin "$pkgdir/usr/share/libalpm/hooks/60-$pkgbase.hook"
  sed "$_subst" ../90-linux.hook |
    install -Dm644 /dev/stdin "$pkgdir/usr/share/libalpm/hooks/90-$pkgbase.hook"
}

package_linux-sunxi-headers() {
  pkgdesc="Header files and scripts for building modules for linux kernel - $_desc"
  provides=("linux-headers=$pkgver")
  conflicts=('linux-headers')

  cd "$srcdir"

  ar x "linux-image-current-sunxi_$_armbver-trunk.${_armbrel}_armhf.deb"
  tar -xf data.tar.xz
  ar x "linux-headers-current-sunxi_$_armbver-trunk.${_armbrel}_armhf.deb"
  tar -xf data.tar.xz

  install -dm755 "$pkgdir/usr/lib/modules/$_kernver"
  cp -r "usr/src/linux-headers-$_kernver" "$pkgdir/usr/lib/modules/$_kernver/build"

  install -Dm644 "boot/config-$_kernver" "$pkgdir/usr/lib/modules/$_kernver/build/.config"
  install -Dm644 "boot/System.map-$_kernver" "$pkgdir/usr/lib/modules/$_kernver/build/System.map"
}