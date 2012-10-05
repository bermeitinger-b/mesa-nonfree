# Maintainer: Jan de Groot <jgc@archlinux.org>
# Maintainer: Andreas Radke <andyrtr@archlinux.org>

pkgbase=mesa
pkgname=('libglapi' 'libgl' 'mesa' 'osmesa' 'libgbm' 'libgles' 'libegl' 'khrplatform-devel' 'ati-dri' 'intel-dri' 'svga-dri' 'nouveau-dri')

_git=true
_gitdate=20121005
#_git=false

if [ "${_git}" = "true" ]; then
    pkgver=8.99.git_$_gitdate
  else
    pkgver=8.0.4
fi
pkgrel=1
arch=('i686' 'x86_64')
makedepends=('glproto>=1.4.16' 'libdrm>=2.4.39' 'libxxf86vm>=1.1.2' 'libxdamage>=1.1.3' 'expat>=2.1.0' 'libx11>=1.5.0' 'libxt>=1.1.3' 
             'gcc-libs>=4.7.1-6' 'dri2proto>=2.8' 'python2' 'libxml2' 'imake' 'llvm' 'systemd' 'libvdpau>=0.5')
url="http://mesa3d.sourceforge.net"
license=('custom')
options=('!libtool')
source=(LICENSE
        pthread_fix.diff)
if [ "${_git}" = "true" ]; then
	# mesa git shot from 9.0 branch - see for state: http://cgit.freedesktop.org/mesa/mesa/log/?h=9.0
	#source=(${source[@]} 'ftp://ftp.archlinux.org/other/mesa/mesa-41d14eaf193c6b1eb87fe1998808a887f1c6c698.tar.gz')
	source=(${source[@]} "MesaLib-git${_gitdate}.zip"::"http://cgit.freedesktop.org/mesa/mesa/snapshot/mesa-542f6feda9bf18267dbd337943a5e871400d425a.tar.gz")
  else
	source=(${source[@]} "ftp://ftp.freedesktop.org/pub/mesa/${pkgver}/MesaLib-${pkgver}.tar.bz2"
	#source=(${source[@]} "ftp://ftp.freedesktop.org/pub/mesa/8.0/MesaLib-8.0-rc2.tar.bz2"
)
fi
md5sums=('5c65a0fe315dd347e09b1f2826a1df5a'
         '03956ac54a44467678120f485b626633'
         '52760839a596df5058fcbb63a2bb10da')

build() {
    cd ${srcdir}/?esa-*

    # build fix from master http://cgit.freedesktop.org/mesa/mesa/commit/?id=dd4fde8f674f5e3efa19e929f97de4ecfd82391b
    patch -Np1 -i ${srcdir}/pthread_fix.diff

    COMMONOPTS="--prefix=/usr \
    --sysconfdir=/etc \
    --with-dri-driverdir=/usr/lib/xorg/modules/dri \
    --with-gallium-drivers=r300,r600,radeonsi,nouveau,svga,swrast \
    --with-dri-drivers=i915,i965,r200,radeon,nouveau,swrast \
    --enable-gallium-llvm \
    --enable-egl \
    --enable-gallium-egl \
    --with-egl-platforms=x11,drm \
    --enable-shared-glapi \
    --enable-gbm \
    --enable-glx-tls \
    --enable-dri \
    --enable-glx \
    --enable-osmesa \
    --enable-gles1 \
    --enable-gles2 \
    --enable-texture-float \
    --enable-xa \
    --enable-vdpau "

# not default:
#    --enable-gallium-egl enable optional EGL state tracker (not required for
#                          EGL support in Gallium with OpenGL and OpenGL ES)
#                          [default=disable]
#    --enable-xa             enable build of the XA X Acceleration API                          [default=no]


if [ "${_git}" = "true" ]; then
    ./autogen.sh \
      $COMMONOPTS
  else
    autoreconf -vfi
    ./configure \
      $COMMONOPTS
fi

    make
}

package_libglapi() {
  depends=('glibc')
  pkgdesc="free implementation of the GL API -- shared library. The Mesa GL API module is responsible for dispatching all the gl* functions"

  make -C ${srcdir}/?esa-*/src/mapi/shared-glapi DESTDIR="${pkgdir}" install

  install -m755 -d "${pkgdir}/usr/share/licenses/libglapi"
  install -m644 "${srcdir}/LICENSE" "${pkgdir}/usr/share/licenses/libglapi/"
}

package_libgl() {
  depends=('libdrm>=2.4.39' 'libxxf86vm>=1.1.2' 'libxdamage>=1.1.3' 'expat>=2.1.0' 'libglapi' 'gcc-libs')
  pkgdesc="Mesa 3-D graphics library and DRI software rasterizer"

  # fix linking because of splitted package
  make -C ${srcdir}/?esa-*/src/mapi/shared-glapi DESTDIR="${pkgdir}" install

  # libGL & libdricore
  make -C ${srcdir}/?esa-*/src/glx DESTDIR="${pkgdir}" install
  make -C ${srcdir}/?esa-*/src/mesa/libdricore DESTDIR="${pkgdir}" install

  # fix linking because of splitted package - cleanup
  make -C ${srcdir}/?esa-*/src/mapi/shared-glapi DESTDIR="${pkgdir}" uninstall


  make -C ${srcdir}/?esa-*/src/gallium/targets/dri-swrast DESTDIR="${pkgdir}" install

  # See FS#26284
  install -m755 -d "${pkgdir}/usr/lib/xorg/modules/extensions"
  ln -s libglx.xorg "${pkgdir}/usr/lib/xorg/modules/extensions/libglx.so"

  install -m755 -d "${pkgdir}/usr/share/licenses/libgl"
  install -m644 "${srcdir}/LICENSE" "${pkgdir}/usr/share/licenses/libgl/"
}

package_mesa() {
  # check also gl.pc
  depends=('libgl' 'libx11>=1.5.0' 'libxext>=1.3.1' 'libxdamage' 'libxfixes' 'libxcb' 'libxxf86vm')
  optdepends=('opengl-man-pages: for the OpenGL API man pages')
  pkgdesc="Mesa 3-D graphics libraries and include files"

  make -C ${srcdir}/?esa-*/src/mesa DESTDIR="${pkgdir}" install-glHEADERS
  make -C ${srcdir}/?esa-*/src/mesa/drivers/dri DESTDIR="${pkgdir}" install-driincludeHEADERS  
  make -C ${srcdir}/?esa-*/src/mesa DESTDIR="${pkgdir}" install-pkgconfigDATA
  make -C ${srcdir}/?esa-*/src/mesa/drivers/dri DESTDIR="${pkgdir}" install-pkgconfigDATA  
  make -C ${srcdir}/?esa-*/src/mesa/drivers/dri/common DESTDIR="${pkgdir}" install-sysconfDATA

  make -C ${srcdir}/?esa-*/src/gallium/targets/xa-vmwgfx DESTDIR="${pkgdir}" install

  install -m755 -d "${pkgdir}/usr/share/licenses/mesa"
  install -m644 "${srcdir}/LICENSE" "${pkgdir}/usr/share/licenses/mesa/"
}

package_osmesa() {
  depends=('libglapi' 'gcc-libs')
  optdepends=('opengl-man-pages: for the OpenGL API man pages')
  pkgdesc="Mesa 3D off-screen rendering library"

  # fix linking because of splitted package
  make -C ${srcdir}/?esa-*/src/mapi/shared-glapi DESTDIR="${pkgdir}" install

  make -C ${srcdir}/?esa-*/src/mesa/drivers/osmesa DESTDIR="${pkgdir}" install

  # fix linking because of splitted package - cleanup
  make -C ${srcdir}/?esa-*/src/mapi/shared-glapi DESTDIR="${pkgdir}" uninstall

  install -m755 -d "${pkgdir}/usr/share/licenses/osmesa"
  install -m644 "${srcdir}/LICENSE" "${pkgdir}/usr/share/licenses/osmesa/"
}

package_libgbm() {
  depends=('systemd' 'libglapi' 'libdrm')
  pkgdesc="Mesa gbm library"

  # fix linking because of splitted package
  make -C ${srcdir}/?esa-*/src/mapi/shared-glapi DESTDIR="${pkgdir}" install

  make -C ${srcdir}/?esa-*/src/gbm DESTDIR="${pkgdir}" install

  # fix linking because of splitted package - cleanup
  make -C ${srcdir}/?esa-*/src/mapi/shared-glapi DESTDIR="${pkgdir}" uninstall

  install -m755 -d "${pkgdir}/usr/share/licenses/libgbm"
  install -m644 "${srcdir}/LICENSE" "${pkgdir}/usr/share/licenses/libgbm/"
}

package_libgles() {
  depends=('libglapi' 'libdrm' 'khrplatform-devel')
  pkgdesc="Mesa GLES libraries and headers"

  # fix linking because of splitted package
  make -C ${srcdir}/?esa-*/src/mapi/shared-glapi DESTDIR="${pkgdir}" install

  make -C ${srcdir}/?esa-*/src/mapi/es1api DESTDIR="${pkgdir}" install
  make -C ${srcdir}/?esa-*/src/mapi/es2api DESTDIR="${pkgdir}" install

  # fix linking because of splitted package - cleanup
  make -C ${srcdir}/?esa-*/src/mapi/shared-glapi DESTDIR="${pkgdir}" uninstall

  install -m755 -d "${pkgdir}/usr/share/licenses/libgles"
  install -m644 "${srcdir}/LICENSE" "${pkgdir}/usr/share/licenses/libgles/"
}

package_libegl() {
  # check also egl.pc
  depends=('libx11' 'libxext' 'libxdamage' 'libxfixes' 'libxxf86vm' 'libxcb' 'libgbm' 'khrplatform-devel')
  pkgdesc="Mesa EGL libraries and headers"

  make -C ${srcdir}/?esa-*/src/gallium/targets/egl-static DESTDIR="${pkgdir}" install
  install -m755 -d "${pkgdir}/usr/share/doc/libegl"
  install -m644 ${srcdir}/?esa-*/docs/egl.html "${pkgdir}/usr/share/doc/libegl/"
  
  # fix linking because of splitted package
  make -C ${srcdir}/?esa-*/src/mapi/shared-glapi DESTDIR="${pkgdir}" install
  make -C ${srcdir}/?esa-*/src/gbm DESTDIR="${pkgdir}" install
  
  make -C ${srcdir}/?esa-*/src/egl DESTDIR="${pkgdir}" install

  # fix linking because of splitted package - cleanup
  make -C ${srcdir}/?esa-*/src/gbm DESTDIR="${pkgdir}" uninstall
  make -C ${srcdir}/?esa-*/src/mapi/shared-glapi DESTDIR="${pkgdir}" uninstall

  install -m755 -d "${pkgdir}/usr/share/licenses/libegl"
  install -m644 "${srcdir}/LICENSE" "${pkgdir}/usr/share/licenses/libegl/"
  
  # fix file conflicts
  rm -rf ${pkgdir}/usr/include/KHR
}

package_khrplatform-devel() {
  pkgdesc="Khronos platform development package"

  install -m755 -d "${pkgdir}/usr/include/KHR"
  install -m644 ${srcdir}/?esa-*/include/KHR/khrplatform.h "${pkgdir}/usr/include/KHR/" 

  install -m755 -d "${pkgdir}/usr/share/licenses/khrplatform-devel"
  install -m644 "${srcdir}/LICENSE" "${pkgdir}/usr/share/licenses/khrplatform-devel/"
}

package_ati-dri() {
  depends=("libgl=${pkgver}")
  pkgdesc="Mesa DRI radeon/r200 + Gallium3D r300,r600 drivers for AMD/ATI Radeon"
  conflicts=('xf86-video-ati<6.9.0-6')

  # fix linking because of splitted package
  make -C ${srcdir}/?esa-*/src/mesa/libdricore DESTDIR="${pkgdir}" install
  
  # classic mesa drivers for radeon,r200
  make -C ${srcdir}/?esa-*/src/mesa/drivers/dri/radeon DESTDIR="${pkgdir}" install
  make -C ${srcdir}/?esa-*/src/mesa/drivers/dri/r200 DESTDIR="${pkgdir}" install
  # gallium3D driver for r300,r600,radeonsi
  make -C ${srcdir}/?esa-*/src/gallium/targets/dri-r300 DESTDIR="${pkgdir}" install
  make -C ${srcdir}/?esa-*/src/gallium/targets/dri-r600 DESTDIR="${pkgdir}" install
  make -C ${srcdir}/?esa-*/src/gallium/targets/dri-radeonsi DESTDIR="${pkgdir}" install
  # vdpau driver
  make -C ${srcdir}/?esa-*/src/gallium/targets/vdpau-r300 DESTDIR="${pkgdir}" install
  make -C ${srcdir}/?esa-*/src/gallium/targets/vdpau-r600 DESTDIR="${pkgdir}" install
  make -C ${srcdir}/?esa-*/src/gallium/targets/vdpau-radeonsi DESTDIR="${pkgdir}" install

  # fix linking because of splitted package - cleanup
  make -C ${srcdir}/?esa-*/src/mesa/libdricore DESTDIR="${pkgdir}" uninstall

  install -m755 -d "${pkgdir}/usr/share/licenses/ati-dri"
  install -m644 "${srcdir}/LICENSE" "${pkgdir}/usr/share/licenses/ati-dri/"
}

package_intel-dri() {
  depends=("libgl=${pkgver}")
  pkgdesc="Mesa DRI drivers for Intel"

  # fix linking because of splitted package
  make -C ${srcdir}/?esa-*/src/mesa/libdricore DESTDIR="${pkgdir}" install

  make -C ${srcdir}/?esa-*/src/mesa/drivers/dri/i915 DESTDIR="${pkgdir}" install
  make -C ${srcdir}/?esa-*/src/mesa/drivers/dri/i965 DESTDIR="${pkgdir}" install

  # fix linking because of splitted package - cleanup
  make -C ${srcdir}/?esa-*/src/mesa/libdricore DESTDIR="${pkgdir}" uninstall
  
  install -m755 -d "${pkgdir}/usr/share/licenses/intel-dri"
  install -m644 "${srcdir}/LICENSE" "${pkgdir}/usr/share/licenses/intel-dri/"
}

package_nouveau-dri() {
  depends=("libgl=${pkgver}")
  pkgdesc="Mesa classic DRI + Gallium3D drivers for Nouveau"

  # fix linking because of splitted package
  make -C ${srcdir}/?esa-*/src/mesa/libdricore DESTDIR="${pkgdir}" install

  # classic mesa driver for nv10 , nv20 nouveau_vieux_dri.so
  make -C ${srcdir}/?esa-*/src/mesa/drivers/dri/nouveau DESTDIR="${pkgdir}" install
  # gallium3D driver for nv30 - nv40 - nv50 nouveau_dri.so
  make -C ${srcdir}/?esa-*/src/gallium/targets/dri-nouveau DESTDIR="${pkgdir}" install
  # vdpau driver
  make -C ${srcdir}/?esa-*/src/gallium/targets/vdpau-nouveau DESTDIR="${pkgdir}" install
  
  # fix linking because of splitted package - cleanup
  make -C ${srcdir}/?esa-*/src/mesa/libdricore DESTDIR="${pkgdir}" uninstall
  
  install -m755 -d "${pkgdir}/usr/share/licenses/nouveau-dri"
  install -m644 "${srcdir}/LICENSE" "${pkgdir}/usr/share/licenses/nouveau-dri/"
}

package_svga-dri() {
  depends=('gcc-libs' 'libdrm' 'expat')
  pkgdesc="Gallium3D VMware guest GL driver"

  make -C ${srcdir}/?esa-*/src/gallium/targets/dri-vmwgfx DESTDIR="${pkgdir}" install
  
  install -m755 -d "${pkgdir}/usr/share/licenses/svga-dri"
  install -m644 "${srcdir}/LICENSE" "${pkgdir}/usr/share/licenses/svga-dri/"
}
