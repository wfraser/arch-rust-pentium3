# Maintainer: Bill Fraser <wfraser@codewise.org>
pkgname=rust-nightly-pentium3
pkgver=1.65.0_2022.09.03.g84f0c3f79
pkgrel=1
pkgdesc="Rust for Pentium III machines"
url="https://github.com/rust-lang/rust"
arch=(i686)
license=('MIT' 'Apache')
depends=(llvm-libs gcc-libs)
makedepends=(cmake ninja python git lib32-openssl lib32-libgit2 lib32-zlib lib32-curl)
provides=(rust)
conflicts=(rustup cargo)
_src="https://static.rust-lang.org/dist/rustc-nightly-src.tar.xz"
# to compile a specific nightly date, use a src url like this, and comment out `pkgver()` below.
#_src="https://static.rust-lang.org/dist/2022-09-03/rustc-nightly-src.tar.xz"
source=($_src $_src.asc pentium3.patch)
validpgpkeys=('108F66205EAEB0AAA8DD5E1C85AB96E6FA1BE5FE')
sha256sums=("$(curl -sL $_src.sha256 | cut -d\  -f1)" "SKIP" "SKIP")

pkgver() {
    cd rustc-nightly-src
    ver="$(expr "$(cat version)" : '\(.*\)-nightly')"
    date="$(expr "$(cat version)" : '.* \(.*\))')"
    rev="$(expr "$(cat version)" : '.* (\(.*\) ')"
    echo "${ver}_${date//\-/.}.g${rev}"
}

build() {
    cd rustc-nightly-src

    patch -p1 < $srcdir/pentium3.patch

    # Notes:
    # --llvm-ldflags=-lz:
    #   LLVM gets built with wrong flags when cross-compilining
    #   See https://github.com/rust-lang/rust/pull/72696
    # --set=llvm.targets=X86:
    #   LLVM by default gets built with the ability to codegen for a whole laundry list of
    #   architectures; we only need X86.
    # --set=llvm.link-jobs=1:
    #   My poor build machine tends to run out of memory linking LLVM, particularly if debug mode
    #   is enabled.
    # --disable-llvm-static-stdcpp:
    #   Statically linking libstdc++ for LLVM ends up pulling in the host version, which has
    #   unsuitable instructions. Instead, dynamically link to the target's version.

    ./configure \
        --prefix=/usr \
        --target=i586-unknown-linux-gnu \
        --host=i586-unknown-linux-gnu \
        --release-channel=nightly \
        --disable-docs \
        --enable-extended \
        --tools=cargo,clippy \
        --set=llvm.targets=X86 \
        --llvm-ldflags=-lz \
        --set=llvm.link-jobs=1 \
        --disable-llvm-static-stdcpp \
    ;

    export PKG_CONFIG_ALLOW_CROSS=1
    export CFLAGS_i586_unknown_linux_gnu="-m32 -march=pentium3"
    export CXXFLAGS_i586_unknown_linux_gnu="-m32 -march=pentium3"
    export RUSTFLAGS=""

    # pre 2021-11-13 component names:
    #python x.py dist -v src/librustc library/std cargo clippy

    python x.py dist -v rustc rust-std cargo clippy
}

package() {
    cd rustc-nightly-src

    # Running `x.py install` here doesn't work:
    # 1. it builds a bunch of stuff all over again
    # 2. when building Cargo, it fails to find the libstd it built, and fails.
    # DESTDIR="$pkgdir/" python x.py install src/librustc library/std cargo clippy

    # But building with `x.py dist` above works fine, so here let's just unpack the dist tarballs
    # into the proper location:

    # dist tarballs have paths like ${component}-nightly-i586-unknown-linux-gnu/${component}/bin/...
    mkdir $pkgdir/usr
    for dist in build/dist/*.tar.xz; do
        tar -C $pkgdir/usr --strip-components=2 -xvf $dist
    done

    rm -rf "$pkgdir/usr/share/doc"

    install -d "$pkgdir/usr/share/licenses/$pkgname/"
    install -m644 COPYRIGHT LICENSE-* "$pkgdir/usr/share/licenses/$pkgname/"

    # FIXME: the binaries specify an ISA level that's too high
    # for now, just remove the section that note is in
    echo "fixing ISA level"
    for f in $pkgdir/usr/bin/*; do
        # file will crash with SIGSYS ("bad system call") if we have libfakeroot.so preloaded
        # due to some seccomp thing
        LD_PRELOAD="" file $f
        if LD_PRELOAD="" file --no-sandbox $f | grep --quiet ELF; then
            echo "removing .note.gnu.property from $f"
            strip --remove-section=.note.gnu.property $f
        fi
    done
}

# vim: ft=bash
