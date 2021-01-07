# Maintainer: Bill Fraser <wfraser@codewise.org>
pkgname=rust-nightly-pentium3
pkgver=1.50.0_2020.12.06
pkgrel=1
pkgdesc="Rust for Pentium III machines"
url="https://github.com/rust-lang/rust"
arch=(i686 x86_64)
license=('MIT' 'Apache')
depends=(llvm-libs gcc-libs)
makedepends=(cmake ninja python git lib32-openssl lib32-libgit2 lib32-zlib)
provides=(rust)
conflicts=(rustup cargo)
_src="https://static.rust-lang.org/dist/rustc-nightly-src.tar.xz"
source=($_src $_src.asc pentium3.patch)
validpgpkeys=('108F66205EAEB0AAA8DD5E1C85AB96E6FA1BE5FE')
sha256sums=("$(curl -sL $_src.sha256 | cut -d\  -f1)" "SKIP" "SKIP")

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
            print "targets=\"X86\""  # Not the whole laundry list of weird architectures.
            print "ldflags=\"-lz\""  # LLVM gets built with wrong flags when cross compiling
                                     # see https://github.com/rust-lang/rust/pull/72696
            next
        }
        { print }
        ' config.toml > config.toml.new
    mv config.toml.new config.toml

    export PKG_CONFIG_ALLOW_CROSS=1
    export CFLAGS_i586_unknown_linux_gnu="-m32 -march=pentium3"
    export CXXFLAGS_i586_unknown_linux_gnu="-m32 -march=pentium3"
    export RUSTFLAGS=""

    python x.py dist -v src/librustc library/std cargo clippy
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
}

# vim: ft=bash
