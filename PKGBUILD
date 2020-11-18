# Maintainer: Felix Schindler <aur at felixschindler dot net>

pkgname=llvm-cling-git
pkgver=0.7.r20.g3d789b13
pkgrel=1
_ocaml_ver=4.11.1
pkgdesc="Interactive C++ interpreter with custom clang and llvm"
arch=('x86_64')
url="https://root.cern.ch/cling"
license=('custom:Cling Release License')
makedepends=(
  'cmake'
  'git'
  'jupyter'
  'libedit'
  'libffi'
  'libffi'
  'libxml2'
  'ncurses'
  'ninja'
  "ocaml=$_ocaml_ver"
  'ocaml-ctypes'
  'ocaml-findlib'
  'python'
  'python-recommonmark'
  'python2'
)
provides=(
  "cling=$pkgver"
  "cling-dev=$pkgver"
  "cling-dev-git=$pkgver"
  "cling-jupyter=$pkgver"
  "cling-jupyter-dev-git=$pkgver"
  "cling-jupyter-dev=$pkgver"
)
conflicts=(
  'cling'
  'cling-dev'
  'cling-dev-git'
  'cling-jupyter'
  'cling-jupyter-dev'
  'cling-jupyter-dev-git'
)
depends=('libffi')
options=('staticlibs')
source=(
  llvm::git+http://root.cern.ch/git/llvm.git#branch=cling-patches
  clang::git+http://root.cern.ch/git/clang.git#branch=cling-patches
  cling::git+http://root.cern.ch/git/cling.git#branch=master
)
sha256sums=('SKIP'
            'SKIP'
            'SKIP')

pkgver() {
  cd "${srcdir}"/cling
  git describe --long | sed 's/^v//;s/\([^-]*-g\)/r\1/;s/-/./g'
}

prepare() {
    cd "$srcdir/llvm" && mkdir -p build
    if [ ! -h "$srcdir/llvm/tools/clang" ]; then
        ln -s "$srcdir/clang" "$srcdir/llvm/tools/clang"
    fi
    if [ ! -h "$srcdir/llvm/tools/cling" ]; then
        ln -s "$srcdir/cling" "$srcdir/llvm/tools/cling"
    fi
}

_install_prefix="/opt/cling"

build() {
  cd "$srcdir/llvm/build"

  cmake .. -G Ninja \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=$_install_prefix \
    -DLLVM_TARGETS_TO_BUILD="host;NVPTX" \
    -DLLVM_BUILD_LLVM_DYLIB=OFF \
    -DLLVM_ENABLE_RTTI=ON \
    -DLLVM_ENABLE_FFI=ON \
    -DLLVM_BUILD_DOCS=OFF \
    -DLLVM_BUILD_TOOLS=OFF \
    -DLLVM_ENABLE_SPHINX=OFF \
    -DLLVM_ENABLE_DOXYGEN=OFF \
    -DFFI_INCLUDE_DIR=$(pkg-config --cflags-only-I libffi | cut -c3-) \
    -DLLVM_BINUTILS_INCDIR=/usr/include

  ninja all ocaml_doc
}

_python2_optimize() {
  python2 -m compileall "$@"
  python2 -O -m compileall "$@"
}

_python3_optimize() {
  python3 -m compileall "$@"
  python3 -O -m compileall "$@"
  python3 -OO -m compileall "$@"
}

package_llvm-cling-git() {
  cd "$srcdir/llvm/build"

  # from llvm
  DESTDIR="$pkgdir" ninja install

  # Include lit for running lit-based tests in other projects
  pushd ../utils/lit
  python3 setup.py install --root="$pkgdir/$_install_prefix" -O1
  popd
  mv -f "$pkgdir/$_install_prefix/usr/bin/lit" "$pkgdir/$_install_prefix/bin/"
  mv -f "$pkgdir/$_install_prefix/usr/lib/python3.8" "$pkgdir/$_install_prefix/lib/"
  rmdir "$pkgdir/$_install_prefix/usr/bin"
  rmdir "$pkgdir/$_install_prefix/usr/lib"
  rmdir "$pkgdir/$_install_prefix/usr"

  # Remove documentation sources
  rm -rf "$pkgdir"/usr/share/doc/

  # fix ocaml location
  mv -f "$pkgdir/usr/lib/ocaml" "$pkgdir/$_install_prefix/lib/"

  # from llvm-libs
  # Symlink LLVMgold.so from /usr/lib/bfd-plugins
  # https://bugs.archlinux.org/task/28479
  install -d "$pkgdir/$_install_prefix/lib/bfd-plugins"
  ln -s ../LLVMgold.so "$pkgdir/$_install_prefix/lib/bfd-plugins/LLVMgold.so"

  mkdir -p "$pkgdir/etc/ld.so.conf.d/"
  echo "$_install_prefix/lib" > "$pkgdir/etc/ld.so.conf.d/90-$pkgname.conf"

  # from cling
  install -d "$pkgdir/usr/bin"
  ln -s "/opt/cling/bin/cling" "$pkgdir/usr/bin/cling"
  install -d "$pkgdir/usr/include"
  ln -s "/opt/cling/include/cling" "$pkgdir/usr/include/cling"

  install -Dm644 "$srcdir/llvm/tools/cling/LICENSE.TXT" "$pkgdir/usr/share/licenses/$pkgname/LICENSE"

  # from cling-jupyter
  cd "$srcdir/cling/tools/Jupyter/kernel"
  python3 setup.py install --prefix=/usr --root="$pkgdir"
  jupyter-kernelspec install --prefix="$pkgdir/usr" .
}
