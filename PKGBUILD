# Maintainer: Bill Fraser <wfraser@codewise.org>
pkgname=rust-nightly-pentium3
pkgver=1.50.0_2020.12.06
pkgrel=1
pkgdesc="Rust for Pentium III machines"
url="https://github.com/rust-lang/rust"
arch=(i686 x86_64)
license=('MIT' 'Apache')
depends=(llvm-libs gcc-libs)
makedepends=(python git lib32-openssl lib32-libgit2 lib32-zlib)
provides=(rust)
conflicts=(rustup cargo)
_src="https://static.rust-lang.org/dist/rustc-nightly-src.tar.xz"
source=($_src $_src.asc pentium3.patch)
validpgpkeys=('108F66205EAEB0AAA8DD5E1C85AB96E6FA1BE5FE')
sha256sums=("$(curl -sL $_src.sha256 | cut -d\  -f1)" "SKIP" "SKIP")

build_targets="src/librustc library/std cargo clippy"
architecture="pentium3"

pkgver() {
    cd rustc-nightly-src
    ver="$(expr "$(cat version)" : '\(.*\)-nightly')"
    date="$(expr "$(cat version)" : '.* \(.*\))')"
    echo "${ver}_${date//\-/.}"
}

build() {
    cd rustc-nightly-src

    patch -p1 < $srcdir/pentium3.patch

    ./configure \
        --prefix=/usr \
        --target=i586-unknown-linux-gnu \
        --host=i586-unknown-linux-gnu \
        --release-channel=nightly \
        --disable-docs

    awk '
        /^#extended = /{ print "extended = true"; next }
        /^#tools = /{ print "tools = [\"cargo\", \"clippy\"]"; next }
        /^\[llvm\]/{
            print
            print "targets=\"X86\""
            print "ldflags=\"-lz\""
            next
        }
        { print }
        ' config.toml > config.toml.new
    mv config.toml.new config.toml

    export PKG_CONFIG_ALLOW_CROSS=1
    export CFLAGS_i586_unknown_linux_gnu="-m32 -march=$architecture"
    export CXXFLAGS_i586_unknown_linux_gnu="-m32 -march=$architecture"
    export RUSTFLAGS=""

    python x.py dist -v $build_targets
}

package() {
    cd rustc-nightly-src
    DESTDIR="$pkgdir/" python x.py install $build_targets
}

# vim: ft=bash
