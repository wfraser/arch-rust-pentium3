# rustc for Pentium III on Arch Linux

This repository contains a PKGBUILD to build a Rust compiler that can run on Pentium III machines.

It is a modified version of the [rust-nightly] AUR package. It builds `rustc`, `cargo`, and
`clippy` with a host and target triple of `i586-unknown-linux-gnu`, because Rust's tier-1
`i686-unknown-linux-gnu` target is actually defined as a `pentium4` CPU, which means it uses SSE2
instructions which cause it to crash on an *actual* i686, better known as Pentium III.

I also patch the Rust sources to say that the `i586-unknown-linux-gnu` is actually a `pentium3`
CPU, but it's still using the `i586-...` LLVM target, so it's not clear whether that'll actually
result in Pentium III optimizations, or just a Pentium II (i586) binary.

In this effort I also used the [FreeBSD rust-bootstrap] package as a valuable reference, because it
faces and solves many of the same problems as this one, namely, cross-bootstrapping the compiler
for different architectures.

# Dependencies

There are two packages in the `other_pkgs/` dir which need to be built and installed on the host
machine in order to build. They're 32-bit versions of the `http-parser` and `libgit2` packages.

On the target (P3) machine, you'll need a matching libgit2 to the one on the host. Currently
(November 2021) this is kind of a pain, because 64-bit arch has v1.2.0 but Arch Linux 32 only has
v1.1.0, and they're incompatible. You'll need to get the PKGBUILD for it and update the version
until an actual update lands in the repository.

[rust-nightly]: https://aur.archlinux.org/packages/rust-nightly/
[FreeBSD rust-bootstrap]: https://www.freshports.org/lang/rust-bootstrap/
