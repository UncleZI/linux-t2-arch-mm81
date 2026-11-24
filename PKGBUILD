# Maintainer: UncleZI
# Previous Maintainers: Noa Himesaka, Redecorating
# Contributors: There are many, see `grep -h "From:" *.patch|sort|uniq -c`.
#               Additionally, MrARM and Ronald Tschalär wrote apple-bce and
#               apple-ibridge drivers, respectively.
#
# Custom build for Mac Mini 2018 (macmini8,1).
# Uses a trimmed config from localmodconfig, with T2-critical modules
# re-enabled manually (T2BCE_*, APPLETB_*, BT_HCIBCM4377, macsmc_hwmon).

pkgbase=linux-t2-mm81
pkgver=7.3.5.arch1
pkgrel=1
pkgdesc='Linux kernel for T2 Mac Mini 2018 (macmini8,1)'
url='https://github.com/archlinux/linux'
arch=(
  x86_64
)
license=(GPL-2.0-only)

makedepends=(
  bc binutils cpio gettext git glibc libelf libgcc openssl pahole perl
  python rust rust-bindgen rust-src tar xxhash xz zlib zstd
)

options=(
  !debug
  !strip
)
_srcname=linux-${pkgver%.*}
_srctag=v${pkgver%.*}-${pkgver##*.}
T2_PATCH_HASH=56ef7488988b4fd7173eab73740e14794534dd61
source=(
  https://cdn.kernel.org/pub/linux/kernel/v${pkgver%%.*}.x/${_srcname}.tar.{xz,sign}
  $url/releases/download/$_srctag/linux-$_srctag.patch.zst{,.sig}
  patches::git+https://github.com/t2linux/linux-t2-patches#commit=${T2_PATCH_HASH}
)
source_x86_64=(config.x86_64)
validpgpkeys=(
  ABAF11C65A2970B130ABE3C479BE3E4300411886  # Linus Torvalds
  647F28654894E3BD457199BE38DBBDC86092693E  # Greg Kroah-Hartman
  83BC8889351B5DEBBB68416EB8AC08600F108CDF  # Jan Alexander Steffens (heftig)
)
sha256sums=('8f293f83d54f0b7b9be56863ebc5e4ea79d73e530b4ae55c572be97ef39f0990'
            'SKIP'
            'ab8cc9a56046bfdcbe8741283cb642fcb1ab25de4ac2b66fe655dd34e3ddc3b6'
            'SKIP'
            '914d933fbfb8865a9edf93115516c1b25f15d44d5ede0fcb80eaef1015cc0163')
sha256sums_x86_64=('3478af24984303c85552231e1e643a6de707d7fa72eee5d29c0f6c037eff5d21')
b2sums=('333bad17d6e6ae06935b08e8715cc617f59c0fdbab52dd1026a16b1125c8e693482946dc46f5b6e1f7c24a3919ddc44a8dd39f045dcb3d2da8b8001c113f13c0'
        'SKIP'
        'b4517a6cf6abb495af1df4884f66ffe79833fa8fe2cae08a00a88ee2ae508c3e68ce1f028faedf9dfcd0c8e687bbd7f9df5ed701417b9e23058f178256975f02'
        'SKIP'
        'a2cd453784e08db080f7d5f3298e303a5f49a11278b377633dd74d40b607976700dceebd26314acbc3447f2c504e88b0a12ea1447a600b0a513af18116573558')
b2sums_x86_64=('1f373a3d4167bf8d0c7b6476fcb4724910a92487c8720be15659defe1bec7c061ac4f3b1d3a27bf9aced93e42b987800929b2c77c74da22d1f1095a06da796e1')

export KBUILD_BUILD_TIMESTAMP="$(date -Ru${SOURCE_DATE_EPOCH:+d @$SOURCE_DATE_EPOCH})"

prepare() {
  cd $_srcname

  echo "Setting version..."
  echo "-t2-mm81" > localversion.10-codename
  echo "-$pkgrel" > localversion.20-pkgrel

  t2linux_patches=$(ls $srcdir/patches | grep -e \.patch$)
  mv $srcdir/patches/*.patch $srcdir/
  local src
  for src in "${source[@]}" $t2linux_patches; do
    src="${src%%::*}"
    src="${src##*/}"
    src="${src%.zst}"
    [[ $src = *.patch ]] || continue
    echo "Applying patch $src..."
    patch -Np1 < "../$src"
  done

  echo "Setting config..."
  cp ../config.$CARCH .config
  cat $srcdir/patches/extra_config >> .config
  make olddefconfig
  diff -u ../config.$CARCH .config || :

  make -s kernelrelease > version
  echo "Prepared $pkgbase version $(<version)"
}

build() {
  cd $_srcname
  make all
  make -C tools/bpf/bpftool vmlinux.h feature-clang-bpf-co-re=1
}

_package() {
  pkgdesc="The $pkgdesc kernel and modules"
  depends=(
    coreutils
    initramfs
    kmod
  )
  optdepends=(
    "$pkgbase-headers: headers and scripts for building modules"
    'linux-firmware: firmware images needed for some devices'
    'scx-scheds: to use sched-ext schedulers'
    'wireless-regdb: to set the correct wireless channels of your country'
  )
  provides=(
    KSMBD-MODULE
    NTSYNC-MODULE
    VIRTUALBOX-GUEST-MODULES
    WIREGUARD-MODULE
    linux
  )
  conflicts=(
    apple-gmux-t2-dkms-git
    linux-t2
  )
  replaces=(
    apple-gmux-t2-dkms-git
    virtualbox-guest-modules-arch
    wireguard-arch
    linux-t2
  )

  cd $_srcname
  local modulesdir="$pkgdir/usr/lib/modules/$(<version)"

  echo "Installing boot image..."
  install -Dm644 "$(make -s image_name)" "$modulesdir/vmlinuz"
  echo "$pkgbase" | install -Dm644 /dev/stdin "$modulesdir/pkgbase"

  echo "Installing modules..."
  ZSTD_CLEVEL=19 make INSTALL_MOD_PATH="$pkgdir/usr" INSTALL_MOD_STRIP=1 \
    DEPMOD=/doesnt/exist modules_install

  rm "$modulesdir"/build
}

_package-headers() {
  pkgdesc="Headers and scripts for building modules for the $pkgdesc kernel"
  depends=(
    binutils glibc libelf libgcc openssl pahole xxhash zlib zstd
  )
  provides=(
    LINUX-HEADERS
    linux-headers
  )
  conflicts=(
    linux-t2-headers
    linux-headers
  )
  replaces=(
    linux-t2-headers
  )

  cd $_srcname
  local builddir="$pkgdir/usr/lib/modules/$(<version)/build"

  local karch
  case $CARCH in
    x86_64) karch=x86 ;;
    *) echo "Unknown CARCH $CARCH"; exit 1 ;;
  esac

  echo "Installing build files..."
  install -Dt "$builddir" -m644 .config Makefile Module.symvers System.map \
    localversion.* version vmlinux tools/bpf/bpftool/vmlinux.h
  install -Dt "$builddir/kernel" -m644 kernel/Makefile
  install -Dt "$builddir/arch/$karch" -m644 arch/$karch/Makefile
  cp -t "$builddir" -a scripts
  ln -srt "$builddir" "$builddir/scripts/gdb/vmlinux-gdb.py"

  if [[ $(scripts/config -s CONFIG_HAVE_STACK_VALIDATION) = y ]]; then
    install -Dt "$builddir/tools/objtool" tools/objtool/objtool
  fi

  if [[ $(scripts/config -s CONFIG_DEBUG_INFO_BTF_MODULES) = y ]]; then
    install -Dt "$builddir/tools/bpf/resolve_btfids" tools/bpf/resolve_btfids/resolve_btfids
  fi

  echo "Installing headers..."
  cp -t "$builddir" -a include
  cp -t "$builddir/arch/$karch" -a arch/$karch/include
  install -Dt "$builddir/arch/$karch/kernel" -m644 arch/$karch/kernel/asm-offsets.s

  install -Dt "$builddir/drivers/md" -m644 drivers/md/*.h
  install -Dt "$builddir/net/mac80211" -m644 net/mac80211/*.h
  install -Dt "$builddir/drivers/media/i2c" -m644 drivers/media/i2c/msp3400-driver.h
  install -Dt "$builddir/drivers/media/usb/dvb-usb" -m644 drivers/media/usb/dvb-usb/*.h
  install -Dt "$builddir/drivers/media/dvb-frontends" -m644 drivers/media/dvb-frontends/*.h
  install -Dt "$builddir/drivers/media/tuners" -m644 drivers/media/tuners/*.h
  install -Dt "$builddir/drivers/iio/common/hid-sensors" -m644 drivers/iio/common/hid-sensors/*.h

  echo "Installing KConfig files..."
  find . -name 'Kconfig*' -exec install -Dm644 {} "$builddir/{}" \;

  if [[ $(scripts/config -s CONFIG_RUST) = y ]]; then
    echo "Installing Rust files..."
    install -Dt "$builddir/rust" -m644 rust/*.rmeta
    install -Dt "$builddir/rust" rust/*.so
  fi

  echo "Installing unstripped VDSO..."
  make INSTALL_MOD_PATH="$pkgdir/usr" vdso_install link=

  echo "Removing unneeded architectures..."
  local arch
  for arch in "$builddir"/arch/*/; do
    [[ $arch = */$karch/ ]] && continue
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
    case "$(file -Sib "$file")" in
      application/x-sharedlib\;*)      strip -v $STRIP_SHARED "$file" ;;
      application/x-archive\;*)        strip -v $STRIP_STATIC "$file" ;;
      application/x-executable\;*)     strip -v $STRIP_BINARIES "$file" ;;
      application/x-pie-executable\;*) strip -v $STRIP_SHARED "$file" ;;
    esac
  done < <(find "$builddir" -type f -perm -u+x ! -name vmlinux -print0)

  echo "Stripping vmlinux..."
  strip -v $STRIP_STATIC "$builddir/vmlinux"

  echo "Adding symlink..."
  mkdir -p "$pkgdir/usr/src"
  ln -sr "$builddir" "$pkgdir/usr/src/$pkgbase"
}

pkgname=(
  "$pkgbase"
  "$pkgbase-headers"
)
for _p in "${pkgname[@]}"; do
  eval "package_$_p() {
    $(declare -f "_package${_p#$pkgbase}")
    _package${_p#$pkgbase}
  }"
done

# vim:set ts=8 sts=2 sw=2 et:
