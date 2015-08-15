# Maintainer: Gaetan Bisson <bisson@archlinux.org>
# Contributor: hokasch <hokasch at operamail dot com>
# Contributor: Dennis Brendel <buddabrod at gmail dot com>
# Contributor: Mathias Buren <mathias.buren at gmail dot com>
# Contributor: Benjamin Mtz (Cruznick) <cruznick at archlinux dot us>

pkgname=compat-wireless-patched
pkgver=3.6.2_1_snp # NO CRAP! (-c)
_upver="${pkgver//_/-}"
pkgrel=3
pkgdesc='Compat wireless driver patched for better injection'
url='http://wireless.kernel.org/en/users/Download/stable/'
arch=('i686' 'x86_64')
license=('GPL')
depends=('linux')
makedepends=('linux-api-headers' 'linux-headers')
source=("http://www.orbit-lab.org/kernel/compat-wireless-${pkgver:0:1}-stable/v${pkgver:0:3}/compat-wireless-${_upver}.tar.bz2" \
        'mac80211.compat08082009.wl_frag+ack_v1.patch')
sha1sums=('98f9b5f39100493f72d4e65e9899dd0751f8948d'
          '85f7a1b141549b774f5631fba259bc414aeeffb8')

_extramodules=extramodules-3.6-ARCH
_kernver=$(cat /usr/lib/modules/${_extramodules}/version) # TODO make this a lower boundary and utilize in reality pacman to get freshest paths

install=install

# For a list of supported drivers, run ./scripts/driver-select from the source tarball.
# To only build specific ones, define the _selected_drivers variable; for instance:
#         _selected_drivers='atheros intel wl12xx'
# You can also export this variable in ~/.bashrc or /etc/profile to automate updating this package

build() {
	cd "${srcdir}/compat-wireless-${_upver}"

	# modprobe -l dropped in kmod
  sed 's:modprobe -l \([^ )`]*\):find /usr/lib/modules/*/kernel -name "\1.ko*" | sed "s|.*/kernel||":' -i scripts/*
  sed 's:\$(MODPROBE) -l \([^ )`]*\):find /usr/lib/modules/*/kernel -name "\1.ko*" | sed "s|.*/kernel||":' -i Makefile

	# rfkill.h does not use compat-3.1.h # TODO : IS THIS RLY NEEDED ?
	echo '#define br_port_exists(dev) (dev->priv_flags & IFF_BRIDGE_PORT)' >> net/wireless/core.h

  # Patch time!
	patch -p1 -i ../mac80211.compat08082009.wl_frag+ack_v1.patch 

  # Make time! 
	[[ -n ${_selected_drivers} ]] && scripts/driver-select ${_selected_drivers}
	make KLIB=/usr/lib/modules/"${_kernver}"
}

package() {
	cd "${srcdir}/compat-wireless-${_upver}"
  mkdir ${pkgdir}/usr/
	make INSTALL_MOD_PATH="${pkgdir}/usr" KMODDIR=../"${_extramodules}" install-modules
	find "${pkgdir}" -name '*.ko' -exec gzip -9 {} \;

	install -d "${pkgdir}"/usr/sbin
	install scripts/*{enable,load} "${pkgdir}"/usr/sbin

	install -d "${pkgdir}"/usr/lib/compat-wireless
	install scripts/modlib.sh "${pkgdir}"/usr/lib/compat-wireless

	install -d "${pkgdir}"/usr/lib/udev/rules.d
	install udev/compat_firmware.sh	"${pkgdir}"/usr/lib/udev
	install udev/50-compat_firmware.rules "${pkgdir}"/usr/lib/udev/rules.d
}
