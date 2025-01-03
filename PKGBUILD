# Maintainer: Frederic Bezies <fredbezies@gmail.com>
# Contributor: ajs124 < aur at ajs124 dot de>
# Contributor: Devin Cofer <ranguvar{AT]archlinux[DOT}us>
# Contributor: Tobias Powalowski <tpowa@archlinux.org>
# Contributor: David Runge <dvzrv@archlinux.org>
# Contributor: Sébastien "Seblu" Luttringer <seblu@seblu.net>

pkgbase=qemu-git
_gitname=qemu
pkgname=(
  qemu-git
  qemu-arch-extra-git
  qemu-block-{iscsi,gluster}-git
  qemu-guest-agent-git
)
pkgdesc="A generic and open source machine emulator and virtualizer. Git version."
pkgver=9.2.0.r28.ga5ba0a7e4e
pkgrel=1
epoch=24
arch=(i686 x86_64)
license=(GPL-2.0-or-later LGPL-2.1-or-later)
url="https://wiki.qemu.org/"
# TODO: consider providing rdma-core
# TODO: consider providing lzfse
makedepends=(
  alsa-lib
  brltty
  bzip2
  cairo
  capstone
  cmocka
  curl
  cdrtools
  dtc
  edk2-ovmf
  fuse3
  gcc-libs
  gdk-pixbuf2
  git
  glib2
  glusterfs
  gnutls
  gtk3
  jack
  libaio
  libbpf
  libcacard
  libcap-ng
  libepoxy
  libiscsi
  libnfs
  libpng
  libpulse
  libsasl
  libseccomp
  libslirp
  libssh
  liburing
  libusb
  libx11
  libxkbcommon
  lzo
  mesa
  meson
  ncurses
  ndctl
  numactl
  pam
  pixman
  python
  python-sphinx
  python-sphinx_rtd_theme
  rust
  rust-bindgen
  sdl2
  sdl2_image
  snappy
  spice-protocol
  spice
  systemd
  usbredir
  vde2
  virglrenderer
  vte3 libvte-2.91.so
  xfsprogs
  zlib
  zstd 
)
options=(!lto)
source=(qemu-guest-agent.service
        65-kvm.rules)
sha256sums=('c39bcde4a09165e64419fd2033b3532378bba84d509d39e2d51694d44c1f8d88'
            'a66f0e791b16b03b91049aac61a25950d93e962e1b2ba64a38c6ad7f609b532c')

case $CARCH in
  i?86) _corearch=i386 ;;
  x86_64) _corearch=x86_64 ;;
esac

pkgver() {
  cd "${srcdir}"
  git describe --long --tags | sed 's/\([^-]*-g\)/r\1/;s/-/./g' | cut -c2-47
}

prepare() {
  cd "${srcdir}/.."
  mkdir -p build-full
  mkdir -p extra-arch-full/usr/{bin,share/qemu}
}

build() {
  _build full
}

_build() (
  
  cd ${srcdir}/../build-$1

  unset CFLAGS
  unset CXXFLAGS

  ${srcdir}/configure \
    --prefix=/usr \
    --sysconfdir=/etc \
    --localstatedir=/var \
    --libexecdir=/usr/lib/qemu \
    --smbd=/usr/bin/smbd \
    --disable-curl \
    --enable-capstone \
    --enable-fdt=system \
    --enable-plugins \
    --enable-rust \
    --enable-sdl \
    --disable-vfio-user-server \
    --enable-xen \
    --enable-debug \
    --enable-debug-info \
    --target-list=i386-softmmu,x86_64-softmmu,ppc-softmmu,ppc64-softmmu,mips64el-softmmu,aarch64-softmmu \
    --enable-werror #\
    #"${@:2}"

  ninja
)

package_qemu-git() {
  pkgdesc="QEMU Git version."
  depends=(
    alsa-lib libasound.so
    bzip2 libbz2.so
    cairo
    capstone
    curl libcurl.so
    dtc
    fuse3
    gcc-libs
    gdk-pixbuf2 libgdk_pixbuf-2.0.so
    glib2 libgio-2.0.so libglib-2.0.so libgmodule-2.0.so libgobject-2.0.so
    gnutls
    gtk3 libgdk-3.so libgtk-3.so
    jack libjack.so
    libaio
    libbpf libbpf.so
    libcacard
    libcap-ng libcap-ng.so
    libepoxy
    libjpeg libjpeg.so
    libnfs
    libpng
    libpulse libpulse.so
    libsasl
    libseccomp libseccomp.so
    libslirp libslirp.so
    libssh libssh.so
    libusb libusb-1.0.so
    liburing liburing.so
    libx11
    libxdp
    libxkbcommon libxkbcommon.so
    lzo
    mesa
    ncurses libncursesw.so
    ndctl
    numactl libnuma.so
    pam libpam.so
    pixman libpixman-1.so
    seabios
    sdl2
    sdl2_image
    snappy
    spice libspice-server.so
    systemd-libs libudev.so
    usbredir
    virglrenderer
    vde2
    vte3 libvte-2.91.so
    zlib
    zstd libzstd.so
  )
  optdepends=(
    'brltty: for braille device support'
    'qemu-arch-extra-git: extra architectures support'
  )
  conflicts=('qemu-headless' 'qemu' 'qemu-desktop')
  provides=('qemu-headless' 'qemu')

  _package full
}

_package() {
  optdepends+=('ovmf: Tianocore UEFI firmware for qemu'
               'samba: SMB/CIFS server support'
               'qemu-block-iscsi-git: iSCSI block support'
               'qemu-block-gluster-git: glusterfs block support')
  install=qemu.install
  options=(!strip !emptydirs)

  DESTDIR="$pkgdir" ninja -C "${srcdir}/../build-$1" install "${@:2}"

  # systemd stuff
  install -Dm644 65-kvm.rules "$pkgdir/usr/lib/udev/rules.d/65-kvm.rules"

  # remove conflicting /var/run directory
  cd "$pkgdir"
  rm -r var

  cd usr/lib

  # bridge_helper needs suid
  # https://bugs.archlinux.org/task/32565
  chmod u+s qemu/qemu-bridge-helper

  # remove split block modules
  rm qemu/block-{iscsi,gluster}.so

  cd ../bin

  # remove extra arch
  for _bin in qemu-*; do
    [[ -f $_bin ]] || continue

    case ${_bin#qemu-} in
      # guest agent
      ga) rm "$_bin"; continue ;;

      # tools
      edid|img|io|keymap|nbd|pr-helper|storage-daemon) continue ;;

      # core emu
      system-${_corearch}) continue ;;
    esac

    mv "$_bin" "$srcdir/../extra-arch-$1/usr/bin"
  done

  cd ../share/qemu
  for _blob in *; do
    [[ -f $_blob ]] || continue

    case $_blob in
      # provided by seabios package
      bios.bin|bios-256k.bin|vgabios-cirrus.bin|vgabios-qxl.bin|\
      vgabios-stdvga.bin|vgabios-vmware.bin|vgabios-virtio.bin|vgabios-bochs-display.bin|\
      vgabios-ramfb.bin|bios-microvm.bin|vgabios-ati.bin) rm "$_blob"; continue ;;

      # provided by edk2-ovmf package
      edk2-*) rm "$_blob"; continue ;;

      # iPXE ROMs
      efi-*|pxe-*) continue ;;

      # core blobs
      bios-microvm.bin|kvmvapic.bin|linuxboot*|multiboot.bin|sgabios.bin|vgabios*) continue ;;

      # Trace events definitions
      trace-events*) continue ;;
    esac

    mv "$_blob" "$srcdir/../extra-arch-$1/usr/share/qemu"
  done

  # provided by edk2-ovmf package
  rm -r firmware

  cd ..
}

package_qemu-arch-extra-git() {
  pkgdesc="QEMU for foreign architectures. Git version."
  depends=(
    dtc
    fuse3
    gcc-libs
    gnutls
    libaio
    libbpf libbpf.so
    glib2 libgio-2.0.so libglib-2.0.so libgobject-2.0.so libgmodule-2.0.so
    libjpeg libjpeg.so
    libpng
    libsasl
    libseccomp libseccomp.so
    libslirp libslirp.so
    liburing liburing.so
    lzo
    ndctl
    numactl libnuma.so
    pam libpam.so
    pixman libpixman-1.so
    snappy
    systemd-libs
    libudev.so
    qemu-git
    vde2
    zlib
    zstd
    libzstd.so
  )
  optdepends=(
    'edk2-armvirt: for aarch64 UEFI support'
    'edk2-ovmf: for ia32 and x64 UEFI support'
  )
  options=(!strip)
  provides=(qemu-arch-extra)
  conflicts=(qemu-arch-extra qemu-emulators-full)

  mv -v "$srcdir/../extra-arch-full/usr" "$pkgdir"
}

package_qemu-block-iscsi-git() {
  pkgdesc="QEMU iSCSI block module. Git version."
  depends=(glibc libiscsi)
  conflicts=(qemu-block-iscsi)
  provides=(qemu-block-iscsi)

  install -vDm 755 "$srcdir/../build-full/block-iscsi.so" -t "$pkgdir/usr/lib/qemu/"
}

package_qemu-block-gluster-git() {
  pkgdesc="QEMU GlusterFS block module. Git version."
  depends=(glibc glusterfs)
  conflicts=(qemu-block-gluster)
  provides=(qemu-block-gluster)

  install -vDm 755 "$srcdir/../build-full/block-gluster.so" -t "$pkgdir/usr/lib/qemu/"
}

package_qemu-guest-agent-git() {
  pkgdesc="QEMU Guest Agent. Git version."
  depends=(gcc-libs glib2 libudev.so liburing)
  conflicts=(qemu-guest-agent)
  provides=(qemu-guest-agent)
  install=qemu-guest-agent.install

  install -vDm 755 "$srcdir/../build-full/qga/qemu-ga" -t "$pkgdir/usr/bin/"
  install -vDm 644 "$srcdir/qemu-guest-agent.service" -t "$pkgdir/usr/lib/systemd/system/"
  install -vDm 755 "$srcdir/scripts/qemu-guest-agent/fsfreeze-hook" -t "$pkgdir/etc/qemu/"
}

# vim:set ts=2 sw=2 et:
