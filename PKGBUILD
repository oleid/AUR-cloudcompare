#!/usr/hint/bash
# Maintainer  : bartus <arch-user-repo(at)bartus.33mail.com>
# Contributor : Johannes Sauer <joh.sauer(at)gmail(dot)com>
# Contributor : Danilo Bargen <aur at dbrgn dot ch>
# Contributor : Olaf Leidinger <oleid@mescharet.de>
# shellcheck disable=SC2034,SC2154 # allow unused/uninitialized variables.

name=cloudcompare
pkgname=${name}
_fragment="#tag=v2.11.0"
pkgver="${_fragment###tag=v}"
pkgrel=3
pkgdesc="A 3D point cloud (and triangular mesh) processing software"
arch=('i686' 'x86_64')
url="http://www.danielgm.net/cc/"
license=('GPL2')
depends=('cgal' 'dlib' 'ffmpeg' 'glew' 'glu' 'mesa' 'mpir' 'pdal' 'qt5-base' 'qt5-tools' 'qt5-svg' 'shapelib' 'tbb' 'vxl')
makedepends=('clang' 'cmake' 'doxygen' 'git' 'laz-perf' 'libharu' 'ninja' 'pcl' 'proj' 'python')
optdepends=('pcl')
source=("${name}::git+https://github.com/CloudCompare/CloudCompare.git${_fragment}"
        "${name}-cork::git+https://github.com/CloudCompare/cork.git"
        constexpr.patch
        cork.patch
        pcl.patch
        CloudCompare.desktop
        ccViewer.desktop)
sha256sums=('SKIP'
            'SKIP'
            '984e6186f6483534a52cb153b65dee016904eb9efdb89211c2c0042eea2417ff'
            '3f8692fbff2b92ebf422f2c4de46f2f9164cd37879092c66f171b32fea464227'
            '183f05dee316bdf7c6e9935e2a21b79b2172f61df5b0b6954bcd0d74751afc64'
            '14096df9cf7aca3099d5df1585d1cf669544e9b10754dce3d2507100dd7034fe'
            '821ac2540e1196774e26f8033946ce7b36223dae7a2a7c78f4a901b4177f68cc')

prepare() {
  git -C "${srcdir}/${name}" submodule update --init --recursive
  git -C "${srcdir}/${name}" apply -v "${srcdir}"/{constexpr,pcl}.patch
  git -C "${srcdir}/${name}-cork" apply -v "${srcdir}"/cork.patch
}

pkgver() {
  git -C "${srcdir}/${name}" describe --tags | sed -r 's/^v//;s/([^-]*-g)/r\1/;s/-/./g'
}

build() {
# shellcheck disable=SC2191
  CMAKE_FLAGS=(
        -Wno-dev
        -DCMAKE_CXX_STANDARD=14
        -DCMAKE_CXX_FLAGS=-fpermissive
        -DCMAKE_INSTALL_PREFIX=/usr
        -DCMAKE_INSTALL_LIBDIR=lib
        -DCMAKE_BUILD_TYPE=Release
        -DCOMPILE_CC_CORE_LIB_WITH_CGAL=ON
        -DCOMPILE_CC_CORE_LIB_WITH_TBB=ON
        -DPLUGIN_EXAMPLE_GL:BOOL=ON
        -DPLUGIN_EXAMPLE_IO:BOOL=ON
        -DPLUGIN_EXAMPLE_STANDARD:BOOL=ON
        -DPLUGIN_GL_QEDL:BOOL=ON
        -DPLUGIN_GL_QSSAO:BOOL=ON
        -DPLUGIN_IO_QADDITIONAL:BOOL=ON
        -DPLUGIN_IO_QCORE:BOOL=ON
        -DPLUGIN_IO_QCSV_MATRIX:BOOL=ON
        -DPLUGIN_IO_QE57:BOOL=ON
        -DPLUGIN_IO_QFBX:BOOL=OFF # requires update of AUR/fbx-sdk (https://www.autodesk.com/content/dam/autodesk/www/adn/fbx/2020-1-1/fbx202011_fbxsdk_linux.tar.gz)
        -DPLUGIN_IO_QPDAL:BOOL=ON
        -DPLUGIN_IO_QPHOTOSCAN:BOOL=ON
        -DPLUGIN_IO_QRDB:BOOL=OFF # requires rdblib (package for AUR from http://www.riegl.com/products/software-packages/rdblib/)
        -DPLUGIN_STANDARD_QANIMATION:BOOL=ON
        -DPLUGIN_STANDARD_QBROOM:BOOL=ON
        -DPLUGIN_STANDARD_QCANUPO:BOOL=ON # requires dlib
        -DDLIB_ROOT:PATH="/usr" # required by qcanupo plugin
        -DPLUGIN_STANDARD_QCOMPASS:BOOL=ON
        -DPLUGIN_STANDARD_QCORK:BOOL=ON # require mpir, cork (cork-git is not enough)
        -DMPIR_INCLUDE_DIR:PATH=/usr/include # required by qcork plugin
        -DCORK_INCLUDE_DIR:PATH="${srcdir}/${name}-cork/src" # required by qcork plugin
        -DCORK_RELEASE_LIBRARY_FILE:PATH="${srcdir}/${name}-cork/lib/libcork.a" # required by qcork plugin
        -DMPIR_RELEASE_LIBRARY_FILE:PATH=/usr/lib/libmpir.so # require by qcork plugin
        -DPLUGIN_STANDARD_QCSF:BOOL=ON
        -DPLUGIN_STANDARD_QFACETS:BOOL=ON # requires shapelib
        -DOPTION_USE_SHAPE_LIB:BOOL=ON
        -DPLUGIN_STANDARD_QHOUGH_NORMALS:BOOL=ON
        -DPLUGIN_STANDARD_QHPR:BOOL=ON
        -DPLUGIN_STANDARD_QM3C2:BOOL=ON
        -DPLUGIN_STANDARD_QPCL:BOOL=ON
        -DPLUGIN_STANDARD_QPCV:BOOL=ON
        -DPLUGIN_STANDARD_QPOISSON_RECON:BOOL=ON
        -DPLUGIN_STANDARD_QRANSAC_SD:BOOL=ON
        -DPLUGIN_STANDARD_QSRA:BOOL=ON
        -DOPTION_USE_DXF_LIB:BOOL=ON # required by qsra plugin
        -DEIGEN_ROOT_DIR=/usr/include/eigen3
  )
  msg2 "Build Cork lib"
  (cd "${srcdir}/${name}-cork"; CFLAGS+=" -fPIC" CXXFLAGS+=" -fPIC" make)
  msg2 "Build CloudCompare"
  (cd "${srcdir}/${name}-cork"; make)
  cmake -B build -S "${srcdir}/${name}" -G Ninja "${CMAKE_FLAGS[@]}" -DCMAKE_CXX_FLAGS="$CXXFLAGS -Wno-deprecated-declarations"
# shellcheck disable=SC2086 # allow slitting for MAKEFLAGS carrying multiple flags.
  ninja -C build ${MAKEFLAGS:--j1}
}

package() {
  DESTDIR="$pkgdir" ninja -C build install

  # install *.desktop files
  install -D -m 644 "${srcdir}"/*.desktop -t "${pkgdir}"/usr/share/applications/

  # copy icons for *.desktop files
  for size in 16 32 64 256; do
    install -D -m 644 ${name}/qCC/images/icon/cc_icon_${size}.png "${pkgdir}"/usr/share/icons/hicolor/${size}x${size}/apps/cc_icon.png
    install -D -m 644 ${name}/qCC/images/icon/cc_viewer_icon_${size}.png "${pkgdir}"/usr/share/icons/hicolor/${size}x${size}/apps/cc_viewer_icon.png
  done 
  install -D -m 644 ${name}/qCC/images/icon/cc_icon.svg "${pkgdir}"/usr/share/icons/hicolor/scalable/apps/cc_icon.svg
  install -D -m 644 ${name}/qCC/images/icon/cc_viewer_icon.svg "${pkgdir}"/usr/share/icons/hicolor/scalable/apps/cc_viewer_icon.svg
}
# vim:set sw=2 ts=2 et:
