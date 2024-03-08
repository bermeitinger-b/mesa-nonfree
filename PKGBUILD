# Maintainer: Laurent Carlier <lordheavym@gmail.com>
# Maintainer: Felix Yan <felixonmars@archlinux.org>
# Contributor: Jan de Groot <jgc@archlinux.org>
# Contributor: Andreas Radke <andyrtr@archlinux.org>

pkgbase=mesa
pkgname=(
	'vulkan-mesa-layers'
	'opencl-clover-mesa'
	'opencl-rusticl-mesa'
	'vulkan-radeon'
	'libva-mesa-driver'
	'mesa'
)
pkgver=24.0.2
pkgrel=3
epoch=10
pkgdesc="An open-source implementation of the OpenGL specification"
url="https://www.mesa3d.org/"
arch=('x86_64')
license=('MIT AND BSD-3-Clause AND SGI-B-2.0')
makedepends=(
	'clang'
	'expat'
	'libdrm'
	'libelf'
	'libglvnd'
	'libva'
	'libx11'
	'libxdamage'
	'libxml2'
	'libxrandr'
	'libxshmfence'
	'libxxf86vm'
	'llvm'
	'lm_sensors'
	'rust'
	'spirv-llvm-translator'
	'spirv-tools'
	'systemd'
	'vulkan-icd-loader'
	'xcb-util-keysyms'
	'zstd'

	# shared between mesa and lib32-mesa
	'clang'
	'cmake'
	'elfutils'
	'glslang'
	'libclc'
	'meson'
	'python-mako'
	'python-ply'
	'rust-bindgen'
	'xorgproto'

	# gallium-omx deps
	'libomxil-bellagio'
)
source=(
	https://mesa.freedesktop.org/archive/mesa-${pkgver}.tar.xz{,.sig}
	radeon_bo_can_reclaim_slab.diff
	vulkan-dispatch-table-add-an-uncompacted-version.patch
	LICENSE
)
sha256sums=('94e28a8edad06d8ed2b83eb53f253b9eb5aa62c3080f939702e1b3039b56c9e8'
	'SKIP'
	'3fd1ad8cd29319502a6f80ecb96bb9a059e5de83a8b6e39f23de8d93921fd922'
	'1733ec76f735788837833e7571b641fc64b56ec3176b93e9234fc0b5428ee6d8'
	'7052ba73bb07ea78873a2431ee4e828f4e72bda7d176d07f770fa48373dec537')
b2sums=('f69e0b3edb7b8611f528a2e04104fe14b2fe8c799921be2d112dba744133813a19f90aa11d39f3f87a282e518003c7cc7966143d25e845f1f4489c461b22f661'
	'SKIP'
	'e7c3451a342cc648149375ce58697ae24273d47060e74ca2948d45ea8fe29b104f1daae4c91968fb6f37d41963d176987abf9ee21acfba0172a9b5d30300a72e'
	'e057a085bf7a9faceaa90b29555626d79e8c818e84a9424ade53dd21c512b2ea37dabb1d8ecccdf0f8fa69a4c6e7b66a6fe970b65baf1d368d6b9cc94ba532c7'
	'1ecf007b82260710a7bf5048f47dd5d600c168824c02c595af654632326536a6527fbe0738670ee7b921dd85a70425108e0f471ba85a8e1ca47d294ad74b4adb')
validpgpkeys=(
	'8703B6700E7EE06D7A39B8D6EDAE37B02CEB490D' # Emil Velikov <emil.l.velikov@gmail.com>
	'946D09B5E4C9845E63075FF1D961C596A7203456' # Andres Gomez <tanty@igalia.com>
	'E3E8F480C52ADD73B278EE78E1ECBE07D7D70895' # Juan Antonio Suárez Romero (Igalia, S.L.) <jasuarez@igalia.com>
	'A5CC9FEC93F2F837CB044912336909B6B25FADFA' # Juan A. Suarez Romero <jasuarez@igalia.com>
	'71C4B75620BC75708B4BDB254C95FAAB3EB073EC' # Dylan Baker <dylan@pnwbakers.com>
	'57551DE15B968F6341C248F68D8E31AFC32428A6' # Eric Engestrom <eric@engestrom.ch>
)
prepare() {
	cd mesa-$pkgver

	# Proposed crash fix from https://gitlab.freedesktop.org/mesa/mesa/-/issues/10613#note_2290167
	patch -Np1 -i ../radeon_bo_can_reclaim_slab.diff

	# https://gitlab.freedesktop.org/mesa/mesa/-/merge_requests/27834
	patch -Np1 -i ../vulkan-dispatch-table-add-an-uncompacted-version.patch

	# Include package release in version string so Chromium invalidates
	# its GPU cache; otherwise it can cause pages to render incorrectly.
	# https://bugs.launchpad.net/ubuntu/+source/chromium-browser/+bug/2020604
	echo "$pkgver-arch$epoch.$pkgrel" >VERSION
}

build() {
	local meson_options=(
		-D android-libbacktrace=disabled
		-D b_ndebug=true
		-D dri3=enabled
		-D egl=enabled
		-D gallium-drivers=radeonsi,swrast,zink
		-D gallium-extra-hud=true
		-D gallium-nine=true
		-D gallium-omx=bellagio
		-D gallium-opencl=icd
		-D gallium-rusticl=true
		-D gallium-va=enabled
		-D gallium-vdpau=disabled
		-D gallium-xa=disabled
		-D gbm=enabled
		-D gles1=disabled
		-D gles2=enabled
		-D glvnd=true
		-D glx=dri
		-D intel-clc=disabled
		-D libunwind=disabled
		-D llvm=enabled
		-D lmsensors=enabled
		-D microsoft-clc=disabled
		-D osmesa=true
		-D platforms=x11
		-D rust_std=2021
		-D shared-glapi=enabled
		-D opencl-spirv=true
		-D valgrind=disabled
		-D video-codecs=all
		-D vulkan-drivers=amd
		-D vulkan-layers=device-select,overlay
	)

	# Build only minimal debug info to reduce size
	CFLAGS+=' -g1'
	CXXFLAGS+=' -g1'

	arch-meson mesa-$pkgver build "${meson_options[@]}"
	meson configure build # Print config
	meson compile -C build

	# fake installation to be seperated into packages
	# outside of fakeroot but mesa doesn't need to chown/mod
	DESTDIR="${srcdir}/fakeinstall" meson install -C build
}

_install() {
	local src f dir
	for src; do
		f="${src#fakeinstall/}"
		dir="${pkgdir}/${f%/*}"
		install -m755 -d "${dir}"
		mv -v "${src}" "${dir}/"
	done
}

_libdir=usr/lib

package_vulkan-mesa-layers() {
	pkgdesc="Mesa's Vulkan layers"
	depends=(
		'libdrm'
		'libxcb'
		'wayland'

		'python'
	)
	conflicts=('vulkan-mesa-layer')
	replaces=('vulkan-mesa-layer')

	_install fakeinstall/usr/share/vulkan/explicit_layer.d
	_install fakeinstall/usr/share/vulkan/implicit_layer.d
	_install fakeinstall/$_libdir/libVkLayer_*.so
	_install fakeinstall/usr/bin/mesa-overlay-control.py

	install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_opencl-clover-mesa() {
	pkgdesc="OpenCL support with clover for mesa drivers"
	depends=(
		'clang'
		'expat'
		'libdrm'
		'libelf'
		'spirv-llvm-translator'
		'zstd'

		'libclc'
	)
	optdepends=('opencl-headers: headers necessary for OpenCL development')
	provides=('opencl-driver')
	replaces=("opencl-mesa<=23.1.4-1")
	conflicts=('opencl-mesa')

	_install fakeinstall/etc/OpenCL/vendors/mesa.icd
	_install fakeinstall/$_libdir/libMesaOpenCL*
	_install fakeinstall/$_libdir/gallium-pipe

	install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_opencl-rusticl-mesa() {
	pkgdesc="OpenCL support with rusticl for mesa drivers"
	depends=(
		'clang'
		'expat'
		'libdrm'
		'libelf'
		'lm_sensors'
		'spirv-llvm-translator'
		'zstd'

		'libclc'
	)
	optdepends=('opencl-headers: headers necessary for OpenCL development')
	provides=('opencl-driver')
	replaces=("opencl-mesa<=23.1.4-1")
	conflicts=('opencl-mesa')

	_install fakeinstall/etc/OpenCL/vendors/rusticl.icd
	_install fakeinstall/$_libdir/libRusticlOpenCL*

	install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_vulkan-radeon() {
	pkgdesc="Radeon's Vulkan mesa driver"
	depends=(
		'libdrm'
		'libelf'
		'libx11'
		'libxshmfence'
		'llvm-libs'
		'systemd'
		'xcb-util-keysyms'
		'zstd'
	)
	optdepends=('vulkan-mesa-layers: additional vulkan layers')
	provides=('vulkan-driver')

	_install fakeinstall/usr/share/drirc.d/00-radv-defaults.conf
	_install fakeinstall/usr/share/vulkan/icd.d/radeon_icd*.json
	_install fakeinstall/$_libdir/libvulkan_radeon.so

	install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_libva-mesa-driver() {
	pkgdesc="VA-API drivers"
	depends=(
		'expat'
		'libdrm'
		'libelf'
		'libx11'
		'libxshmfence'
		'llvm-libs'
		'zstd'
	)
	provides=('libva-driver')

	_install fakeinstall/$_libdir/dri/*_drv_video.so

	install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}

package_mesa() {
	depends=(
		'libdrm'
		'libelf'
		'libglvnd'
		'libxdamage'
		'libxshmfence'
		'libxxf86vm'
		'llvm-libs'
		'lm_sensors'
		'vulkan-icd-loader'
		'zstd'

		'libomxil-bellagio'
	)
	optdepends=(
		'opengl-man-pages: for the OpenGL API man pages'
	)
	provides=(
		'mesa-libgl'
		'opengl-driver'
	)
	conflicts=('mesa-libgl')
	replaces=('mesa-libgl')

	_install fakeinstall/usr/share/drirc.d/00-mesa-defaults.conf
	_install fakeinstall/usr/share/glvnd/egl_vendor.d/50_mesa.json

	# ati-dri, nouveau-dri, intel-dri, svga-dri, swrast, swr
	_install fakeinstall/$_libdir/dri/*_dri.so

	_install fakeinstall/$_libdir/bellagio
	_install fakeinstall/$_libdir/d3d
	_install fakeinstall/$_libdir/lib{gbm,glapi}.so*
	_install fakeinstall/$_libdir/libOSMesa.so*

	_install fakeinstall/usr/include
	_install fakeinstall/$_libdir/pkgconfig

	# libglvnd support
	_install fakeinstall/$_libdir/libGLX_mesa.so*
	_install fakeinstall/$_libdir/libEGL_mesa.so*

	# indirect rendering
	ln -sr "$pkgdir"/$_libdir/libGLX_{mesa,indirect}.so.0

	install -m644 -Dt "${pkgdir}/usr/share/licenses/${pkgname}" LICENSE
}
